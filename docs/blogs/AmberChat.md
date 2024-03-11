# Accelerate Inference of Fully Transparent Open-Source LLMs from LLM360 on Qualcomm® Cloud AI 100 DL2q Instances

Many popular Large Language Models (LLMs) are closed and/or limited by
their licensing to specific use-cases, which limits the democratization
of AI progress. [LLM360](https://www.llm360.ai/) is a joint effort
of Petuum, [MBZUAI](https://mbzuai.ac.ae/), and Cerebras, with a focus
on providing fully accessible and open-source LLMs.
[Amber](https://huggingface.co/LLM360/Amber) and
[AmberChat](https://huggingface.co/LLM360/AmberChat) are two such
open-source LLMs released under the LLM360 initiative.
[Amber-7B](https://huggingface.co/LLM360/Amber) is a foundation English
language model, following the Llama2-7B architecture, and
[AmberChat-7B](https://huggingface.co/LLM360/AmberChat) is the
instruction fine-tuned version for chat applications.

As part of Qualcomm Technologies' "Developer First" strategy and in
general support of open-source initiatives, the Qualcomm Cloud AI 100
inference accelerator now supports the Amber and AmberChat models, which
are accelerated using microscaling formats and on-device caching of
key-value tensors. You can run these models on Qualcomm Cloud AI 100
Standard cards using the [DL2q
instances](https://aws.amazon.com/ec2/instance-types/dl2q/) on AWS EC2
in a few simple steps. Check out the [model recipe on our Qualcomm Cloud
AI 100 Github
page](https://github.com/quic/cloud-ai-sdk/tree/1.12/models/language_processing/decoder/LlamaForCausalLM)
to run the Amber or AmberChat models.

### Qualcomm Cloud AI 100 Optimizations for LLM Inference:

A variety of compute and memory optimizations in both the hardware and
software deliver the best-in-class performance-per-TCO\$ for LLMs on
Qualcomm Cloud AI 100. While these are covered in earlier articles
[\[2\]](https://developer.qualcomm.com/blog/power-efficient-acceleration-large-language-models-qualcomm-cloud-ai-sdk)
[\[3\]](https://developer.qualcomm.com/blog/train-anywhere-infer-qualcomm-cloud-ai-100),
two key optimizations to highlight specifically for these models are:

-   **Microscaling Format:** The model weights are compressed to MXFP6
    using the new microscaling formats [(Open Compute Project MX
    Specifications
    \[4\])](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf)
    to reduce weight memory footprint and to boost the effective memory
    bandwidth of the system. The computations are performed in FP16. For
    more details, check out our [blog on Microscaling Formats
    \[5\]](https://developer.qualcomm.com/blog/qualcomm-cloud-ai-100-accelerates-large-language-model-inference-2x-using-microscaling-mx)

-   **On-device Caching** of key-value tensors for past tokens over the
    model's entire context length to:

    * Avoid re-computation during the autoregressive generation in LLMs
    and
    * Avoid movement of numerous large key-value tensors back and forth
    between host and device.

As LLM inference especially in the decode phase is primarily
memory-bound, these optimizations significantly boost the LLM
performance and improve the end-user experience in an inference
application deployed using these LLMs.

### Performance numbers on Qualcomm Cloud AI 100 Standard SKU:

The performance of AmberChat-7B on a single Cloud AI 100 Standard SKU
for a batchsize of 1 is listed in the table below. Note that Amber and
AmberChat have the same architecture, so their performance should be
identical. A single DL2q instance contains 8 Standard SKU cards, so we
can run 8 replicas of these models on a single instance, 1 on each
card. We can further increase the throughput performance by
increasing the batchsize.


| Model | Device | Precision<br>of Weights | Replicas|Batch Size | Compiled<br>Prompt Length | Maximum<br>Context Length | Prefill<br>Latency (sec) | Decode<br>Throughput (tok/s) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| AmberChat-7B | 1x STD SKU | MXFP6 | 1 | 1 | 128 | 2048 | 1.33 | 14.9 |
| | 8x STD SKU DL2q | MXFP6 | 8 | 1 | 128 | 2048 | 1.33 | 119.2 |

### Sample Prompts and Generated Responses:

Here are some sample prompts and the corresponding responses generated
by AmberChat-7B. We used the prompt template from the model's
HuggingFace Model Card:

**Prompt Template:**

A chat between a curious human and an artificial intelligence assistant.
The assistant gives helpful, detailed, and polite answers to the
human's questions.

\### Human: {prompt}

\### Assistant:

**Prompt:** How do I mount a TV to drywall safely?

**Response:** To mount a TV to dry wall safely, you will need the
following tools:

1. A drill
2. Wall anchors
3. Screws
4. A stud finder
5. A level
6. A screwdriver
7. A stud mounting bracket
8. A TV mounting bracket

Here are the steps to mount a TV to dry wall safely:

1. Use a stud finder to locate the studs in the wall where you want to mount the TV.
2. Use a level to make sure the TV is level before mounting it.
3. Drill holes in the dry wall where the wall anchors will go.
4. Insert the wall anchors into the holes and screw them into the studs.
5. Use screws to secure the TV mounting bracket to the wall anchors.
6. Place the TV on the mounting bracket and adjust the level until it is straight.
7. Secure the TV to the mounting bracket with screws.
8. Use a screwdriver to tighten the screws and make sure the TV is secure.

It is important to make sure the TV is securely mounted to the wall to prevent it from falling and causing injury or damage.

<br>

**Prompt:** Happy is to sad as calm is to \_.

**Response:** Angry.

<br>

**Prompt:** Got any creative ideas for a 10 year old's birthday?

**Response:** Sure, how about a DIY spa day at home? The birthday girl
can invite her friends over and each girl can create their own spa
treatment using items like face masks, bath salts, and nail polish. You
could even set up a DIY manicure and pedicure station with nail polish
and nail art supplies. The girls can also play relaxing music and enjoy
some snacks while they pamper themselves. It's a fun and creative way
to celebrate a birthday!

### References:

\[1\] Z. Liu, et al. "[*LLM360: Towards Fully Transparent Open-Source
LLMs*](https://arxiv.org/abs/2312.06550)", arXiv preprint
arXiv:2312.06550, 2023.

\[2\] *"Power-efficient acceleration for large language models --
Qualcomm Cloud AI SDK"*,
<https://developer.qualcomm.com/blog/power-efficient-acceleration-large-language-models-qualcomm-cloud-ai-sdk>

\[3\] "*Train anywhere, Infer on Qualcomm Cloud AI 100*",
<https://developer.qualcomm.com/blog/train-anywhere-infer-qualcomm-cloud-ai-100>

\[4\] Open Compute Project (OCP) Microscaling Formats (MX)
Specifications.
<https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf>

\[5\] "*Qualcomm Cloud AI 100 Accelerates Large Language Model Inference
by \~2x Using Microscaling (Mx) Formats*",
<https://developer.qualcomm.com/blog/qualcomm-cloud-ai-100-accelerates-large-language-model-inference-2x-using-microscaling-mx>

*Qualcomm branded products are products of Qualcomm Technologies, Inc. and/or its subsidiaries.*