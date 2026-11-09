# My Ollama benchmark, 9 November 2026

I ran this benchmark against the Ollama server in my Debian 13 AI VM.

| Item | Value |
|---|---|
| Ollama endpoint | LAN API on port `11434` |
| Ollama version | `0.40.15` |
| GPU | NVIDIA Quadro RTX 4000, 8 GB |
| Context | `num_ctx=8192` |
| Concurrency | `1` |
| Quantization | Q4_K_M for all tested models |

For the controlled model comparison, I tested only models that were already installed. I did not download, create or delete any models, and I left the server configuration unchanged. I sent requests one at a time with a four-second pause. After the last test, I loaded the production model again and restored its runtime `keep_alive` to 24 hours. I later tested normal household activity with Frigate and LLM Vision running together.

## How I tested

For each model, I sent:

1. A first text request, including the loading time when the model was not resident.
2. A hot text request in streaming mode to measure TTFT and generation speed.
3. A hot vision request using a synthetic image with no personal data.

I used the same French summarisation prompt for every text request, with temperature 0 and at most 96 output tokens. The vision request used at most 64 output tokens.

![Synthetic benchmark image](../images/ollama-benchmark-test.png)

I expected this image answer:

```text
cercle=rouge; carre=bleu; triangle=vert; texte=A7
```

I measured TTFT from the start of each request to the first non-empty streamed fragment. Ollama reports durations in nanoseconds, so I calculated generation speed with this formula:

```text
eval_count × 1,000,000,000 / eval_duration
```

## Synthetic image results

| Model | Cold load | Full cold request | Hot text TTFT | Hot text total | Generation | Vision TTFT | Vision total | Vision score | VRAM from `/api/ps` |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `qwen3-vl:8b-instruct` | 18.99 s | 21.91 s | 0.378 s | 3.332 s | 31.4 tok/s | 1.943 s | 2.551 s | 4/4 | 5.74 GiB |
| `qwen3-vl:4b-instruct` | 12.21 s | 13.53 s | 0.217 s | 1.294 s | 87.7 tok/s | 18.027 s | 18.252 s | 4/4 | 4.03 GiB |
| `gemma4:e2b` | 118.29 s | 140.20 s | 0.132 s | 0.791 s | 138.3 tok/s | 0.292 s | 0.432 s | 4/4 | 0.36 GiB* |
| `qwen3.5:9b` | 24.17 s | 26.51 s | 0.448 s | 1.994 s | 44.0 tok/s | 0.886 s | 1.275 s | 3/4 | 5.35 GiB |

\* The Gemma VRAM figure does not match its installed model file. The file is approximately 4.27 GiB, while `/api/ps` reported only 0.36 GiB. I use `nvidia-smi` to confirm total GPU use instead of relying on this figure.

## Synthetic image notes

### `qwen3-vl:8b-instruct`

This model returned the correct image answer and behaved consistently while hot. It was the slowest text model in my test at roughly 31 to 34 tok/s. Its cold load took 18.99 seconds, which is why I prefer to keep it loaded.

### `qwen3-vl:4b-instruct`

This model loaded fastest and generated text at about 88 tok/s. Its image answer was correct, but prompt evaluation paused for 17.95 seconds before generation began. I ran several consecutive image requests to see whether that delay returned after the vision encoder was warm.

The real-photo test answered that question. After the first image loaded the model, its average TTFT on the next two images was 1.25 seconds.

### `gemma4:e2b`

Once loaded, Gemma was the fastest model. Text TTFT was 0.132 seconds and generation reached 138.3 tok/s. It also returned the correct image answer with a 0.292-second TTFT. Cold loading took 118.29 seconds, and the first request took 140.20 seconds in total. I would test it with my real Home Assistant and Frigate prompts before using it, and only if it can stay loaded.

My later real-photo run loaded Gemma in 11.79 seconds. Its first-call load time varied sharply between the two runs, so I do not consider the 118.29-second result a stable estimate.

### `qwen3.5:9b`

This model generated text faster than my production 8B, but read `A7` as `H2`. This was the only failed synthetic image check. It scored 9/9 in my real-photo run, so the OCR miss did not repeat on the tram image.

## My synthetic test rankings

### Hot text speed

