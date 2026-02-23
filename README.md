# tinyorin

Purpose-tuned AI inference — from GPU to FPGA to custom silicon.

[tinygrad](https://github.com/tinygrad/tinygrad) beat Qualcomm's SNPE by 2× on openpilot's supercombo (Snapdragon 845). When you optimize a compiler for a narrow workload on known silicon, you outperform vendor SDKs that must be general-purpose. We're applying that same idea across targets — and building toward a tape-out.

**[tinyorin](https://github.com/tinyorin/tinyorin)** is a fork of tinygrad, customized for AI inference on Jetson hardware. First target: beat TensorRT on Orin Nano 8GB — then scale across the Jetson family.

---

## Why This Works

TensorRT is intentionally conservative — fixed kernel libraries, heuristic scheduling, no per-shape PTX. It has to work well enough for every model on every GPU.

tinyorin doesn't. It ships custom PTX for *your* kernels on *your* hardware:

```
tinygrad scheduler → shape-aware kernel selection
                    ↓
custom PTX (per-shape fusion, WMMA for tensor cores)
                    ↓
driver JIT → SASS
```

On Orin Nano (1024 CUDA cores, 32 tensor cores), this means:
- Register blocking tuned for your exact conv/transpose shapes
- WMMA tensor core loops unrolled for your matmul dimensions
- Shared memory layouts fused for model-specific op sequences

TensorRT can't do this. It's the same gap tinygrad exploited on Qualcomm — except NVIDIA gives us better tools to do it. Nsight Compute, Nsight Systems, and full PTX/SASS documentation make every optimization measurable. On Qualcomm, tinygrad worked partially blind. On Jetson, we can see exactly where the cycles go.

---

## Repositories

| | |
|---|---|
| [tinyorin](https://github.com/tinyorin/tinyorin) | Fork of tinygrad — customized NV backend, shape-aware PTX kernel tuning |
| tinyzynq (next) | Custom HLS datapaths on Xilinx Kria SoM — toward tape-out |

---

## The Arc

```
Precedent — tinygrad on Qualcomm SD845
  Beat SNPE by 2× on openpilot supercombo
  Proved: workload-specific open compiler > general-purpose vendor SDK

tinyorin                          (active)
  Profile TRT → identify hot kernels → write shape-perfect PTX → fuse ops
  Target: 1.5-2× on Jetson Orin Nano 8GB

tinyzynq                          (next)
  First target Kria SoM, characterize custom HLS datapaths on FPGA
  Extract datapath requirements → HLS → RTL → characterize vs ASIC target

custom ASIC                       (target)
  HLS kernels → RTL → synthesis → PDK → tape-out
```

*You cannot design good silicon without first understanding why existing hardware is slow for your workload.* tinygrad on Qualcomm proved the compiler thesis. tinyorin builds that understanding on GPU; tinyzynq takes it to FPGA; then tape-out.

