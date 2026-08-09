# Validation of the NLP pipeline on 0+ material

The tools used to compute the metrics — `razdel` for tokenisation and sentence
segmentation, spaCy `ru_core_news_sm` for tagging and parsing — are trained on
general Russian corpora, predominantly adult news and fiction. This directory
documents a manual check of how they behave on 0+ children's literature and
bounds the effect of their errors on the results reported in the paper.

## Sample

40 passages of 5 consecutive sentences, one passage per text, drawn from 40 of
the 100 books: 200 sentences, 2,042 tokens, 309 of them tagged for part of
speech.

Allocation is equal between subcorpora (20 passages each) rather than
proportional to their size, because the statistic that matters is whether
accuracy differs between RUS-O and RUS-T, and that comparison is most powerful
with balanced groups. Verse is oversampled (4 passages per subcorpus against a
5.6% share of the corpus) because line breaks without terminal punctuation are
the hard case for sentence segmentation.

Passages start no closer than 3 sentences to either end of a text, so that
dedications, chapter headings and closing formulae are excluded.

`sample_manifest.csv` records which passage came from which text and over which
sentence indices.

## Procedure

One annotator, working blind to the system output. Judgements were made in
three passes over the same sentences — boundaries, then tokens, then
part of speech and coordination — so that each pass could rely on the
decisions of the previous one.

Because there was a single annotator, no inter-annotator agreement is
reported.

## Decision rules

Gold labels follow UD conventions for Russian, since that is what the models
are trained on. The rules below cover the cases UD leaves open or that are
specific to this register.

**Sentence boundaries.** A line break in verse is not a boundary unless
punctuation marks one. Direct speech introduced by a dash ends where the
reported clause ends. An ellipsis opens a new sentence only if the next word
begins a new predication. A run of interjections (*Бух! Бах!*) is a sequence of
sentences.

**Tokenisation.** Hyphenated single words are one token (*из-за*, *кто-то*,
*кап-кап*); hyphenated appositions are two (*заяц-беляк*). Diminutives are one
token regardless of suffix chain. Lengthened onomatopoeia (*мяяяяу*) is one
token.

**Part of speech.** Onomatopoeia is `INTJ`. A diminutive takes the part of
speech of its base: *зайчонок* is `NOUN`. Invented character names are `PROPN`.
`X` is used only where the part of speech is genuinely undetermined.

**Coordination.** Two independent judgements per sentence. `coord_any`: any
coordination is present, phrasal included (*мама и папа*). `coord_clause`: two
or more clauses are coordinated, each with its own subject. Homogeneous
predicates under one subject (*встал и пошёл*) count as `coord_any` but not as
`coord_clause`.

## Results

`validation_results.csv`. `p_symmetry` tests whether accuracy differs between
the two subcorpora; a large value means the tool is equally reliable on both,
so the contrasts reported in the paper cannot be an artefact of unequal
annotation quality.

| Task | n | Accuracy | 95% CI | RUS-O | RUS-T | p (symmetry) |
|---|---|---|---|---|---|---|
| Sentence segmentation | 200 | 1.000 | — | 1.000 | 1.000 | — |
| Tokenisation | 2,042 | .9985 | [.9962, 1.000] | 1.000 | .9969 | .11 |
| POS tagging | 309 | .864 | [.828, .897] | .831 | .892 | .16 |
| POS, UD-ambiguous pairs allowed | 309 | .922 | — | .901 | .940 | .29 |
| Coordination flag, superseded rule | 200 | .740 | [.680, .795] | .640 | .840 | **.002** |

Confidence intervals are bootstrapped by resampling texts, not rows: tokens
within a passage are not independent.

### Segmentation and tokenisation

All 200 sentence boundaries were correct, including the 40 verse sentences.
2,039 of 2,042 tokens were correct; all three errors were substitutions rather
than splits, so the total token count matched the gold count exactly. The
volume, lexical-diversity and readability metrics therefore rest on a layer
with an error rate below 0.2%.

### Part of speech

86.4% strict, 92.2% if pairs left unsettled by UD conventions for Russian are
treated as acceptable variants: predicatives tagged `ADV`, `ADJ` or `VERB`
(*жарко*, *можно*, *надо*) and determiner-pronouns tagged `DET` or `PRON`.
Those pairs account for 18 of the 42 errors.

The most frequent substantive error is characteristic of the genre:
sentence-initial capitalised common nouns naming animal or object protagonists
are read as proper nouns (*Кораблик*, *Котята*). Accuracy on the 8 tokens the
annotator flagged as diminutives is 5/8, with every error of that kind; the 2
onomatopoeia tokens are too few to estimate anything.

Recomputing the POS ratios from the gold annotation shifts them by at most
.023, and the between-group differences are essentially unchanged — nouns
−.087 against −.086, adjectives +.017 against +.019. The null results of the
POS block cannot be attributed to tagging noise.

### Coordination

Validation led to one metric being redefined. `coordination_rules.csv` compares
four candidate rules against the manual annotation on the same 200 sentences.

| Rule | Accuracy | Precision | Recall | p (symmetry) |
|---|---|---|---|---|
| A. any `conj` or `cc` (superseded) | .740 | .433 | .975 | **.002** |
| B. A, excluding sentence-initial discourse conjunctions | .805 | .507 | .950 | **.004** |
| **C. predicate coordination (adopted)** | **.875** | **.692** | .675 | **.67** |
| D. clause coordination with distinct subjects | .790 | .417 | .125 | .39 |

Rule A recalls almost everything but is imprecise: it also fires on
sentence-initial *А*, *И*, *Но*, which the parser attaches to the root as
`cc`, and on asyndetic enumeration. Sentence-initial discourse conjunctions
account for 43% of its false positives, and they are more frequent in RUS-O
than in RUS-T, which is why its accuracy is unequal across subcorpora in the
direction of the reported effect.

Rule C — `conj` with verbal dependent and head — is more accurate, far more
precise, and equally reliable on both subcorpora. `coordination_recomputed.csv`
gives all four rules over all 100 texts; under rule C the contrast is .251
against .177, d = +0.79, p_adj = .002, marginally stronger than under rule A.

Coordination of full clauses, each with its own subject, occurs in about 3% of
sentences and at the same rate in both subcorpora, so it is too rare to serve
as a metric on this material.

## Scope

The validation covers sentence segmentation, tokenisation, part-of-speech
tagging and the coordination metric. The 17 indices of the extended
morphological block are produced by a separate analyser and were not
validated.

The sample is sized to estimate tool accuracy, not to re-test the between-group
contrasts; with 100 sentences per subcorpus it is not powered to reproduce them
independently.

## Files

`annotation_records.csv` holds one row per annotated unit with the gold label,
the system prediction and the flags, but no word forms, so accuracy and
confidence intervals can be recomputed without exposing the texts. The
annotation sheets themselves contain running text and are not distributed.

The notebooks are the tooling: `00` draws the sample and builds the annotation
sheets, `00b` builds the part-of-speech sheets, `01` scores the completed
annotation.
