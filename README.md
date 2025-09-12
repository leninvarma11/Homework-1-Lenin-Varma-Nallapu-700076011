# Homework-1  Explanation
# Lenin Varma Nallapu
# 700760311


# 1 Question

### **1. U.S. ZIP codes**

**Regex:** `\b\d{5}(?:[ -]\d{4})?\b`

* `\b` → word boundary ensures we match whole tokens.
* `\d{5}` → matches exactly 5 digits.
* `(?:[ -]\d{4})?` → optional +4 part with **hyphen or space**.
* Matches: `12345`, `12345-6789`, `12345 6789`.

### **2. Words not starting with capital letters**

**Regex:** `\b(?![A-Z])[a-z]+(?:['’-][a-z]+)*\b`

* `(?![A-Z])` → negative lookahead to avoid words starting with capital letters.
* `[a-z]+` → first part of the word (lowercase letters).
* `(?:['’-][a-z]+)*` → allows internal apostrophes or hyphens (like `don’t`, `state-of-the-art`).
* `\b` → ensures whole word match.

### **3. Numbers (optional sign, commas, decimals, scientific)**

**Regex:** `\b[+-]?(?:(?:\d{1,3}(?:,\d{3})+|\d+)(?:.\d+)?|.\d+)(?:[eE][+-]?\d+)?\b`

* `[+-]?` → optional plus or minus sign.
* `\d{1,3}(?:,\d{3})+|\d+` → handles thousands separators or plain digits.
* `(?:.\d+)?` → optional decimal part.
* `(?:[eE][+-]?\d+)?` → optional scientific notation.
* `\b` → ensures the number is a complete token.

### **4. Email spelling variants**

**Regex:** `(?i)\bE(?:[- ]?)mail\b`

* `(?i)` → case-insensitive.
* `\b` → word boundary.
* `E(?:[- ]?)mail` → matches `e-mail`, `e mail`, or `email`.
* For en-dash included: `(?i)\bE(?:\s|[-\u2013])?mail\b`.

### **5. Interjection “go” with repetition & punctuation**

**Regex:** `\bgo+\b[!.,?]?`

* `go+` → matches `g` followed by **1 or more o’s** (`go`, `goo`, `gooo`...).
* `\b` → ensures word boundaries.
* `[!.,?]?` → optional trailing punctuation.

### **6. Lines ending with a question mark & quotes/brackets**

**Regex:** `(?m)^[^\n]?[)"]\u201D\u2019\s]$`

* `(?m)` → multiline mode, so `^` and `$` match **line start/end**.
* `^[^\n]?` → start of line, optional character.
* `[)\"\u201D\u2019\s]` → matches closing brackets, quotes, or spaces.
* `$` → ensures match is at **line end**.


# 2 Question


### **1. Paragraph Setup**

python
paragraph = "Yesterday, I went to New York City. It’s a busy place, full of energy! However, I didn’t stay long."

* A small sample paragraph of **3–4 sentences** is chosen.
* It contains punctuation, contractions, and a multiword expression (*New York City*).


### **1a. Naïve Space-Based Tokenization**

python
naive_tokens = paragraph.split()

* Splits the paragraph **only on spaces**.
* Example output: `['Yesterday,', 'I', 'went', 'to', 'New', 'York', 'City.', ...]`
* **Problem:** punctuation stays attached to words (`Yesterday,`, `City.`), MWEs are split, contractions not handled.

### **1b. Manual Correction**
python
manual_tokens = [
    'Yesterday', ',', 'I', 'went', 'to', 'New York City', '.',
    'It', '’s', 'a', 'busy', 'place', ',', 'full', 'of', 'energy', '!',
    'However', ',', 'I', 'did', 'n’t', 'stay', 'long', '.'
]

* Corrects **punctuation** by separating commas, periods, exclamation marks.
* Handles **clitics/contractions**: `It’s → It + ’s`, `didn’t → did + n’t`.
* Keeps **multiword expressions** together: `New York City` as one token.

### **2. Tool-Based Tokenization (spaCy)**

python
nlp = spacy.load("en_core_web_sm")
doc = nlp(paragraph)
spacy_tokens = [t.text for t in doc]

* Uses **spaCy**, an NLP library with pre-trained models for English.
* SpaCy handles:

  * punctuation separation
  * clitics like `It’s → It + ’s`
  * token boundaries consistently

**Example output:**
`['Yesterday', ',', 'I', 'went', 'to', 'New', 'York', 'City', '.', ...]`

* Note: SpaCy splits *New York City* into 3 tokens, whereas manual grouped it as 1.

### **Comparing Manual vs SpaCy**

python
set(manual_tokens) - set(spacy_tokens)
set(spacy_tokens) - set(manual_tokens)

* Shows **differences in tokenization**:

  * Manual captures MWEs as a single token.
  * SpaCy sticks strictly to surface token rules.

### **3. Multiword Expressions (MWEs)**

python
mwes = [
    "New York City - place name, should be one token.",
    "social media - fixed phrase with non-compositional meaning.",
    "kick the bucket - idiom meaning 'to die'."
]

