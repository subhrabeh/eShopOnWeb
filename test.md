# Customer Record Deduplication — Approach Comparison

We tested 7 deduplication approaches against 9 sample customer records (name, address, email, DOB) containing 6 known duplicate pairs with variations like nicknames (Bob/Robert), typos (Smith/Smyth), abbreviations (St/Street), and DOB mismatches.

---

## Approaches Tested

**1. TF-IDF Vector (Whole Record)**
Concatenates all fields into a single text blob, vectorizes using character n-gram TF-IDF, and computes cosine similarity. DOB scored separately. Simple to implement but treats all fields equally and struggles with nickname variations at the character level.

**2. Fuzzy String Matching (Field-Level)**
Compares each field independently using fuzzy ratios (token_sort_ratio for names, partial_ratio for addresses, ratio for emails). Weighted combination with DOB. Most lenient of the rule-based approaches — good at catching partial matches and abbreviations.

**3. TF-IDF Vector (Field-Level)**
Same TF-IDF vectorization as Approach 1 but applied per field with separate vectorizers and weighted combination. More precise than the blob approach but shares the same character n-gram limitation for nicknames.

**4. Record Linkage / Probabilistic (Fellegi-Sunter)**
Uses the `recordlinkage` library with Jaro-Winkler and Levenshtein string comparators plus date comparison. Weighted scoring inspired by the Fellegi-Sunter model. Most resilient to nickname variations even without a nickname map — Jaro-Winkler scored Bob↔Robert at 0.874.

**5. Unsupervised ML (ECM Classifier)**
Expectation-Maximization classifier from `recordlinkage`. Learns match/non-match distributions from the feature data alone — no labels needed. Automatically finds a decision boundary based on the statistical separation of duplicate vs non-duplicate feature patterns.

**6a. Supervised ML (Random Forest)**
Trained on consensus labels (pairs flagged by ≥3 of the 4 rule-based approaches). Uses all 9 comparison features (Jaro-Winkler, Levenshtein, fuzzy scores, DOB). Learns non-linear decision boundaries. Top features: address similarity (0.22) and DOB match (0.20).

**6b. Supervised ML (Logistic Regression)**
Same training data as Random Forest but learns a linear weighted combination. Provides interpretable coefficients showing each feature's contribution. DOB had the highest coefficient (1.76), followed by name Levenshtein (1.08).

---

## Results Summary

| Pair | Records | DOB | Blob | Fuzzy | VecFld | RecLink | ECM | RF | LR |
|------|---------|-----|------|-------|--------|---------|-----|----|----|
| 0↔1 | John Smith ↔ Jon Smyth | exact | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.91) | ✓ (.90) |
| 0↔3 | John Smith ↔ John Smith | exact | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (1.0) | ✓ (.91) |
| 1↔3 | Jon Smyth ↔ John Smith | exact | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.78) | ✓ (.88) |
| 2↔4 | Jane Doe ↔ Janet Doe | exact | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.93) | ✓ (.91) |
| 5↔6 | Bob Johnson ↔ Robert Johnson | exact | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (1.0) | ✓ (.91) |
| 7↔8 | Alice Brown ↔ Alice Brown | diff | ✗ | ✓ | ✗ | ✗ | ✓ | ✗ (.26) | ✓ (.63) |

| Approach | Pairs Found |
|----------|-------------|
| 1 - TF-IDF Blob | 5 |
| 2 - Fuzzy Field | 6 |
| 3 - TF-IDF Field | 5 |
| 4 - Record Linkage | 5 |
| 5 - ECM (unsupervised ML) | 6 |
| 6a - Random Forest | 5 |
| 6b - Logistic Regression | 6 |

All 7 approaches unanimously agreed on 5 core duplicate pairs.

The contested pair — Alice Brown [7] ↔ Alice Brown [8] — has identical name and similar address but different DOBs (1992 vs 1993). Only Fuzzy, ECM, and Logistic Regression flagged it. This is the kind of edge case that would go to manual review in production.

---

## Key Takeaways

- **Nickname handling matters.** Without a nickname map, only Record Linkage (Jaro-Winkler) and Fuzzy still caught Bob↔Robert. The vector approaches missed it entirely.
- **DOB is a strong signal but not absolute.** Data entry errors in DOB happen. Approaches that weight DOB too heavily will miss legitimate duplicates.
- **No single approach is best.** A production system should combine multiple signals — use an ensemble or voting strategy where pairs agreed on by majority get auto-merged, and borderline cases go to human review.
- **ML adds value when you have features, not necessarily labels.** ECM (unsupervised) performed as well as the best rule-based approach without any training labels. Supervised ML matched the consensus it was trained on, with Logistic Regression being slightly more aggressive than Random Forest on edge cases.