1. `gemma4:e2b`
2. `qwen3-vl:4b-instruct`
3. `qwen3.5:9b`
4. `qwen3-vl:8b-instruct`

### Vision accuracy, then measured latency

1. `gemma4:e2b`
2. `qwen3-vl:8b-instruct`
3. `qwen3-vl:4b-instruct`
4. `qwen3.5:9b`

### Cold loading

1. `qwen3-vl:4b-instruct`
2. `qwen3-vl:8b-instruct`
3. `qwen3.5:9b`
4. `gemma4:e2b`

## Real-photo follow-up

For my second run, I used three public photographs from the [COCO validation set](https://cocodataset.org/). I link to the source files here instead of copying them into the wiki.

| Case | Source image | Checks |
|---|---|---|
| Pets | [COCO 000000039769](http://images.cocodataset.org/val2017/000000039769.jpg) | Two cats, sofa or couch, two remote controls |
| Indoor person | [COCO 000000000139](http://images.cocodataset.org/val2017/000000000139.jpg) | One person, kitchen or open-plan kitchen, standing or working near the counter |
| Tram and OCR | [COCO 000000006040](http://images.cocodataset.org/val2017/000000006040.jpg) | Tram or streetcar, route number 2, running on rails |

I awarded one point for each correct field, for a maximum of 9. The first photo for each model also loaded that model. I calculated the warm averages from the second and third photos.

| Model | Quality | First load | First-photo TTFT | Warm TTFT average | Warm total average | Main errors |
|---|---:|---:|---:|---:|---:|---|
| `qwen3.5:9b` | 9/9 | 15.43 s | 16.37 s | 0.75 s | 1.18 s | None |
| `qwen3-vl:8b-instruct` | 7/9 | 17.04 s | 19.49 s | 2.37 s | 2.99 s | Called the kitchen a dining room and described the person as eating |
| `qwen3-vl:4b-instruct` | 6/9 | 11.45 s | 12.84 s | 1.25 s | 1.44 s | Reported two people in a living room and described them as eating |
| `gemma4:e2b` | 5/9 | 11.79 s | 12.31 s | 0.26 s | 0.39 s | Missed both remotes, misread the indoor scene and read route 504 instead of 2 |

My accuracy ranking for the real photographs:

1. `qwen3.5:9b`
2. `qwen3-vl:8b-instruct`
3. `qwen3-vl:4b-instruct`
4. `gemma4:e2b`

The two image tests contain 13 checks in total. I treat the combined count as a small sanity check, not a complete measure of model quality.

| Model | Synthetic | Real photos | Combined |
|---|---:|---:|---:|
| `qwen3.5:9b` | 3/4 | 9/9 | 12/13 |
| `qwen3-vl:8b-instruct` | 4/4 | 7/9 | 11/13 |
| `qwen3-vl:4b-instruct` | 4/4 | 6/9 | 10/13 |
| `gemma4:e2b` | 4/4 | 5/9 | 9/13 |

## My repeated localhost benchmark over SSH

I then ran the benchmark client directly on Debian against `127.0.0.1:11434` to remove LAN latency. Each model received one request after switching, three repeated hot text requests and the three real-photo requests. I sampled GPU and CPU use every 200 ms.

| Model | Text TTFT median | Text TTFT p95 | Generation median | Vision TTFT median | Vision total median | Peak VRAM from `nvidia-smi` |
|---|---:|---:|---:|---:|---:|---:|
| `qwen3.5:9b` | 0.380 s | 0.387 s | 43.96 tok/s | 0.719 s | 1.175 s | 6 610 MiB |
| `qwen3-vl:8b-instruct` | 0.368 s | 0.372 s | 32.35 tok/s | 2.188 s | 2.741 s | 7 244 MiB |
| `qwen3-vl:4b-instruct` | 0.214 s | 0.216 s | 86.71 tok/s | 1.125 s | 1.338 s | 5 236 MiB |
| `gemma4:e2b` | 0.107 s | 0.112 s | 140.51 tok/s | 0.250 s | 0.411 s | 3 264 MiB |

The first request after each switch included model loading. It may still have benefited from the Linux disk cache, so I consider these switch times rather than fully cold starts.

| Model | Ollama load time | First-token time after switch | First request total |
|---|---:|---:|---:|
| `qwen3.5:9b` | 7.69 s | 8.07 s | 10.97 s |
| `qwen3-vl:4b-instruct` | 10.46 s | 10.67 s | 12.20 s |
| `gemma4:e2b` | 12.24 s | 12.52 s | 13.42 s |
| `qwen3-vl:8b-instruct` | 16.78 s | 17.20 s | 21.07 s |

The NVIDIA driver reported much more VRAM than Ollama `/api/ps`, especially for Gemma. I therefore use the driver figure when judging whether another model will fit.

| Model | Ollama `/api/ps` | Peak `nvidia-smi` |
|---|---:|---:|
| `qwen3-vl:4b-instruct` | 4.03 GiB | 5 236 MiB |
| `gemma4:e2b` | 0.36 GiB | 3 264 MiB |
| `qwen3.5:9b` | 5.35 GiB | 6 610 MiB |
| `qwen3-vl:8b-instruct` | 5.74 GiB | 7 244 MiB |

During hot text generation, the 8B averaged 92.9% total CPU use across the four vCPUs and 45.6% GPU use. The other models averaged roughly 61% to 63% CPU use. This suggests that my 8B path is strongly limited by CPU work or partial offload. The runner command does not expose the GPU-layer count, and the benchmark account could not read the Docker logs, so I could not confirm the exact cause.

## My real usage tests with Frigate and LLM Vision

I walked around the house and carried out normal activities while Frigate and Home Assistant LLM Vision were active. Ollama stayed on `qwen3-vl:8b-instruct` with an 8192-token context. I observed existing application traffic without adding benchmark requests during these sessions.

For the second session, I enabled the Ollama provider's debug logger in Frigate 0.18.2 to record image counts and request options:

```yaml
logger:
  logs:
    frigate.genai.plugins.ollama: debug
```

I matched those logs with Ollama's request timings. Times below are local to Europe/Paris on 9 November 2026. API durations are measured on the Ollama server; they do not include the full time until a result appears in an application.

| Session | Observed inference calls | HTTP 200 | HTTP 400 | Median successful API duration | Longest successful API duration |
|---|---:|---:|---:|---:|---:|
| First | 14 | 10 | 4 | 4.04 s | 17.27 s |
| Second| 8 | 6 | 2 | 9.335 s | 26.27 s |

These samples contain different requests, so I do not interpret the second session's higher median as a performance regression. HTTP 200 confirms a completed request, not an accurate description.

### What the Frigate debug logs showed

I matched six of the second session's eight calls to Frigate. The other two used `/api/chat`; I have not confirmed their source with Home Assistant logs.

| Frigate request | Images | Input tokens | API duration | Result |
|---|---:|---:|---:|---|
| Object description | 1 | 1 085 | 3.40 s | HTTP 200 |
| Structured review | 9 | 10 383 | 0.35 s | Context exceeded |
| Object description | 7 | 7 348 | 15.27 s | HTTP 200 |
| Object description | 6 | 6 370 | 26.27 s | HTTP 200 |
| Structured review | 37 | 39 541 | 2.14 s | Context exceeded |
| Structured review | 3 | 4 266 | 17.56 s | HTTP 200 |

Both rejected calls in this session came from structured reviews. Frigate sent `num_ctx: 8192`, `num_predict: 256`, `temperature: 0.1` and `keep_alive: 24h` on all six calls. The configured context matched the one loaded in Ollama, but the nine-image and 37-image requests exceeded it.

The first session's four rejected requests contained 10426, 36579, 18772 and 44636 tokens. Matching timestamps and token counts confirmed that these also came from Frigate. My longer Frigate log export contained 393 context-limit errors between 6 and 9 November, so the problem predates these tests.

The six-image request reached Frigate's provider at 17:23:00.508 and began in the Ollama runner at 17:23:11.826. Its runner processing took about 14.96 seconds. The roughly 11.3-second delay before processing is consistent with waiting behind the preceding request; I cannot assign every part of that delay to a specific stage from these logs alone.

### GPU readings and camera errors

I sampled `nvidia-smi` approximately every ten seconds during the real usage sessions. These are sampled maxima, so shorter peaks may have been missed.

| Reading | First session | Second session |
|---|---:|---:|
| VRAM throughout sampled readings | 7 296 MiB | 7 296 MiB |
| Highest sampled GPU utilisation | 100% | 100% |
| Highest sampled GPU temperature | 62 °C | 64 °C |
| Highest sampled GPU power | 125.21 W | 123.39 W |

The second Frigate export also showed three FFmpeg crashes across two camera streams, with hardware-frame synchronisation and download errors. My configuration uses `preset-vaapi`. These errors need a separate decoding investigation; the logs do not establish that Ollama caused them.

### What I plan to test next

I will temporarily disable `review.genai` on the cameras where it is enabled and measure object descriptions on their own. I will then compare single-image object descriptions using `use_snapshot: true`. That setting affects object descriptions, not the images sent for review summaries, and removes the movement information available from a sequence.

I am keeping the 8192-token context, one parallel request and one loaded model for that comparison. I have not applied the proposed review or snapshot changes yet. I also want to test decoding without hardware acceleration on one affected camera while watching CPU use.

I have kept private camera images, credentials and raw logs out of this wiki. These sessions measure request completion and latency; I have not scored the descriptions for accuracy or measured streaming TTFT.

## What I am running now

For now, I am keeping `qwen3-vl:8b-instruct` in production. It already works with Frigate and Home Assistant and finished second on the real photographs. Before replacing it, I want to repeat the comparison with a few anonymised camera frames.

`qwen3.5:9b` is my strongest replacement candidate. It scored 9/9, reached a median vision TTFT of 0.719 seconds and used 6 610 MiB of actual VRAM. The 8B used 7 244 MiB and reached 2.188 seconds. The 9B still missed `A7` in the synthetic OCR check, so I will test it with my actual Frigate and Home Assistant prompts before switching.

The 4B saves about 2 GiB compared with the 8B and stays fast after its first image, but its indoor-scene errors make it a weaker choice for my alerts. Gemma is the fastest and lightest model, but it had the lowest accuracy on the real photographs.

## My configuration choices

### Keep-alive

A 24-hour keep-alive saves me approximately 19 seconds whenever `qwen3-vl:8b-instruct` would otherwise need to reload. I keep it because the model is used regularly and the GPU remains stable.

Requests without an explicit `keep_alive` expired after about five minutes. I still need to check that my running Docker container has `OLLAMA_KEEP_ALIVE=24h`; otherwise the 24-hour lifetime may come only from a client request.

### Context

My server loaded `num_ctx=8192`. The controlled text prompts used 222 to 245 tokens, and the Qwen image prompt used up to 1079. My later Frigate tests reached 7348 input tokens on a successful request and exceeded 8192 on several rejected requests. I am keeping the current context while investigating image counts; reducing it to 4096 would reject some requests that currently succeed.

### Parallelism and memory

I keep `OLLAMA_NUM_PARALLEL` and `OLLAMA_MAX_LOADED_MODELS` at `1`. My `nvidia-smi` state showed `7296 MiB / 8192 MiB` in use, leaving `896 MiB` free. I do not have enough spare GPU memory for more parallel requests or another loaded model.

## What this benchmark does not prove

- I took one hot measurement and used one synthetic image per model. These results describe my test run, not a statistical average.
- My real-photo follow-up used three photographs and one prompt per photograph. That sample is too small to measure production accuracy.
- My repeated SSH run measured three hot text requests per model, but only one request for each photograph.
- My synthetic image test checks basic shapes, colours and OCR. The later household sessions measure real application traffic, but I have not evaluated the accuracy of their descriptions.
- I left Frigate and Home Assistant active, so their requests may have caused some contention.
- Each model uses a different tokenizer, so tokens per second do not compare useful text output exactly.
- During the first run, I could not read total GPU use for the alternative models without system access to `nvidia-smi`.

Official references:

- [Ollama API metrics](https://docs.ollama.com/api/usage)
- [Ollama Chat API](https://docs.ollama.com/api/chat)
- [Ollama FAQ: keep-alive, context and parallelism](https://github.com/ollama/ollama/blob/main/docs/faq.mdx)
- [Frigate GenAI documentation](https://docs.frigate.video/configuration/genai/genai_config/)
- [Frigate object descriptions and snapshot behaviour](https://docs.frigate.video/configuration/genai/genai_objects/)
- [Frigate GPU troubleshooting](https://docs.frigate.video/troubleshooting/gpu/)

