# Codebook — KidLit v0.1

Data dictionary for the five files in `data/`. Sections in order:
`metadata.csv`, `metrics_main.csv`, `metrics_morph.csv`, and the two results
tables.

---

## `metadata.csv`

Bibliographic metadata for the **KidLit v0.1** corpus: 100 contemporary
Russian-language children's books carrying the publisher's "0+" age marker,
balanced by origin type (50 original works, 50 translations into Russian).

This file accompanies the paper *KidLit by Numbers: Linguistic Profiles of
Original and Translated Russian 0+ Books* (ICNLSP 2026).

### File format

| Property | Value |
|---|---|
| Format | CSV, comma-separated, `"` quoting |
| Encoding | UTF-8, no BOM |
| Line endings | LF (`\n`) |
| Header | Yes, first row |
| Rows | 100 (one per text) |
| Columns | 12 |
| Missing values | Empty field (no `NA`, `NULL` or `-` placeholders) |

**Language convention.** Column names and all controlled-vocabulary values are
in English. Proper names — titles, authors, translators, publishers — are given
in their original script and are not transliterated. Russian titles, author
names and publisher names therefore appear in Cyrillic; original titles of
translated works appear in the script of the source language.

### Columns

#### `text_id`
**Type:** string · **Missing:** none · **Unique:** yes (primary key)

Stable identifier of the text, of the form `{origin_type}-{NN}`, where `NN` is a
zero-padded two-digit number running from `01` to `50` within each subcorpus —
e.g. `RUS-O-07`, `RUS-T-42`.

Use this column to join `metadata.csv` with the per-text metric files. The
numeric part alone is **not** unique across the corpus: it repeats between the
two subcorpora, so always join on the full `text_id`.

#### `origin_type`
**Type:** categorical · **Missing:** none

Origin status of the text; the grouping variable for all comparisons reported in
the paper.

| Value | n | Meaning |
|---|---|---|
| `RUS-O` | 50 | Original work written in Russian by a Russian-language author |
| `RUS-T` | 50 | Russian translation of a foreign-language work, produced by a professional translator for the Russian book market |

#### `title`
**Type:** string · **Missing:** none

Title of the Russian edition, in Cyrillic, as printed on the title page. All 100
values are distinct.

#### `original_title`
**Type:** string · **Missing:** 58 (all 50 `RUS-O` rows + 8 `RUS-T` rows)

Title of the source-language work, in the script of the source language.

Empty by definition for `RUS-O`. Empty for eight `RUS-T` rows where the Russian
edition does not print the original title and it could not be verified from an
independent source; see `data_quality_report`. Capitalisation follows the source
edition and is therefore inconsistent across rows (some titles are set in full
capitals on the cover); it has not been normalised.

#### `author`
**Type:** string · **Missing:** none

Author of the work, in Cyrillic. For translations this is the author of the
source-language original, transcribed into Cyrillic as given in the Russian
edition — not the translator.

Multiple authors are separated by `, ` within a single field. 43 distinct
authors across the 50 `RUS-O` texts and 44 across the 50 `RUS-T` texts: some
authors contribute more than one text, so rows are not fully independent for
modelling purposes.

Name order follows the Russian edition and is therefore not uniform: most values
are given first-name-first, a minority surname-first.

#### `translator`
**Type:** string · **Missing:** 52 (all 50 `RUS-O` rows + 2 `RUS-T` rows)

Translator of the Russian edition, in Cyrillic.

Empty by definition for `RUS-O`. Empty for two `RUS-T` rows where the edition
does not credit a translator. 35 distinct translators cover the 48 credited
translations; several translators contribute more than one text.

#### `source_language`
**Type:** categorical · **Missing:** none

Language the text was written in. `Russian` for every `RUS-O` row by definition.
For `RUS-T`, the language of the source work — seven languages in total:

| Value | n (`RUS-T`) |
|---|---|
| `English` | 24 |
| `French` | 10 |
| `German` | 8 |
| `Italian` | 4 |
| `Polish` | 2 |
| `Dutch` | 1 |
| `Finnish` | 1 |

#### `year`
**Type:** integer · **Missing:** none · **Range:** 2015–2026

Year of first publication for `RUS-O`; year the source text was created for
`RUS-T`. Not the year of the Russian edition used for digitisation.

