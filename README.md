# Dijkstra protocol parameters: specification for the constitution update, guardrails and initial settings

**Status:** Draft v1.1
**Authors:** Carlos Lopez de Lara (IOG), Aleksandr Vershilov (Tweag)
**Date:** 2026-09-08
**Supersedes:** [dijkstra-protocol-parameters-for-constitution-update](https://docs.google.com/document/d/1M649pDQtquYr4n5QBx_Q7vfzyFINQssN8Cpp553BGvA/edit?usp=sharing) (v0.2, 2026-09-01)
**Audience:** Parameters Committee, HFWG, Civics Committee, IOG, Tweag

This document specifies every protocol parameter that the Dijkstra hard fork (PV12) introduces as updatable, in the form the ledger will accept it, and frames the guardrails and initial values still to be agreed for each. It is written to serve three deliverables from one source:

1. the **Constitution Update and Guardrail Appendix** governance action that must name each new parameter,
2. the **guardrails script** that ships with that action, and
3. the **initial settings** carried in Dijkstra genesis or set by the first parameter update after the hard fork.

Section 5 says how to read the parameter tables. Sections 6 to 10 are descriptive: they record what the upstream specifications define, and nothing more. Sections 11 and 12 are open, awaiting the guardrails and initial values the Parameters Committee agrees with the technical teams. Section 13 tracks the decisions still outstanding.

---

## 1. Change log

| Version | Date       | Change                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|:------- |:---------- |:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| v1.1    | 2026-09-10 | Changes type of the `ppPerasHealingFactor` parameter to reflect concrete bounded CDDL type                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| v1.0    | 2026-09-03 | **Replaces  [dijkstra-protocol-parameters-for-constitution-update](https://docs.google.com/document/d/1M649pDQtquYr4n5QBx_Q7vfzyFINQssN8Cpp553BGvA/edit?usp=sharing) and changes the document's purpose from working note to specification.** The previous document paraphrased the scope and sent the reader to the CIPs for the exact wording. This one keeps the CIPs authoritative and stops paraphrasing: it reproduces them verbatim at a pinned revision, refreshed whenever upstream changes, so the text here can be relied on and diffed against the source, with anything it notices raised on the CIP rather than corrected locally. |
| v1.2    | 2026-09-21 | Added `ppPerasQuorumThresholdSafetyMargin` parameter required for Peras security                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

---

## 2. Authority: this document is downstream of the specifications

Every parameter in this document has an upstream specification: CIP-0164 for the nine Leios parameters, CIP-0023 for `minPoolMargin`, CIP-0050 for `maxPledgeLeverage`, and CIP-0140 for Peras.

**The four reference-script parameters have no CIP.** Their upstream is ledger [ADR-9](https://github.com/IntersectMBO/cardano-ledger/blob/master/docs/adr/2024-08-14_009-refscripts-fee-change.md) (accepted, 2024-08-14), which plays the role a CIP plays elsewhere.

Where this document and its upstream disagree, **the upstream wins**, and this document is what gets corrected. Anything raised here about a parameter's definition, semantics or symbol is upstreamed rather than settled here. Alternative implementations hold delivery milestones against specification conformance, so a bound that encodes a reading its upstream does not state would break that equivalence.

The parameter surface described here is fixed at the era boundary and cannot be extended afterwards without another era-to-era hard fork, because the era's `PParams` shape and CDDL are frozen when Dijkstra begins.

---

## 3. Why a constitution update before the hard fork

A parameter introduced at a hard fork holds its genesis value until the constitution names it and defines guardrails for it, introducing parameters in the Constitution ahead of the hardfork paves the way for the rollout. It is not a hard dependency, but it's desireable.

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
| Peras (CIP-0140) | 5 (activation at intra-era PV13) | `NOT STARTED` (issue #5966) | Open, could defer to PV13 (Section 10) |
| Plutus V4 | Cost model entry only, no new parameter | `MERGED` | No |

**Dijkstra introduces 20 new parameters in the code.** 15 are there today at parameter-update keys 34 to 48; the 7 Peras ones need a ledger PR to add them and assign their keys, and their list is not settled (Section 10). All 22 have to be in the code at PV12, whether or not they do anything yet, because the era's parameter set is fixed at the era boundary.

**14 of the 20 need naming in the constitution ahead of the hard fork**: the 9 Leios, the 4 reference-script and `maxPledgeLeverage`. Those are live at PV12 and unusable until the constitution names them and defines guardrails for them.

**The other 6 could be deferred to a later constitution update**: `minPoolMargin` (Section 8) and the 5 Peras parameters (Section 10). Both activate at the intra-era PV13. Deferring their inclussion in the Constiuttion is an option, these parameters are inactive on PV12 so it could be beneficial to take the v12 to v13 window to analyse bounds, including the first chance to see Peras and Leios running together. **Both are still decisions to be made**.

CIPs PR #1213 would add one or two more to every figure above if it merges (Section 6.2), may also come with PV13.

---

## 5. How to read the parameter tables

Sections 6 to 9 use the same columns.

**Two name columns.** The specification's name for the parameter, headed by whichever specification it comes from, and the ledger name. Appendix I must use the ledger name (Section 3), so that is what guardrails attach to.

**Update key.** The integer a "Parameter Update" action carries; the action holds no name at all. 15 of Dijkstra's new parameters have keys today, 34 to 48. The Peras parameters have none assigned yet.

**Groups.** The governance groups the parameter belongs to.

**Status.** `MERGED` = on cardano-ledger `master`. `OPEN PR` = implemented but not merged. `NOT STARTED` = no implementation.

**Unit on chain.** The unit the value carries in a governance action, which is not always the unit its specification uses. Differences are called out on the row. Dimensionless parameters have no unit, so the column gives the CDDL type instead: `unit_interval` is 0 to 1, `nonnegative_interval` has no upper bound, and both are carried as exact rationals rather than decimals. Bounds on them are conventionally written as decimals, as the ratified text does in TC-01, "*treasuryCut* must not be lower than 0.1 (10%)".

Section 10 (Peras) has it's own layout, not too different that what's use on the rest.

---

## 6. CIP-0164, Leios

**Source.** CIP-0164 as it stands with CIP PR [CIPs#1250](https://github.com/cardano-foundation/CIPs/pull/1250), scheduled to be merged before September 11 2026.

**Implementation status: `MERGED`.** All nine are on cardano-ledger `master` via PR [#6002](https://github.com/IntersectMBO/cardano-ledger/pull/6002), merged 2026-09-03. That PR implements CIP-0164 as corrected by #1250 and maps each parameter to the CIP's symbol, so the set and its semantics are the CIP's; the identifiers are the ledger's. The names in Section 6.3 are current as of the 2026-09-02 renames.

**Governance grouping.** All nine are **NetworkGroup** and **SecurityGroup**, so an update touching any of them needs SPO approval as well as DRep approval at the `dvtPPNetworkGroup` threshold. The placement comes from the ledger; CIP-0164 does not state it (open issue 1). The constitution carries its own group lists that this update has to extend, the critical-parameter list in Appendix I section 2.1 and the group lists in Appendix I section 9.

### 6.1 CIP-0164 Table 3, verbatim

A verbatim copy of Table 3 at PR #1250 head `e173ea52`, row order included, so the two can be diffed mechanically. Cell padding is collapsed for legibility; every cell's content is the CIP's. 

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

| CIP-0164 name | Symbol | Ledger name | Update key | Unit on chain | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| Committee size | $N_c$ | `leiosCommitteeSize` | 43 | seats | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Quorum stake threshold | $\tau$ | `leiosQuorumStakeThreshold` | 44 | `unit_interval` | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Header diffusion period length | $L_\text{hdr}$ | `leiosAnnouncementPeriodLength` | 40 | **milliseconds** | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Voting period length | $L_\text{vote}$ | `leiosVotePeriodLength` | 41 | **milliseconds** | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Diffusion period length | $L_\text{diff}$ | `leiosDiffusionPeriodLength` | 42 | **milliseconds** | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum endorser block size | $S_\text{EB}$ | `maxEndorserBlockReferencesSize` | 45 | bytes | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum total transaction size per endorser block | $S_\text{EB-tx}$ | `maxEndorserBlockTxsSize` | 46 | bytes | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum reference script size per endorser block | $S_\text{EB-ref}$ | `maxRefScriptSizePerEndorserBlock` | 48 | bytes | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum Plutus memory per endorser block | none | `maxEndorserBlockExecutionUnits[memory]` | 47 | memory units | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| Maximum Plutus steps per endorser block | none | `maxEndorserBlockExecutionUnits[steps]` | 47 | step units | **NetworkGroup**, **SecurityGroup** | `MERGED` |

The `[memory]` and `[steps]` notation follows the constitution's treatment of `maxBlockExecutionUnits`, guardrailed separately per index. On chain the two are one value at key 47, a pair ordered **memory first** (`ex_units = [mem, steps]`); these two rows are the only place this table departs from Table 3's order. An update to that key carries both components.

**Units.** CIP-0164 gives the three timing parameters in seconds; The implementation uses milliseconds, so 1 s, 4 s and 7 s are 1000 ms, 4,000ms and 7,000ms. **This document writes every timing value in milliseconds.** Aligning the CIP's Units column is open issue 2.

### 6.4 Relations: the constraints these parameters must jointly satisfy

Leios security constrains the timing parameters jointly, against each other and against network characteristics that are measured rather than governed. Tables 1 and 2 are those characteristics, verbatim.

**Line links point at [CIPs PR #1250](https://github.com/cardano-foundation/CIPs/pull/1250), head `e173ea52`, not at `master`.** 

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

**R1. Equivocation detection window.** [L572-L578](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L572-L578). Voting starts $3 \times L_\text{hdr}$ after an RB announces an EB, one $L_\text{hdr}$ for each step of the CIP's Detection Mechanism list: "Initial header propagation", "Conflicting header propagation" and "Equivocation evidence propagation". Honest nodes then refuse to vote for any EB from an RB slot where equivocation was detected ([L585-L588](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L585-L588)), and EB diffusion continues during this window ([L593](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L593)).

**R2. Voting period.** [L610-L612](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L610-L612). Verbatim: "The voting period must accommodate EB diffusion (transmission and processing):"

$$3 \times L_\text{hdr} + L_\text{vote} > \Delta_\text{EB}^{\text{O}}$$

**R3. Diffusion period.** [L637-L639](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L637-L639). Verbatim: "The diffusion period must satisfy:"

$$L_\text{diff} \geq \Delta_\text{EB}^{\text{W}} + \Delta_\text{reapply} - \Delta_\text{RB} - 3 \times L_\text{hdr} - L_\text{vote}$$

**R4. EB reapplication constraint.** [L685-L687](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L685-L687). Verbatim: "Reapplying a certified EB cannot cost more than standard transaction processing."

$$\Delta_\text{reapply} < \Delta_\text{applyTxs}$$

**R5. Certified EB transmission constraint.** [L693-L696](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L693-L696). Verbatim: "Any certified EB referenced by an RB must be transmitted (but not necessarily be processed) before that RB needs to be processed."

$$\Delta_\text{EB}^{\text{W}} < 3 \times L_\text{hdr} + L_\text{vote} + L_\text{diff} + (\Delta_\text{RB} - \Delta_\text{applyTxs})$$

**R6. Quorum threshold** [L500](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L500) and [L2556-L2565](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L2556-L2565). From below, $\tau$ must exceed the adversarial stake Praos tolerates by a sizable margin, because $\tau - \sigma_a$ is the honest stake provably holding every certified EB when voting ends, and that remainder is the coverage $\Delta_\text{EB}^{\text{W}}$ is measured from. Verbatim: "Below $\tau = 0.5$ the bound fails outright, since two disjoint sets of voters could each reach a quorum and certify conflicting EBs; a threshold in the 50s clears that only nominally and leaves nothing for the assumption above." From above:

$$0.5 < \tau < \sigma(N_c)$$

where $\sigma(N_c)$ is the cumulative active stake of the top $N_c$ pools, read off the prevailing stake distribution. $\sigma(N_c) - \tau$ is the abstention budget: the seated stake that can decline to vote before no quorum can form.

**R7. Certificate inclusion delay in slots.** [L404-L419](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L404-L419). The total delay $3 \times L_\text{hdr} + L_\text{vote} + L_\text{diff}$ is a wall-clock duration, and chain inclusion needs a whole number of slots, so it is divided by `slotLength` and **rounded up**. At mainnet's `slotLength` of one second the division is exact for the CIP's feasible values.

**R8. Committee size against the stake distribution.** [L2622-L2628](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L2622-L2628). $\sigma(N_c)$ must exceed $\tau$ or the abstention budget is negative and no quorum can form. Verbatim: "This is a hard floor rather than an operating point: at exactly $\tau$ every seated pool must vote and be reachable within $L_\text{vote}$, so a practical choice leaves budget for pools that are offline, [keyless](#key-registration), or slow."

**R9. Individual transaction size is unchanged.** [L657-L662](https://github.com/cardano-foundation/CIPs/blob/e173ea5250e99e93db3436205f32f755169f1866/CIP-0164/README.md?plain=1#L657-L662). Verbatim: "Note that $S_\text{EB-tx}$ does not change the maximum size of individual transactions. The existing `maxTxSize` parameter remains unchanged and continues to limit individual transaction sizes."

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

| ADR-9 name | Ledger name | Update key | Conway hardcoded value | What it bounds | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| "Limit per block" | `maxRefScriptSizePerBlock` | 34 | 1,048,576 bytes (1 MiB) | Total bytes of all reference scripts used by all transactions in a block | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| "Limit per transaction" | `maxRefScriptSizePerTx` | 35 | 204,800 bytes (200 KiB) | Total bytes of reference scripts a single transaction may use | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| "Size increment" (`sizeIncrement`) | `refScriptCostStride` | 36 | 25,600 bytes (25 KiB) | Size increment within which the per-byte price stays constant before the next escalation step | **NetworkGroup**, **SecurityGroup** | `MERGED` |
| "Multiplier" (`multiplier`) | `refScriptCostMultiplier` | 37 | 1.2 | Growth factor applied to the per-byte price at each stride boundary | **NetworkGroup**, **SecurityGroup** | `MERGED` |

ADR-9 names these four descriptively rather than as identifiers, so the first column quotes its labels, with `sizeIncrement` and `multiplier` being the names its code snippet uses. The values come from the same place, verbatim: "Limit per transaction: `200KiB` (or `204800` bytes)", "Limit per block: `1MiB` (or `1048576` bytes)", "Size increment: `25KiB` (or 25,600 bytes)" and "Multiplier: `1.2`". Zero is rejected by the encoding for the stride and the multiplier, before any guardrail is consulted.

**`minFeeRefScriptCostPerByte` is already governed.** A Conway parameter, key 33, **EconomicGroup** and **SecurityGroup**, carrying MFRS-01 (must not exceed 1,000) and MFRS-02 (must not be negative). **The ratified constitution spells it `minFeeRefScriptCoinsPerByte`, in all seven places it appears, and the parameter-update name is `minFeeRefScriptCostPerByte`.** By the constitution's own rule in Section 3 the parameter-update name governs.

---

## 8. CIP-0023, Min Pool Margin `minPoolMargin`, also in constitution update?

| CIP-0023 name | Ledger name | Update key | Unit on chain | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- |
| `minPoolMargin` (identical) | `minPoolMargin` | 39 | `unit_interval` | **EconomicGroup** | `MERGED`, logic not started |

**Definition** (CIP-0023, verbatim): "`minPoolMargin` defines the lower bound for the pool margin (variable fee), i.e., the minimum allowable percentage of rewards a pool can take. Pool registration and update certificates MUST have `margin >= minPoolMargin`." Bounding the `margin` field of `pool_params`, itself a `unit_interval`, fixes the parameter's domain at 0 to 1 without CIP-0023 having to state it. It complements `minPoolCost`, the fixed-fee floor, which is already governed by MPC-01 to MPC-03.

**Mechanics.** The backward-compatible route CIP-0023 itself recommends is the one being taken, rather than the MUST-reject rule. Verbatim from CIP-0023: "if a pool's margin is less than `minPoolMargin`, the protocol-level `minPoolMargin` overrides the pool's registered `margin` during reward calculation. This minimizes disruption and lets legacy pool certificates remain valid". Pool registrations and updates with a lower margin are therefore not rejected, and existing certificates stay valid. The same clamping is planned for the pool `cost` field against `minPoolCost`.

**Status.** The parameter is merged, and nothing reads it: `ppMinPoolMarginL` appears in the ledger only in the parameter definition, the changelog and a genesis test. The reward-calculation logi two-sided.c is **not started**, ledger issue [#5954](https://github.com/IntersectMBO/cardano-ledger/issues/5954), and cardano-node#6634 puts activation at the intra-era PV13.

**Could be deferred, on the same footing as Peras (Section 10).** The parameter is in the code at PV12 and holds key 39, but nothing reads it until PV13, so naming it now would make governable a value that has no effect, and would mean agreeing guardrails before the behaviour they bound exists. Deferring costs nothing meanwhile, since PARAM-01 holds it at its genesis value of 0. Still a decision to be made.

---

## 9. CIP-0050, Pledge Leverage `maxPledgeLeverage`

| CIP-0050 name | Ledger name | Update key | Unit on chain | Groups | Status |
| :---- | :---- | :---- | :---- | :---- | :---- |
| "Maximum pledge leverage", $L$ | `maxPledgeLeverage` | 38 | `nonnegative_interval` or `nil` | **TechnicalGroup** | `MERGED` |

**This is the only nullable parameter in the set.** The value may be a non-negative ratio or `null`, and `null` is how it is introduced at the hard fork, which is equivalent to how every era before Dijkstra behaves. No parameter currently named in the constitution is nullable, so the guardrails script has no precedent.

**Definition.** A cap on pool leverage, the ratio of a pool's total stake to its pledge, applied inside the rewards calculation.

**Mechanics.** Eligible stake in the rewards formula becomes the minimum of the pool's stake, the saturation point z0 = 1/k, and L times its pledge. Stake above L times pledge earns nothing, for the operator or its delegators. With the parameter set, a zero-pledge pool earns zero rewards on delegated stake. Pools whose pledge supports their stake are unaffected and keep the a0 boost.

**Range** (CIP-0050, verbatim): "**Range of *L*:** 1 ≤ *L* ≤ 10,000 (dimensionless ratio). An *L* of 10,000 represents an extremely high allowed leverage (i.e. pledge need only be 0.01% of the stake), effectively similar to the status quo with a very weak pledge influence. An *L* of 1 represents a very strict requirement where a pool's stake cannot exceed its pledge (100% pledge) if it is to earn full rewards." Its simulations swept L over {10, 100, 1000, 10,000} and concluded, verbatim: "The sweet spot lies at the lower end of the tested range (~10 to ~100), where Sybil protection is strongest yet the number of viable pools remains healthy."

**Status.** Merged, with the reward-calculation semantics active. Introduced unset, so per ledger PR #5943 "for the feature to really become active the first real value must be set via a governance action". The constitution entry is therefore a precondition for the feature doing anything at all.

---

## 10. Peras Parameters at PV12, also in constitution update?


Note on the stability of the parameters:

> [!NOTE]
>
> Upstream is [CIP-0140](https://github.com/cardano-foundation/CIPs/tree/master/CIP-0140), that defined `Params` record defines seven protocol parameters: $U$ round length, $L$ block-selection offset, $A$ certificate expiration, $R$ chain-ignorance period, $K$ cool-down period, $B$ certification boost and $\tau$ quorum. Committee size $n$ appears only in its feasible-values table. Some of those parameters are dependent and some are not governable.
>
> **The parameter list is final but not yet stabilized in all sources.**
> 
> Sources are:
> 
> - [Peras decision log](https://github.com/tweag/cardano-peras/blob/main/doc/adr/0003-protocol-parameters.md) that discusses each parameter and if it should be a gouvernable or not;
> - [cardano-peras#283](https://github.com/tweag/cardano-peras/issues/283) issue that tracks updating and tracking parameters in all places;
> - [cardano-tracker#5966](https://github.com/IntersectMBO/cardano-ledger/issues/5966) ledger issue that is tracking the addition on parameters;
> - This document;
> - [CIP-140](https://cips.cardano.org/cip/CIP-0140);
>
> At this point peras decision log, peras tracker and ledger tracker are in sync and CIP update is a work in progress.
> 
> Nothing has landed in the ledger, so none of these has a key or a settled name, and a PR is needed before the CDDL freezes. Whichever set is agreed, these are consensus parameters of the same kind as the Leios nine, so the placement to expect is **TechnicalGroup** for the DRep threshold and **SecurityGroup** so that changing any of them also requires an SPO vote.

| Parameter | Symbol | Units | Description | Default |
| :---- | :---- | :---- | :---- | ----: |
| `ppPerasMinCandidateBlockAge` | $L$ | `SlotInterval` | The minimum age of a candidate block for being voted upon. | 30 |
| `ppPerasCertBoost` | $B$ | `Word16` | The extra chain weight that a certificate gives to a block. | 15 |
| `ppPerasTargetCommitteeSize` | $n$ | `Word16` | The number of members on the voting committee. | 900 |
| `ppPerasBootstrapRound` | $R_\text{bootstrap}$ | StrictMaybe Word64 | Peras round number used to manually bootstrap Peras voting for the first time and to resynchronize voting after unexpected failures. | Nothing |
| `ppPerasHealingFactor` | `h` | `PositiveInterval` | coefficient in the $T_\text{heal}$ formula | 2 |
| `ppPerasQuorumThresholdSafetyMargin` | $\tau_\mathsf{margin}$ | `PositiveInterval` | extra margin on the top of the required quorum number | 0.1 | 

> [!NOTE]
> 
> Committee size $n$ is a governed quantity in CIP-0140's feasible-values table but not a field of its `Params` record, and no implementation list carries it. There are some discussions how to track committee size and a committee selection algorithm with few various approaches, that affect the certificate size. $n$ parameter is required only in one of those, but so far Peras team believes that including this parameter is the cleanest and safest approach, no matter what decision will take an effect.

Other parameters that are mentioned in CIP could not be included as they are either non-changable without implementation modifications or have only one reasonable value.

`ppPerasBootstrapRound` was not concidered in CIP-140 because CIP did not cover the Peras enablement. The Peras voting rules are defined in a way that if there were no Peras votes on the chain Peras could not start voting. There are two ways how to bootstrap Peras, either to modify voting rules and introduce special cases that would lead to additional research, or introduce a special parameter that would allow to unblock Peras. Peras team decided that such parameter is better solution because it largely simplify codebases (incl. alternative nodes), rules and introduces a way to unblock Peras in case of an unforceen situation.

`ppPerasQuorumThresholdSafetyMargin` is required to mitigate potential security due to possible transfer of the votes to the malicious nodes.

> [!NOTE]
>
> As per CIP-140, when peras will be enabled we will have to change security parameter $k$ to keep the same safety properties. However in this update we do not propose to change those as Peras in not planned to be enabled in the Hard Fork.

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

The ratified Appendix I uses a parameter initialism plus a sequence number (MBBS for `maxBlockBodySize`, MFRS for `minFeeRefScriptCoinsPerByte`, MPC for `minPoolCost`, PPI for `poolPledgeInfluence`), with an extra letter for indexed parameters (MBEU-S, MBEU-M). Following that convention we propose:

| Ledger name | Prefix |
| :---- | :---- |
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
| `ppPerasMinCandidateBlockAge` | PMCBA |
| `ppPerasCertBoost` | PCB |
| `ppPerasTargetCommitteeSize` | PTCS |
| `ppPerasHealingFactor` | PHF |
| `ppPerasQuorumThresholdSafetyMargin` | PQTSM |
| Era length | PE |


**Pending Peras Parameters.**

### 11.3 Guardrails, per parameter WIP

One block per parameter, to be filled in. Each carries a placeholder row showing the shape: replace it, and number upwards from `-01` as the ratified text does (MBBS-01, MBBS-02 and so on).

#### `leiosAnnouncementPeriodLength` ($L_\text{hdr}$), milliseconds WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LAPL-01 | (y) | Must be between 0 and 5000 milliseconds | 0ms allows to turn-off announcements, 5000ms is the delta assumption of Praos. Meaningful values are around ~1000ms |
| LAPL-02 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |
#### `leiosVotePeriodLength` ($L_\text{vote}$), milliseconds WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LVPL-01 | (y) | must be between 0ms and 20000ms | 0ms allows to turn off voting; a reasonable maximum would be 20000ms, which is the expected block time at the current activeSlotCoefficient = 0.05, the whole pipeline must be shorter than the expected value of block time |
| LVPL-02 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |
#### `leiosDiffusionPeriodLength` ($L_\text{diff}$), milliseconds WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LDPL-01 | (y)| must not be greater than 2^32-1ms | 2^32-1ms turns certification off |
| LDPL-02 | (x)| must be greater than [PENDING]| Depends on sizes and network topology |
| LDPL-03 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `leiosCommitteeSize` ($N_c$), seats WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LCS-01 | (y) |  be a pool count that together control [eg 95%?] of the active stake | Current meaningful value 900-1000 |
| LCS-02 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `leiosQuorumStakeThreshold` ($\tau$), `unit_interval` WIP

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| LQST-01 | (y) | must be greater than 1/2 and smaller or equal to 1 | source and reasoning |
| LQST-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `maxEndorserBlockReferencesSize` ($S_\text{EB}$), bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MEBRS-01 | (y) | Must be between 0 and 1000000 (1MB); higher than what is used in the CIP to give more flexibility in parameterizing for anticipated transaction structure on mainnet (e.g. many small txs require more references to saturate the S_EB-tx parameter) | source and reasoning |

#### `maxEndorserBlockTxsSize` ($S_\text{EB-tx}$), bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MEBTS-01 | (y) | Must be between 0 and 12000000 (12MB); value range shown feasible in simulations | source and reasoning |

#### `maxRefScriptSizePerEndorserBlock` ($S_\text{EB-ref}$), bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MRSEB-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `maxEndorserBlockExecutionUnits[memory]`, memory units

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MEBEU-M-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `maxEndorserBlockExecutionUnits[steps]`, step units

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MEBEU-S-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `maxRefScriptSizePerBlock`, bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MRSB-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `maxRefScriptSizePerTx`, bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MRST-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `refScriptCostStride`, bytes

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| RSCS-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `refScriptCostMultiplier`, `positive_interval`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| RSCM-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `maxPledgeLeverage` ($L$), `nonnegative_interval`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MPL-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### `minPoolMargin`, `unit_interval`, conditional on Section 8

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| MPM-01 | (y), (x) or (~ - reason) | must / must not / should / should not … | source and reasoning |

#### Peras

##### `ppPerasMinCandidateBlockAge`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PMCBA-01 | (y), (x) or (~ - reason) | ppPerasMinCandidateBlockAge must not be lower than 30 slots | CIP-140 |
| PMCBA-02 | (y), (x) or (~ - reason) | ppPerasMinCandidateBlockAge must not be larger than 30 slots | CIP-140 |

Parameter is fixed to the value proposed in CIP-140.

##### `ppPerasCertBoost`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PCB-01 | (y), (x) or (~ - reason) | ppPerasCertBoost must not be lower than 15 | CIP-140 |
| PCB-02 | (y), (x) or (~ - reason) | ppPerasCertBoost must not be larger than 15 | CIP-140 |

Parameter is fixed to the value proposed in CIP-140.

##### `ppPerasTargetCommitteeSize`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PTCS-01 | (y), (x) or (~ - reason) | ppPerasTargetCommitteeSize must not be lower than 500 | CIP-140 |
| PTCS-02 | (y), (x) or (~ - reason) | ppPerasTargetCommitteeSize must not be larger than 1000 | CIP-140 |

##### `ppPerasHealingFactor`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PHF-01 | (y), (x) or (~ - reason) | ppPerasTargetCommitteeSize must not be lower than 1 | CIP-140 |
| PHF-02 | (y), (x) or (~ - reason) | ppPerasTargetCommitteeSize must not be larger than 3 | CIP-140 |

##### `ppPerasQuorumThresholdSafetyMargin`

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PQTSM-01 | (y), (x) or (~ - reason) |  `ppPerasQuorumThresholdSafetyMargin` must be lower than 0.25 | Total quorum size should not extend 1 |

##### Era length

| ID | Class | Guardrail | Basis / rationale |
| :---- | :---- | :---- | :---- |
| PE-01 | (y), (x) or (~ - reason) | epoch lenght must be a divisible by the Peras round length ($U=90$) | CIP-140 |

---

## 12. Initial settings

**To be filled in.**

---

## 13. Open issues, prioritized

Ordered by what they block. Each names the decision, who owns it, and where it should be settled. Downstream drafting work, such as transcribing the agreed changes into the constitution itself, is out of scope here and tracked separately.

**1. No specification states the governance grouping of any parameter.** CIP-0164, CIP-0023 and CIP-0050 are all silent on it, and the placement comes from the ledger alone. The CIPs should carry it, as the reference other implementers build against. Separately, an open question for the Parameters Committee: is the grouping as implemented the one they want? Owner: the CIP authors for the omission, the Parameters Committee for the grouping itself. Venue: CIPs.

**2. CIP-0164 states the timing parameters in seconds; on chain they are milliseconds.** Routine alignment, blocking nothing here, since this document writes milliseconds throughout and the ledger types all three `Milliseconds32`. The CIP's Units column will be updated to give the unit the value is carried in. Owner: CIP-0164 authors. Venue: CIPs#1250.

**3. `minPoolMargin`.** Introduced in the Dijkstra era, inactive until the intra-era hard fork to PV13. Should it be introduced in the Constitution at this time (Section 8)? Owner: Parameters Committee. Venue: this document's review.

**4. The Peras parameters.** Introduced in the Dijkstra era, inactive until the intra-era hard fork to PV13. Should they be introduced in the Constitution at this time (Section 10)? Owner: Tweag by Modus Create as Peras owner. Venue: cardano-node#6634.

### Closed

---
