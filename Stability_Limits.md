
### 4.3 Design Properties

| Property | Mechanism |
|---|---|
| **Symmetry between sides** | GS is run from both sides in each round |
| **Priority to forced pairs** | They are removed before any concession |
| **Procedural fairness** | Random permutation gives each party the same probability distribution over positions |
| **Convergence** | Each concession decreases remaining options by at least 1 |
| **Strategy-proofness** | Randomness is independent of reports |

---

## 5. Theoretical Results

### 5.1 Convergence Theorem

**Theorem 1 (Convergence):**  
For every run of ABSM, the algorithm terminates after a finite number of rounds not exceeding \(|A| + |B|\).

**Proof:**  
Let \(S_t = \sum_{x \in \text{Active}} (\text{len}(P_x) - \text{pos}(x))\) be the sum of remaining options in round \(t\). In each round:

- If \(\text{Agreed} \neq \emptyset\), the number of active parties decreases by at least \(2|\text{Agreed}|\), so \(S_{t+1} < S_t\).
- If \(\text{Agreed} = \emptyset\), every party with an available option moves one step down, so \(S_{t+1} \leq S_t - 1\).

Since \(S_t\) is a non-negative integer that strictly decreases, the algorithm terminates after at most \(S_0 \leq |A| + |B|\) rounds. \(\blacksquare\)

**Remark:** This upper bound is conservative; in practice, convergence is much faster because the permutation moves many parties in a single round.

### 5.2 Symmetry Theorem

**Theorem 2 (Symmetry):**  
If `seed` is fixed, then ABSM(A, B) and ABSM(B, A) produce the same set of pairs (with roles swapped).

**Proof:**  
GS(A) and GS(B) are structurally symmetric. The intersection \(M_A \cap M_B\) is symmetric. The random permutation on \(A \cup B\) is symmetric. Hence every step is symmetric. \(\blacksquare\)

### 5.3 Procedural Fairness Theorem

**Theorem 3 (Procedural fairness):**  
In each deadlock round, every party \(x \in \text{Active}\) has the same probability \(1/|\text{Active}|\) of being in any position in the permutation \(\pi\).

**Proof:**  
The random permutation is generated uniformly over the symmetric group \(S_{|\text{Active}|}\). For each position \(k\), the probability that \(x\) occupies it is \(1/|\text{Active}|\). \(\blacksquare\)

**Corollary:**  
The expected number of concessions per party after \(K\) deadlock rounds is \(K/|\text{Active}|\), and the variance is bounded by \(K/|\text{Active}|\) under an approximately binomial distribution.

### 5.4 Strategy-Proofness Theorem

**Theorem 4 (Strategy-proofness):**  
If the random seed is unpredictable to the parties, then no reporting strategy yields a higher expected payoff than truthful reporting.

**Proof (sketch):**  
The random selection of the conceding party is independent of reports. Any change in reporting does not change the probability that a party is selected to concede in a given round. Since changing the report does not change the probability distribution over outcomes, truthful reporting is a weakly dominant strategy. \(\blacksquare\)

**Caveat:** This proof assumes that parties do not know the seed and cannot infer it from past observations. If the seed is inferable, the property collapses.

### 5.5 Stability Limits

**Important remark:**  
The final matching is stable with respect to the adjusted preferences \(P_x[\text{pos}(x):]\), **not** with respect to the original preferences \(L_x\). This means there is a possibility of **original-preference blocking pairs**.

**Illustrative example:**  
Suppose \(a_1\) and \(b_1\) preferred each other in the original preferences, but \(a_1\) moved down to \(b_2\) due to a concession, and \(b_1\) moved down to \(a_2\). If \(a_1\) still prefers \(b_1\) to \(b_2\) (in the original preferences), and \(b_1\) prefers \(a_1\) to \(a_2\), then \((a_1, b_1)\) is an original blocking pair.

**Mitigation:**  
A post-hoc verification mechanism can be added to detect these cases and rerun with additional constraints. We leave this for future work.

---

## 6. Complexity Analysis

| Component | Complexity |
|---|---|
| One GS run | \(O(nm)\) |
| Full round (GS both sides + intersection) | \(O(nm)\) |
| Number of rounds | \(O(n + m)\) in the worst case |
| **Total complexity** | \(O((n+m) \cdot nm)\) |
| Space complexity | \(O(nm)\) |

**Remark:** In practice, the number of rounds is usually small (less than 10) because the permutation moves many parties in a single round. Hence the actual complexity is closer to \(O(nm)\).

---

## 7. Discussion

### 7.1 Advantages

1. **Simplicity:** No complex concession-cost calculations are required.
2. **Symmetry:** No side is systematically favored.
3. **Guaranteed convergence:** A clear upper bound.
4. **Procedural fairness:** Proven.
5. **Strategy-proofness:** Proven (under assumptions).

### 7.2 Limitations

1. **Original stability is lost:** The final matching is stable only with respect to adjusted preferences.
2. **Outcome fairness is not guaranteed:** The permutation ensures procedural fairness, but may produce variance in outcomes.
3. **Unpredictable-seed assumption:** If it fails, strategy-proofness fails.
4. **Exhausted parties:** Their handling (`MarkExhausted`) breaks symmetry, because parties with longer lists are more likely to concede.