The two subcorpora differ on this variable: `RUS-O` texts are on average more
recent (mean 2021.2, range 2015–2026) than `RUS-T` texts (mean 2018.4, range
2015–2025). The corpus is not balanced on publication year.

#### `publisher`
**Type:** string · **Missing:** none

Publisher of the Russian edition, in Cyrillic (Latin where the imprint itself
uses Latin script). Values have been normalised so that one imprint has one
spelling — e.g. `АСТ` rather than the mixture of `АСТ` and `Издательство АСТ`
found in the working file.

40 distinct publishers overall: 26 in `RUS-O`, 21 in `RUS-T`, with 7 imprints
appearing in both subcorpora. The largest single share is 7 of 50 texts
(`Поляндрия`, in `RUS-T`).

#### `age_marker`
**Type:** categorical · **Missing:** none

Publisher's age marker. Constant `0+` across the corpus — this is an inclusion
criterion, not a variable. Retained so the file is self-describing.

Texts whose edition carries an equivalent formulation rather than the literal
symbol ("для дошкольного возраста", "для чтения взрослыми детям") are recorded
as `0+`.

#### `genre`
**Type:** categorical · **Missing:** none

Broad form of the text.

| Value | `RUS-O` | `RUS-T` | Meaning |
|---|---|---|---|
| `prose` | 46 | 43 | Continuous prose narrative |
| `verse` | 4 | 7 | Text set in verse |

A coarse two-way split; it does not distinguish finer subcategories such as
fairy tale, story or didactic book.

#### `n_words`
**Type:** integer · **Missing:** none

Word-token count of the full text, produced by the tokenisation step of the
processing pipeline (`razdel`), counting word tokens only and excluding
punctuation. This is the same quantity reported as "word count" in the paper.

| Statistic | `RUS-O` | `RUS-T` |
|---|---|---|
| Mean | 2067.0 | 698.1 |
| Median | 1345.5 | 546.5 |
| Minimum | 188 | 81 |
| Maximum | 11461 | 3819 |

Included here as a convenience field so that basic corpus description does not
require loading the metric matrices; it is duplicated in the main metric file.

### Notes on scope

The full texts of the 100 books are under copyright and are **not** distributed.
This release contains bibliographic metadata and derived quantitative measures
only, which is sufficient to reproduce every statistic reported in the paper but
does not permit reconstruction of the texts.


---

## `metrics_main.csv`

Per-text values of the main metric block: 100 rows, 81 columns
(`text_id`, `origin_type`, and 79 metrics). Same file conventions as
`metadata.csv` — UTF-8 without BOM, comma-separated, LF line endings, header
row, no missing values anywhere.

Join to `metadata.csv` on `text_id`.

35 of the 79 metrics are the main block reported in Table 3 of the paper; they
are marked **[paper]** below. The remaining 44 were computed by the same
pipeline and are released as well: they cost nothing to distribute and support
reanalysis, but they were not part of the tested family and carry no adjusted
p-values.

### Key columns

| Column | Description |
|---|---|
| `text_id` | Primary key, matches `metadata.csv` (`RUS-O-01` … `RUS-T-50`) |
| `origin_type` | `RUS-O` or `RUS-T`, 50 each |

### Volume and counts

| Column | Description |
|---|---|
| `n_words` | **[paper]** Word tokens, punctuation excluded |
| `n_sentences` | **[paper]** Sentences after `razdel` segmentation |
| `pages` | **[paper]** Physical pages of the printed edition, from the metadata — not a text-derived quantity |
| `n_chars` | Characters including spaces |
| `n_chars_no_spaces` | Characters excluding spaces |
| `n_unique_words` | Distinct word forms |
| `n_syllables` | Syllables, counted with `pyphen` |
| `n_lex_words` | Content-word tokens (function words removed) |
| `n_lex_unique` | Distinct content-word forms |
| `n_monosyllabic` / `n_polysyllabic` | Tokens of one syllable / three or more |
| `n_short_sents` / `n_long_sents` | Sentences below / above the pipeline's length thresholds |

### Word and sentence length

| Column | Description |
|---|---|
| `avg_word_len` | **[paper]** Mean word length in characters |
| `avg_syllables` | **[paper]** Mean syllables per word |
| `avg_sent_words` | **[paper]** Mean sentence length in words |
| `avg_sent_chars` | **[paper]** Mean sentence length in characters |
| `med_word_len`, `med_sent_len`, `min_sent_len`, `max_sent_len` | Median and extreme values of the same quantities |

