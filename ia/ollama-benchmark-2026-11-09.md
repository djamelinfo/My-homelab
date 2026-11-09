# Ollama on a Quadro RTX 4000: model benchmarks and Frigate tuning

My AI server has an 8 GB Quadro RTX 4000 and serves both Frigate and Home Assistant LLM Vision. I wanted to see how my production model, `qwen3-vl:8b-instruct`, compared with the smaller models, then check what happened during normal activity around the house.

The controlled tests favoured `qwen3.5:9b` for image accuracy on my small sample. The household tests exposed a more immediate problem: Frigate could send enough images to exceed the context before Ollama generated any answer.

## Test setup

| Component | Value |
|---|---|
| Test date | 9 November 2026 |
| AI VM | Debian 13 on Proxmox, 4 vCPUs |
| GPU | NVIDIA Quadro RTX 4000, 8 GB |
| Ollama | `0.40.15`, Docker |
| Quantization | Q4_K_M for all four models |
| Context | 8192 tokens |
| Parallel requests / loaded models | 1 / 1 |

For the repeated comparison, I ran the client on Debian against `127.0.0.1:11434`. Each model received a request after loading, three warm text requests and three photograph requests. GPU and CPU readings were sampled every 200 ms. Frigate and Home Assistant remained active, so application traffic could still compete with the benchmark.

The text task used the same French summarisation prompt, temperature 0 and a maximum of 96 output tokens. TTFT is the time from sending a request to receiving its first non-empty streamed fragment. Generation speed comes from Ollama's output token count divided by its evaluation duration.

## Warm performance

| Model | Median text TTFT | Median text generation | Median vision TTFT | Median vision total | Peak VRAM |
|---|---:|---:|---:|---:|---:|
| `qwen3.5:9b` | 0.380 s | 43.96 tok/s | 0.719 s | 1.175 s | 6610 MiB |
| `qwen3-vl:8b-instruct` | 0.368 s | 32.35 tok/s | 2.188 s | 2.741 s | 7244 MiB |
| `qwen3-vl:4b-instruct` | 0.214 s | 86.71 tok/s | 1.125 s | 1.338 s | 5236 MiB |
| `gemma4:e2b` | 0.107 s | 140.51 tok/s | 0.250 s | 0.411 s | 3264 MiB |

Gemma generated text fastest and used the least VRAM. The 4B also had substantially more memory headroom than the production 8B. The 9B returned the first vision token in about a third of the 8B's time.

During warm text generation, the 8B averaged 92.9% CPU use across the four vCPUs and 45.6% GPU use. CPU work or partial offload may explain some of its lower throughput; these readings do not identify the exact bottleneck.

I use `nvidia-smi` for the memory figures. Ollama's `/api/ps` reported lower values, particularly for Gemma, and did not represent total GPU memory consumption in these runs.

## Loading a model

| Model | Ollama load time | First-token time after switch | First request total |
|---|---:|---:|---:|
| `qwen3.5:9b` | 7.69 s | 8.07 s | 10.97 s |
| `qwen3-vl:4b-instruct` | 10.46 s | 10.67 s | 12.20 s |
| `gemma4:e2b` | 12.24 s | 12.52 s | 13.42 s |
| `qwen3-vl:8b-instruct` | 16.78 s | 17.20 s | 21.07 s |

These are model-switch measurements. Linux could still have model files in its disk cache, so they are not fully cold starts. They explain why I keep the production model resident with a 24-hour keep-alive in Frigate's requests.

## Image checks

