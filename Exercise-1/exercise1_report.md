# Exercise 1: Tokenization and Subword Vocabulary Analysis

**Course:** Natural Language Processing
**Dataset:** Cebuano Bible (`ceb_bible.xlsx`) — 105,722 rows, ~4.57M characters of raw text

---

## Task 1: Text Normalization

Before tokenization, the raw corpus was cleaned through the following steps:

**1. Row Filtering**
The dataset contains mixed content: actual verse text, verse number markers (e.g., `1`, `17-18`), footnotes prefixed with `#`, and `NaN` rows. A filtering function discarded all non-text rows using regex and null checks, retaining only genuine Cebuano verse text.

**Justification:** Verse numbers and footnotes are metadata, not natural language. Including them would pollute the vocabulary with digits and English glosses.

**2. Lowercasing**
All text was converted to lowercase.

**Justification:** Cebuano is not case-sensitive in meaning. Treating `Dios` and `dios` as distinct tokens would artificially inflate vocabulary size without linguistic benefit.

**3. Unicode NFC Normalization**
Applied `unicodedata.normalize('NFC', text)`.

**Justification:** Ensures that characters with diacritics (e.g., accented vowels) are represented consistently as a single composed code point, preventing duplicate tokens caused by different Unicode encodings of the same character.

**4. Punctuation Removal**
Removed all non-word, non-whitespace characters via `re.sub(r'[^\w\s]', '', text)`.

**Justification:** Punctuation marks (periods, commas, quotation marks, colons) carry syntactic information but are not relevant for vocabulary or subword analysis in this exercise.

**5. Digit Removal**
Removed all remaining digit characters via `re.sub(r'\d+', '', text)`.

**Justification:** Any residual digits (verse number fragments embedded in text) do not belong to the Cebuano lexicon and would add noise.

**6. Whitespace Normalization**
Collapsed multiple whitespace characters into single spaces and stripped leading/trailing whitespace.

**Justification:** Ensures clean, consistent word boundaries for whitespace-based tokenization.

---

## Task 2: Vocabulary Creation

After normalization, the corpus was split on whitespace to produce word tokens.

| Metric                              | Value   |
| ----------------------------------- | ------- |
| Total word tokens                   | 777,442 |
| Vocabulary size (unique word types) | 18,416  |

The type-token ratio (TTR) is approximately **2.37%**, indicating high lexical repetition consistent with a religious text using a limited set of recurring words (e.g., names, conjunctions, divine titles).

---

## Task 3: Content Word Analysis

Function words (pronouns, conjunctions, prepositions, particles) were excluded using a manually constructed Cebuano stopword list. Content words were identified as all remaining tokens with length > 2.

### Top 10 Most Frequent Content Words

| Rank | Word   | Count | Relative Frequency |
| ---- | ------ | ----- | ------------------ |
| 1    | iyang  | 7,754 | 0.9974%            |
| 2    | ginoo  | 6,969 | 0.8964%            |
| 3    | dios   | 6,140 | 0.7898%            |
| 4    | dili   | 5,596 | 0.7198%            |
| 5    | tawo   | 4,722 | 0.6074%            |
| 6    | akong  | 4,520 | 0.5814%            |
| 7    | ilang  | 4,361 | 0.5609%            |
| 8    | inyong | 3,475 | 0.4470%            |
| 9    | gayod  | 3,459 | 0.4449%            |
| 10   | imong  | 3,413 | 0.4390%            |

The dominance of *ginoo* ("Lord") and *dios* ("God") confirms the religious domain of the corpus. Possessive pronouns (*iyang*, *akong*, *ilang*, *inyong*, *imong*) appear frequently due to the narrative and instructional nature of biblical text.

### Bottom 10 Least Frequent Content Words

| Rank | Word      | Count | Relative Frequency |
| ---- | --------- | ----- | ------------------ |
| 1    | matingob  | 1     | 0.000129%          |
| 2    | tanommga  | 1     | 0.000129%          |
| 3    | sanay     | 1     | 0.000129%          |
| 4    | dumalaha  | 1     | 0.000129%          |
| 5    | mugna     | 1     | 0.000129%          |
| 6    | klasi     | 1     | 0.000129%          |
| 7    | nagaturok | 1     | 0.000129%          |
| 8    | nagpabasa | 1     | 0.000129%          |
| 9    | nagasanga | 1     | 0.000129%          |
| 10   | pishon    | 1     | 0.000129%          |

These hapax legomena (words appearing exactly once) include morphologically derived verbs (*nagaturok*, *nagasanga*, *nagpabasa*), archaic terms (*mugna*), loanwords (*klasi*, *pishon*), and one apparent tokenization artifact (*tanommga*, a split-error from a line break). These are strong candidates for Task 6's rare word analysis.

---

## Task 4: Research on Subword Tokenization Methods

