# Extremely Low-Cost Text Embeddings on Qualcomm® Cloud AI 100 DL2q Instances

Text embedding models map text into dense vectors for use in tasks such
as semantic similarity search, document retrieval, text classification
and clustering. Hence, they are well-suited for use with vector
databases for RAG-type applications. The two BGE (BAAI General
Embedding) models,
[BAAI/bge-large-en-v1.5](https://huggingface.co/BAAI/bge-large-en-v1.5)
and
[BAAI/bge-base-en-v1.5](https://huggingface.co/BAAI/bge-base-en-v1.5),
are recent and popular English-language text embedding models, ranked in
the top 10 of the [HuggingFace MTEB Leaderboard
\[1\]](https://huggingface.co/spaces/mteb/leaderboard) (as of 30 Jan
2024). [A recent article \[2\]
showcased](https://www.philschmid.de/inferentia2-embeddings) how one can
deploy the *bge-base-en-v1.5* embedding model on an *inf2.xlarge* EC2
instance for the cost of *\$0.006 per 1 million tokens*.

With an industry-leading performance per Watt, Qualcomm Cloud AI 100 can
run the BGE embedding models at an extremely low cost-per-token. Using
the recipe listed on [Cloud AI SDK
Github](https://github.com/quic/cloud-ai-sdk/tree/1.12/models/language_processing/encoder),
we can export, compile, and benchmark *bge-base-en-v1.5* and
*bge-large-en-v1.5* (and a whole line-up of other transformer
encoder-type models) with a single command. You can run inference on
these compiled models via command line or using our
[Python](https://github.com/quic/cloud-ai-sdk/tree/1.12/samples/python)
or
[C++](https://github.com/quic/cloud-ai-sdk/tree/1.12/samples/cpp/cpp_qpc_inference)
APIs.

Qualcomm Cloud AI 100 Standard inference accelerators are now generally
available via the [DL2q instances](https://aws.amazon.com/ec2/instance-types/dl2q/) on AWS EC2.
Each *dl2q.24xlarge* instance has 8 such accelerators, with a staggering
1.4 PetaFLOPS of FP16 performance, or 2.8 PetaOPS of INT8 performance.
Each accelerator has 14 AI cores. For best throughput performance, we
can compile each BGE embedding model for 2 cores with a batchsize of 1,
and serve 7 replicas per card, or 56 independent replicas per
*dl2q.24xlarge* instance. This simplifies and provides immense
flexibility for deployment at scale, since we don't have to deal with
batching of requests to achieve the highest performance.

The throughput and latency numbers for the two BGE embedding models are
shown in the table below, when compiled for the full sequence length of
512 defined by these models, without network overhead. The calculations
of price in \$/1M tokens is shown for both on-demand and 3-year
reserved, all upfront, pricing, based on the AWS EC2 US-West (Oregon)
region *(as of 30 Jan 2024)*.

| Pricing Model | Price ($/hr) | Embedding Model | Sequence Length (tokens) | Latency (ms) | Throughput (inferences/sec) | Total Throughput (tokens/sec) | M Tokens per $ | $ / 1M tokens | $ / 1B toeksn |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 3 yr, all up front | \$3.3537 | *BAAI/bge-base-en-v1.5* | 512 | 15.2 | 457 | 1,871,872 | 2,009 | \$0.000498 | \$0.498 |
|on-demand | \$8.9194 | *BAAI/bge-base-en-v1.5* | 512 | 15.2 | 457 | 1,871,872 | 756 | \$0.001324 | \$1.324 |
| 3 yr, all up front | \$3.3537 | *BAAI/bge-large-en-v1.5* | 512 | 62.6 | 111 | 455,885 | 489 | \$0.002043 | \$2.043 |
| on-demand | \$8.9194 | *BAAI/bge-large-en-v1.5* | 512 | 62.6 | 111 | 455,885 | 184 | \$0.005435 | \$5.435 |

Even with on-demand pricing, Qualcomm Cloud AI 100 offers the extremely
low-cost performance at **\$0.0013 per 1M tokens** for the
*bge-base-en-v1.5* model, and **\$0.0054 per 1M** tokens for the
bge-large-en-v1.5 model. For large-scale deployments, using the 3-year
reserved instances, we can get as low as **\$0.0005 per 1M tokens** and
**\$0.002 per 1M tokens**, respectively. In other words, **for less than
50 cents, you can embed 1 billion tokens** worth of text data!

## Conclusion

Embedding text at scale is crucial for many practical applications of
Generative AI, and the industry leading performance/Watt of Qualcomm
Cloud AI 100 delivers extremely low-cost text embedding with top
embedding models such as *bge-base-en-v1.5* and *bge-large-en-v1.5*,
offering as low as **\~\$0.0005 per 1M tokens** and **\~\$0.002 per 1M
tokens**, respectively.

## References:

\[1\] [HuggingFace Massive Text Embedding Benchmark (MTEB) Leaderboard](https://huggingface.co/spaces/mteb/leaderboard)

\[2\] [\"*Deploy Embedding Models on AWS inferentia2 with Amazon SageMaker*\"](https://www.philschmid.de/inferentia2-embeddings)