I used one synthetic image for colours, shapes and text, followed by three photographs from the [COCO validation set](https://cocodataset.org/).

![Synthetic benchmark image](../images/ollama-benchmark-test.png)

The synthetic image had four expected fields:

```text
cercle=rouge; carre=bleu; triangle=vert; texte=A7
```

| Photograph | Checks |
|---|---|
| [Cats on a sofa](http://images.cocodataset.org/val2017/000000039769.jpg) | Two cats, sofa or couch, two remote controls |
| [Person indoors](http://images.cocodataset.org/val2017/000000000139.jpg) | One person, kitchen or open-plan kitchen, standing or working near the counter |
| [Tram](http://images.cocodataset.org/val2017/000000006040.jpg) | Tram or streetcar, route number 2, running on rails |

Each correct field earned one point. These 13 checks compare the models on this image set; they are too few to establish accuracy for camera alerts.

| Model | Synthetic image | Photographs | Total | Errors in the sample |
|---|---:|---:|---:|---|
| `qwen3.5:9b` | 3/4 | 9/9 | 12/13 | Read `A7` as `H2` |
| `qwen3-vl:8b-instruct` | 4/4 | 7/9 | 11/13 | Called the kitchen a dining room and described the person as eating |
| `qwen3-vl:4b-instruct` | 4/4 | 6/9 | 10/13 | Reported two people in a living room and described them as eating |
| `gemma4:e2b` | 4/4 | 5/9 | 9/13 | Missed the remotes, misread the indoor scene and read route 504 instead of 2 |

For my image workloads, the 9B is the most promising alternative from this comparison. The 8B remains my production model. The smaller models offer faster responses, but their mistakes on the indoor photograph matter for the kind of descriptions I use.

## My real usage tests with Frigate and LLM Vision

I walked around the house and carried out normal activities while both applications were active. Ollama stayed on `qwen3-vl:8b-instruct`; no synthetic benchmark requests were added during these sessions. Frigate's debug logs supplied the image counts and token usage below.

### Where the context errors came from

In the first two sessions, Ollama completed 16 of 22 observed inference calls. All six rejected calls were matched to Frigate and exceeded the 8192-token context.

The second session showed how much the image count mattered:

| Frigate request | Images | Input tokens | Ollama API duration | Result |
|---|---:|---:|---:|---|
| Object description | 1 | 1085 | 3.40 s | Completed |
| Structured review | 9 | 10383 | 0.35 s | Context exceeded |
| Object description | 7 | 7348 | 15.27 s | Completed |
| Object description | 6 | 6370 | 26.27 s | Completed |
| Structured review | 37 | 39541 | 2.14 s | Context exceeded |
| Structured review | 3 | 4266 | 17.56 s | Completed |

The failed reviews already requested `num_ctx: 8192`. Keeping the model loaded could avoid reload delays, but it could not make those inputs fit.

### Reducing the review image budget

I changed the provider configuration to give Frigate a smaller planning budget while retaining the full context in Ollama:

```yaml
provider_options:
  keep_alive: 24h
  options:
    num_ctx: 4096

runtime_options:
  options:
    num_ctx: 8192
    num_predict: 256
    temperature: 0.1
```

In Frigate 0.18.2, review planning reads the provider context value and the runtime options override the options sent to Ollama. With this configuration, the next test produced five reviews with three images each. All five completed, using 4161 to 4273 input tokens.

Object descriptions use a separate image selection path. Three ten-image object requests still failed, with 10426 to 10594 tokens. Changing the review budget therefore addressed only part of the problem.

### Using a snapshot for FrontCam

I then enabled `use_snapshot: true` under `cameras.FrontCam.objects.genai`. This sends a single snapshot for that camera's object descriptions. It does not change review summaries, and a single image carries less information about movement than a sequence.

The latest Frigate log export contained these eleven requests:

| Request type | Calls | Images per call | Input tokens | Result |
|---|---:|---:|---:|---|
| Object description | 4 | 1 | 1085 to 1204 | All completed |
| Object description | 1 | 6 | 6436 | Completed |
| Object description | 1 | 8 | 8487 | Context exceeded |
| Structured review | 5 | 3 | 4158 to 4269 | All completed |

Ten requests completed and one was rejected. All five review responses completed without a JSON parsing error. The remaining failure was an eight-image object request. The snapshot change on FrontCam therefore improved the tested configuration without eliminating every oversized request.

### Queuing and GPU use

One parallel request keeps memory use manageable but makes bursts visible. In the earlier six-image example, the API call lasted 26.27 seconds while the runner processed it in about 14.96 seconds. The roughly 11.3 seconds before processing were consistent with waiting behind another request.

VRAM stayed around 7296 MiB during the household sessions. In the latest monitoring window, the highest sampled temperature was 73 °C. These GPU readings were taken approximately every ten seconds.


## Configuration I kept

I use an 8192-token Ollama context, one loaded model and one parallel request. At roughly 7296 MiB of VRAM usage, the card has about 896 MiB free. Frigate requests a 24-hour keep-alive, uses the smaller review planning budget and sends snapshots for FrontCam object descriptions.


## References

- [Ollama API metrics](https://docs.ollama.com/api/usage)
- [Ollama Chat API](https://docs.ollama.com/api/chat)
- [Ollama FAQ: keep-alive, context and parallelism](https://github.com/ollama/ollama/blob/main/docs/faq.mdx)
- [Frigate GenAI configuration](https://docs.frigate.video/configuration/genai/genai_config/)
- [Frigate object descriptions and snapshots](https://docs.frigate.video/configuration/genai/genai_objects/)
- [Frigate 0.18.2 Ollama provider implementation](https://github.com/blakeblackshear/frigate/blob/v0.18.2/frigate/genai/plugins/ollama.py)
- [Frigate 0.18.2 review frame selection](https://github.com/blakeblackshear/frigate/blob/v0.18.2/frigate/data_processing/post/review_descriptions.py)
