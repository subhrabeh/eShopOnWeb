# Customer Record Deduplication — Approach Comparison

We tested 7 deduplication approaches against 9 sample customer records (name, address, email, DOB, brand) containing duplicate pairs across 5 retail brands with variations like nicknames (Bob/Robert), typos (Smith/Smyth), abbreviations (St/Street), cross-brand duplicates, and DOB mismatches.

---

## Sample Records

| # | Name | Address | Email | DOB | Brand |
|---|------|---------|-------|-----|-------|
| 0 | John Smith | 123 Main St | john@example.com | 1985-03-15 | A |
| 1 | Jon Smyth | 123 Main Street | jon@example.com | 1985-03-15 | B |
| 2 | Jane Doe | 456 Oak Ave | jane@example.com | 1990-07-22 | A |
| 3 | John Smith | 124 Main St | johnsmith@example.com | 1985-03-15 | A |
| 4 | Janet Doe | 456 Oak Avenue | janet@example.com | 1990-07-22 | C |
| 5 | Bob Johnson | 789 Pine Rd | bob@example.com | 1978-11-30 | D |
| 6 | Robert Johnson | 789 Pine Road | robert@example.com | 1978-11-30 | D |
| 7 | Alice Brown | 321 Elm St | alice@example.com | 1992-01-10 | E |
| 8 | Alice Brown | 321 Elm Street | aliceb@example.com | 1993-06-05 | B |

Variations include: nicknames (Bob/Robert, Jon/John, Jane/Janet), typos (Smith/Smyth), abbreviations (St/Street, Rd/Road, Ave/Avenue), near-identical addresses (123 vs 124), DOB mismatch (records 7 & 8), and cross-brand duplicates (A↔B, A↔C, E↔B).

---

## Approaches Tested

All approaches now include brand as a comparison field (exact match: 1.0 same brand, 0.0 different).

**1. TF-IDF Vector (Whole Record)**
Concatenates all fields into a single text blob, vectorizes using character n-gram TF-IDF, and computes cosine similarity. DOB and brand scored separately. Weights: 50% text, 35% DOB, 15% brand.

**2. Fuzzy String Matching (Field-Level)**
Compares each field independently using fuzzy ratios (token_sort_ratio for names, partial_ratio for addresses, ratio for emails). Weights: 30% name, 20% address, 10% email, 25% DOB, 15% brand.

**3. TF-IDF Vector (Field-Level)**
Same TF-IDF vectorization as Approach 1 but applied per field with separate vectorizers. Same weights as Approach 2.

**4. Record Linkage / Probabilistic (Fellegi-Sunter)**
Uses `recordlinkage` library with Jaro-Winkler and Levenshtein comparators plus date comparison and exact brand match. Field weights: name=2.0, address=1.5, email=1.0, dob=2.5, brand=1.0.

**5. Unsupervised ML (ECM Classifier)**
Expectation-Maximization classifier from `recordlinkage`. Learns match/non-match distributions from the feature data alone — no labels needed. Now includes brand as an input feature.

**6a. Supervised ML (Random Forest)**
Trained on consensus labels (pairs flagged by ≥3 of the 4 rule-based approaches). Uses 10 comparison features including brand match. Top features: address similarity (0.25) and address Jaro-Winkler (0.24).

**6b. Supervised ML (Logistic Regression)**
Same training data as Random Forest but learns a linear weighted combination. DOB had the highest coefficient (1.57), followed by name Levenshtein (1.00) and brand (0.52).

---

## Results With Nickname Mapping + Brand

| Pair | Records | DOB | Brand | Blob | Fuzzy | VecFld | RecLink | ECM | RF | LR |
|------|---------|-----|-------|------|-------|--------|---------|-----|----|----|
| 0↔1 | John Smith ↔ Jon Smyth | exact | A≠B | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.73) | ✓ (.85) |
| 0↔3 | John Smith ↔ John Smith | exact | A=A | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.97) | ✓ (.92) |
| 1↔3 | Jon Smyth ↔ John Smith | exact | B≠A | ✗ | ✓ | ✗ | ✓ | ✓ | ✗ (.22) | ✓ (.82) |
| 2↔4 | Jane Doe ↔ Janet Doe | exact | A≠C | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.94) | ✓ (.87) |
| 5↔6 | Bob Johnson ↔ Robert Johnson | exact | D=D | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.97) | ✓ (.91) |
| 7↔8 | Alice Brown ↔ Alice Brown | diff | E≠B | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ (.25) | ✓ (.57) |

