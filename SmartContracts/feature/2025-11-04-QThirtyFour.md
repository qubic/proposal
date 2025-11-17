# Proposal: “4-of-30” Lottery — Final Specification with Adaptive Fast-Recovery

## 1. Purpose and Scope

This document defines the business, economic, and operational parameters of the on-chain lottery “4-of-30.” It establishes:

* Prize structure and minimum payouts;
* Revenue allocation and reserve policy;
* An adaptive Fast-Recovery (FR) regime that accelerates jackpot rebuild to a configurable target (“Target Carry”) at high ticket volumes;
* Shareholder remuneration from the Distribution component;
* Machine-readable configuration for implementation in the smart contract (SC).

---

## 2. Product Overview

**Game format.** A player selects 4 unique numbers from 30.
**Prize tiers.** k=2, k=3, k=4 (jackpot).
**Jackpot.** The jackpot (carry) is a separate, accumulating balance. Upon a k=4 win, the winner receives **100% of the current jackpot**. Jackpot replenishment is automatic per Sections 5–7.

---

## 3. Revenue Allocation (per round)

Let ticket price be (P). Let (N) denote tickets sold in the round, and (R = N \cdot P).

* **Winners (k=2/k=3)**: 68% of (R)
* **Distribution (partners/shareholders)**: 20% of (R) —**shareholder income of the RL smart contract**
* **Dev**: 10% of (R)
* **Burn**: 2% of (R)

The jackpot (k=4) is **not** funded from the 68% winners pool; it grows via FR redirects and overflow as defined below.

> Note: During FR, a **temporary** fraction of Distribution may be redirected to the jackpot (per “Financial Mechanics During FR”); once FR is OFF, Distribution returns to the **full 20% of (R)** for RL shareholders.

---

## 4. Minimum Payout Floors

* **k=2 floor:** ≥ 0.5 × (P) per winner.
* **k=3 floor:** ≥ 5 × (P) per winner.
  If a tier pool is insufficient, top-ups may be sourced from **Reserve.General**, subject to safety limits: ≤ 10% of total reserve per round; ≤ 25 × (P) per winner; soft floor of reserve equivalent to 20 × (P).

---

## 5. Reserves and Overflow

* **Overflow.** Any unawarded amount within k=2/k=3 after payouts is overflow. Overflow is split between **Reserve** and **Jackpot (carry)** with mode-dependent coefficients (baseline vs FR).
* **Reserve buckets.**

  * **Reserve.General:** used to meet payout floors;
  * **Reserve.JackpotRebuild:** used to reseed jackpot after k=4 wins up to the configured target (subject to availability).
* **Safety limits.** As defined in Section 4.

---

## 6. Target Carry and Jackpot Reseed

* **Target Carry** is configurable (e.g., 0.5B, 1B, 2B qu).
* After a k=4 payout, the SC may reseed the jackpot from **Reserve.JackpotRebuild** up to Target Carry (or available balance if lower). The remainder is rebuilt through FR and overflow per Sections 7–8.

---

## 7. Adaptive Fast-Recovery (FR)

**Objective.** When ticket volume is high, FR temporarily accelerates jackpot growth so that the jackpot is expected to reach Target Carry **not later** than the next average k=4 occurrence.

**Adaptive trigger.** Each round, the SC evaluates:

1. the remaining gap to Target Carry;
2. the expected growth rate of the jackpot under FR;
3. the expected number of rounds before the next average k=4 (as a function of (N)).

**Activation rule.** If, at current (N), the expected FR-driven growth over the expected time to next k=4 is sufficient to close the gap, **FR=ON**.
Additionally, within a **post-k4 window** (default 50 rounds), FR may activate under looser conditions to support early rebuild.

**Deactivation rule (hysteresis).** If the jackpot is at or above Target Carry for **3 consecutive rounds**, **FR=OFF**.

---

## 8. Financial Mechanics During FR

While FR=ON, a small, **equal** temporary redirect from Dev and Distribution augments the jackpot, complemented by a minor temporary contribution from k=3 and an overflow bias to carry. Players face no additional fees.

