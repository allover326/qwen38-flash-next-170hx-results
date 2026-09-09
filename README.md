# Qwen3.8-Flash-Next on 2× CMP 170HX

`Qwen/Qwen3.8-Flash-Next` — **180B total parameters, 6B activated** — served on **two**
community-unlocked NVIDIA CMP 170HX cards.

GA100 silicon, sm_80, 64 GiB HBM2e per card. **No fp8 tensor cores. No NVLink. Two cards.**

> ### 107 tok/s decode · ~5,900 tok/s prefill · 262,144-token context · 457 tok/s at 8 streams

---

## Decode

| | tok/s |
|---|---|
| single stream | **107** |
| without speculative decoding | 73 |

## Prefill — flat across the entire context

No decay from 21k to 249k tokens:

| prompt tokens | tok/s |
|---|---|
| 21,051 | 5,926 |
| 85,969 | 6,059 |
| 171,870 | 5,914 |
| **249,247** | **5,781** |

## Concurrency

| streams | aggregate tok/s | per-stream tok/s |
|---|---|---|
| 1 | 117 | 126 |
| 2 | 156 | 86 |
| 4 | 281 | 88 |
| 8 | **457** | 65 |

Per-stream throughput holds **51%** of single-stream at 8 concurrent requests.

## Context and KV

| | |
|---|---|
| context length | **262,144** (full open-weights native) |
| KV pool | **682,666 tokens** |
| concurrent full-context requests | **2.6×** |
| needle recall | **3/3 at 248,989 real prompt tokens** (5%, 50%, 95% depth) |

A KV pool large enough to *hold* N tokens is not the same claim as *retrieving* from depth N.
This is the second one.

## Footprint

~**99 GiB** of weights resident across the two cards — a 180B-parameter model, its 51B-parameter
n-gram embedding table, a vision tower and a speculative-decoding head, on 128 GiB of total VRAM.

---

Ampere has no fp8 tensor cores, and this architecture was not designed with 64 GiB cards in mind.
Getting here took work on both the inference runtime and the checkpoint itself. **That
implementation is not published.**

## Caveats

- Speed, footprint and stability only — quality has not been benchmarked against another model.
- 262,144 is the open-weights native context; the hosted API offers more. Extension beyond native
  length is untested here.
- One hardware configuration, one set of runs. Nothing here is claimed to generalize.

## License

Measurements, CC0. Model weights are under the Qwen Community License 1.0.