### Lexical diversity

| Column | Description |
|---|---|
| `ttr` | **[paper]** Type-token ratio over the whole text |
| `sttr` | **[paper]** Standardised TTR, 100-token window |
| `hapax_ratio` | **[paper]** Share of word forms occurring exactly once |
| `yule_k` | **[paper]** Yule's K, concentration of frequent vocabulary |
| `shannon_h` | **[paper]** Shannon entropy of the word-form distribution, bits |
| `top1000_ratio` | **[paper]** Share of tokens in the 1000 most frequent words of Russian (Lyashevskaya & Sharov 2009) |
| `hapax_count` | Absolute number of hapax legomena |
| `hapax_dis_ratio` | Share of word forms occurring exactly twice |
| `simpson_d` | Simpson's diversity index |

### Readability

| Column | Description |
|---|---|
| `flesch_ru` | **[paper]** Flesch reading ease, Russian adaptation — higher means easier |
| `fog_index` | **[paper]** Gunning FOG — higher means harder |
| `ari` | **[paper]** Automated Readability Index — higher means harder |
| `coleman_liau` | **[paper]** Coleman-Liau — higher means harder |
| `flesch_kincaid`, `smog` | Two further indices, computed but not reported in the paper |

### Syntax

| Column | Description |
|---|---|
| `avg_tree_depth` | **[paper]** Mean depth of the dependency tree |
| `predicate_coord_ratio` | **[paper]** Share of sentences containing a `conj` relation whose dependent and head are both verbal. This is the predicate-coordination ratio reported in the paper |
| `coord_any_ratio` | **Superseded.** Share of sentences containing any `conj` or `cc` relation — the definition used in an earlier version of the analysis. Manual validation showed it also fires on sentence-initial discourse conjunctions (*а*, *и*, *но*), giving a precision of .433 and, critically, unequal accuracy across subcorpora (.64 against .84, p = .002). Released for transparency; not part of the tested family |
| `sent_initial_conj_ratio` | Share of sentences opening with *а*, *и* or *но*. Computed while diagnosing the metric above; not reported in the paper |
| `max_tree_depth` | Maximum dependency depth in the text |
| `avg_dependents` | Mean number of dependents per token. This is the "dependents per token" mentioned in Section 6 of the paper but absent from Table 3 |

### Part-of-speech profile

Shares of word tokens carrying each Universal Dependencies tag, as assigned by
`ru_core_news_sm`. Within a text the `pos_*` columns sum to 1.

A tag that does not occur in a text yields a share of exactly `0`. In the
working files such cases were stored as empty cells rather than zeros; they
have been filled with `0` in this release. `pos_punct_ratio` from the working
files is **not** included: it was present for only one of the two subcorpora
and non-empty for 2 of 100 texts, and represents punctuation tags leaking into
a word-token ratio.

| Column | Description |
|---|---|
| `pos_noun_ratio` | **[paper]** Nouns |
| `pos_verb_ratio` | **[paper]** Verbs |
| `pos_adj_ratio` | **[paper]** Adjectives |
| `pos_adv_ratio` | **[paper]** Adverbs |
| `pos_propn_ratio` | **[paper]** Proper nouns |
| `pos_pron_ratio` | **[paper]** Pronouns |
| `animate_noun_ratio` | **[paper]** Share of animate nouns among nouns |
| `personal_pron_ratio` | **[paper]** Personal pronouns. **Identically zero in all 100 texts.** This is a pipeline fault, not a linguistic finding: the paper reports the metric as returning zero and omits it from Table 3. It is released for transparency and must not be used |
| `pos_adp_ratio`, `pos_sconj_ratio`, `pos_cconj_ratio`, `pos_part_ratio`, `pos_det_ratio`, `pos_aux_ratio`, `pos_num_ratio`, `pos_intj_ratio`, `pos_x_ratio` | Remaining UD tags: adpositions, subordinating and coordinating conjunctions, particles, determiners, auxiliaries, numerals, interjections, other |

### Grammar

| Column | Description |
|---|---|
| `past_tense_ratio`, `present_tense_ratio`, `future_tense_ratio` | **[paper]** Shares of finite verbs in each tense |
| `imperative_ratio` | **[paper]** Share of verbs in the imperative |
| `quest_sent_ratio` | **[paper]** Share of interrogative sentences |
| `excl_sent_ratio` | **[paper]** Share of exclamatory sentences |
| `active_voice_ratio` | **[paper]** Share of verb forms in the active voice |
| `passive_voice_ratio` | Share in the passive voice |