| Approach | Pairs Found |
|----------|-------------|
| 1 - TF-IDF Blob | 4 |
| 2 - Fuzzy Field | 5 |
| 3 - TF-IDF Field | 4 |
| 4 - Record Linkage | 5 |
| 5 - ECM (unsupervised ML) | 6 |
| 6a - Random Forest | 4 |
| 6b - Logistic Regression | 6 |

All 7 approaches agreed on 4 core pairs. The brand field caused Jon Smyth [1] ↔ John Smith [3] (cross-brand B≠A) to drop from unanimous to 4/7 agreement. Alice Brown [7] ↔ [8] (diff DOB + cross-brand E≠B) was only caught by ECM and Logistic Regression.

---

## Results Without Nickname Mapping + Brand

| Pair | Records | DOB | Brand | Blob | Fuzzy | VecFld | RecLink | ECM | RF | LR |
|------|---------|-----|-------|------|-------|--------|---------|-----|----|----|
| 0↔1 | John Smith ↔ Jon Smyth | exact | A≠B | ✗ | ✓ | ✗ | ✓ | ✓ | ✗ (.13) | ✓ (.77) |
| 0↔3 | John Smith ↔ John Smith | exact | A=A | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.83) | ✓ (.93) |
| 1↔3 | Jon Smyth ↔ John Smith | exact | B≠A | ✗ | ✓ | ✗ | ✓ | ✓ | ✗ (.08) | ✓ (.74) |
| 2↔4 | Jane Doe ↔ Janet Doe | exact | A≠C | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.75) | ✓ (.81) |
| 5↔6 | Bob Johnson ↔ Robert Johnson | exact | D=D | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (.75) | ✓ (.87) |
| 7↔8 | Alice Brown ↔ Alice Brown | diff | E≠B | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ (.14) | ✗ (.50) |

| Approach | With Nicknames | Without Nicknames |
|----------|---------------|-------------------|
| 1 - TF-IDF Blob | 4 | 3 |
| 2 - Fuzzy Field | 5 | 5 |
| 3 - TF-IDF Field | 4 | 3 |
| 4 - Record Linkage | 5 | 5 |
| 5 - ECM (unsupervised ML) | 6 | 6 |
| 6a - Random Forest | 4 | 3 |
| 6b - Logistic Regression | 6 | 5 |

Without nicknames, unanimous agreement dropped to 3 pairs. Alice Brown [7]↔[8] was only caught by ECM (1/7) — even Logistic Regression dropped it (prob fell from 0.57 to 0.50, right at the decision boundary). The combination of different DOB + different brand + no nickname help made this the hardest case.

---

## Impact of Brand Field

Adding brand as a comparison field had a notable tightening effect across all approaches:

- Same-brand pairs (0↔3 Brand A=A, 5↔6 Brand D=D) scored higher and were caught more confidently.
- Cross-brand pairs (0↔1 A≠B, 2↔4 A≠C, 7↔8 E≠B) saw reduced scores. The 15% brand weight penalty pushed borderline cases below thresholds.
- Jon Smyth [1] ↔ John Smith [3] went from 7/7 agreement (without brand) to 4/7 (with brand) because the B≠A brand mismatch combined with the name variation was enough to trip up the vector and RF approaches.
- In a multi-brand retail context, cross-brand duplicates are common (same customer shops at multiple brands). Brand should be weighted low enough that it boosts confidence for same-brand matches without blocking legitimate cross-brand duplicates.

---

## Key Takeaways

- **Nickname handling matters.** Without a nickname map, TF-IDF vector approaches lost matches. Fuzzy and ML-based approaches were far more resilient.
- **Brand is a useful signal but must be weighted carefully.** In multi-brand retail, customers frequently appear across brands. Over-weighting brand will suppress legitimate cross-brand duplicates.
- **DOB is a strong signal but not absolute.** Data entry errors in DOB happen. Approaches that weight DOB too heavily will miss legitimate duplicates (Alice Brown case).
- **No single approach is best.** A production system should combine multiple signals — use an ensemble or voting strategy where pairs agreed on by majority get auto-merged, and borderline cases go to human review.
- **ECM (unsupervised ML) was the most resilient approach overall.** It found 6 pairs in every scenario — with/without nicknames, with brand. It doesn't need training labels and adapts to the data distribution automatically.
- **Fuzzy and Record Linkage are the most consistent rule-based approaches.** Both held at 5 pairs across nickname variations.
- **For production, consider a hybrid:** use Fuzzy or Record Linkage for candidate generation, then ML (ECM or Logistic Regression) for final scoring, with a three-bucket system (auto-merge / manual review / reject).