### 4.1 Byte Pair Encoding (BPE)

**Developers:** Rico Sennrich, Barry Haddow, and Alexandra Birch (University of Edinburgh, 2016).
**Used in:** GPT-2, GPT-3, GPT-4, RoBERTa.

**Main Idea**
BPE originates from a data compression algorithm. In tokenization, the training corpus is first split into individual characters. The algorithm then iteratively identifies the most frequent adjacent pair of symbols (characters or sub-strings), merges them into a single new token, and records that merge rule. This process repeats for a fixed number of merge operations (which determines vocabulary size). During inference, the learned merge rules are applied in order to segment new words.

**Procedure**

1. Initialize vocabulary with all unique characters plus a special end-of-word symbol.
2. Count all adjacent symbol pairs across the corpus.
3. Merge the most frequent pair into a new symbol.
4. Repeat steps 2-3 until the target vocabulary size is reached.
5. At inference time, apply merge rules greedily in the learned order.

**Limitations**

- The merging is greedy and deterministic — it always produces the same segmentation, which can be suboptimal for low-resource or morphologically rich languages.
- The final vocabulary is heavily influenced by corpus frequency, so rare words or novel words may be split into many small pieces.
- Does not model uncertainty over possible segmentations.

**Reference:** Sennrich, R., Haddow, B., & Birch, A. (2016). Neural machine translation of rare words with subword units. *ACL 2016*.

---

### 4.2 WordPiece

**Developers:** Mike Schuster and Kaisuke Nakamura (Google, 2012); later adapted by Jacob Devlin et al. for BERT (2019).
**Used in:** BERT, DistilBERT, ELECTRA, mBERT.

**Main Idea**
WordPiece is similar to BPE in that it iteratively merges symbol pairs, but the selection criterion differs: instead of choosing the most frequent pair, WordPiece selects the pair whose merge maximizes the likelihood of the training corpus under a unigram language model. Non-initial subwords are prefixed with `##` to indicate they are continuations of a word.

**Procedure**

1. Initialize vocabulary with all characters.
2. For each candidate pair, compute the score: `freq(AB) / (freq(A) * freq(B))`.
3. Merge the pair with the highest score.
4. Repeat until vocabulary size is reached.
5. At inference, apply a greedy longest-match-first strategy from left to right within each word.

**Limitations**

- The `##` prefix convention is language-model-specific and not portable across frameworks without post-processing.
- Longest-match-first inference is greedy and not globally optimal.
- The likelihood-based criterion can be slower to compute than frequency-based BPE.

**Reference:** Devlin, J., Chang, M., Lee, K., & Toutanova, K. (2019). BERT: Pre-training of deep bidirectional transformers for language understanding. *NAACL 2019*.

---

### 4.3 Unigram Language Model

**Developer:** Taku Kudo (Google, 2018).
**Used in:** SentencePiece (XLNet, ALBERT, T5, mBART, multilingual NMT systems).

**Main Idea**
Unlike BPE and WordPiece which build vocabulary bottom-up, the Unigram model starts with a large candidate vocabulary and prunes it. It models tokenization as a probabilistic process: given a word, every possible segmentation has a probability equal to the product of its subword token probabilities (assuming independence). The algorithm iteratively removes tokens whose removal causes the least decrease in total corpus likelihood, until the target vocabulary size is reached. At inference, the most probable segmentation (via the Viterbi algorithm) is used.

**Procedure**

1. Initialize a large vocabulary (e.g., all substrings up to a length limit).
2. Use the Expectation-Maximization (EM) algorithm to estimate token probabilities.
3. Compute the loss increase if each token is removed.
4. Remove the bottom p% of tokens (those whose removal hurts likelihood least).
5. Repeat until vocabulary size is reached.
6. At inference, decode using Viterbi to find the highest-probability segmentation.

**Limitations**

- More computationally expensive to train than BPE due to EM iterations.
- The probabilistic model assumes token independence, which is linguistically unrealistic.
- The initial large vocabulary and pruning strategy may behave poorly on very small corpora.

**Reference:** Kudo, T. (2018). Subword regularization: Improving neural network translation models with multiple subword candidates. *ACL 2018*.

---

## Task 5: Subword Tokenization

All three tokenizers were trained using the Hugging Face `tokenizers` library on the preprocessed Cebuano corpus with a target vocabulary size of 1,000 tokens. This compact vocabulary size was chosen to constrain the models, effectively forcing them to learn shared morphemic subwords and affixes rather than memorizing whole words, which clearly highlights the algorithmic differences in segmentation during analysis.

### Results

| Algorithm | Total Tokens | Vocabulary Size |
| --------- | ------------ | --------------- |
| BPE       | 1,120,210    | 1,000           |
| Unigram   | 1,301,687    | 1,000           |
| WordPiece | 1,189,793    | 1,000           |