**8.1 Equal redirects (Dev/Distribution).**

* **Dev redirect:** `dev_redirect_pp = 1.0% of R` → jackpot (carry).
* **Distribution redirect:** `dist_redirect_pp = 1.0% of R` → jackpot (carry).
* **High-load supplement (optional):** If (N \geq 1000), add `dev_redirect_extra_pp = 0.35% of R` and `dist_redirect_extra_pp = 0.35% of R` (total +0.7% of R into carry).

**8.2 Temporary k=3 contribution (recovery-rake).**

* `winners_rake_of_winners_total_fr = 5%` of the 68% winners block is redirected to carry **from k=3 only**.
* The remaining winners block (`win_eff`) is split with a slight tilt toward k=2 for stability during FR: **k=3 = 35% of `win_eff`**, **k=2 = 65% of `win_eff`**. Floors remain intact.

**8.3 Overflow bias under FR.**

* `alpha_when_active = 0.05`, i.e., **95% of tier overflow → carry**, **5% → reserve**.

**8.4 Post-FR reversion.**

* When FR turns OFF, Dev and Distribution return to 10%/20% of (R) (no redirect), winners split reverts to baseline, and overflow split reverts to baseline policy.

---

## 9. Shareholder Remuneration (Distribution)

* **Source.** The Distribution block (20% of (R)).
* **FR effect.** During FR, the `dist_redirect_pp [ + dist_redirect_extra_pp ]` portion is temporarily routed to carry; once FR is OFF, Distribution returns to the full 20% of (R).

---

## 10. Parameters (Defaults and Governance Ranges)

* **Target Carry:** 1,000,000,000 qu (governable).
* **post_k4_window_rounds:** 50 (range: 20–80).
* **hysteresis_rounds_at_or_above_target:** 3 (range: 2–5).
* **dev_redirect_pp:** 0.01 (range: 0.005–0.02).
* **dist_redirect_pp:** 0.01 (range: 0.005–0.02).
* **high_load_redirect_extra (N≥1000):** dev=0.0035, dist=0.0035 (each; range: 0.0025–0.005).
* **winners_rake_of_winners_total_fr:** 0.05 (range: 0.03–0.08).
* **tiers_pool_shares_fr (of `win_eff`):** k3=0.35, k2=0.65 (sum=1.0).
* **alpha_when_active:** 0.05 (range: 0.03–0.20).
* **Reserve safety limits:** per Sections 4–5.

---

## 11. Operational Order of Settlement (Per Round, High Level)

1. Compute baseline blocks: **Dev 10%**, **Distribution 20%**, **Burn 2%**, **Winners 68%**.
2. If **FR=ON**:

   * Redirect `dev_redirect_pp * R [ + dev_redirect_extra_pp * R ]` to carry (reducing Dev by the same amount);
   * Redirect `dist_redirect_pp * R [ + dist_redirect_extra_pp * R ]` to carry (reducing Distribution by the same amount);
   * From Winners, redirect `winners_rake_of_winners_total_fr` (5% of 68% (R)) to carry from k=3;
   * Split `win_eff` as `{k3:35%, k2:65%}`; apply floors;
   * Split overflow with `alpha_when_active=0.05` (95% → carry, 5% → reserve).
3. If **k=4** occurs: pay **100% of carry** to winner(s); then optionally reseed from **Reserve.JackpotRebuild** up to Target Carry (subject to availability).
4. If **FR OFF**: apply baseline tier shares and baseline overflow split.

---

## 15. Summary

This specification delivers a transparent “4-of-30” lottery with strict 100% jackpot payout and a principled, adaptive FR mechanism. FR leverages temporary, equal redirects from Dev and Distribution, a modest k=3 contribution, and an FR-biased overflow to restore the jackpot to Target Carry at scale, while preserving player fairness, shareholder remuneration, and operational discipline.

**Distribution (20% of (R)) constitutes RL smart-contract shareholder income**; during FR, a small portion is temporarily redirected to the jackpot and then fully restored to shareholders once FR is OFF.
