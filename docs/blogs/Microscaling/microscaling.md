# Accelerate Large Language Model Inference by ~2x Using Microscaling (Mx) Formats
Posted By Colin Verrilli

*MxFP, defined by the Microscaling Formats (Mx) Alliance, is enabled on DL2q instance of AWS EC2 and is evaluated on several large language models.*

## Large Language Model Challenges
When performing inference on large language models (LLMs), the generation of tokens (beyond the first) requires DRAM memory bandwidth that far exceeds the required compute. Each subsequent token that is generated requires reading all the model weights from DRAM. In addition, the remembered state (KV cache) of the prompt processing and of all previously generated tokens must be read from DRAM. For this reason, these workloads are severely memory bandwidth constrained in the subsequent token generation phase. The more memory bandwidth available, the faster the generation can occur.

In addition to bandwidth, memory *capacity* can also be a constraining factor in deploying LLMs. The largest of these models have hundreds of billions of parameters which easily exceeds the capacity of inference accelerators. These large models are typically split across several accelerator cards in such a way that each card will hold a portion of the model weights. The storing of KV cache also requires more DRAM capacity. The capacity requirements increase linearly with model size (d_model), context length and batch size.

A third performance limiter with LLM inference is communication bandwidth. Since these workloads are split across many cards, intermediate activations must be broadcast between the cards several times during the execution of each decoder block of every token inference. The size of the data to be communicated is proportional to the model size (d_model and number of decoder blocks), the batch size and the sequence length. The communication impact is especially important during prompt processing where the data for many tokens need to be communicated. In this case, the ratio of communication to DRAM access is much higher.

The three above performance constraints can all be addressed with data compression techniques. Weight compression reduces the required bandwidth to read the weights. It also can reduce the footprint of the weights in DRAM. Activation compression can reduce the amount of actual data that needs to be communicated over the inter-card links.

## Microscaling Formats
Earlier this year, AMD, Arm, Intel, Meta, Microsoft, NVIDIA, and Qualcomm Technologies, Inc. formed the Microscaling Formats (MX) Alliance with the goal of creating and standardizing next-generation 6- and 4-bit data types for AI training and inferencing. The key enabling technology that enables sub 8-bit formats to work, referred to as *microscaling*, builds on a foundation of years of design space exploration and research. MX enhances the robustness and ease-of-use of existing 8-bit formats such as FP8 and INT8, thus lowering the barrier for broader adoption of single digit bit training and inference.

The initial [MX specification](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf) introduces four concrete floating point and integer-based data formats (MXFP8, MXFP6, MXFP4, and MXINT8) that are compatible with current AI stacks, support implementation flexibility across both hardware and software, and enable fine-grain microscaling at the hardware level. [Extensive studies](https://arxiv.org/abs/2310.10537) demonstrate that MX formats can be easily deployed for many diverse real-world cases such as large language models, computer vision, and recommender systems. MX technology also enables LLM pre-training at 6- and 4-bit precisions without any modifications to conventional training recipes.

One of the primary intentions of MX is to allow vendors to build more power and area efficient hardware for both inference and training accelerators. This is a long-term goal, however, a short-term benefit of this standard and the associated data science is that weights and activations can be "direct cast" (compressed / quantized) into these formats post-training to address the issues described above.

## Qualcomm Cloud AI 100 and MX
Qualcomm's Cloud AI 100 inference accelerator provides flexible programming that permits adaptation to a wide range of new workloads and new acceleration techniques. One such technique recently implemented is the use of MXFP6 as a weight compression format. When the user selects this compilation option, the compiler will automatically compress the weights from FP32 or FP16 into MXFP6 format during the offline compilation phase. Compressing in this form saves 61% of the size of the weights and thus reduces the pressure on DRAM capacity.

At inference run time, the Qualcomm Cloud AI 100 performs on-the-fly decompression in software using its vector engine with an optimized decompression kernel. decompression can be performed in parallel with the weight fetching and computations, so the overhead is mostly hidden. Again, up to 61% of the DRAM bandwidth can be saved. After decompression, the calculations are performed as before in FP16 precision. The use of FP16 is acceptable since the LLMs still remain DRAM constrained so that the compute is not a bottleneck. FP16 also allows to retain the higher precision activations which overcomes loss of accuracy from the quantization.

Not yet available, but in-plan is the use of MXINT8 for compression and decompression of activations that need to be communicated over inter-card links. MXINT8 has the property that FP16 can generally be "direct cast" to MXINT8 without overall loss in accuracy of the workload (see referenced [white paper](https://arxiv.org/abs/2310.10537)). In cases where the link bandwidth is constrained, the benefit gained in reduced transfer latencies overcomes the overhead of compressing and decompressing in software. This feature is expected to make a significant improvement in time-to-first-token latencies for large models and for large prompts.

## Experimental Results
Use of MXFP6 for weight compression allows the Qualcomm Cloud AI 100 to have all the benefits of reduced precision (like FP8 or INT8) without the need for special hardware, without the loss of workload accuracy and without the need to modify the training process. It is entirely transparent to the user.

Table 1 shows experimental results for various workloads on the Qualcomm Cloud AI 100 Standard card. The throughput improvement when using MXFP6 weights is given. For the 13B model, without MXFP6 the network will not fit, but with MXFP6, the model fits and benefits from the performance improvements.

*Table 1 - Qualcomm Cloud AI 100 STD Throughput Improvement*

| Model | BS | PL | GL | CL | FP16 <br />(tok/sec) | MXFP6 <br />(tok/sec) | MXFP6 % Gain <br />over FP16 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| gpt-j-6b | 1 | 1024 | 1024 | 2048 | 8.08 | 13.9 | 72.03 |
| opt-2.7b | 1 | 1024 | 1024 | 2048 | 15.44 | 27.72 | 79.53 |
| opt-13b | 1 | 1024 | 1024 | 2048 | n/a | 6.98 | n/a |
 

<br />

Table 2 shows experimental results for various workloads on the Qualcomm Cloud AI 100 Ultra card. The throughput improvement when using MXFP6 weights is given.  
<br />

*Table 2 - Qualcomm Cloud AI 100 Ultra Throughput Improvement*

| Model | BS | PL | GL | CL | FP16 <br />(tok/sec) | MXFP6 <br />(tok/sec) | MXFP6 % Gain <br />over FP16 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| opt-6.7b | 1 | 1024 | 1024 | 2048 | 19.49 | 41.29 | 111.85 |
| Llama2-7b | 1 | 1024 | 1024 | 2048 | 26.49 | 43.78 | 65.27 |
| xgen-7b | 1 | 1024 | 1024 | 2048 | 26.5 | 43.46 | 64 |
| gpt-j-6b | 1 | 1024 | 1024 | 2048 | 31.14 | 50.99 | 63.74 |
| opt-2.7b | 1 | 1024 | 1024 | 2048 | 64.17 | 104.25 | 62.46 |


The MXFP6 weight compression feature is available in the 1.12 release of the Qualcomm Cloud AI 100 software and runs on a DL2q instance in AWS EC2 today. MxFP delivers up to 79% increase in tokens per second for 2K context length and enables models such as OPT-13B to fit on a single card in the DLq2 instance. Each DL2q instance can run 8 instances of OPT-13B.

[Visit Qualcomm® Cloud AI 100 Github](https://github.qualcomm.com/qranium/cloud-ai) to learn more and download SDK.


<br />

*Snapdragon and Qualcomm branded products are products of Qualcomm Technologies, Inc. and/or its subsidiaries.*