**Observations:**

- BPE produces the fewest tokens, meaning it achieves the most aggressive merging, resulting in longer subword pieces on average.
- Unigram produces the most tokens, indicating finer-grained segmentation under the probabilistic model with this vocabulary size.
- WordPiece falls between the two. Its likelihood-based merge criterion leads to moderate compression.
- All three use the same vocabulary size (1,000), so the differences reflect algorithmic behavior rather than vocabulary capacity.

---

## Task 6: Comparative Analysis

Three words were selected that are (1) rare in the corpus (frequency = 1) and (2) morphologically complex, featuring Cebuano verbal affixes and reduplication.

### Selected Words

| Word               | Frequency | Morphological Notes                                                                                                                                                 |
| ------------------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nagapabungolbungol | 1         | Progressive causative verb with reduplication:*naga-* (progressive) + *pa-* (causative) + reduplicated root *bungol* (deaf) = pretending to be deaf           |
| gipapanalanginan   | 1         | Causative passive verb:*gi-* (perfective passive) + *pa-* (causative) + *panalangin* (prayer/blessing) + *-an* (locative suffix) = was caused to be blessed |
| nagabalhinbalhin   | 1         | Progressive verb with reduplication:*naga-* (progressive) + reduplicated root *balhin* (move) = kept moving around                                              |

### Segmentation Comparison

| Word               | BPE                                                                    | Unigram                                                                                                | WordPiece                                                                          |
| ------------------ | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- |
| nagapabungolbungol | `nagapa` \| `b` \| `ung` \| `ol` \| `b` \| `ung` \| `ol` | `nagapa` \| `b` \| `u` \| `ng` \| `o` \| `l` \| `b` \| `u` \| `ng` \| `o` \| `l` | `nagapa` \| `##bu` \| `##ng` \| `##ol` \| `##bu` \| `##ng` \| `##ol` |
| gipapanalanginan   | `gipa` \| `panalanginan`                                           | `gipa` \| `panalangin` \| `a` \| `n`                                                           | `gipa` \| `##pan` \| `##ala` \| `##ng` \| `##ina` \| `##n`             |
| nagabalhinbalhin   | `naga` \| `bal` \| `hin` \| `bal` \| `hin`                   | `naga` \| `balhin` \| `balhin`                                                                   | `naga` \| `##ba` \| `##l` \| `##hin` \| `##ba` \| `##l` \| `##hin`   |

### Discussion

**BPE** produces moderate segmentation. It correctly isolates the productive prefix *naga-* in *nagabalhinbalhin* and the *gipa-* causative passive prefix in *gipapanalanginan*. Most notably, BPE keeps *panalanginan* intact as a single token in the second word, preserving the root morpheme. However, for *nagapabungolbungol*, it merges the prefix *naga-* with *pa-* into `nagapa` and then fragments the reduplicated root *bungol* into individual pieces (`b`, `ung`, `ol`). BPE's greedy merge strategy favors frequent prefixes but struggles with less frequent roots.

**Unigram** produces the most fragmented output for *nagapabungolbungol* (11 tokens), splitting the root into near-character-level pieces. However, it performs well on *gipapanalanginan* by recovering the morphologically meaningful *panalangin* as a single unit, and on *nagabalhinbalhin* by cleanly preserving both instances of the reduplicated root *balhin*. This reflects the probabilistic model's strength: it can recover high-probability substrings even when overall segmentation is fine-grained.

**WordPiece** falls in between. It uses the `##` continuation prefix to mark non-initial subwords. For *nagabalhinbalhin*, it isolates the *naga-* prefix but fragments each *balhin* into three pieces (`##ba`, `##l`, `##hin`). For *gipapanalanginan*, it crosses morpheme boundaries with segments like `##pan`, `##ala`, `##ng`, `##ina`, `##n`, which do not correspond to any meaningful morphological units. Its likelihood-based criterion tends to produce moderate-length subwords without linguistic alignment.

**Handling of Affixes**
All three tokenizers recognize the frequent *naga-* prefix as a unit (either as `naga` or merged as `nagapa`). The causative *pa-* is consistently merged with its adjacent prefix (*nagapa-* or *gipa-*) rather than being isolated, since *pa-* alone is too short to be reliably learned as a separate token. The suffix *-an* is not cleanly recovered by any tokenizer — BPE absorbs it into the root (*panalanginan*), Unigram splits it into `a` + `n`, and WordPiece distributes it across fragments. Reduplication patterns (*bungolbungol*, *balhinbalhin*) are handled most cleanly by Unigram, which preserves the repeated root as a unit.

None of the three tokenizers achieve true morphological segmentation; they approximate it through frequency correlation rather than linguistic knowledge. A morphologically-aware tokenizer trained with explicit affix rules would be needed for fully accurate Cebuano segmentation.