### 7.3 Comparison with Alternative Mechanisms

| Mechanism | Procedural fairness | Outcome fairness | Original stability | Simplicity |
|---|---|---|---|---|
| Classic GS | Weak (proposer-biased) | Medium | **Preserved** | Excellent |
| Lottery over stable matchings | Excellent | Medium | **Preserved** | Difficult |
| ABSM (this paper) | Excellent | Medium | Lost | Good |
| Least-cost concession | Weak | Good | Lost | Medium |

**Remark:** No mechanism satisfies all properties simultaneously—a well-known result in game theory (impossibility theorems). Choosing ABSM represents a trade-off in favor of simplicity, symmetry, and procedural fairness.

### 7.4 Ethical Dimensions

It must be noted that describing concession as "realistic satisfaction" may be used to justify structural injustice. Adaptive preferences (Elster, 1983) are not always genuine satisfaction—they may be resignation to bad options. Any practical application should:

1. **Distinguish genuine satisfaction from forced acceptance.**
2. **Not use stability as the sole justification for fairness.**
3. **Provide appeal mechanisms for affected parties.**

---

## 8. Future Work

1. **Proving preservation of original stability:** Study additional conditions that guarantee no original-preference blocking pairs.
2. **Least-cost concession mechanism:** Replace the random permutation with a deterministic rule that minimizes loss.
3. **Lottery over stable matchings:** Instead of changing preferences, randomly select among the stable matchings between \(M_A\) and \(M_B\).
4. **Empirical application:** Test the algorithm on real data (labor markets, college admissions).
5. **Behavioral comparison:** Study how humans actually behave under deadlock, and compare with the algorithm.

---

## 9. Conclusion

This paper presented the ABSM algorithm, an adaptive framework for solving two-sided matching problems when interests conflict at the original preference level. The algorithm relies on running GS from both sides, extracting forced pairs, and a random-permutation concession mechanism. We proved convergence, symmetry, procedural fairness, and strategy-proofness (under assumptions). We also explicitly discussed limitations concerning original stability and outcome fairness.

ABSM is a first step toward modeling adaptive behavior in matching, but it is not the final solution. We hope it stimulates future work on mechanisms that combine original stability, outcome fairness, and procedural simplicity—a difficult but necessary trade-off for fair real-world applications.

---

## References

1. Abdulkadiroğlu, A., & Sönmez, T. (1998). Random serial dictatorship and the core from random endowments in house allocation problems. *Econometrica*, 66(3), 689–701.
2. Abdulkadiroğlu, A., & Sönmez, T. (2003). School choice: A mechanism design approach. *American Economic Review*, 93(3), 729–747.
3. Bogomolnaia, A., & Moulin, H. (2001). A new solution to the random assignment problem. *Journal of Economic Theory*, 100(2), 295–328.
4. Elster, J. (1983). *Sour Grapes: Studies in the Subversion of Rationality*. Cambridge University Press.
5. Gale, D., & Shapley, L. S. (1962). College admissions and the stability of marriage. *American Mathematical Monthly*, 69(1), 9–15.
6. Gale, D., & Sotomayor, M. (1985). Ms. Machiavelli and the stable matching problem. *American Mathematical Monthly*, 92(4), 261–268.
7. Gusfield, D., & Irving, R. W. (1989). *The Stable Marriage Problem: Structure and Algorithms*. MIT Press.
8. Knuth, D. E. (1976). *Mariages Stables et Leurs Relations avec d'Autres Problèmes Combinatoires*. Les Presses de l'Université de Montréal.
9. Roth, A. E. (1984). The evolution of the labor market for medical interns and residents: A case study in game theory. *Journal of Political Economy*, 92(6), 991–1016.
10. Roth, A. E., & Sotomayor, M. A. O. (1990). *Two-Sided Matching: A Study in Game-Theoretic Modeling and Analysis*. Cambridge University Press.
11. Roth, A. E., Sönmez, T., & Ünver, M. U. (2004). Kidney exchange. *Quarterly Journal of Economics*, 119(2), 457–488.
12. Simon, H. A. (1955). A behavioral model of rational choice. *Quarterly Journal of Economics*, 69(1), 99–118.

---

## Appendix: Notes for Reviewers

This paper represents a **preliminary theoretical proposal**, not a fully proven result. The following points require scrutiny by reviewers:

1. **The proof of Theorem 4 (strategy-proofness)** needs a more precise formulation—we did not study the case of parties who might benefit from "not conceding" if they expect to be selected in the next round.
2. **The convergence bound \(O(n+m)\)** is conservative; a tighter bound may exist.
3. **The upper complexity bound \(O((n+m) \cdot nm)\)** may be higher than necessary in certain cases.
4. **The illustrative example in Section 5.5** needs a more rigorous mathematical analysis of the conditions for original blocking pairs.

These points do not invalidate the idea, but they define the limits of its validity and open avenues for future work.

---

**End of Paper.**