* MWEs are **fixed phrases** or idioms where meaning cannot be inferred from individual words.
* Treating them as a single token preserves **semantic meaning** for NLP tasks.

### **4. Reflection**

python
reflection = """..."""

* Key points:

  * Handling **contractions and MWEs** is challenging.
  * Some languages are harder than English due to **rich morphology**.
  * **Punctuation** can be tricky (apostrophes, quotes).
  * Manual tokenization may differ from automatic tools depending on **semantic grouping**.
  * Tokenization balances **linguistic accuracy** and **tool consistency**.
 

# 3 Question

 **Helper Functions**
1. **`add_eow(word)`**

   * Adds an **end-of-word marker `_`** and splits the word into characters.
   * Example: `"low"` → `['l', 'o', 'w', '_']`.

2. **`get_stats(tokens)`**

   * Counts **all bigrams** (pairs of adjacent symbols) in the corpus.
   * Returns a `Counter` of bigram frequencies.

3. **`merge_pair(tokens, pair)`**

   * Merges the **most frequent bigram** in all words.
   * Updates the corpus with the new merged token.

4. **`segment(word, merges)`**

   * Segments a **new word** using the learned merges.
   * Applies merges step by step to produce **subword tokens**.

**3.1 Manual BPE on Toy Corpus**
* **Corpus:**
  `["low","low","low","low","low","lowest","lowest","newer","newer","wider","wider","wider","new","new"]`
* **Steps:**

  1. Add end-of-word markers: `'low' → l o w _`
  2. Compute **bigram counts**
  3. Perform **first 3 merges manually**:

     * Merge the most frequent bigram at each step.
     * Example output: Step 1 merge `('l','o') → lo`, Step 2 merge `('lo','w') → low`, etc.
* **Goal:** Observe **how the vocabulary evolves**.

**3.2 Mini BPE Learner (Toy Corpus)**
* Automates **BPE merges** (10 steps).
* Prints:

  * **Top bigram** at each step
  * **Vocabulary size** after each merge
* **Segmentation:**

  * Tests words like `"new"`, `"newer"`, `"lowest"`, `"widest"`, `"newestest"`.
  * Shows how **subwords** are assigned for out-of-vocabulary words.

**3.3 Training BPE on Paragraph**
* **Paragraph:** About AI and machine learning.
* Steps:

  1. Convert text to **lowercase**, remove punctuation, split into words.
  2. Add **end-of-word marker `_`**.
  3. Train **30 merges** using the same bigram-counting procedure.
* **Outputs:**

  * **Top 5 merges:** Most frequent subword merges.
  * **Longest tokens:** Shows how subword units grow with merges.
  * **Segmentation of 5 words**: `"artificial"`, `"researchers"`, `"bias"`, `"transforming"`, `"solutions"`.

    * Example: `"researchers"` → `['re', 'search', 'ers', '_']`.

**Key Concepts Illustrated**

1. **Subword tokenization**

   * Splits words into **smaller units** to handle rare or OOV words.
   * Example: `"newestest"` can be segmented as `['new', 'est', 'est', '_']`.

2. **Bigram merging**

   * Most frequent adjacent pairs are merged first.
   * Vocabulary grows gradually from **characters → subwords → full words**.

3. **Application**

   * Useful in NLP for **neural machine translation, language modeling**, and **handling rare words**.
   * Reduces OOV issues while preserving meaningful subword morphemes (`-er`, `-ing`).


# 4 Question

 **1. Minimum Edit Distance**

* **Model A (Sub=1, Ins=1, Del=1)**

  * Substitutions, insertions, and deletions all cost the same (1).
  * Minimum distance = **3**.

* **Model B (Sub=2, Ins=1, Del=1)**

  * Substitutions are **more expensive** than insertions/deletions.
  * Minimum distance = **4**.

**Explanation:**

* In Model A, it's cheaper to substitute letters directly.
* In Model B, the algorithm avoids substitutions when possible and uses cheaper insertions/deletions, increasing total cost.

**2. One Valid Edit Sequence**

**Model A (cost 3):**

1. Insert `a` after `S` → `Saunday`
2. Insert `t` after `Sa` → `Satunday`
3. Substitute `n → r` → `Saturday`

**Model B (cost 4):**

1. Insert `a` after `S` → `Saunday`
2. Insert `t` after `Sa` → `Satunday`
3. Delete `n` → `Satuday`
4. Insert `r` before `d` → `Saturday`

**Explanation:**

* Both sequences transform `Sunday` into `Saturday`.
* The difference arises from the **cost of substitution**:

  * Model A prefers substitution.
  * Model B prefers deletion + insertion instead of the expensive substitution.


 **3. Reflection**

* The two models produce **different distances** because substitution cost differs.
* In Model A, substitutions are cheap and useful for mismatched letters.
* In Model B, substitution is costly, so the algorithm favors **insertions/deletions**.
* Model choice affects applications:

  * **Spell checking:** Substitutions should be cheap since typos are often letter swaps.
  * **DNA alignment:** Substitutions (mutations) may be biologically significant, so a higher cost is more realistic.
* Understanding cost models helps design algorithms that reflect the domain-specific **importance of edits**.




