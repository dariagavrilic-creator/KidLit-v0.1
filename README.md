# KidLit v0.1 — data and code

Reproducibility package for

> *KidLit by Numbers: Linguistic Profiles of Original and Translated Russian 0+
> Books.* Daria Gavrilik and Aleksandra Ostraia. Proceedings of the 9th
> International Conference on Natural Language and Speech Processing
> (ICNLSP 2026), Trento.

KidLit v0.1 is a balanced corpus of 100 contemporary Russian-language
children's books with "0+" age marker: 50 original works
(**RUS-O**) and 50 translations into Russian (**RUS-T**). The paper compares
the two subcorpora on 35 metrics in a main block and 17 in an extended
morphological block.

## The texts are not distributed

The 100 books are in copyright. We do not redistribute the texts. The restriction extends to any
representation from which the texts could be reconstructed, so no tokenised or
morphologically annotated form of the running text is included here either.

What is distributed is everything needed to check the analysis: the
bibliography of the corpus, the per-text value of every metric, the code that
produced those values, and the records of the manual validation. **Every
number in the paper can be recomputed from these files without access to the
texts.**

## Contents

```
data/
  metadata.csv              100 × 12   bibliographic metadata
  metrics_main.csv          100 × 81   per-text values, main block
  metrics_morph.csv         100 × 19   per-text values, morphological block
  results_main.csv           35 × 16   Table 3 of the paper
  results_morph.csv          17 × 16   Table 4 of the paper
  codebook.md                          data dictionary for all five files
notebooks/
  00_merge_source_tables.ipynb         corpus assembly
  01_metrics_rus_o.ipynb               metric computation, RUS-O
  02_metrics_rus_t.ipynb               metric computation, RUS-T
  03_comparative_statistics.ipynb      between-group tests and figures
  04_morphological_analysis.ipynb      extended morphological block
  05_reproduce_paper_tables.ipynb      results tables from the released data
validation/
  validation_results.csv               accuracy of each tool, per subcorpus
  annotation_records.csv               2,951 annotated units, labels only
  sample_manifest.csv                  which passage came from which text
  coordination_rules.csv               four coordination rules against gold
  coordination_recomputed.csv          the four rules over all 100 texts
  00_draw_sample.ipynb                 draws the sample, builds the sheets
  00b_pos_workbooks.ipynb              part-of-speech annotation sheets
  01_score_annotation.ipynb            scoring
```

## Reproducing the paper

`notebooks/05_reproduce_paper_tables.ipynb` reads `metrics_main.csv` and
`metrics_morph.csv` and regenerates `results_main.csv` and
`results_morph.csv` — Tables 3 and 4 — end to end. It needs only pandas,
numpy, scipy and statsmodels: no language models, no access to the books.

```bash
pip install pandas numpy scipy statsmodels jupyter
cd notebooks && jupyter nbconvert --to notebook --execute 05_reproduce_paper_tables.ipynb
```

Notebooks `00`–`04` document how the metrics were derived from the running
text. They cannot be re-run from this repository, since the texts are not
here; each one says so in its header.

## Notes on the data

**Join key.** Every table keys on `text_id` (`RUS-O-01` … `RUS-T-50`). In the
working files the two subcorpora were numbered independently from 1 to 50, so
the numeric part alone is not unique.

**Coordination.** The metric reported in the paper is
`predicate_coord_ratio`: the share of sentences containing a `conj` relation
whose dependent and head are both verbal. An earlier definition counting any
`conj` or `cc` relation is released as `coord_any_ratio` but is superseded —
manual validation showed it also fires on sentence-initial discourse
conjunctions, with precision .433 and unequal accuracy across subcorpora
(.64 against .84, p = .002). See `validation/coordination_rules.csv`.

**Part-of-speech shares.** A tag absent from a text yields a share of exactly
0. The working files stored those cases as empty cells; they are filled with 0
here.

**`personal_pron_ratio`** is identically zero in all 100 texts. This is a
pipeline fault — the `PronType` feature is not populated by
`ru_core_news_sm` — not a linguistic finding. It is released for transparency
and must not be used.

## Validation of the NLP pipeline

A stratified sample of 40 passages (200 sentences, 2,042 tokens), one per text
from 40 of the 100 books, was annotated manually and blind to the system
output. Full method and results are in `validation/README.md`.

| Task | n | Accuracy | RUS-O | RUS-T | p (symmetry) |
|---|---|---|---|---|---|
| Sentence segmentation (razdel) | 200 | 1.000 | 1.000 | 1.000 | — |
| Tokenisation (razdel) | 2,042 | .9985 | 1.000 | .9969 | .11 |
| POS tagging (spaCy) | 309 | .864 | .831 | .892 | .16 |

## Citation

See `CITATION.cff`.

## Licence

Data and documentation: CC BY 4.0 (`LICENSE-data`).
Code: MIT (`LICENSE`).

Neither licence extends to the underlying books, which are not part of this
release.
