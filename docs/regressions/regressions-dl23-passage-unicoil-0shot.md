# Anserini Regressions: TREC 2023 Deep Learning Track (Passage)

**Model**: uniCOIL (with doc2query-T5 expansions) zero-shot

This page describes baseline experiments, integrated into Anserini's regression testing framework, on the [TREC 2023 Deep Learning Track passage ranking task](https://trec.nist.gov/data/deep2023.html) using the MS MARCO V2 passage corpus.
Here, we cover experiments with the uniCOIL model trained on the MS MARCO V1 passage ranking test collection, applied in a zero-shot manner, with doc2query-T5 expansions.

The uniCOIL model is described in the following paper:

> Jimmy Lin and Xueguang Ma. [A Few Brief Notes on DeepImpact, COIL, and a Conceptual Framework for Information Retrieval Techniques.](https://arxiv.org/abs/2106.14807) _arXiv:2106.14807_.

For additional instructions on working with the MS MARCO V2 passage corpus, refer to [this page](../../docs/experiments-msmarco-v2.md).

Note that the NIST relevance judgments provide far more relevant passages per topic, unlike the "sparse" judgments provided by Microsoft (these are sometimes called "dense" judgments to emphasize this contrast).

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/dl23-passage-unicoil-0shot.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/dl23-passage-unicoil-0shot.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --index --verify --search --regression dl23-passage-unicoil-0shot
```

We make available a version of the corpus that has already been processed with uniCOIL, i.e., we have applied doc2query-T5 expansions, performed model inference on every document, and stored the output sparse vectors.
Thus, no neural inference is involved.

From any machine, the following command will download the corpus and perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --download --index --verify --search --regression dl23-passage-unicoil-0shot
```

The `run_regression.py` script automates the following steps, but if you want to perform each step manually, simply copy/paste from the commands below and you'll obtain the same regression results.

## Corpus Download

Download, unpack, and prepare the corpus:

```bash
# Download
wget https://rgw.cs.uwaterloo.ca/JIMMYLIN-bucket0/data/msmarco_v2_passage_unicoil_0shot.tar -P collections/

# Unpack
tar -xvf collections/msmarco_v2_passage_unicoil_0shot.tar -C collections/

# Rename (indexer is expecting corpus under a slightly different name)
mv collections/msmarco_v2_passage_unicoil_0shot collections/msmarco-v2-passage-unicoil-0shot
```

To confirm, `msmarco_v2_passage_unicoil_0shot.tar` is 41 GB and has an MD5 checksum of `1949a00bfd5e1f1a230a04bbc1f01539`.
With the corpus downloaded, the following command will perform the remaining steps below:

```bash
python src/main/python/run_regression.py --index --verify --search --regression dl23-passage-unicoil-0shot \
  --corpus-path collections/msmarco-v2-passage-unicoil-0shot
```

## Indexing

Sample indexing command:

```bash
bin/run.sh io.anserini.index.IndexCollection \
  -collection JsonVectorCollection \
  -input /path/to/msmarco-v2-passage-unicoil-0shot \
  -generator DefaultLuceneDocumentGenerator \
  -index indexes/lucene-index.msmarco-v2-passage-unicoil-0shot/ \
  -threads 24 -impact -pretokenized -storeRaw \
  >& logs/log.msmarco-v2-passage-unicoil-0shot &
```

The path `/path/to/msmarco-v2-passage-unicoil-0shot/` should point to the corpus downloaded above.

The important indexing options to note here are `-impact -pretokenized`: the first tells Anserini not to encode BM25 doclengths into Lucene's norms (which is the default) and the second option says not to apply any additional tokenization on the uniCOIL tokens.
Upon completion, we should have an index with 138,364,198 documents.

For additional details, see explanation of [common indexing options](../../docs/common-indexing-options.md).

## Retrieval

Topics and qrels are stored [here](https://github.com/castorini/anserini-tools/tree/master/topics-and-qrels), which is linked to the Anserini repo as a submodule.
The regression experiments here evaluate on the 82 topics for which NIST has provided judgments as part of the [TREC 2023 Deep Learning Track](https://trec.nist.gov/data/deep2023.html).

After indexing has completed, you should be able to perform retrieval as follows:

```bash
bin/run.sh io.anserini.search.SearchCollection \
  -index indexes/lucene-index.msmarco-v2-passage-unicoil-0shot/ \
  -topics tools/topics-and-qrels/topics.dl23.unicoil.0shot.tsv.gz \
  -topicReader TsvInt \
  -output runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q.topics.dl23.unicoil.0shot.txt \
  -impact -pretokenized &

bin/run.sh io.anserini.search.SearchCollection \
  -index indexes/lucene-index.msmarco-v2-passage-unicoil-0shot/ \
  -topics tools/topics-and-qrels/topics.dl23.unicoil.0shot.tsv.gz \
  -topicReader TsvInt \
  -output runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rm3.topics.dl23.unicoil.0shot.txt \
  -impact -pretokenized -rm3 -collection JsonVectorCollection &

bin/run.sh io.anserini.search.SearchCollection \
  -index indexes/lucene-index.msmarco-v2-passage-unicoil-0shot/ \
  -topics tools/topics-and-qrels/topics.dl23.unicoil.0shot.tsv.gz \
  -topicReader TsvInt \
  -output runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rocchio.topics.dl23.unicoil.0shot.txt \
  -impact -pretokenized -rocchio -collection JsonVectorCollection &
```

Evaluation can be performed using `trec_eval`:

```bash
bin/trec_eval -c -M 100 -m map -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -M 100 -m recip_rank -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m ndcg_cut.10 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m recall.100 -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m recall.1000 -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q.topics.dl23.unicoil.0shot.txt

bin/trec_eval -c -M 100 -m map -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rm3.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -M 100 -m recip_rank -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rm3.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m ndcg_cut.10 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rm3.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m recall.100 -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rm3.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m recall.1000 -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rm3.topics.dl23.unicoil.0shot.txt

bin/trec_eval -c -M 100 -m map -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rocchio.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -M 100 -m recip_rank -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rocchio.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m ndcg_cut.10 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rocchio.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m recall.100 -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rocchio.topics.dl23.unicoil.0shot.txt
bin/trec_eval -c -m recall.1000 -l 2 tools/topics-and-qrels/qrels.dl23-passage.txt runs/run.msmarco-v2-passage-unicoil-0shot.unicoil-0shot-cached_q+rocchio.topics.dl23.unicoil.0shot.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **MAP@100**                                                                                                  | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
|:-------------------------------------------------------------------------------------------------------------|-----------|-----------|-----------|
| [DL23 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.1437    | 0.1363    | 0.1491    |
| **MRR@100**                                                                                                  | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
| [DL23 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.6424    | 0.5697    | 0.6385    |
| **nDCG@10**                                                                                                  | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
| [DL23 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.3855    | 0.3776    | 0.3938    |
| **R@100**                                                                                                    | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
| [DL23 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.3293    | 0.3126    | 0.3351    |
| **R@1000**                                                                                                   | **uniCOIL (with doc2query-T5) zero-shot**| **+RM3**  | **+Rocchio**|
| [DL23 (Passage)](https://microsoft.github.io/msmarco/TREC-Deep-Learning)                                     | 0.5541    | 0.5541    | 0.5742    |