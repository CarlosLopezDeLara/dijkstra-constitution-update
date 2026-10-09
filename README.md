# Dijkstra protocol parameters: specification for the constitution update, guardrails and initial settings

**Status:** Draft v1.3
**Authors:** Carlos Lopez de Lara (IOG), Aleksandr Vershilov (Tweag)
**Date:** 2026-10-07
**Supersedes:** [dijkstra-protocol-parameters-for-constitution-update](https://docs.google.com/document/d/1M649pDQtquYr4n5QBx_Q7vfzyFINQssN8Cpp553BGvA/edit?usp=sharing) (v0.2, 2026-09-01)
**Audience:** Parameters Committee, HFWG, Civics Committee, IOG, Tweag

This document specifies every protocol parameter that the Dijkstra hard fork (PV12) introduces as updatable, in the form the ledger will accept it, and frames the guardrails and initial values still to be agreed for each. It is written to serve three deliverables from one source:

1. the **Constitution Update and Guardrail Appendix** governance action that must name each new parameter,
2. the **guardrails script** that ships with that action, and
3. the **initial settings** carried in Dijkstra genesis or set by the first parameter update after the hard fork.

Section 5 says how to read the parameter tables. Sections 6 to 10 are descriptive: they record what the implementation and the upstream specifications define, and nothing more. Sections 11 and 12 are open, awaiting the guardrails and initial values the Parameters Committee agrees with the technical teams. Section 13 tracks the decisions still outstanding.

---

## 1. Change log

| Version | Date | Change |
| :---- | :---- | :---- |
| v1.3 | 2026-10-07 | Aligns the document with cardano-ledger `master` (`0a72d43f59`) and the merged CIP-0164. **Peras (Section 10):** the six parameters merged in ledger PR #6067; names now the parameter-update names without the `pp` prefix (`perasMinCandidateBlockAge` and so on); update keys 49 to 54; groups NetworkGroup and SecurityGroup as implemented, not the TechnicalGroup previously expected; types corrected (`perasBootstrapRound` is `nil` or a 32-bit unsigned integer, `perasQuorumThresholdSafetyMargin` is a `unit_interval`); descriptions taken from the ledger; proposed defaults moved to Section 12. **Counts (Sections 4 and 5):** 21 parameters at keys 34 to 54. **Section 9:** `maxPledgeLeverage` is no longer the only nullable parameter. **Sections 11.2 and 11.3:** Peras names updated; PHF-01/02 named the wrong parameter. **CIP-0164 (Sections 6 and 13):** PR #1250 merged on 2026-09-15 as `a2cac180`; Tables 1 to 3 and the Table 3 note re-verified verbatim against it; line links repointed. **CDDL type column:** added to the parameter tables in Sections 6.3, 7.2, 8, 9 and 10, giving each parameter's CDDL type and the range it allows; Section 5 explains it. **Section 11.3, `maxRefScriptSizePerBlock`:** guardrails MRSB-01 to MRSB-07 added, matching the constitution draft; MRSB-05 classed (~) as a cross-parameter comparison. **Section 11.3, `maxRefScriptSizePerTx`:** guardrails MRST-01 to MRST-10 added, matching the constitution draft, including MRST-08 (must not be decreased). **Section 11.3, `maxRefScriptSizePerEndorserBlock`:** guardrails MRSEB-01 to MRSEB-07 added, matching the constitution draft. **Section 11.3, `refScriptCostStride`:** guardrails RSCS-01 to RSCS-07 added, matching the constitution draft. **Section 11.3, `refScriptCostMultiplier`:** guardrails RSCM-01 to RSCM-05 added, matching the constitution draft. **Section 11.3, `minFeeRefScriptCostPerByte`:** new MFRS-05 (must not be zero); the constitution update corrects the ratified spelling `minFeeRefScriptCoinsPerByte` (Sections 7.2 and 11.2). **Section 11.3, `minPoolMargin`:** guardrails MPM-01 to MPM-05 added, matching the constitution draft, conditional on Section 8. **Section 11.3, `maxPledgeLeverage`:** guardrails MPL-01 to MPL-05 added, matching the constitution draft: the full CIP-0050 range 1 to 10,000 when set, null explicitly allowed, no prescribed initial value. **Section 11.3, `leiosAnnouncementPeriodLength`:** LAPL-01 to LAPL-05 replace the earlier "0 to 5000 ms" row, matching the constitution draft: upper bound 5,000 ms kept; lower bound [PENDING] and positive. **Section 11.3, `leiosVotePeriodLength`:** LVPL-01 to LVPL-05 replace the earlier "0 to 20000 ms" row, matching the constitution draft: upper bound 20,000 ms kept, lower bound [PENDING]. LAPL-05 and LVPL-05 allow the timing parameters to be changed individually, each change evaluated against the other two. **Section 11.3, `leiosDiffusionPeriodLength`:** LDPL-01 to LDPL-05 replace the earlier rows, matching the constitution draft; the 2^32-1 ms upper bound is replaced by a [PENDING] one. **Section 11.3, `leiosCommitteeSize`:** LCS-01 to LCS-08 replace the earlier rows, matching the constitution draft; the stake-coverage rule is (x), and there is deliberately no (y) upper bound. **Section 11.3, `leiosQuorumStakeThreshold`:** LQST-01 to LQST-09 replace the earlier rows, matching the constitution draft: greater than 0.5 and lower than 1.0 (no longer "smaller or equal to 1"), a 0.6 to 0.9 "should" band; duplicate ID removed. **Section 11.3, `maxEndorserBlockReferencesSize`:** MEBRS-01 to MEBRS-06 replace the earlier row, matching the constitution draft; 1,000,000 bytes kept as the upper bound. **Section 11.3, `maxEndorserBlockTxsSize`:** MEBTS-01 to MEBTS-08 replace the earlier row, matching the constitution draft; 12,000,000 bytes kept as the upper bound; MEBTS-04 (at least `maxTxSize`) as a correctness rule and MEBTS-08 stating the usefulness floor. **Section 11.3, `maxEndorserBlockExecutionUnits[memory/steps]`:** MEBEU-M-01 to M-08 and MEBEU-S-01 to S-08 added, matching the constitution draft. **Linear Leios kill switch:** `maxEndorserBlockTxsSize` = 0 is the designated way to deactivate Linear Leios (MEBTS-02 and MEBTS-04 carry the exception, MEBTS-02 becomes (x)); the other timing and size floors stay positive. **Section 11.3, joint Linear Leios timing relations:** new LLS-01 to LLS-03 (R2, R5 and the pipeline bound), with a new LLS prefix in Section 11.2; LAPL-05, LVPL-05 and LDPL-05 refer to them. **Section 11.3, `poolRetireMaxEpoch`:** ratified PRME-02 amended to PRME-02a (y) must not be lower than 1, matching the Dijkstra ledger's new well-formedness check. **Section 11.4:** new, listing the terms the guardrails rely on, including the two the constitution update now defines (Leios Committee, Leios Certificate). **Section 11.3, PE-01:** typos corrected ("lenght", "a divisible"); meaning unchanged. **`minPoolCost`:** ratified MPC-03 deprecated and marked "MPC-03a: Deprecated", so that `minPoolCost` can be set to 0 once `minPoolMargin` provides the floor (Sections 8 and 11.3). **Section 2 (Authority):** rewritten. The implementation decides names, keys, types and groups; the CIPs and ADR-9 decide semantics; this document decides guardrails and initial values; the constitution update follows this document. Previously the upstream specifications were said to win outright. **Metadata:** header version and date corrected, change log ordered newest first. |
| v1.2 | 2026-09-21 | Added `perasQuorumThresholdSafetyMargin` parameter required for Peras security |
| v1.1 | 2026-09-10 | Changes type of the `perasHealingFactor` parameter to reflect concrete bounded CDDL type |
| v1.0 | 2026-09-03 | **Replaces  [dijkstra-protocol-parameters-for-constitution-update](https://docs.google.com/document/d/1M649pDQtquYr4n5QBx_Q7vfzyFINQssN8Cpp553BGvA/edit?usp=sharing) and changes the document's purpose from working note to specification.** The previous document paraphrased the scope and sent the reader to the CIPs for the exact wording. This one keeps the CIPs authoritative and stops paraphrasing: it reproduces them verbatim at a pinned revision, refreshed whenever upstream changes, so the text here can be relied on and diffed against the source, with anything it notices raised on the CIP rather than corrected locally. |

---

## 2. Authority: what this document follows, and what follows it

Three sources bear on each parameter, and each decides a different thing:

1. **The implementation decides the facts on chain.** cardano-ledger `master` and the Dijkstra CDDL fix each parameter's name, update key, type and governance groups. Where the implementation and a CIP disagree on any of these, this document follows the implementation, records the difference, and raises it on the CIP. CIP-0164 giving the timing parameters in seconds while the ledger carries milliseconds (open issue 2), and the Peras parameter set differing from CIP-0140's `Params` record (Section 10), are both cases of this.
2. **The upstream specifications decide what each parameter means.** CIP-0164 for the nine Leios parameters, CIP-0023 for `minPoolMargin`, CIP-0050 for `maxPledgeLeverage`, and CIP-0140 for Peras. **The four reference-script parameters have no CIP.** Their upstream is ledger [ADR-9](https://github.com/IntersectMBO/cardano-ledger/blob/master/docs/adr/2024-08-14_009-refscripts-fee-change.md) (accepted, 2024-08-14), which plays the role a CIP plays elsewhere.
3. **This document decides the guardrails and initial values** (Sections 11 and 12), as agreed by the Parameters Committee with the technical teams.

**The constitution update follows this document.** Its parameter names, descriptions and guardrails must match the ones recorded here; a difference between the two is a defect in one of them, to be resolved rather than left standing.

Anything this document notices about a parameter's definition, semantics or symbol is raised upstream, on the CIP or with the ledger team, rather than settled here. Alternative implementations hold delivery milestones against specification conformance, so a bound that encodes a reading neither the implementation nor its upstream specification states would break that equivalence.

The parameter surface described here is fixed at the era boundary and cannot be extended afterwards without another era-to-era hard fork, because the era's `PParams` shape and CDDL are frozen when Dijkstra begins.

---

## 3. Why a constitution update before the hard fork

A parameter introduced at a hard fork holds its genesis value until the constitution names it and defines guardrails for it, introducing parameters in the Constitution ahead of the hardfork paves the way for the rollout. It is not a hard dependency, but it's desirable.

> PARAM-01 (y) Any protocol parameter that is not explicitly named in this
> document must not be changed by a "Parameter Update" action
>
> HARDFORK-05 (x) Any new updatable protocol parameters that are
> introduced with a hard fork must be included in this Appendix and
> suitable guardrails defined for those parameters
>
> NEW-CONSTITUTION-01a (x) A New Constitution or Guardrails Script
> governance action must be submitted to define any required guardrails
> for new parameters that are introduced via a Hard Fork governance action

And on naming:

> Note that, to avoid ambiguity, this Appendix uses the parameter name
> that is used in "Parameter Update" actions rather than any other
> convention.

[Dijkstra era scope](https://github.com/IntersectMBO/cardano-node/issues/6634) commits to submitting that Constitution Update ahead of the Dijkstra hard fork, once two conditions hold: the Dijkstra CDDL is ready, and the teams agree on guardrails for each new parameter. It will carry a new guardrails script.

---

## 4. Scope

**What this table covers.** Dijkstra features that introduce a new updatable protocol parameter. See [cardano-node#6634](https://github.com/IntersectMBO/cardano-node/issues/6634) for the full era scope.

| Feature | New updatable params | Implementation status | Constitution change |
| :---- | :---- | :---- | :---- |
| CIP-0164 Leios | 9 | `MERGED` (PR #6002, 2026-09-03) | **Yes** |
| Reference script pricing and limits | 4 (active on Conway as hardcoded values) | `MERGED` | **Yes** |
| CIP-0023 Fair Min Fees | 1 (`minPoolMargin`) | `MERGED`, logic `NOT STARTED`, activation at intra-era PV13 | Open, could defer to PV13 (Section 8) |
| CIP-0050 Pledge Leverage | 1 (`maxPledgeLeverage`) | `MERGED` | **Yes** |
| Peras (CIP-0140) | 6 (activation at intra-era PV13) | `MERGED` (PR #6067, 2026-09-28) | Open, could defer to PV13 (Section 10) |
| Plutus V4 | Cost model entry only, no new parameter | `MERGED` | No |

**Dijkstra introduces 21 new parameters in the code**, at parameter-update keys 34 to 54: the 9 Leios, the 4 reference-script, `minPoolMargin`, `maxPledgeLeverage` and the 6 Peras parameters. All 21 have to be in the code at PV12, whether or not they do anything yet, because the era's parameter set is fixed at the era boundary.

**14 of the 21 need naming in the constitution ahead of the hard fork**: the 9 Leios, the 4 reference-script and `maxPledgeLeverage`. Those are live at PV12 and unusable until the constitution names them and defines guardrails for them.

**The other 7 could be deferred to a later constitution update**: `minPoolMargin` (Section 8) and the 6 Peras parameters (Section 10). Both activate at the intra-era PV13. Deferring their inclusion in the Constitution is an option, these parameters are inactive on PV12 so it could be beneficial to take the v12 to v13 window to analyse bounds, including the first chance to see Peras and Leios running together. **Both are still decisions to be made**.

CIPs PR #1213 would add one or two more to every figure above if it merges (Section 6.2), may also come with PV13.

---

## 5. How to read the parameter tables

Sections 6 to 9 use the same columns.

**Two name columns.** The specification's name for the parameter, headed by whichever specification it comes from, and the ledger name. Appendix I must use the ledger name (Section 3), so that is what guardrails attach to.

**Update key.** The integer a "Parameter Update" action carries; the action holds no name at all. All 21 of Dijkstra's new parameters have keys, 34 to 54.

**Groups.** The governance groups the parameter belongs to.

**Status.** `MERGED` = on cardano-ledger `master`. `OPEN PR` = implemented but not merged. `NOT STARTED` = no implementation.

**Unit on chain.** The unit the value carries in a governance action, which is not always the unit its specification uses. Differences are called out on the row. Fractions and ratios are carried as exact rationals rather than decimals. Bounds on them are conventionally written as decimals, as the ratified text does in TC-01, "*treasuryCut* must not be lower than 0.1 (10%)".

**CDDL type (range).** The type the Dijkstra CDDL gives the parameter's entry in `protocol_param_update` (`eras/dijkstra/impl/cddl/data/dijkstra.cddl`, keys 33 to 54), with the range that type allows. The range is the outer limit on any guardrail: a value outside it cannot be encoded at all, so the ledger rejects it before the guardrails script is consulted. `uint .size 4` is 0 to 4,294,967,295 and `uint .size 2` is 0 to 65,535. `unit_interval` is 0 to 1; `nonnegative_interval` is 0 or more with no upper bound; `positive_interval` is greater than 0 with no upper bound. `ex_units` is a pair, memory first, each component 0 to 2⁶³−1. `nil` means the parameter can be unset.

Section 10 (Peras) uses the same columns with the CIP-0140 symbol in place of a spec name, plus the ledger's description of each parameter, since there is no CIP table to reproduce.

---

## 6. CIP-0164, Leios

**Source.** CIP-0164 as merged with CIP PR [CIPs#1250](https://github.com/cardano-foundation/CIPs/pull/1250) on 2026-09-15, commit `a2cac180`.

**Implementation status: `MERGED`.** All nine are on cardano-ledger `master` via PR [#6002](https://github.com/IntersectMBO/cardano-ledger/pull/6002), merged 2026-09-03. That PR implements CIP-0164 as corrected by #1250 and maps each parameter to the CIP's symbol, so the set and its semantics are the CIP's; the identifiers are the ledger's. The names in Section 6.3 are current as of the 2026-09-02 renames.

**Governance grouping.** All nine are **NetworkGroup** and **SecurityGroup**, so an update touching any of them needs SPO approval as well as DRep approval at the `dvtPPNetworkGroup` threshold. The placement comes from the ledger; CIP-0164 does not state it (open issue 1). The constitution carries its own group lists that this update has to extend, the critical-parameter list in Appendix I section 2.1 and the group lists in Appendix I section 9.

### 6.1 CIP-0164 Table 3, verbatim

A verbatim copy of Table 3 at CIPs commit `a2cac180` (PR #1250 as merged), row order included, so the two can be diffed mechanically. Cell padding is collapsed for legibility; every cell's content is the CIP's. 

<div align="center">

| Parameter | Symbol | Units | Description | Rationale |
| ---- | :----: | :----: | ---- | ---- |
| Committee size | $N_c$ | seats | Number of top-stake pools seated on the epoch's voting committee | Directly bounds votes per EB and certificate size. Governed from the stake distribution so that covered stake $\sigma(N_c)$ exceeds $\tau$ with headroom; see [feasible values](#table-7). |
| Quorum stake threshold | $\tau$ | fraction | Minimum fraction of total active stake that must be represented by votes in a certificate | Safety-critical. Must satisfy $0.5 < \tau < \sigma(N_c)$, and leave a sizable $\tau - \sigma_a$ of honest stake for the $\Delta_\text{EB}^{\text{W}}$ assumption; see [choosing the quorum threshold](#choosing-quorum-threshold). |
| <a id="l-hdr" href="#l-hdr"></a>Header diffusion period length | $L_\text{hdr}$ | seconds | Duration for RB headers to propagate network-wide | Per [equivocation detection](#equivocation-detection): must accommodate header propagation for equivocation detection. |
| <a id="l-vote" href="#l-vote"></a>Voting period length | $L_\text{vote}$ | seconds | Duration during which committee members can vote on endorser blocks | Per [voting period](#voting-period): must accommodate EB propagation and validation time. Set to minimum value that ensures honest parties can participate in voting |
| <a id="l-diff" href="#l-diff"></a>Diffusion period length | $L_\text{diff}$ | seconds | Additional period after voting to ensure network-wide EB availability | Per [diffusion period](#diffusion-period): derived from the fundamental safety constraint. Leverages the network assumption that data known to >25% of nodes propagates fully within this time |
| Maximum endorser block size | $S_\text{EB}$ | bytes | Maximum size of an endorser block itself | Limits EB size to ensure timely diffusion; prevents issues with many small transactions |
| Maximum total transaction size per endorser block | $S_\text{EB-tx}$ | bytes | Maximum total size of transactions that can be endorsed by an endorser block | Limits total transaction payload to ensure timely diffusion within stage length |
| Maximum reference script size per endorser block | $S_\text{EB-ref}$ | bytes | Maximum total size of reference scripts used by the transactions endorsed by an endorser block | The EB analogue of the Praos per-block `maxRefScriptSizePerBlock`: bounds the script bytes that must be loaded from the UTxO and evaluated per EB, which the EB's own size does not account for. |
| Maximum Plutus steps per endorser block | - | step units | Maximum computational steps allowed for Plutus scripts in a single endorser block | Limits computational resources per EB to ensure timely validation |
| Maximum Plutus memory per endorser block | - | memory units | Maximum memory allowed for Plutus scripts in a single endorser block | Limits memory resources per EB to ensure timely validation |

<em>Table 3: Protocol Parameters</em>

</div>

The note attached to Table 3 in the CIP, verbatim:

> [!NOTE]
>
> While per-transaction limits _could_ be increased based on the
> [evidence](#resource-requirements), the choice was made to deliberately **only
> introduce per-endorser-block limits** to avoid escalation of a threat:
> throughput-lowering attacks on certification would violate liveness of high
> plutus budget transactions.
>
> For example, a 26% stake attacker can trivially attack leios throughput with a
> 75% certification threshold. If an application would rely on higher (than what
> is available in praos) plutus demand transactions, those would not get
> included at all during such an attack.
>
> Despite, an application can work around this by splitting intense plutus work
> into multiple transactions and chaining them into, potentially, the same
> endorser block.

### 6.2 In-flight variant: CIPs PR #1213 would add one or two parameters to the nine

A second open PR would change the parameter set. CIP PR [#1213](https://github.com/cardano-foundation/CIPs/pull/1213) raises the Plutus resources a single transaction may use when it is carried in an endorser block. Its rows are for "Additional computational steps allowed for Plutus scripts on transactions in endorser blocks" and the matching allowance for memory, each of which "Expands the limit on computational resources per tx in an EB", and its note calls them "purely additive". The per-endorser-block budgets stay as they are; what changes is the ceiling on any one transaction inside such a block. In CIP terms that is two further rows in Table 3. In the terms this document works in, it is **two protocol parameters beyond the nine**, each needing a name, a key, guardrails and an initial value. Which of the two depends on the ledger.

Verbatim, the two rows it adds, under its own Table 3 header so they render:

| Parameter | Symbol | Units | Description | Rationale |
| ---- | :----: | :----: | ---- | ---- |
| Maximum _additional_ Plutus steps per tx in endorser block | - | step units | Additional computational steps allowed for Plutus scripts on transactions in endorser blocks | Expands the limit on computational resources per tx in an EB |
| Maximum _additional_ Plutus memory per tx in endorser block | - | memory units | Maximum memory allowed for Plutus scripts in a single endorser block | Expands the limit on memory resources per tx in an EB |


**Consequence for this specification.** If #1213 gets merged for PV12, the constitution update must name and bound an additional per-transaction Plutus budget for EBs. Whether that is one parameter or two depends on the ledger.

**Where this stands.** The ledger implements the nine, in #1250's shape. The parameters #1213 proposes are under active investigation and consideration, likely differed to a later era.

### 6.3 The nine parameters

Columns as set out in Section 5. Naming convention on the ledger side, per PR #6002: the `leios` prefix marks the parameters governing the protocol's rounds and its voting committee, and the size limits are named after what they bound, in the manner of the existing `maxTxSize` and `maxRefScriptSizePerBlock`. That is why the CIP's "Maximum endorser block size" is `maxEndorserBlockReferencesSize` on chain: what it bounds is the list of transaction references.

Ten CIP rows map to nine ledger parameters: the Plutus step and memory budgets are one value at key 47, the same shape as the existing `maxBlockExecutionUnits`.

| CIP-0164 name | Symbol | Ledger name | Update key | Unit on chain | CDDL type (range) | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| Committee size | $N_c$ | `leiosCommitteeSize` | 43 | seats | `uint .size 2` (0 to 65,535) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Quorum stake threshold | $\tau$ | `leiosQuorumStakeThreshold` | 44 | fraction of total active stake | `unit_interval` (0 to 1) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Header diffusion period length | $L_\text{hdr}$ | `leiosAnnouncementPeriodLength` | 40 | **milliseconds** | `uint .size 4` (0 to 4,294,967,295) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Voting period length | $L_\text{vote}$ | `leiosVotePeriodLength` | 41 | **milliseconds** | `uint .size 4` (0 to 4,294,967,295) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Diffusion period length | $L_\text{diff}$ | `leiosDiffusionPeriodLength` | 42 | **milliseconds** | `uint .size 4` (0 to 4,294,967,295) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum endorser block size | $S_\text{EB}$ | `maxEndorserBlockReferencesSize` | 45 | bytes | `uint .size 4` (0 to 4,294,967,295) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum total transaction size per endorser block | $S_\text{EB-tx}$ | `maxEndorserBlockTxsSize` | 46 | bytes | `uint .size 4` (0 to 4,294,967,295) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum reference script size per endorser block | $S_\text{EB-ref}$ | `maxRefScriptSizePerEndorserBlock` | 48 | bytes | `uint .size 4` (0 to 4,294,967,295) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum Plutus memory per endorser block | none | `maxEndorserBlockExecutionUnits[memory]` | 47 | memory units | `ex_units`, memory component (0 to 2⁶³−1) | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum Plutus steps per endorser block | none | `maxEndorserBlockExecutionUnits[steps]` | 47 | step units | `ex_units`, steps component (0 to 2⁶³−1) | **NetworkGroup**, **SecurityGroup** | `MERGED` |

The `[memory]` and `[steps]` notation follows the constitution's treatment of `maxBlockExecutionUnits`, guardrailed separately per index. On chain the two are one value at key 47, a pair ordered **memory first** (`ex_units = [mem, steps]`); these two rows are the only place this table departs from Table 3's order. An update to that key carries both components.

**Units.** CIP-0164 gives the three timing parameters in seconds; The implementation uses milliseconds, so 1 s, 4 s and 7 s are 1000 ms, 4,000ms and 7,000ms. **This document writes every timing value in milliseconds.** Aligning the CIP's Units column is open issue 2.

### 6.4 Relations: the constraints these parameters must jointly satisfy

Leios security constrains the timing parameters jointly, against each other and against network characteristics that are measured rather than governed. Tables 1 and 2 are those characteristics, verbatim.

**Line links point at CIPs commit `a2cac180` (PR #1250 as merged), not at `master`.** 

<div align="center">

| Characteristic | Symbol | Description | Observed Range by Simulations |
| ---- | :----: | ---- | :----: |
| <a id="delta-hdr" href="#delta-hdr"></a>Header propagation | $\Delta_\text{hdr}$ | Time for constant size headers (< 1,500 bytes) to propagate network-wide | < 1 second |
| <a id="delta-rb" href="#delta-rb"></a>RB diffusion | $\Delta_\text{RB}$ | Complete ranking block propagation and adoption time | < 5 seconds |
| <a id="delta-eb-H" href="#delta-eb-H"></a>EB optimistic diffusion | $\Delta_\text{EB}^{\text{O}}$ | EB **diffusion** time (transmission + processing) under favorable network conditions | 1-3 seconds |
| <a id="delta-eb-A" href="#delta-eb-A"></a>EB worst-case transmission | $\Delta_\text{EB}^{\text{W}}$ | EB **transmission** time for certified EBs starting from >25% network coverage (processing already completed during voting) | 15-20 seconds |

<em>Table 1: Network Characteristics</em>

</div>

<div align="center">

| Characteristic | Symbol | Description | Observed Range by Simulations |
| ---- | :----: | ---- | :----: |
| <a id="delta-reapply" href="#delta-reapply"></a>EB reapplication | $\Delta_\text{reapply}$ | Certified EB reapplication with minimal checks and UTxO updates | < 1 second |
| <a id="delta-applytxs" href="#delta-applytxs"></a>Transaction validation | $\Delta_\text{applyTxs}$ | Standard Praos transaction validation time for RB processing | ~1 second |

<em>Table 2: Ledger Characteristics</em>

</div>

The constraints specified in the CIP must be taken into account when setting initial values for the Leios parameters and their guardrails:

**R1. Equivocation detection window.** [L576-L582](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L576-L582). Voting starts $3 \times L_\text{hdr}$ after an RB announces an EB, one $L_\text{hdr}$ for each step of the CIP's Detection Mechanism list: "Initial header propagation", "Conflicting header propagation" and "Equivocation evidence propagation". Honest nodes then refuse to vote for any EB from an RB slot where equivocation was detected ([L589-L592](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L589-L592)), and EB diffusion continues during this window ([L597](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L597)).

**R2. Voting period.** [L614-L616](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L614-L616). Verbatim: "The voting period must accommodate EB diffusion (transmission and processing):"

$$3 \times L_\text{hdr} + L_\text{vote} > \Delta_\text{EB}^{\text{O}}$$

**R3. Diffusion period.** [L641-L643](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L641-L643). Verbatim: "The diffusion period must satisfy:"

$$L_\text{diff} \geq \Delta_\text{EB}^{\text{W}} + \Delta_\text{reapply} - \Delta_\text{RB} - 3 \times L_\text{hdr} - L_\text{vote}$$

**R4. EB reapplication constraint.** [L689-L691](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L689-L691). Verbatim: "Reapplying a certified EB cannot cost more than standard transaction processing."

$$\Delta_\text{reapply} < \Delta_\text{applyTxs}$$

**R5. Certified EB transmission constraint.** [L697-L700](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L697-L700). Verbatim: "Any certified EB referenced by an RB must be transmitted (but not necessarily be processed) before that RB needs to be processed."

$$\Delta_\text{EB}^{\text{W}} < 3 \times L_\text{hdr} + L_\text{vote} + L_\text{diff} + (\Delta_\text{RB} - \Delta_\text{applyTxs})$$

**R6. Quorum threshold** [L504](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L504) and [L2568-L2577](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L2568-L2577). From below, $\tau$ must exceed the adversarial stake Praos tolerates by a sizable margin, because $\tau - \sigma_a$ is the honest stake provably holding every certified EB when voting ends, and that remainder is the coverage $\Delta_\text{EB}^{\text{W}}$ is measured from. Verbatim: "Below $\tau = 0.5$ the bound fails outright, since two disjoint sets of voters could each reach a quorum and certify conflicting EBs; a threshold in the 50s clears that only nominally and leaves nothing for the assumption above." From above:

$$0.5 < \tau < \sigma(N_c)$$

where $\sigma(N_c)$ is the cumulative active stake of the top $N_c$ pools, read off the prevailing stake distribution. $\sigma(N_c) - \tau$ is the abstention budget: the seated stake that can decline to vote before no quorum can form.

**R7. Certificate inclusion delay in slots.** [L408-L423](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L408-L423). The total delay $3 \times L_\text{hdr} + L_\text{vote} + L_\text{diff}$ is a wall-clock duration, and chain inclusion needs a whole number of slots, so it is divided by `slotLength` and **rounded up**. At mainnet's `slotLength` of one second the division is exact for the CIP's feasible values.

**R8. Committee size against the stake distribution.** [L2634-L2640](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L2634-L2640). $\sigma(N_c)$ must exceed $\tau$ or the abstention budget is negative and no quorum can form. Verbatim: "This is a hard floor rather than an operating point: at exactly $\tau$ every seated pool must vote and be reachable within $L_\text{vote}$, so a practical choice leaves budget for pools that are offline, [keyless](#key-registration), or slow."

**R9. Individual transaction size is unchanged.** [L661-L666](https://github.com/cardano-foundation/CIPs/blob/a2cac18039d7bab6c0dc1a806ad8a96b8b0546fc/CIP-0164/README.md?plain=1#L661-L666). Verbatim: "Note that $S_\text{EB-tx}$ does not change the maximum size of individual transactions. The existing `maxTxSize` parameter remains unchanged and continues to limit individual transaction sizes."

---

## 7. Ledger ADR-9, Reference script pricing and limits

### 7.1 Origin: why the values exist, and why they become parameters now

Source throughout: ledger [ADR-9](https://github.com/IntersectMBO/cardano-ledger/blob/master/docs/adr/2024-08-14_009-refscripts-fee-change.md), accepted 2024-08-14.

**The problem.** Deserialising a script carries an overhead large enough that a script can be made expensive to deserialise and cheap to execute. Used as a reference script, that was an attack vector, and ADR-9 records it as exacerbated by there being no limit on reference-script bytes per transaction beyond the transaction size itself. The result was a DDoS route: many transactions costing the attacker very little and expensive for a node to validate. The underlying gap, per ADR-9, is that reference-script overhead "was not properly accounted for when this feature was introduced in the Babbage era".

**The first attempt, and why it fell short.** Conway introduced `minFeeRefScriptCostPerByte`, a flat rate per reference-script byte. ADR-9 records that it was deliberately set to a moderate value rather than a deterrent one, because a rate high enough to stop the attack would have made reference scripts nearly as expensive as regular scripts, with the intention of tuning it later.

**What forced the current design.** The attack was carried out on mainnet on 25 June 2024. A flat rate cannot be both cheap for ordinary use and a deterrent at scale, so ADR-9 replaced it with a curve, verbatim: "Instead of using the same linear cost for the whole size we split this total size into `25KiB` chunks and each subsequent chunk will get a linear pricing cost that is higher than the previous one by a multiplier of `1.2`. In other words pricing for the first `25KiB` will be as with the initial approach, just the value of `minFeeRefScriptCostPerByte`. The following `25KiB` will have the price of `minFeeRefScriptCostPerByte * multiplier` and so on." Price grows exponentially while size grows linearly, so the common case stays cheap, which ADR-9 puts at up to 25 KiB per transaction, and climbs steeply above it: the last 25 KiB before the per-transaction cap is priced at 3.6 times the base rate. The chunk count restarts for each transaction. Two hard caps bound the total, 200 KiB per transaction and 1 MiB per block.

**Why the values were hardcoded, and why that ends now.** Adding protocol parameters was out of reach that late in the Conway release cycle, so the four values were compiled in, with the intent stated in ADR-9 verbatim: they "will be turned into proper protocol parameters in the next era". Dijkstra is that era, and that is the whole of the change. The same four values are stated in Dijkstra genesis and become updatable by governance, with the fee curve and the caps behaving exactly as they do today.

### 7.2 The four parameters

**Governance grouping.** All four are **NetworkGroup** and **SecurityGroup**, so SPO-voted.

| ADR-9 name | Ledger name | Update key | CDDL type (range) | Conway hardcoded value | What it bounds | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| "Limit per block" | `maxRefScriptSizePerBlock` | 34 | `uint .size 4` (0 to 4,294,967,295) | 1,048,576 bytes (1 MiB) | Total bytes of all reference scripts used by all transactions in a block | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| "Limit per transaction" | `maxRefScriptSizePerTx` | 35 | `uint .size 4` (0 to 4,294,967,295) | 204,800 bytes (200 KiB) | Total bytes of reference scripts a single transaction may use | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| "Size increment" (`sizeIncrement`) | `refScriptCostStride` | 36 | `positive_word32` (1 to 4,294,967,295) | 25,600 bytes (25 KiB) | Size increment within which the per-byte price stays constant before the next escalation step | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| "Multiplier" (`multiplier`) | `refScriptCostMultiplier` | 37 | `positive_interval` (greater than 0) | 1.2 | Growth factor applied to the per-byte price at each stride boundary | **NetworkGroup**, **SecurityGroup** | `MERGED` |

ADR-9 names these four descriptively rather than as identifiers, so the first column quotes its labels, with `sizeIncrement` and `multiplier` being the names its code snippet uses. The values come from the same place, verbatim: "Limit per transaction: `200KiB` (or `204800` bytes)", "Limit per block: `1MiB` (or `1048576` bytes)", "Size increment: `25KiB` (or 25,600 bytes)" and "Multiplier: `1.2`". Zero is rejected by the encoding for the stride and the multiplier, before any guardrail is consulted.

**`minFeeRefScriptCostPerByte` is already governed.** A Conway parameter, key 33, **EconomicGroup** and **SecurityGroup**, carrying MFRS-01 (must not exceed 1,000) and MFRS-02 (must not be negative). **The ratified constitution spells it `minFeeRefScriptCoinsPerByte`, in all seven places it appears, and the parameter-update name is `minFeeRefScriptCostPerByte`.** By the constitution's own rule in Section 3 the parameter-update name governs, so the constitution update corrects the spelling in all seven places. MFRS-01 to MFRS-04 keep their content; only the name changes. The update also adds MFRS-05 (Section 11.3).

---

## 8. CIP-0023, Min Pool Margin `minPoolMargin`, also in constitution update?

| CIP-0023 name | Ledger name | Update key | Unit on chain | CDDL type (range) | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| `minPoolMargin` (identical) | `minPoolMargin` | 39 | fraction | `unit_interval` (0 to 1) | **EconomicGroup** | `MERGED`, logic not started |

**Definition** (CIP-0023, verbatim): "`minPoolMargin` defines the lower bound for the pool margin (variable fee), i.e., the minimum allowable percentage of rewards a pool can take. Pool registration and update certificates MUST have `margin >= minPoolMargin`." Bounding the `margin` field of `pool_params`, itself a `unit_interval`, fixes the parameter's domain at 0 to 1 without CIP-0023 having to state it. It complements `minPoolCost`, the fixed-fee floor, which is already governed by MPC-01 and MPC-02; this update deprecates MPC-03 (Section 11.3), so that `minPoolCost` can eventually be set to 0 with `minPoolMargin` as the floor.

**Mechanics.** The backward-compatible route CIP-0023 itself recommends is the one being taken, rather than the MUST-reject rule. Verbatim from CIP-0023: "if a pool's margin is less than `minPoolMargin`, the protocol-level `minPoolMargin` overrides the pool's registered `margin` during reward calculation. This minimizes disruption and lets legacy pool certificates remain valid". Pool registrations and updates with a lower margin are therefore not rejected, and existing certificates stay valid. The same clamping is planned for the pool `cost` field against `minPoolCost`.

**Status.** The parameter is merged, and nothing reads it: `ppMinPoolMarginL` appears in the ledger only in the parameter definition, the changelog and a genesis test. The reward-calculation logic is **not started**, ledger issue [#5954](https://github.com/IntersectMBO/cardano-ledger/issues/5954), and cardano-node#6634 puts activation at the intra-era PV13.

**Could be deferred, on the same footing as Peras (Section 10).** The parameter is in the code at PV12 and holds key 39, but nothing reads it until PV13, so naming it now would make governable a value that has no effect, and would mean agreeing guardrails before the behaviour they bound exists. Deferring costs nothing meanwhile, since PARAM-01 holds it at its genesis value of 0. Still a decision to be made.

---

## 9. CIP-0050, Pledge Leverage `maxPledgeLeverage`

| CIP-0050 name | Ledger name | Update key | Unit on chain | CDDL type (range) | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| "Maximum pledge leverage", $L$ | `maxPledgeLeverage` | 38 | ratio | `max_pledge_leverage`: `nonnegative_interval` (0 or more) or `nil` | **TechnicalGroup** | `MERGED` |

**One of two nullable parameters in the set**, with `perasBootstrapRound` (Section 10). The value may be a non-negative ratio or `null`, and `null` is how it is introduced at the hard fork, which is equivalent to how every era before Dijkstra behaves. No parameter currently named in the constitution is nullable, so the guardrails script has no precedent.

**Definition.** A cap on pool leverage, the ratio of a pool's total stake to its pledge, applied inside the rewards calculation.

**Mechanics.** Eligible stake in the rewards formula becomes the minimum of the pool's stake, the saturation point z0 = 1/k, and L times its pledge. Stake above L times pledge earns nothing, for the operator or its delegators. With the parameter set, a zero-pledge pool earns zero rewards on delegated stake. Pools whose pledge supports their stake are unaffected and keep the a0 boost.

**Range** (CIP-0050, verbatim): "**Range of *L*:** 1 ≤ *L* ≤ 10,000 (dimensionless ratio). An *L* of 10,000 represents an extremely high allowed leverage (i.e. pledge need only be 0.01% of the stake), effectively similar to the status quo with a very weak pledge influence. An *L* of 1 represents a very strict requirement where a pool's stake cannot exceed its pledge (100% pledge) if it is to earn full rewards." Its simulations swept L over {10, 100, 1000, 10,000} and concluded, verbatim: "The sweet spot lies at the lower end of the tested range (~10 to ~100), where Sybil protection is strongest yet the number of viable pools remains healthy."

**Status.** Merged, with the reward-calculation semantics active. Introduced unset, so per ledger PR #5943 "for the feature to really become active the first real value must be set via a governance action". The constitution entry is therefore a precondition for the feature doing anything at all.

---

## 10. Peras Parameters at PV12, also in constitution update?

**Source.** [CIP-0140](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0140). Its `Params` record defines seven protocol parameters: $U$ round length, $L$ block-selection offset, $A$ certificate expiration, $R$ chain-ignorance period, $K$ cool-down period, $B$ certification boost and $\tau$ quorum. Committee size $n$ appears only in its feasible-values table. Some of those parameters are dependent and some are not governable, so the implemented set differs from the CIP's (see below). The CIP update to match the implemented set is a work in progress.

**Implementation status: `MERGED`.** All six are on cardano-ledger `master` via PR [#6067](https://github.com/IntersectMBO/cardano-ledger/pull/6067), merged 2026-09-28, which closed ledger issue [#5966](https://github.com/IntersectMBO/cardano-ledger/issues/5966). Names, update keys, types and groups below are the ledger's: `eras/dijkstra/impl/src/Cardano/Ledger/Dijkstra/PParams.hs` and `eras/dijkstra/impl/cddl/data/dijkstra.cddl`, keys 49 to 54. The descriptions are the ledger's doc comments. Other sources that track the set: the [Peras decision log](https://github.com/tweag/cardano-peras/blob/main/doc/adr/0003-protocol-parameters.md), which discusses each parameter and whether it should be governable, and [cardano-peras#283](https://github.com/tweag/cardano-peras/issues/283).

**Governance grouping.** All six are **NetworkGroup** and **SecurityGroup**, so an update touching any of them needs SPO approval as well as DRep approval at the `dvtPPNetworkGroup` threshold, the same as the nine Leios parameters. The placement comes from the ledger; CIP-0140 does not state it (open issue 1). Earlier versions of this document expected TechnicalGroup for the DRep threshold.

| CIP-0140 symbol | Ledger name | Update key | Unit on chain | CDDL type (range) | Description (ledger) | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| $L$ | `perasMinCandidateBlockAge` | 49 | slots | `slot_interval` = `uint .size 4` (0 to 4,294,967,295) | Minimum age for a block to be eligible for voting in a Peras round. | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| $h$ | `perasHealingFactor` | 50 | ratio | `positive_interval` (greater than 0) | Healing time coefficient used to derive the length of a cooldown period. | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| $B$ | `perasCertBoost` | 51 | chain weight | `uint .size 2` (0 to 65,535) | Extra chain weight that a Peras Certificate gives to a boosted block. | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| $n$ | `perasTargetCommitteeSize` | 52 | members | `uint .size 2` (0 to 65,535) | Target size of the voting committee. | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| $R_\text{bootstrap}$ | `perasBootstrapRound` | 53 | round number | `nil` or `uint .size 4` (0 to 4,294,967,295) | Round number used to manually bootstrap voting at a specific time. | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| $\tau_\mathsf{margin}$ | `perasQuorumThresholdSafetyMargin` | 54 | fraction | `unit_interval` (0 to 1) | Extra safety margin added on top of the 75% quorum threshold baseline. Must be between 0 and 0.25, inclusive. | **NetworkGroup**, **SecurityGroup** | `MERGED` |

The names are the parameter-update names. `peras…L` in the ledger source is the Haskell accessor, not the name a "Parameter Update" action uses (Section 3). Proposed initial values are in Section 12.

> [!NOTE]
>
> Committee size $n$ is a governed quantity in CIP-0140's feasible-values table but not a field of its `Params` record. There are some discussions how to track committee size and a committee selection algorithm with few various approaches, that affect the certificate size. $n$ parameter is required only in one of those, but so far Peras team believes that including this parameter is the cleanest and safest approach, no matter what decision will take an effect. The ledger now carries it as `perasTargetCommitteeSize`.

Other parameters that are mentioned in CIP could not be included as they are either non-changeable without implementation modifications or have only one reasonable value. In particular the quorum threshold $\tau$ is not a parameter: the ledger fixes it at a 75% baseline, and `perasQuorumThresholdSafetyMargin` is added on top of it.

`perasBootstrapRound` was not considered in CIP-140 because CIP did not cover the Peras enablement. The Peras voting rules are defined in a way that if there were no Peras votes on the chain Peras could not start voting. There are two ways how to bootstrap Peras, either to modify voting rules and introduce special cases that would lead to additional research, or introduce a special parameter that would allow to unblock Peras. Peras team decided that such parameter is better solution because it largely simplify codebases (incl. alternative nodes), rules and introduces a way to unblock Peras in case of an unforeseen situation. It is nullable, like `maxPledgeLeverage` (Section 9), and is unset when introduced.

`perasQuorumThresholdSafetyMargin` is required to mitigate potential security due to possible transfer of the votes to the malicious nodes. The ledger's doc comment bounds it to 0 to 0.25 inclusive, but no ledger rule enforces that bound; only the `unit_interval` type (0 to 1) is enforced. A guardrail is therefore the only enforcement of the 0.25 limit (Section 11.3).

> [!NOTE]
>
> As per CIP-140, when Peras will be enabled we will have to change security parameter $k$ to keep the same safety properties. However in this update we do not propose to change those as Peras is not planned to be enabled in the Hard Fork.

---

## 11. Guardrails

**Open.** Bounds are for the Parameters Committee to agree with the technical teams and protocol designers.

### 11.1 How the ratified constitution expresses a guardrail

Each guardrail carries a symbol saying whether the Guardrails Script can enforce it. The three, verbatim from Appendix I:

> Symbol and Explanation
>
> -   (y) The Guardrail Script can be used to enforce the Guardrail
>
> -   (x) The Guardrail Script cannot be used to enforce the Guardrail
>
> -   (~ - reason) The Guardrail Script cannot be used to enforce the
>     Guardrail for the reason given, but future ledger changes could
>     enable this.

For further details on the Guardrails how they operate and how to set them always refer to the on-chain ratified Constitution at https://bafkreieyuknozbtewyurfqoagvplvykadn6a4u6wglupavdz46bbsnnl6e.ipfs.inbrowser.link/

### 11.2 Guardrail identifier naming

The ratified Appendix I uses a parameter initialism plus a sequence number (MBBS for `maxBlockBodySize`, MFRS for `minFeeRefScriptCostPerByte`, spelt `minFeeRefScriptCoinsPerByte` in the ratified text, MPC for `minPoolCost`, PPI for `poolPledgeInfluence`), with an extra letter for indexed parameters (MBEU-S, MBEU-M). Following that convention we propose:

| Ledger name | Prefix |
| :---- | :---- |
| Joint Linear Leios timing relations | LLS |
| `leiosAnnouncementPeriodLength` | LAPL |
| `leiosVotePeriodLength` | LVPL |
| `leiosDiffusionPeriodLength` | LDPL |
| `leiosCommitteeSize` | LCS |
| `leiosQuorumStakeThreshold` | LQST |
| `maxEndorserBlockReferencesSize` | MEBRS |
| `maxEndorserBlockTxsSize` | MEBTS |
| `maxEndorserBlockExecutionUnits[memory]` | MEBEU-M |
| `maxEndorserBlockExecutionUnits[steps]` | MEBEU-S |
| `maxRefScriptSizePerEndorserBlock` | MRSEB |
| `maxRefScriptSizePerBlock` | MRSB |
| `maxRefScriptSizePerTx` | MRST |
| `refScriptCostStride` | RSCS |
| `refScriptCostMultiplier` | RSCM |
| `minPoolMargin` | MPM |
| `maxPledgeLeverage` | MPL |
| `perasMinCandidateBlockAge` | PMCBA |
| `perasCertBoost` | PCB |
| `perasTargetCommitteeSize` | PTCS |
| `perasHealingFactor` | PHF |
| `perasQuorumThresholdSafetyMargin` | PQTSM |
| Era length | PE |


**Pending Peras Parameters.**

### 11.3 Guardrails, per parameter WIP

One block per parameter, to be filled in. Each carries a placeholder row showing the shape: replace it, and number upwards from `-01` as the ratified text does (MBBS-01, MBBS-02 and so on).

#### Joint Linear Leios timing relations (LLS)

Placed before the three timing parameters, as the ratified Appendix I places group-wide guardrails before parameter-specific ones. The constitution text defines the measured terms used here (Endorser Block diffusion time, certified Endorser Block transmission time, Block diffusion time, transaction validation time) as characteristics established by benchmarking and simulation, not as fixed values, so all three guardrails are (x). They constrain the resulting values; any one timing parameter may be changed on its own.

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LLS-01 | (x) | 3 × `leiosAnnouncementPeriodLength` + `leiosVotePeriodLength` must exceed the Endorser Block diffusion time, so that Leios Committee members can receive and validate an Endorser Block before voting on it ends. This must continue to hold after any change to the timing parameters or to `maxEndorserBlockReferencesSize`, `maxEndorserBlockTxsSize` or `maxEndorserBlockExecutionUnits`, since larger Endorser Blocks take longer to diffuse | R2 (CIP-0164 voting period, L614-616); Δ_EB^O. TSC notes LLS-01, plus their note that larger EBs tighten R2 |
| LLS-02 | (x) | 3 × `leiosAnnouncementPeriodLength` + `leiosVotePeriodLength` + `leiosDiffusionPeriodLength` + (Block diffusion time − transaction validation time) must exceed the certified Endorser Block transmission time, so that every honest node receives a certified Endorser Block in time to process a Block that includes its certificate within the Block diffusion time | R5, CIP-0164 Constraint 2 (L695-700), whose formula this states; the security argument at L706-715. Together with R4 (MEBTS-05, MEBEU-M-05, MEBEU-S-05) it implies R3, so R3 needs no guardrail of its own. At the CIP's values the deadline is 14 + (5 − 1) = 18 s, against Table 1's Δ_EB^W of 15 to 20 s. TSC notes LLS-02 |
| LLS-03 | (x - "should") | 3 × `leiosAnnouncementPeriodLength` + `leiosVotePeriodLength` + `leiosDiffusionPeriodLength` should be well below the expected interval between Blocks (20 seconds at the current active slot coefficient of 0.05 and slot length of one second). Intervals between Blocks are random, so the longer this total, the larger the share of Blocks produced before a certificate can be included: at 14 seconds, about half of all Blocks | CIP-0164 L2466-2473: at a 14-slot delay, 0.95¹³ ≈ 51% of Blocks carry transactions directly. TSC notes LLS-04, which estimates e^(−T/20) ≈ 50% |

The TSC notes' LLS-03 (R8, committee coverage above τ) is not repeated here: LCS-04 and LQST-05 state it.

#### `leiosAnnouncementPeriodLength` ($L_\text{hdr}$), milliseconds WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LAPL-01 | (y) | must not be lower than [PENDING] milliseconds | Lower bound; value to be agreed. Must be positive: 0 would remove the 3 × L_hdr wait that equivocation detection relies on (CIP-0164 L576-597; TSC notes). The Linear Leios kill switch is `maxEndorserBlockTxsSize` = 0 (MEBTS-02), not this parameter. Meaningful values are around 1,000 ms (CIP-0164 feasible value 1 s) |
| LAPL-02 | (y) | must not exceed 5,000 milliseconds | The delta assumption of Praos; agreed by this document's earlier draft and the TSC notes |
| LAPL-03 | (y) | must not be negative | Ratified pattern |
| LAPL-04 | (x) | must exceed the time it takes a Block header to propagate network-wide, with headroom for adverse network conditions, as measured by benchmarking and simulation on a mainnet-representative topology | L_hdr is the header propagation bound behind equivocation detection (R1; CIP-0164 Table 1, Δ_hdr < 1 s). Headers must reach all honest nodes, not a percentile of them. Headroom is stated in words; no figure has been proposed |
| LAPL-05 | (x) | any change must be evaluated together with the values of `leiosVotePeriodLength` and `leiosDiffusionPeriodLength` that will apply alongside it, and validated via simulation against LLS-01 to LLS-03. The three parameters may be changed individually or together | The relations are joint, so each change is checked against the other two, but a single parameter may be adjusted on its own when evidence supports it (for example, one more second of L_vote to improve the certification rate) |
#### `leiosVotePeriodLength` ($L_\text{vote}$), milliseconds WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LVPL-01 | (y) | must not be lower than [PENDING] milliseconds | Lower bound; value to be agreed. Must be positive. 0 is not the kill switch: EBs would still be produced and diffused with no voting window; the designated switch is `maxEndorserBlockTxsSize` = 0 (MEBTS-02). CIP-0164 feasible value 4 s |
| LVPL-02 | (y) | must not exceed 20,000 milliseconds | The expected inter-block time at the current `activeSlotCoefficient` of 0.05 and one-second slots; agreed by this document's earlier draft and the TSC notes. Expected, so probabilistic: RB gaps are random. The binding limit is the whole pipeline 3 × L_hdr + L_vote + L_diff, to be addressed with the joint timing guardrails |
| LVPL-03 | (y) | must not be negative | Ratified pattern |
| LVPL-04 | (x) | must be long enough for Leios Committee members to receive and validate an Endorser Block and for their votes to diffuse to Block producers, as shown by benchmarking and simulation on a mainnet-representative topology | The voting period's purpose (R2; CIP-0164 voting period) in words, so the guardrail does not depend on an undefined Δ term (TSC notes) |
| LVPL-05 | (x) | any change must be evaluated together with the values of `leiosAnnouncementPeriodLength` and `leiosDiffusionPeriodLength` that will apply alongside it, and validated via simulation against LLS-01 to LLS-03. The three parameters may be changed individually or together | As LAPL-05 |
#### `leiosDiffusionPeriodLength` ($L_\text{diff}$), milliseconds WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LDPL-01 | (y) | must not be lower than [PENDING] milliseconds | Lower bound; value to be agreed. The CIP-0164 relations give a minimum of about 4,000 ms at its example values, and the CIP uses 7,000 ms for safety (`CIP-0164.md:2462`); the TSC notes put R5's range at 4,000 to 9,000 ms across Table 1's Δ_EB^W range. 0 is not an off switch here: it would let a certificate be included before the certified EB has reached all honest nodes (R5) |
| LDPL-02 | (y) | must not exceed [PENDING] milliseconds | A real upper bound for normal operation, replacing the earlier "must not be greater than 2^32-1ms". That bound was the type's maximum, so the script could never reject anything with it, and no source defines 2^32-1 ms as "certification off" (CIP-0164, ledger and consensus `main` checked 2026-10-08); it only prevents certification in practice. L_diff carries no deactivation value; the designated kill switch is `maxEndorserBlockTxsSize` = 0 (MEBTS-02). The whole pipeline 3 × L_hdr + L_vote + L_diff must fit well within the expected 20 s block gap |
| LDPL-03 | (y) | must not be negative | Ratified pattern |
| LDPL-04 | (x) | must be long enough for a certified Endorser Block to reach all honest nodes before a Block that includes its certificate needs to be processed, as shown by benchmarking and simulation on a mainnet-representative topology | The diffusion period's purpose (R3, R5; CIP-0164 diffusion period) in words, so the guardrail does not depend on undefined Δ terms (TSC notes) |
| LDPL-05 | (x) | any change must be evaluated together with the values of `leiosAnnouncementPeriodLength` and `leiosVotePeriodLength` that will apply alongside it, and validated via simulation against LLS-01 to LLS-03. The three parameters may be changed individually or together | As LAPL-05 |

#### `leiosCommitteeSize` ($N_c$), seats WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LCS-01 | (y) | must not be negative | Ratified pattern |
| LCS-02 | (y) | must not be zero | A zero committee can never reach a quorum |
| LCS-03 | (y) | must not be lower than [PENDING] (seats) | Lower bound; value to be agreed. CIP-0164 floors at mainnet epoch 649: σ(N_c) must exceed τ, which is P75 ≈ 302 seats at τ = 0.75, and never falls below P50 = 160 seats since τ > 0.5. Both move with the stake distribution. CIP-0164 feasible value 900 seats (P99 = 890) |
| LCS-04 | (x) | must be set such that, under the prevailing stake distribution, the active stake held by the seated Stake Pools exceeds `leiosQuorumStakeThreshold`, with sufficient headroom for seated Stake Pools that do not vote | R8. Needs the stake distribution, so (x). Replaces the earlier "(y) a pool count that together control [eg 95%?] of the active stake", which the script cannot check (TSC notes) |
| LCS-05 | (x) | any increase must be shown by benchmarking and simulation to leave the resulting vote traffic per Endorser Block diffusing within `leiosVotePeriodLength` | Each seat adds one vote per EB on the wire. **No (y) upper bound, deliberately:** the parameter is liveness-only (CIP-0164: a mis-set N_c costs throughput, never safety), and this guardrail is what limits it. CIP-0164 has simulated vote diffusion up to 1,500 voters per EB |
| LCS-06 | (x) | any change must be supported by an analysis of the prevailing stake distribution showing that LCS-04 continues to hold | The parameter may be changed in either direction (TSC notes) |
| LCS-07 | (x) | `leiosCommitteeSize` and `leiosQuorumStakeThreshold` must be evaluated jointly; any change to either must be accompanied by an analysis confirming that `leiosQuorumStakeThreshold` remains strictly lower than the stake coverage of the seated Stake Pools | R6 and R8 |
| LCS-08 | (x - "should") | should be reviewed whenever `stakePoolTargetNum`, `maxPledgeLeverage` or the stake distribution change materially | Each of these moves stake between pools, and so the coverage of a fixed committee size (TSC notes) |

#### `leiosQuorumStakeThreshold` ($\tau$), `unit_interval` WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LQST-01 | (y) | must be greater than 0.5 (50%) | R6, from below: below 0.5 two disjoint sets of voters could each certify conflicting EBs. Expressed in the script as `minValue 0.5` plus `notEqual 0.5` |
| LQST-02 | (y) | must be lower than 1.0 (100%) | Replaces the earlier "smaller or equal to 1": at 1.0 all active stake would have to vote, including stake outside the committee, so certification would practically never succeed; it also contradicts τ < σ(N_c) (TSC notes). Expressed as `maxValue 1` plus `notEqual 1` |
| LQST-03 | (x - "should") | should not be lower than 0.6 (60%) | CIP-0164: "a threshold in the 50s clears that only nominally and leaves nothing for the assumption above" (TSC notes) |
| LQST-04 | (x - "should") | should not exceed 0.9 (90%) | Liveness: σ(N_c) − τ is the abstention budget, and an adversary holding it can block certification by not voting; CIP-0164 notes a 26% stake attacker can stall throughput at τ = 0.75 (TSC notes). CIP-0164 suggests 0.75 as a starting value |
| LQST-05 | (x) | must be strictly lower than the share of active stake held by the Stake Pools seated under `leiosCommitteeSize`, under the prevailing stake distribution | R6 from above, R8. Needs the stake distribution |
| LQST-06 | (x) | any decrease must be accompanied by a formal security analysis demonstrating that the resulting threshold still ensures the fraction of honest nodes guaranteed to know a certified Endorser Block exceeds the minimum required for safe diffusion within `leiosDiffusionPeriodLength`, under the assumed adversarial stake bound | R6: τ − σ_a is the honest stake the diffusion assumption rests on |
| LQST-07 | (x) | `leiosQuorumStakeThreshold` and `leiosCommitteeSize` must be evaluated jointly; any change to either must be accompanied by a combined security analysis demonstrating that the joint settings satisfy the Leios protocol safety and liveness requirements | Mirrors LCS-07 |
| LQST-08 | (x) | any change must be confirmed by benchmarking and simulation | TSC notes |
| LQST-09 | (x - "should") | should not be changed more than once in any 18-epoch period (approximately 3 months) except in response to a Severity 1 or Severity 2 security issue | The safety-critical parameter of the voting layer |

#### `maxEndorserBlockReferencesSize` ($S_\text{EB}$), bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MEBRS-01 | (y) | must not be negative | Ratified pattern |
| MEBRS-02 | (y) | must not be lower than [PENDING] bytes | Lower bound; value to be agreed. Must be positive; the designated kill switch is `maxEndorserBlockTxsSize` = 0 (MEBTS-02) |
| MEBRS-03 | (y) | must not exceed 1,000,000 bytes | Higher than the CIP-0164 feasible value of 512 kB, to give more flexibility in parameterizing for anticipated transaction structure on mainnet (e.g. many small transactions require more references to saturate S_EB-tx); agreed by this document's earlier draft and the TSC notes |
| MEBRS-04 | (x - "should") | should be large enough to reference transactions filling `maxEndorserBlockTxsSize`, given the expected transaction sizes | CIP-0164 size parameters: S_EB exists because many small transactions create many references; at 1 MB of transactions the hashes alone can exceed 320 KB (TSC notes) |
| MEBRS-05 | (x) | any increase must be confirmed by benchmarking and simulation showing that Endorser Blocks can diffuse network-wide within the voting period (`leiosVotePeriodLength`) | Limits EB size to ensure timely diffusion (CIP-0164 Table 3) |
| MEBRS-06 | (x - "should") | should not be increased by more than [PENDING]% in any single epoch | Rate of change; no access to change history on chain |

#### `maxEndorserBlockTxsSize` ($S_\text{EB-tx}$), bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MEBTS-01 | (y) | must not be negative | Ratified pattern |
| MEBTS-02 | (x) | must not be lower than [PENDING] bytes, except when set to 0 to deactivate Linear Leios | **The usefulness floor**, to be set from simulation well above `maxTxSize` (MEBTS-08). **0 is the designated Linear Leios kill switch**: EBs then endorse no transactions and all transactions are carried in RBs. Cleaner than the alternatives: with `leiosVotePeriodLength` = 0 EBs would still be produced and diffused with no one able to vote on them, and a very large `leiosDiffusionPeriodLength` would let all the Leios work happen and then waste it. (x) because the script cannot express "0 or at least the floor" (it requires all predicates to hold); the script enforces the envelope 0 to 12,000,000 bytes (MEBTS-01, MEBTS-03), the Constitutional Committee the gap. The behaviour at 0 is to be confirmed with the Leios team |
| MEBTS-03 | (y) | must not exceed 12,000,000 bytes | Value range shown feasible in simulations (CIP-0164 feasible value 12 MB); agreed by this document's earlier draft and the TSC notes |
| MEBTS-04 | (~ - no access to existing parameter values) | must not be lower than `maxTxSize`, so that any valid transaction can be endorsed by an Endorser Block, except when set to 0 to deactivate Linear Leios | A correctness rule, not the operating floor: it keeps large transactions from being confined to RBs (TSC notes), and protects the case where `maxTxSize` is later raised. In normal operation MEBTS-02 sits far above it |
| MEBTS-05 | (x) | any change must be confirmed by benchmarking and simulation demonstrating (1) that nodes with the expected reference hardware configuration can validate all endorsed transactions within the Endorser Block reapplication time constraint, and (2) that the resulting network bandwidth requirements remain within the capabilities of the Stake Pool Operator community | R4 (reapplication) and diffusion within the stage length (CIP-0164 Table 3) |
| MEBTS-06 | (x) | does not affect individual transaction size limits; changes to this parameter must not be used as a substitute for adjusting `maxTxSize` | R9 |
| MEBTS-07 | (x - "should") | should not be increased by more than [PENDING]% in any single governance action | Rate of change; same form as ratified MBBS-05 |
| MEBTS-08 | (x - "should") | should be set well above `maxBlockBodySize`, so that Endorser Blocks add meaningful throughput over Blocks alone | States the usefulness intent of MEBTS-02 before a number is agreed |

#### `maxRefScriptSizePerEndorserBlock` ($S_\text{EB-ref}$), bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MRSEB-01 | (y) | must not be negative | Ratified pattern for size limits |
| MRSEB-02 | (y) | must not be zero | A zero limit would force every transaction that uses a reference script into RBs (TSC notes) |
| MRSEB-03 | (y) | must not exceed [PENDING] bytes | Upper bound; value to be agreed. CIP-0164 feasible value 12 MB |
| MRSEB-04 | (y) | must not be lower than [PENDING] bytes | Lower bound; value to be agreed |
| MRSEB-05 | (~ - no access to existing parameter values) | must not be lower than `maxRefScriptSizePerTx`, so that any valid transaction can be included in an Endorser Block | Counterpart of MRST-07. Otherwise large-reference-script transactions are confined to RBs and may be delayed under load (TSC notes, Tenet 1) |
| MRSEB-06 | (x) | any increase must be confirmed by benchmarking and simulation demonstrating that the reference-script validation cost of a maximally-filled Endorser Block allows Leios Committee members to validate it within `leiosVotePeriodLength` on the expected reference hardware | Voters must validate an EB before voting (CIP-0164, voting period); performance results are not available on chain |
| MRSEB-07 | (x - "should") | should not be changed by more than [PENDING]% in any [PENDING]-epoch period | Rate of change (TSC notes); no access to change history on chain |

#### `maxEndorserBlockExecutionUnits[memory]`, memory units

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MEBEU-M-01 | (y) | must not be negative | Ratified pattern |
| MEBEU-M-02 | (y) | must not be lower than [PENDING] units | Lower bound; value to be agreed. CIP-0164 feasible value 7,000M memory units |
| MEBEU-M-03 | (y) | must not exceed [PENDING] units | Upper bound; value to be agreed |
| MEBEU-M-04 | (~ - no access to existing parameter values) | must not be lower than `maxTxExecutionUnits[memory]`, so that any valid transaction can be endorsed by an Endorser Block | CIP-0164 deliberately keeps per-transaction limits unchanged and adds only per-EB limits (Table 3 note) (TSC notes). A correctness rule, not the operating floor |
| MEBEU-M-05 | (x) | any change must be confirmed by benchmarking and simulation on reference hardware demonstrating that an Endorser Block using the full limit can be validated by Leios Committee members within `leiosVotePeriodLength`, and reapplied within the Endorser Block reapplication constraint | Voters validate before voting (R2); R4 (reapplication). TSC notes |
| MEBEU-M-06 | (x - "should") | should be set consistent with `maxEndorserBlockExecutionUnits[steps]` so that the combined resource cost of a fully utilized Endorser Block remains within acceptable node resource constraints | The two components are one parameter at key 47 |
| MEBEU-M-07 | (x - "should") | should not be changed by more than [PENDING]% in any [PENDING]-epoch period | Rate of change (TSC notes) |
| MEBEU-M-08 | (x - "should") | should be set well above `maxBlockExecutionUnits[memory]`, so that Endorser Blocks add meaningful script capacity over Blocks alone | The usefulness intent behind MEBEU-M-02, as MEBTS-08 |

#### `maxEndorserBlockExecutionUnits[steps]`, step units

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MEBEU-S-01 | (y) | must not be negative | Ratified pattern |
| MEBEU-S-02 | (y) | must not be lower than [PENDING] units | Lower bound; value to be agreed. CIP-0164 feasible value 2,000G step units |
| MEBEU-S-03 | (y) | must not exceed [PENDING] units | Upper bound; value to be agreed |
| MEBEU-S-04 | (~ - no access to existing parameter values) | must not be lower than `maxTxExecutionUnits[steps]`, so that any valid transaction can be endorsed by an Endorser Block | CIP-0164 deliberately keeps per-transaction limits unchanged and adds only per-EB limits (Table 3 note) (TSC notes). A correctness rule, not the operating floor |
| MEBEU-S-05 | (x) | any change must be confirmed by benchmarking and simulation on reference hardware demonstrating that an Endorser Block using the full limit can be validated by Leios Committee members within `leiosVotePeriodLength`, and reapplied within the Endorser Block reapplication constraint | Voters validate before voting (R2); R4 (reapplication). TSC notes |
| MEBEU-S-06 | (x - "should") | should be set consistent with `maxEndorserBlockExecutionUnits[memory]` so that the combined resource cost of a fully utilized Endorser Block remains within acceptable node resource constraints | The two components are one parameter at key 47 |
| MEBEU-S-07 | (x - "should") | should not be changed by more than [PENDING]% in any [PENDING]-epoch period | Rate of change (TSC notes) |
| MEBEU-S-08 | (x - "should") | should be set well above `maxBlockExecutionUnits[steps]`, so that Endorser Blocks add meaningful script capacity over Blocks alone | The usefulness intent behind MEBEU-S-02, as MEBTS-08 |

#### `maxRefScriptSizePerBlock`, bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MRSB-01 | (y) | must not be negative | Ratified pattern for size limits |
| MRSB-02 | (y) | must not be zero | A zero limit would reject every transaction that uses a reference script |
| MRSB-03 | (y) | must not exceed [PENDING] bytes | Upper bound; value to be agreed |
| MRSB-04 | (y) | must not be lower than [PENDING] bytes | Lower bound; value to be agreed |
| MRSB-05 | (~ - no access to existing parameter values) | must not be lower than `maxRefScriptSizePerTx`, so that any valid transaction can be included in a Block | Otherwise large-reference-script transactions are confined to EBs and cannot be included at all when Leios certification is off, locking funds (TSC notes, Tenet 5). Class as the ratified MTS-04 |
| MRSB-06 | (x) | any increase must be confirmed by benchmarking and simulation demonstrating that the reference-script validation cost of a maximally-filled Block remains within the processing constraints of nodes on the expected reference hardware | Reference-script DoS protection (Section 7.1); performance results are not available on chain |
| MRSB-07 | (x - "should") | should not be changed by more than [PENDING]% in any [PENDING]-epoch period | Rate of change (TSC notes); no access to change history on chain |

#### `maxRefScriptSizePerTx`, bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MRST-01 | (y) | must not be negative | Ratified pattern for size limits |
| MRST-02 | (y) | must not be zero | A zero limit would reject every transaction that uses a reference script |
| MRST-03 | (y) | must not exceed [PENDING] bytes | Upper bound; value to be agreed |
| MRST-04 | (y) | must not be lower than [PENDING] bytes | Lower bound; value to be agreed |
| MRST-05 | (~ - no access to existing parameter values) | must not exceed `maxRefScriptSizePerBlock`, so that a transaction is never permitted more reference-script data than an entire Block | Counterpart of MRSB-05. Class as the ratified MTS-04 |
| MRST-06 | (x - "should") | any increase should be reviewed together with the reference-script tiered fee parameters (`minFeeRefScriptCostPerByte`, `refScriptCostStride`, `refScriptCostMultiplier`) so that denial-of-service protection is preserved | The fee curve and the caps act together (Section 7.1) |
| MRST-07 | (~ - no access to existing parameter values) | must not exceed `maxRefScriptSizePerEndorserBlock`, so that any valid transaction can be included in an Endorser Block | Otherwise large-reference-script transactions are confined to RBs (TSC notes). Counterpart of MRSEB's lower bound against this parameter |
| MRST-08 | (~ - no access to existing parameter values) | must not be decreased, so that funds locked by scripts that must be supplied as reference scripts remain spendable | TSC notes: "must" rather than "should", to avoid locking funds. Same form as the ratified MTS-03 for `maxTxSize`. Consequence: lowering the cap, for example during a reference-script DoS attack, needs a constitution change |
| MRST-09 | (x) | any increase must be confirmed by benchmarking and simulation demonstrating that the validation cost of a transaction using the maximum permitted reference-script size remains within the processing constraints of nodes on the expected reference hardware | Performance results are not available on chain |
| MRST-10 | (x - "should") | should not be changed by more than [PENDING]% in any [PENDING]-epoch period | Rate of change (TSC notes); no access to change history on chain |

#### `refScriptCostStride`, bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| RSCS-01 | (y) | must not be zero | Also rejected by the encoding (`positive_word32`); stated for the record |
| RSCS-02 | (y) | must not be negative | Also excluded by the type; ratified pattern (e.g. MTS-02) |
| RSCS-03 | (y) | must not exceed [PENDING] bytes | Upper bound; value to be agreed. Conway hardcoded value 25,600 bytes |
| RSCS-04 | (y) | must not be lower than [PENDING] bytes | Lower bound; value to be agreed |
| RSCS-05 | (~ - no access to existing parameter values) | should be lower than `maxRefScriptSizePerTx`, so that transactions using large reference scripts reach the higher pricing steps of the fee curve | A stride at or above the per-transaction cap leaves every transaction in the first pricing step, so the fee is flat per byte, the pre-ADR-9 situation (TSC notes). A sanity check, hence "should" |
| RSCS-06 | (x - "should") | should be set together with `refScriptCostMultiplier` and `minFeeRefScriptCostPerByte` so that the reference-script tiered fee continues to make large reference-script transactions progressively more expensive, preserving denial-of-service protection | The fee curve (Section 7.1) |
| RSCS-07 | (x) | any change must be confirmed by benchmarking and simulation demonstrating that the reference-script denial-of-service protection is preserved | Performance results are not available on chain (TSC notes) |

#### `refScriptCostMultiplier`, `positive_interval`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| RSCM-01 | (y) | must not be negative | Also excluded by the type (`positive_interval`); ratified pattern |
| RSCM-02 | (y) | must not be lower than 1.0, so that the per-byte reference-script price is non-decreasing across tiers and denial-of-service protection is not weakened | A multiplier below 1 would make large reference scripts cheaper per byte, a DoS vector (TSC notes). Exactly 1.0 is allowed deliberately: it flattens the curve to a single per-byte rate, which governance may want in some circumstances |
| RSCM-03 | (y) | must not exceed [PENDING] | Upper bound; value to be agreed. Conway hardcoded value 1.2; TSC notes suggest a sensible maximum around 2 |
| RSCM-04 | (x - "should") | should be set together with `refScriptCostStride` and `minFeeRefScriptCostPerByte` to preserve the intended denial-of-service protection of the reference-script tiered fee | The fee curve (Section 7.1) |
| RSCM-05 | (x) | any change must be confirmed by benchmarking and simulation demonstrating that the reference-script denial-of-service protection is preserved | Performance results are not available on chain (TSC notes) |

#### `maxPledgeLeverage` ($L$), `nonnegative_interval` or `nil`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MPL-01 | (y) | when set, must not be lower than 1 | CIP-0050 range 1 ≤ L ≤ 10,000. Essential: the type allows 0, and L = 0 makes every pool's reward-eligible stake zero, so no pool earns rewards (`SnapShots.hs` `maxPool'`) |
| MPL-02 | (y) | when set, must not exceed 10,000 | CIP-0050 range. The guardrails allow the full CIP range; choosing a value inside it is for governance (MPL-04) |
| MPL-03 | (y) | may be set to null (unset), which removes the leverage cap and restores the reward calculation used before the Dijkstra era; MPL-01 and MPL-02 apply only when it is set to a value | The ledger type is `nonnegative_interval` or `nil`; null is the state at the hard fork and may be restored. The guardrails script is being adapted to handle the nullable value |
| MPL-04 | (x - "should") | should be set with regard to protection against Sybil attack, the economic viability of operating a pool, and the prevailing `poolPledgeInfluence` | Both parameters make pledge matter (a0 as a reward bonus, L as a cap); the TSC notes flag their interaction. Guidance rather than hard-coded joint thresholds |
| MPL-05 | (x - "should") | changes should be announced at least [PENDING] epochs in advance, so that pool operators can adjust their pledge | TSC notes (advance warning). Not on the Appendix I 2.1 critical list, so PARAM-04a's 90-day notice does not apply |

#### `minPoolMargin`, `unit_interval`, conditional on Section 8

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MPM-01 | (y) | must not be negative | Also excluded by the type (`unit_interval`); ratified pattern |
| MPM-02 | (y) | must not exceed 1.0 (100%) | Also excluded by the type; stated for the record, as ratified TC-04 does for `treasuryCut` |
| MPM-03 | (y) | must not exceed [PENDING] | Upper bound; value to be agreed |
| MPM-04 | (x - "should") | should be set with regard to both delegator rewards and the economic viability of operating a pool | CIP-0023's purpose (Section 8) |
| MPM-05 | (~ - no access to existing parameter values) | `minPoolMargin` and `minPoolCost` should not both be zero, so that pool fees always have a floor | Both at zero removes every fee floor (TSC notes). Placed here only, leaving the ratified `minPoolCost` guardrails untouched; `minPoolCost` may already be zero under ratified MPC-01 to MPC-03 |

#### `minFeeRefScriptCostPerByte`, `nonnegative_interval`, existing parameter

Ratified MFRS-01 to MFRS-04 stand, with the parameter name corrected from `minFeeRefScriptCoinsPerByte` (Section 7.2). One guardrail is added:

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MFRS-05 | (y) | must not be zero, since a zero base rate makes the whole reference-script tiered fee zero and removes its denial-of-service protection | Every tier is priced as a multiple of this rate (Section 7.1), so zero switches the protection off regardless of the stride and multiplier (TSC notes) |

#### `minPoolCost`, lovelace, existing parameter

Ratified MPC-01 and MPC-02 stand. MPC-03 is deprecated. Under the ratified rule, a deprecated guardrail's label is never reused and an amended entry takes a new label, so the place of MPC-03 is marked "MPC-03a: Deprecated". This starts a convention for guardrails deprecated without replacement: the ratified text previously had only amendments, where the original disappears and the "a" version holds the new text.

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MPC-03a | none | Deprecated (replaces MPC-03: ~~`minPoolCost` should be set in line with the economic cost for operating a pool~~) | Removed. With `minPoolMargin` introduced (CIP-0023), the intended end state is `minPoolCost` = 0, with the margin as the fee floor; MPC-03 would stand in the way of that. MPM-05 keeps a floor: `minPoolMargin` and `minPoolCost` should not both be zero. MPC-01 already allows 0 |

#### `poolRetireMaxEpoch`, epochs, existing parameter

Ratified PRME-01 stands. PRME-02 is amended, taking the ratified "a" suffix for an amended guardrail:

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PRME-02a | (y) | must not be lower than 1 | Replaces ratified PRME-02 (x - "should") "should not be lower than 1". From Dijkstra the ledger rejects a Parameter Update that sets it to 0 (`ppuWellFormed`, `eras/dijkstra/impl/src/Cardano/Ledger/Dijkstra/PParams.hs:1166`, commit `2602d861bd`), because a pool may only retire in an epoch e with current < e ≤ current + `poolRetireMaxEpoch` (`Shelley/Rules/Pool.hs:309-312`), so 0 would prevent every pool from retiring (TSC notes) |

#### Peras

##### `perasMinCandidateBlockAge`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PMCBA-01 | (y), (x) or (~ - reason) | perasMinCandidateBlockAge must not be lower than 30 slots | CIP-140 |
| PMCBA-02 | (y), (x) or (~ - reason) | perasMinCandidateBlockAge must not be larger than 30 slots | CIP-140 |

Parameter is fixed to the value proposed in CIP-140.

##### `perasCertBoost`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PCB-01 | (y), (x) or (~ - reason) | perasCertBoost must not be lower than 15 | CIP-140 |
| PCB-02 | (y), (x) or (~ - reason) | perasCertBoost must not be larger than 15 | CIP-140 |

Parameter is fixed to the value proposed in CIP-140.

##### `perasTargetCommitteeSize`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PTCS-01 | (y), (x) or (~ - reason) | perasTargetCommitteeSize must not be lower than 500 | CIP-140 |
| PTCS-02 | (y), (x) or (~ - reason) | perasTargetCommitteeSize must not be larger than 1000 | CIP-140 |

##### `perasHealingFactor`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PHF-01 | (y), (x) or (~ - reason) | perasHealingFactor must not be lower than 1 | CIP-140 |
| PHF-02 | (y), (x) or (~ - reason) | perasHealingFactor must not be larger than 3 | CIP-140 |

##### `perasQuorumThresholdSafetyMargin`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PQTSM-01 | (y), (x) or (~ - reason) | `perasQuorumThresholdSafetyMargin` must be lower than 0.25 | Total quorum size should not extend 1 |

##### Era length

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PE-01 | (y), (x) or (~ - reason) | epoch length must be divisible by the Peras round length ($U=90$) | CIP-140 |

### 11.4 Terms the guardrails rely on

The guardrails above use terms that the constitution update defines in Appendix I's terminology, so that each guardrail has a precise meaning:

| Term | Defined as (summary; the constitution text is authoritative) | Source |
| :---- | :---- | :---- |
| Block | Extended: under Leios also called a Ranking Block (RB); "Block" means an RB unless stated otherwise | CIP-0164 |
| Endorser Block (EB) | A supplementary block, announced by an RB, carrying references to additional transactions; its transactions become part of the chain only if it is certified and the certificate is included in a later RB | CIP-0164 |
| Leios Committee | The Stake Pools eligible to vote on EBs during an epoch: the top Stake Pools by active stake, up to `leiosCommitteeSize`, determined once per epoch | CIP-0164 Table 3 (committee size) |
| Leios Certificate | An aggregated proof of Leios Committee votes attesting that votes representing at least `leiosQuorumStakeThreshold` of total active stake were cast for the same EB | CIP-0164 Table 3 (quorum stake threshold) |

The measured terms used by LLS-01 to LLS-03 (Endorser Block diffusion time, certified Endorser Block transmission time, Block diffusion time, transaction validation time) are defined in the constitution's Leios Timing Parameters section rather than in the terminology.

---

## 12. Initial settings

**To be filled in.** Values proposed so far are recorded here until agreed.

### 12.1 Peras, proposed by the Peras team

The ledger defines no defaults; these values would be set in Dijkstra genesis. They take effect only once Peras is activated at the intra-era PV13.

| Ledger name | Proposed initial value |
| :---- | ----: |
| `perasMinCandidateBlockAge` | 30 slots |
| `perasHealingFactor` | 2 |
| `perasCertBoost` | 15 |
| `perasTargetCommitteeSize` | 900 |
| `perasBootstrapRound` | unset (`nil`) |
| `perasQuorumThresholdSafetyMargin` | 0.1 |

---

## 13. Open issues, prioritized

Ordered by what they block. Each names the decision, who owns it, and where it should be settled. Downstream drafting work, such as transcribing the agreed changes into the constitution itself, is out of scope here and tracked separately.

**1. No specification states the governance grouping of any parameter.** CIP-0164, CIP-0023, CIP-0050 and CIP-0140 are all silent on it, and the placement comes from the ledger alone. The CIPs should carry it, as the reference other implementers build against. Separately, an open question for the Parameters Committee: is the grouping as implemented the one they want? Owner: the CIP authors for the omission, the Parameters Committee for the grouping itself. Venue: CIPs.

**2. CIP-0164 states the timing parameters in seconds; on chain they are milliseconds.** Routine alignment, blocking nothing here, since this document writes milliseconds throughout and the ledger types all three `Milliseconds32`. The CIP's Units column will be updated to give the unit the value is carried in. CIPs#1250 merged on 2026-09-15 with the units still in seconds, so this needs a follow-up CIP-0164 PR. Owner: CIP-0164 authors. Venue: CIPs.

**3. `minPoolMargin`.** Introduced in the Dijkstra era, inactive until the intra-era hard fork to PV13. Should it be introduced in the Constitution at this time (Section 8)? Owner: Parameters Committee. Venue: this document's review.

**4. The Peras parameters.** Merged (ledger PR #6067) and introduced in the Dijkstra era, inactive until the intra-era hard fork to PV13. Names, keys and groups are settled, so the decision no longer waits on the implementation. Should they be introduced in the Constitution at this time (Section 10)? Owner: Tweag by Modus Create as Peras owner. Venue: cardano-node#6634.

### Closed

---
