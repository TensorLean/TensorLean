<p align="center">
  <img alt="TensorLean: Formal Verification of Production GPU Kernels" src="assets/tensorlean-banner.png#gh-light-mode-only" width="71%">
  <img alt="TensorLean: Formal Verification of Production GPU Kernels" src="assets/tensorlean-banner-dark.png#gh-dark-mode-only" width="71%">
</p>

---

TensorLean is the first framework for formally verifying production GPU kernels. Developers write a kernel's tensor-level specification, and TensorLean proves that the kernel's PTX code is data-race free, terminates, and is functionally correct against the specification within floating-point tolerance, on every input that satisfies the specification's preconditions and in every execution.

For more details, please refer to our blog at [tensorlean.ai](https://tensorlean.ai).

## Release

We will soon release our paper and open-source TensorLean, including its PTX semantics and the specifications and proofs of the verified kernels.

## Correctness Bugs Found by TensorLean

With TensorLean, we verified 149 open-source kernel instances, including 102 from FlashAttention, proving 83 of them correct and finding bugs in the other 66. Here, we share 39 correctness bugs of various types that TensorLean's verification found in production kernel libraries. All have been reported to developers.

| Library | Issue / PR | Bug Type | Bug Description |
|---|---|---|---|
| [FlashAttention‑4](https://github.com/Dao-AILab/flash-attention) | [#3165](https://github.com/Dao-AILab/flash-attention/issues/3165) | Memory access error | `flash_attn_varlen_func` with a paged KV cache whose page table has no column crashes with an illegal memory access (SM90) |
|  | [#3166](https://github.com/Dao-AILab/flash-attention/issues/3166) | Memory access error | Paged KV cache with `page_table.shape[1] * page_size >= 2**31` crashes with an illegal memory access (SM90): the forward kernel's key count overflows 32 bits |
|  | [#3195](https://github.com/Dao-AILab/flash-attention/issues/3195) / [#3196](https://github.com/Dao-AILab/flash-attention/pull/3196) | Data race | `compute_block_sparsity`: data race on `reduction_buffer` can drop blocks with unmasked keys |
|  | [#3197](https://github.com/Dao-AILab/flash-attention/issues/3197) | Undefined behavior | SM100 backward and MLA forward initialize mbarriers from every lane of a warp (`mbarrier_init` without `elect_one`) |
| [FlashAttention‑3](https://github.com/Dao-AILab/flash-attention) | [#3014](https://github.com/Dao-AILab/flash-attention/issues/3014) / [#3015](https://github.com/Dao-AILab/flash-attention/pull/3015) | Memory access error | Silent data corruption of KV cache in `flash_attn_with_kvcache` with non-interleaved rotary (head dim not 64/96/128/192/256) |
|  | [#3104](https://github.com/Dao-AILab/flash-attention/issues/3104) / [#3105](https://github.com/Dao-AILab/flash-attention/pull/3105) | Uninitialized memory, Memory access error, Data race | Cross-CTA data race and incorrect output and `softmax_lse` in split-KV `flash_attn_varlen_func` with `seqused_q` but no `cu_seqlens_q` |
|  | [#3136](https://github.com/Dao-AILab/flash-attention/issues/3136) / [#3137](https://github.com/Dao-AILab/flash-attention/pull/3137) | Uninitialized memory | `flash_attn_with_kvcache` does not append the new keys that no query attends to |
|  | [#3138](https://github.com/Dao-AILab/flash-attention/issues/3138) | Memory access error | `flash_attn_combine` crashes with "misaligned address" on `out_partial` / `out` views that pass its checks |
|  | [#3142](https://github.com/Dao-AILab/flash-attention/issues/3142) | Memory access error | `flash_attn_with_kvcache` with an empty page table silently out-of-bound reads, and returns a wrong result |
|  | [#3151](https://github.com/Dao-AILab/flash-attention/issues/3151) / [#3152](https://github.com/Dao-AILab/flash-attention/pull/3152) | Memory access error, Undefined behavior | Backward: `int` overflow in the fp32 accumulators' strides, sizes and block offsets (illegal memory access at batch 2 × 32 heads × 512K tokens) |
|  | [#3184](https://github.com/Dao-AILab/flash-attention/issues/3184) | Undefined behavior | `prepare_varlen_num_blocks_kernel` divides by zero when no batch has a key |
|  | [#3192](https://github.com/Dao-AILab/flash-attention/issues/3192) | Undefined behavior | Combine: `int` overflow in `ceil_div(seqlen * num_heads, kBlockM)` leaves `out` and `softmax_lse` unwritten, with no error |
|  | [#3193](https://github.com/Dao-AILab/flash-attention/issues/3193) | Deadlock | Latent deadlock in the Hopper forward kernel: missing `__syncwarp()` after the producer warp's `barrier_O.wait` |
|  | [#3194](https://github.com/Dao-AILab/flash-attention/issues/3194) | Uninitialized memory, Undefined behavior | `flash_attn_combine` silently returns without writing `out` or `softmax_lse` when its kernel fails to launch (e.g. batch size > 65535) |
| [FlashAttention‑2](https://github.com/Dao-AILab/flash-attention) | [#3001](https://github.com/Dao-AILab/flash-attention/pull/3001) | Incorrect algorithm implementation, Numerical bug | Incorrect `softmax_lse` of causal ALiBi attention in `flash_fwd_kernel` and `flash_fwd_splitkv_kernel` |
|  | [#3004](https://github.com/Dao-AILab/flash-attention/pull/3004) | Uninitialized memory, Memory access error | Memory corruption and incorrect `softmax_lse` when `flash_fwd_splitkv_kernel` early returns |
|  | [#3039](https://github.com/Dao-AILab/flash-attention/issues/3039) / [#3040](https://github.com/Dao-AILab/flash-attention/pull/3040) | Memory access error | Incorrect `softmax_lse` from `flash_attn_varlen_func` in GQA decoding with paged KV |
|  | [#3075](https://github.com/Dao-AILab/flash-attention/issues/3075) | Memory access error | `flash_attn_with_kvcache`: wrong output with cache_leftpad and no cache_seqlens (even-MN kernels attend past each sequence's end) |
|  | [#3096](https://github.com/Dao-AILab/flash-attention/issues/3096) | Numerical bug, Memory access error | 32-bit offsets overflow: the LSE, the split-KV accumulators, the split-KV key/value step, the combine kernel's rows |
|  | [#3097](https://github.com/Dao-AILab/flash-attention/issues/3097) | Memory access error | `flash_attn_with_kvcache`: the length of `cache_batch_idx` is not checked |
|  | [#3162](https://github.com/Dao-AILab/flash-attention/issues/3162) | Numerical bug, Memory access error | 32-bit offsets of `softmax_d` and `softmax_lse` overflow in the backward kernels and the forward kernel |
|  | [#3190](https://github.com/Dao-AILab/flash-attention/issues/3190) | Memory access error | The split-KV combine kernel stores `softmax_lse` past its end when `b * h * seqlen_q` is not a multiple of its block size |
|  | [#3191](https://github.com/Dao-AILab/flash-attention/issues/3191) | Numerical bug | `out` is NaN and `softmax_lse` is `inf` in rows with a masked key when `softmax_scale` is not positive or is below ~8.15e-39 |
| [DeepGEMM](https://github.com/deepseek-ai/DeepGEMM) | [#476](https://github.com/deepseek-ai/DeepGEMM/issues/476) / [#477](https://github.com/deepseek-ai/DeepGEMM/pull/477) | Incorrect algorithm implementation | SM90 `fp8_einsum("bhr,hdr->bhd", a, b, d, c)` silently drops `c` and returns `a @ b.T` |
|  | [#490](https://github.com/deepseek-ai/DeepGEMM/issues/490) | Numerical bug, Memory access error | 32-bit scale indices wrap in `pack_fp32_into_ue8m0` (reads) and `transpose_fp32` (writes) |
|  | [#514](https://github.com/deepseek-ai/DeepGEMM/issues/514) / [#515](https://github.com/deepseek-ai/DeepGEMM/pull/515) | Memory access error, Data race | SM90 masked grouped GEMMs write a group's partial last M block into the next group's rows, racing with that group's stores (wrong `d` when `M` is not a multiple of `BLOCK_M`) |
| [Triton](https://github.com/triton-lang/triton) | [#12226](https://github.com/triton-lang/triton/pull/12226) | Numerical bug | FP16 underflow and overflow of E8M0 scales in `_upcast_from_mxfp` |
| [TileLang](https://github.com/tile-ai/tilelang) | [#2699](https://github.com/tile-ai/tilelang/issues/2699) / [#2700](https://github.com/tile-ai/tilelang/pull/2700) | Data race, Miscompilation | `ThreadSync` missed `st.bulk`, causing a data race and wrong result in `chunk_intra_token_parallel` kernel on Blackwell (`sm_120`) |
|  | [#2703](https://github.com/tile-ai/tilelang/issues/2703) | Data race, Miscompilation | `ThreadSync` missed st.bulk, causing a data race and wrong result in `deepseek_nsa_mean_pooling_fwd` kernel from `TileOPs` on Blackwell (sm_120) |
| [Liger Kernel](https://github.com/linkedin/Liger-Kernel) | [#1525](https://github.com/linkedin/Liger-Kernel/pull/1525) | Incorrect algorithm implementation, Numerical bug | RuntimeError on rows wider than 65536 and NaN in `_softmax_multi_block_forward_kernel` |
|  | [#1570](https://github.com/linkedin/Liger-Kernel/issues/1570) | Numerical bug, Memory access error | Experimental embedding kernels overflow int32 row offsets (illegal memory access past 2^31 output elements) |
|  | [#1573](https://github.com/linkedin/Liger-Kernel/issues/1573) | Numerical bug, Memory access error | Row loops `for i in range(0, n_cols, BLOCK_SIZE)` wrap their int32 counter when `n_cols` is just below 2^31 (illegal memory access) |
|  | [#1658](https://github.com/linkedin/Liger-Kernel/issues/1658) | Numerical bug, Uninitialized memory, Memory access error | `_l2_swizzle` in mlp.py counts tiles in int32 (wrong tile mapping when `S` or `hidden_dim` is within a block of 2^31) |
|  | [#1659](https://github.com/linkedin/Liger-Kernel/issues/1659) | Numerical bug, Memory access error | Multi-token attention mask kernels overflow int32 batch offsets (`batch_id * stride_b`) past 2^31 |
|  | [#1660](https://github.com/linkedin/Liger-Kernel/issues/1660) | Numerical bug | Cross entropy, fused CE+TVD and GRPO's log-softmax return NaN when a row's first block of logits is all `-inf` |
|  | [#1661](https://github.com/linkedin/Liger-Kernel/issues/1661) | Numerical bug | Vocab-parallel CE treats every target of the rank whose shard crosses 2^31 as off-rank (`vocab_start + n_cols` wraps in int32) |
| [TileOPs](https://github.com/tile-ai/TileOPs) | [#1584](https://github.com/tile-ai/TileOPs/issues/1584) | Data race | MHCPostKernel: shared-memory write-write race due to wrong stride in x_out_shared indexing |
|  | [#1727](https://github.com/tile-ai/TileOPs/issues/1727) / [#1728](https://github.com/tile-ai/TileOPs/pull/1728) | Data race | BatchNormFwdTrainKernel: intra-block global memory read-write race cause by missing barrier |
|  | [#1729](https://github.com/tile-ai/TileOPs/issues/1729) / [#1730](https://github.com/tile-ai/TileOPs/pull/1730) | Data race | EngramGateConvBwdKernel: inter-block global memory read-write race |

## Contact

contact@tensorlean.ai