### Punctuation

| Column | Description |
|---|---|
| `punct_diversity` | **[paper]** Normalised diversity of punctuation marks used |
| `punct_dot_ratio`, `punct_comma_ratio`, `punct_excl_ratio`, `punct_quest_ratio`, `punct_ellipsis_ratio`, `punct_semicolon_ratio`, `punct_colon_ratio`, `punct_dash_ratio`, `punct_hyphen_ratio`, `punct_quote_open_ratio`, `punct_quote_close_ratio` | Share of each mark among all punctuation in the text |

---

## `metrics_morph.csv`

Per-text values of the extended morphological block: 100 rows, 19 columns
(`text_id`, `origin_type`, 17 indices), computed with `ruts`. No missing
values. Join on `text_id`.

All 17 are reported in Table 4 of the paper. Column names are English; the
Russian names used in the working files are given for traceability.

| Column | Russian source name | Description |
|---|---|---|
| `analyticity_index` | Индекс аналитичности/автосемантичности | Share of function words |
| `verbality_index` | Индекс глагольности | Share of verbs |
| `substantivity_index` | Индекс субстантивности | Share of nouns |
| `pronominality_index` | Индекс местоименности | Share of pronouns |
| `adjectivity_index` | Индекс адъективности | Share of adjectives |
| `nominal_vocab_index` | Индекс именной лексики | Share of nominal vocabulary |
| `noun_verb_ratio` | Соотношение имённости-глагольности | Nouns divided by verbs |
| `genitive_ratio` | Доля словоформ в родительном падеже | Word forms in the genitive |
| `instrumental_ratio` | Доля словоформ в творительном падеже | Word forms in the instrumental |
| `short_adj_ratio` | Доля кратких прилагательных | Short-form adjectives |
| `full_participle_ratio` | Доля полных причастий | Full participles |
| `short_participle_ratio` | Доля кратких причастий | Short participles |
| `predicative_ratio` | Доля предикативов | Predicatives |
| `gerund_ratio` | Доля деепричастий | Gerunds |
| `infinitive_ratio` | Доля инфинитивов | Infinitives |
| `numeral_ratio` | Доля числительных | Numerals |
| `particle_ratio` | Доля частиц | Particles |

---

## `results_main.csv` and `results_morph.csv`

Between-group test results. `results_main.csv` has 35 rows and corresponds to
Table 3 of the paper; `results_morph.csv` has 17 rows and corresponds to
Table 4. Both are regenerated end to end by
`notebooks/05_reproduce_paper_tables.ipynb` from the two metric matrices.

| Column | Description |
|---|---|
| `group` | Metric group as organised in Section 4.2 (`results_main.csv` only) |
| `metric_id` | Column name in the corresponding metric matrix — the join key |
| `metric_label` | Human-readable label as printed in the paper |
| `n_rus_o`, `n_rus_t` | Texts entering the test, 50 and 50 throughout |
| `mean_rus_o`, `mean_rus_t` | Group means |
| `sd_rus_o`, `sd_rus_t` | Group standard deviations, `ddof = 1` |
| `t_statistic` | Two-sample *t* statistic (`results_morph.csv` only) |
| `cohens_d` | Effect size with pooled SD; positive means higher in RUS-O |
| `p_ttest` | Two-sample *t*-test, uncorrected |
| `p_mwu` | Mann-Whitney U test, uncorrected |
| `p_adj_bh` | Benjamini-Hochberg adjusted p, FDR q = 0.05 |
| `p_adj_holm` | Holm adjusted p, family-wise error rate |
| `significance_bh` | `***` p < .001, `**` p < .01, `*` p < .05, `ns` otherwise, on `p_adj_bh` |
| `higher_in` | `RUS-O` or `RUS-T` |

The primary test is Mann-Whitney for the main block and the *t*-test for the
morphological block, matching the paper; the other p-value is supplied for
comparison. Corrections are applied independently within each file: 35 tests in
one family, 17 in the other.

`personal_pron_ratio` is degenerate — identical zeros in both groups — so its
p-values are set to 1.0. It is kept inside the correction family because the
published analysis kept it there. Dropping it would rescale every adjusted
p-value in `results_main.csv` by 34/35 and would change no conclusion.
