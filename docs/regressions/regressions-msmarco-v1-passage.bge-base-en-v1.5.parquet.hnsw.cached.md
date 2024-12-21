# Anserini Regressions: MS MARCO Passage Ranking

**Model**: [BGE-base-en-v1.5](https://huggingface.co/BAAI/bge-base-en-v1.5) with HNSW indexes (using cached queries)

This page describes regression experiments, integrated into Anserini's regression testing framework, using the [BGE-base-en-v1.5](https://huggingface.co/BAAI/bge-base-en-v1.5) model on the [MS MARCO passage ranking task](https://github.com/microsoft/MSMARCO-Passage-Ranking), as described in the following paper:

> Shitao Xiao, Zheng Liu, Peitian Zhang, and Niklas Muennighoff. [C-Pack: Packaged Resources To Advance General Chinese Embedding.](https://arxiv.org/abs/2309.07597) _arXiv:2309.07597_, 2023.

In these experiments, we are using cached queries (i.e., cached results of query encoding).

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/msmarco-v1-passage.bge-base-en-v1.5.parquet.hnsw.cached.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/msmarco-v1-passage.bge-base-en-v1.5.parquet.hnsw.cached.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead and then run `bin/build.sh` to rebuild the documentation.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --index --verify --search --regression msmarco-v1-passage.bge-base-en-v1.5.parquet.hnsw.cached
```

We make available a version of the MS MARCO Passage Corpus that has already been encoded by the BGE-base-en-v1.5 model.

From any machine, the following command will download the corpus and perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --download --index --verify --search --regression msmarco-v1-passage.bge-base-en-v1.5.parquet.hnsw.cached
```

The `run_regression.py` script automates the following steps, but if you want to perform each step manually, simply copy/paste from the commands below and you'll obtain the same regression results.

## Corpus Download

Download the corpus and unpack into `collections/`:

```bash
wget https://rgw.cs.uwaterloo.ca/pyserini/data/msmarco-passage-bge-base-en-v1.5.parquet.tar -P collections/
tar xvf collections/msmarco-passage-bge-base-en-v1.5.parquet.tar -C collections/
```

To confirm, `msmarco-passage-bge-base-en-v1.5.parquet.tar` is 39 GB and has MD5 checksum `b235e19ec492c18a18057b30b8b23fd4`.
With the corpus downloaded, the following command will perform the remaining steps below:

```bash
python src/main/python/run_regression.py --index --verify --search --regression msmarco-v1-passage.bge-base-en-v1.5.parquet.hnsw.cached \
  --corpus-path collections/msmarco-passage-bge-base-en-v1.5.parquet
```

## Indexing

Sample indexing command, building HNSW indexes:

```bash
bin/run.sh io.anserini.index.IndexHnswDenseVectors \
  -threads 16 \
  -collection ParquetDenseVectorCollection \
  -input /path/to/msmarco-passage-bge-base-en-v1.5.parquet \
  -generator ParquetDenseVectorDocumentGenerator \
  -index indexes/lucene-hnsw.msmarco-v1-passage.bge-base-en-v1.5/ \
  -M 16 -efC 100 \
  >& logs/log.msmarco-passage-bge-base-en-v1.5.parquet &
```

The path `/path/to/msmarco-passage-bge-base-en-v1.5.parquet/` should point to the corpus downloaded above.
Upon completion, we should have an index with 8,841,823 documents.

## Retrieval

Topics and qrels are stored [here](https://github.com/castorini/anserini-tools/tree/master/topics-and-qrels), which is linked to the Anserini repo as a submodule.
The regression experiments here evaluate on the 6980 dev set questions; see [this page](../../docs/experiments-msmarco-passage.md) for more details.

After indexing has completed, you should be able to perform retrieval as follows using HNSW indexes:

```bash
bin/run.sh io.anserini.search.SearchHnswDenseVectors \
  -index indexes/lucene-hnsw.msmarco-v1-passage.bge-base-en-v1.5/ \
  -topics tools/topics-and-qrels/topics.msmarco-passage.dev-subset.bge-base-en-v1.5.jsonl.gz \
  -topicReader JsonIntVector \
  -output runs/run.msmarco-passage-bge-base-en-v1.5.parquet.bge-hnsw-cached.topics.msmarco-passage.dev-subset.bge-base-en-v1.5.jsonl.txt \
  -hits 1000 -efSearch 1000 -threads 16 &
```

Evaluation can be performed using `trec_eval`:

```bash
bin/trec_eval -c -m map tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-bge-base-en-v1.5.parquet.bge-hnsw-cached.topics.msmarco-passage.dev-subset.bge-base-en-v1.5.jsonl.txt
bin/trec_eval -c -M 10 -m recip_rank tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-bge-base-en-v1.5.parquet.bge-hnsw-cached.topics.msmarco-passage.dev-subset.bge-base-en-v1.5.jsonl.txt
bin/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-bge-base-en-v1.5.parquet.bge-hnsw-cached.topics.msmarco-passage.dev-subset.bge-base-en-v1.5.jsonl.txt
bin/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.msmarco-passage.dev-subset.txt runs/run.msmarco-passage-bge-base-en-v1.5.parquet.bge-hnsw-cached.topics.msmarco-passage.dev-subset.bge-base-en-v1.5.jsonl.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **AP@1000**                                                                                                  | **BGE-base-en-v1.5**|
|:-------------------------------------------------------------------------------------------------------------|-----------|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.364     |
| **RR@10**                                                                                                    | **BGE-base-en-v1.5**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.358     |
| **R@100**                                                                                                    | **BGE-base-en-v1.5**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.901     |
| **R@1000**                                                                                                   | **BGE-base-en-v1.5**|
| [MS MARCO Passage: Dev](https://github.com/microsoft/MSMARCO-Passage-Ranking)                                | 0.981     |

The above figures are from running brute-force search with cached queries on non-quantized **flat** indexes.
With cached queries on non-quantized HNSW indexes, observed results are likely to differ; scores may be lower by up to 0.01, sometimes more.
Note that HNSW indexing is non-deterministic (i.e., results may differ slightly between trials).

## Reproduction Log[*](../../docs/reproducibility.md)

To add to this reproduction log, modify [this template](../../src/main/resources/docgen/templates/msmarco-v1-passage.bge-base-en-v1.5.parquet.hnsw.cached.template) and run `bin/build.sh` to rebuild the documentation.