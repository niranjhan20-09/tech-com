# Smart Ambulance–Hospital Coordination and Dynamic Route Optimization Using Real-Time Data

**Research Proposal / Topic Package**

Prepared: 1 October 2026
Repository: `niranjhan20-09/tech-com`
Branch: `arena/01a0f5a8-tech-com`

---

## How to use this document

Sections 1–12 below follow exactly the numbering you requested. Every quantitative claim is
anchored to a **verified, indexed source** (PubMed/PMC, ASCE Library, Springer/BMC, Cambridge
University Press, JMIR, ACM/IEEE). Section 13 is the reference list, **tiered by evidentiary
strength** — Tier A sources are safe to cite directly in a journal paper; Tier B sources are
useful for motivation and gap-framing but should be cited as "emerging/preliminary". Section 14
is an evidence-extraction table you can lift straight into a literature matrix.

A flag to carry forward: several of the strongest *systems* papers in this space are
**simulation-only** (no prospective clinical deployment). That is itself one of the main gaps
this proposal exploits — see §5.4.

---

## 1) Title

> **"Smart Ambulance–Hospital Coordination and Dynamic Route Optimization Using Real-Time Data"**

**Suggested subtitle variants** (pick by submission venue):

| Variant | Best for |
|---|---|
| *…Using Real-Time Data* | General / multidisciplinary venues (keeps scope broad) |
| *…A Real-Time Decision-Support Framework for Ambulance Destination Selection and Traffic-Aware Routing* | Health informatics / EMS journals |
| *…Integrating Bed-Availability Forecasting, Location-Based Services, and Dynamic Route Optimization* | Transportation / ITS venues (ASCE, TRB) |
| *…Design and Simulation-Based Evaluation of an Interoperable Prehospital Coordination Platform* | If you deliver a working prototype + DES evaluation |

**Working acronym:** **SAHC-DRO** (Smart Ambulance–Hospital Coordination with Dynamic Route Optimization).

---

## 2) Introduction

Emergency medical services (EMS) operate under a hard and unforgiving constraint: for
time-critical conditions — ST-elevation myocardial infarction (STEMI), acute ischemic stroke,
major trauma, sepsis — clinical outcome is a monotone-decreasing function of elapsed time.
Each minute of untreated large-vessel stroke destroys on the order of two million neurons;
cardiac-arrest survival falls roughly 7–10% per minute without intervention.

The operational chain that converts a 911 call into definitive care has four links:

```
   Dispatch  →  Response (scene)  →  Transport + Destination selection  →  ED handover
```

Ambulance crews and dispatchers currently make the two most consequential decisions in that
chain — **which hospital** and **which route** — with systematically incomplete information:

- **Hospital state is invisible pre-arrival.** Bed availability, ICU/ventilator capacity,
  specialist presence, CT/cath-lab readiness, and current ED crowding are generally not exposed
  to the transporting crew in real time. The crew's default heuristic is therefore *proximity*.
- **Road state is dynamic and unmodelled.** Static distance-based routing (Dijkstra, A\*) treats
  edge weights as constant, when real urban travel times vary markedly with time of day, day of
  week, weather, incidents, and intersection-level congestion.
- **Hospital notification is late and lossy.** Pre-arrival warning is typically a voice radio
  report, which is unstructured, non-persistent, arrives once, and reaches only whoever happens
  to be listening.

The consequences are measurable at population scale. When a receiving ED is at capacity,
ambulances are **diverted**; a 2025 scoping review found diversion adds **1.7–7 minutes** of
transport time and, when episodes exceed 12 hours, is associated with mortality increases of up
to **35%** [8]. High ED crowding independently increases the odds of inpatient death by
**5% (95% CI 2–8%)** [9]. On the receiving end, crew time lost to offload delay is enormous: in
California, mean ambulance patient offload time (APOT-1) across **5.9 million** offloads was
**42.8 minutes**, and **16 of 34** local EMS agencies exceeded the state's 30-minute standard
[24].

Critically, the *inverse* evidence is equally strong: when coordination information **is**
delivered in real time, outcomes improve.

- A regional real-time capacity dashboard (REPAC, Calgary) raised the proportion of time
  hospitals spent in a favourable receiving status from **57.5% → 64.1% → 78.7%** over 12 months
  and cut EMS avoidance from **4.4% → 1.8% → 0.6%** (p<0.001) [3].
- A simulation-based bed-availability forecaster integrated with Google Maps routing achieved
  ED/ICU availability prediction accuracies of **[100%, 88.7%]**, **[100%, 90.5%]**, and
  **[100%, 92.1%]** at 20/40/60-minute horizons and used them to recommend destinations [1].
- A meta-analysis of **21 studies / 3,267 patients** found smartphone-based prehospital
  coordination apps reduced door-to-balloon time by **19.11 minutes (95% CI −26.22 to −12.00)** [20].
- The 2026 AHA/ASA stroke guideline reports, across **371,988** EMS-transported patients, that
  prenotification shortened door-to-imaging to **26 vs 31 minutes** and raised on-time
  thrombolysis from **79.2% → 82.8%** [23].

Simultaneously, three technology curves have matured enough to make an integrated platform
tractable: (i) **cloud + mobile + IoT** for real-time telemetry and two-way crew↔hospital
messaging; (ii) **commercial traffic APIs and ML travel-time models** that beat constant-speed
baselines; and (iii) **health-interoperability standards** (HL7 FHIR, and specifically the FHIR
SANER bed-availability implementation guide and the CDC NHSN Bed Capacity project) that now
define machine-readable capacity reporting [26, 27].

**This research investigates whether integrating real-time hospital availability,
location-based services, and dynamic route optimization into a single coordination platform
improves the efficiency of ambulance-to-hospital coordination and reduces the time required to
identify and reach an appropriate receiving hospital.**

---

## 3) The Research Problem

### 3.1 Concise problem statement

> Ambulance crews and EMS dispatchers must select a destination hospital and a route under
> severe time pressure, but they lack real-time access to (a) emergency capacity and facility
> capability at candidate hospitals, and (b) current and predicted traffic conditions along
> candidate routes. Destination selection therefore defaults to proximity, and routing defaults
> to static shortest-path. When the nearest hospital is crowded, lacks the required capability,
> or is reached through congestion, the patient is diverted after arrival or transported to a
> facility that cannot definitively treat them — adding transport time, delaying definitive
> care, and removing the ambulance from service. Hospitals, symmetrically, receive little or no
> advance warning and cannot pre-position staff, imaging, or catheterisation resources.
>
> **There is no widely-deployed, standards-based system that closes this loop — jointly
> optimising destination choice and route choice against live capacity and live traffic, while
> pushing a structured pre-arrival notification to the selected hospital.**

Formally, the gap is that existing practice optimises:

$$\min_{h \in H}\; d(\text{scene}, h) \qquad\text{(nearest hospital)}$$

whereas the decision that actually governs time-to-definitive-care is:

$$\min_{h \in H,\; p \in P(\text{scene}, h)}\; \underbrace{T_p(t_0)}_{\text{traffic-aware ETA}} + \underbrace{W_h\!\left(t_0 + T_p\right)}_{\text{predicted ED wait}} \quad \text{s.t.} \quad h \models C(\text{patient})$$

where $H$ is the hospital set, $P$ the path set, $T_p$ the time-dependent traversal time along
path $p$, $W_h(\tau)$ the predicted wait/offload delay at $h$ on arrival at time $\tau$, and
$h \models C(\text{patient})$ a capability-match constraint (stroke centre, PCI capability,
trauma level, ICU/ventilator, obstetrics, paediatrics, …).

**Note what the second formulation adds:** the *predictive* term $W_h(t_0 + T_p)$. Because
transport takes 10–40 minutes, the relevant question is not "is there a bed now?" but "will
there be a bed when we arrive?" — this is precisely the advance demonstrated by Xu et al. [1].

### 3.2 Why the problem is important

**Clinical.** Minutes determine tissue and life. For the four classic time-critical pathways:

| Pathway | Metric | Evidence of prenotification / coordination benefit |
|---|---|---|
| STEMI | Door-to-balloon | −19.11 min with smartphone coordination apps (95% CI −26.22 to −12.00) [20] |
| Acute stroke | Door-to-imaging / CT | 26 vs 31 min; on-time thrombolysis 82.8% vs 79.2% [23]; door-to-CT 13 vs 19 min, p<0.001 [22] |
| Major trauma | Time to definitive surgical care | Destination-capability matching is the dominant modifiable factor [11, 17] |
| General ED | Inpatient mortality | High crowding → +5% odds of death (95% CI 2–8%) [9] |

**Operational.** Ambulance-hours are the scarce resource. Every minute an ambulance waits "on
the wall" is a minute unavailable to the next caller. Offload delay is not a rounding error —
statewide means of 42.8 minutes have been documented [24]. Routing improvement compounds
directly: EMVLight demonstrated up to **42.6%** reduction in emergency-vehicle travel time
while *simultaneously* reducing average travel time for general traffic by **23.5%** [13].

**Equity.** Diversion and crowding harms are not evenly distributed. Documented racial
disparities exist in mortality among acute-MI patients affected by diversion [8], and rural
catchments face structurally longer transport times. A system that reasons explicitly about
capability and predicted wait — rather than raw distance — is a lever for equitable destination
assignment.

**Economic.** ED crowding raises length of stay (~0.8%) and cost per admission (~1%) even
after case-mix adjustment [9]; ambulance-hours lost to offload delay carry direct agency cost
(Riverside and San Bernardino Counties logged 20,535 delay-hours in 2012, ≈$3M in lost unit
hours [24]).

### 3.3 Challenges that will arise

**C1 — Data availability and freshness.** Hospital capacity data are heterogeneous, frequently
manual, and inconsistently refreshed. The Dutch national experience is instructive: 75.6% of
EDs adopted the dashboard, but **52.8% of users reported it only occasionally alleviates
inflow**, partly because criteria are subjective and capture only a small slice of ED
presentations [5]. Any system is bounded by the honesty and latency of its inputs.

**C2 — Capability matching is not a scalar.** "Can this hospital take this patient?" is a
high-dimensional constraint (specialty, imaging, cath lab, ICU level, ventilators, obstetrics,
paediatrics, burn/trauma designation, current diversion status, staff on shift). Naive
proximity ignores it entirely; naive capacity scoring loses it.

**C3 — Travel-time uncertainty under congestion.** ETA error is non-Gaussian and correlated
across edges (a single incident perturbs many links). Recent selective-dispatch work models
this explicitly with context-dependent uncertainty sets rather than point estimates [28].

**C4 — Coupling of routing and signal control.** Choosing a route and green-waving that route
are one problem, not two. Traditional preemption *increases* general-traffic delay by ~10%;
joint formulations (EMVLight) recover that loss and turn it into a gain [13]. In most
jurisdictions the ambulance operator does not control the signals — a governance, not
algorithmic, obstacle.

**C5 — Interoperability and standards.** HL7v2 remains the installed base; FHIR is the
strategic direction. CDC NHSN Bed Capacity and the FHIR SANER implementation guide are
converging on standardized machine-readable capacity reporting [26, 27], but real-world
hospital EHR exposure of these endpoints is uneven.

**C6 — Privacy, security, and consent.** Continuous geolocation of ambulances plus
patient-identifiable telemetry is high-sensitivity data. HIPAA/GDPR impose encryption,
access-control, data-minimisation, and audit-trail obligations; resource-constrained IoT
devices are historically weak at all four [29]. Location tracking also raises surveillance and
autonomy concerns.

**C7 — Alarm fatigue and false activation.** Over-alerting hospital teams degrades the very
response the system is meant to accelerate. Meta-analysis found false-activation rates reported
inconsistently and not poolable across studies [20] — an open measurement problem.

**C8 — Human factors and liability.** Paramedics must trust and override the recommendation.
Automated destination recommendation touches clinical-responsibility and malpractice questions
(who is liable if the algorithm's choice is worse than the nearest hospital?).

**C9 — Compute latency.** Decisions must return in seconds. Any exact ILP formulation must be
demonstrated to solve within an operational time budget, or be replaced by a heuristic with
bounded regret.

**C10 — Evaluation without harming patients.** Randomising real ambulance destinations is
ethically fraught. Discrete-event simulation over real historical data is the accepted
substitute, which means **external validity, not internal validity, is the binding limitation**.

---

## 4) Objectives of the Research

### Primary objective

> **To design, implement, and evaluate a real-time coordination platform that jointly
> recommends a destination hospital and a traffic-aware route to EMS crews, and pushes a
> structured pre-arrival notification to the selected hospital.**

### Specific objectives

| # | Objective | Success measure |
|---|---|---|
| O1 | **Characterise** the decision problem and information requirements of ambulance destination selection and routing, via structured review and stakeholder elicitation (paramedics, dispatchers, ED leads) | Documented requirement set + prioritised criteria taxonomy |
| O2 | **Model** destination selection as a constrained, multi-criteria optimisation over live capacity, capability match, and traffic-aware ETA, including *predicted* (not merely current) bed availability at arrival time | Formal model + solvability within operational latency budget |
| O3 | **Implement** a prototype: hospital-side capacity service (FHIR-aligned), ambulance-side mobile client (GPS + crew input), routing service (live-traffic + ML travel-time), and pre-arrival notification channel | Working end-to-end prototype |
| O4 | **Develop and benchmark** the routing module against (a) static distance Dijkstra/A\*, (b) commercial traffic API, (c) ML travel-time prediction, and (d) optimal/exact solution | MAE, RMSE, R², and mean travel-time reduction |
| O5 | **Evaluate** end-to-end system performance by discrete-event simulation driven by real historical EMS and ED data | Reduction in time-to-provider; reduction in diversion rate; ED load-balance (CV) |
| O6 | **Assess** the pre-arrival notification component against standard prenotification metrics | Door-to-provider / door-to-CT / door-to-balloon analogues; false-activation rate |
| O7 | **Analyse** safety, security, privacy, and equity implications, and specify a conformance/governance profile | Threat model, HIPAA/GDPR control mapping, equity impact assessment |
| O8 | **Publish** an open, reusable specification (data model, API contract, scoring function, benchmark scenarios) to enable replication | Public artefact + reference implementation |

---

## 5) Literature Review

### 5.1 Thematic structure

The literature divides into five clusters. (A fuller per-paper extraction is in §14.)

#### Cluster A — Hospital / ED destination selection and capacity dashboards

The earliest and most influential work is **Sprivulis & Gerrard** (Perth, 2005), who showed that
pre-emptive ambulance distribution using an **internet-accessible ED workload schematic** plus
prehospital triage allocation reduced diversion — largely by shifting flow from inner-metro to
outer-metro EDs [4].

**McLeod et al.** (REPAC, Calgary, 2010) is the reference implementation for this cluster. A
dashboard synthesised real-time capacity and acuity across three adult EDs, refreshed every
**120 seconds**, colour-coded receiving status, and — except for protocolised STEMI/stroke/major
trauma — **automatically assigned** a destination. Over three 6-month windows: favourable
status **57.5% → 64.1% → 78.7%**; EMS avoidance **4.4% → 1.8% → 0.6%** (p<0.001) [3].

**Xu et al.** (Taiwan, 2024) contributed the key *temporal* advance: rather than reporting
current beds, a simulation-based forecaster predicted ED/ICU availability at
**20/40/60-minute** horizons. Accuracies reached **[100%, 88.7%]**, **[100%, 90.5%]**, and
**[100%, 92.1%]**, and the system combined the forecast with Google Maps routing distance and an
eight-dimension hospital quality assessment to recommend destinations. In their worked example,
the highest-scoring and closest hospital was **rejected** because the forecast showed no bed
within 60 minutes [1].

Two important *cautionary* results temper this cluster:

- **Baan-Kooman et al.** (Netherlands, 2025): across 62 of 82 Dutch EDs using a national
  traffic-light diversion dashboard, **69.7%** of ED managers still experienced crowding more
  than three times weekly, and **52.8%** said the dashboard only occasionally reduced inflow.
  The authors conclude the dashboard "reflects *perceived* crowding rather than purely objective
  measures" and that diversion merely displaces regional overcrowding [5].
- **Ong et al.** (2025 scoping review): diversion adds **1.7–7 min** transport time; mortality
  rises by up to **35%** for episodes >12 h; evidence on mortality is genuinely mixed, with
  roughly as many neutral/positive studies as negative ones [8].

Together these say: **dashboards help, but a dashboard alone is not an optimisation.** It
redistributes a fixed pie; it does not choose the best slice.

#### Cluster B — Routing, traffic-aware pathfinding, and signal preemption

**Zeng, Yi, Wang & Qu** (ASCE *J. Transp. Eng., Part A: Systems*, 2021) is the anchor citation
for this proposal from the transportation side. It explicitly frames the problem the way this
research does: "inactive traffic operators fail to provide coordination to prioritize the
ambulance, while ignoring the choice of hospitals will lead to inevitable patient transfer
between hospitals." The paper introduces **prehospital screening** (injury diagnosis driving
ambulance-type assignment) and **lane preclearing** (guaranteeing a predefined driving speed),
formulates a **mixed-integer linear programme** with **semi-soft time windows** penalising late
arrival both on-scene and at hospital, and validates on a real **Shenzhen** network, explicitly
analysing the marginal value of each stakeholder's participation [11].

**Kohneh et al.** (ASCE *JTEP-A*, 2024) extend to connected-vehicle technology: a bi-objective
model minimising ERV travel time while maximising safety, enabling contraflow lane use by
mission priority, solved by a hybrid NSGA-II/PSO that outran CPLEX [12].

On learning-based control, **EMVLight** is the state of the art: a decentralised multi-agent RL
framework that jointly optimises EMV routing and traffic-signal pre-emption using dynamic
Dijkstra routing plus multi-agent advantage actor-critic, with a pressure-based reward. Results:
**up to 42.6%** reduction in EMV travel time and **23.5%** shorter average travel time for *all*
vehicles — notably, the traditional pre-emption baseline *increased* general-traffic travel time
by ~10% [13]. Follow-up work reaches **37.7%** reduction via offline Decision Transformer
training that avoids unsafe online exploration [14].

Route-based signal preemption (**Mu et al.**) formulates earliest/latest green-start windows per
intersection and solves a multi-objective programme trading EV residence time against general
vehicle throughput [15].

**Almalki, Aldhahri & Aljojo** (2025) is the closest structural analogue to the destination
module proposed here: a **two-stage** algorithm — (i) *time-based selection*: nearest EMS centre
by a time-weighted Dijkstra; (ii) *service-based selection*: verify bed availability, department
access, and injury-type match, falling back to alternatives when the nearest facility fails —
built over OpenStreetMap. It directly names the failure mode: "when EMS prioritize proximity
over medical suitability, ambulances may choose closer facilities over those with appropriate
specialization" [16].

#### Cluster C — Simulation and operations-research approaches to hospital selection

**Tedesco, Feletti et al.** (*J. Ind. & Prod. Eng.*, 2024) built a DES covering the **entire**
EMS process — ambulance transport through patient departure from the ED — and compared
assignment logics on a real **Milan** EMS system. Their headline finding directly indicts the
proximity default: the best effectiveness-driven logic cut total average EMS process time (GPT)
from **7.28 h to 6.08 h (~20%)**, and criteria based on expected waiting time and resource
saturation significantly outperformed proximity [17].

A related ACM-published DES study found minimising **Time To Provider (TTP)** — from the start
of the ambulance journey to the start of clinical evaluation in the ED — to be the best single
criterion, reducing both throughput times and overcrowding [18].

**Caicedo Rolón & Rivera Cadavid** provide the field's only dedicated literature review of
hospital selection, and their frequency counts are the cleanest statement of the gap: selection
criteria used in prior work were **closeness 83.33%**, **hospital care capacity 62.5%**, and
**shortest queue / most free beds 45.83%**; dominant methods were DES, queuing models, and
MILP solved with CPLEX/Arena; and the authors explicitly recommend that future work incorporate
technological developments *with* the participation of EMS actors [19].

**Azarnoush & Nemati** (*Sci. Rep.* 2026) contribute the "Transportation 4.0" framing this
proposal inherits: an **Intelligent Emergency Information System (IEIS)** with an integer-linear
model minimising total travel time across **two phases** (ambulance→patient, patient→hospital),
solved with CPLEX on a real network (25 street nodes, 9 ambulances, 6 calls, 5 hospitals), and
re-solved under **real-time disruption scenarios** (new call node; ambulance breakdown) to
demonstrate re-planning. Their stated limitations are exactly this proposal's design brief:
travel times treated as deterministic, and **hospital capacity constraints omitted** [2].

#### Cluster D — Machine learning for EMS operations

The definitive synthesis is **Alrawashdeh et al.** (*Prehosp. Disaster Med.*, 2024), a
PRISMA-ScR scoping review (PROSPERO CRD42021271256) of **164 studies** (125 clinical, 39
operational). Two findings matter enormously for gap-framing:

1. Operational ML concentrated on **ambulance allocation (n=21)**, detection (n=5), deployment
   (n=5), and **route optimization (n=5)** — only five route-optimization studies in a corpus of
   164 [19a].
2. The authors call for **standardisation and task-specific evaluation**, noting wide
   heterogeneity in metrics and inputs. Median operational performance: AUC 96.1%, accuracy
   90.0%, sensitivity 94.4%, specificity 87.7%.

On travel time specifically, **Zhao & Vanberkel** (ISCRAM 2025) compared three ANN architectures
against Google Maps and the traditional constant-speed model on real Nova Scotia EMS data,
concluding ANNs align more closely with historical data and suit EMS simulation, while Google
Maps suits live dispatch — a clean statement that the two are **complementary, not competing**
[25].

Broader evidence that ML-augmented operations pay off: ML-driven dynamic redeployment cut
response time by **1.5 minutes** [19a]; an optimization-augmented ML pipeline on San Francisco
911 data reduced mean response time by **up to 30%** versus online benchmarks [30];
gradient-boosted models on >1 million Stockholm EMS missions (2017–2022) identified weather,
call priority, and system strain as dominant drivers [31]; robust optimization for Dhaka reduced
median response time **10–18%** overall and **24–35%** at rush hour [32]; and ML forecasting of
ambulance turnaround time reached **90.16%, 97.02%, 93.09%** accuracy across three hospitals
[33].

#### Cluster E — Prehospital notification and mHealth communication

This is the best-evidenced cluster clinically, because it has been tested on patients rather
than in simulation.

- **STEMI meta-analysis** (JMIR mHealth & uHealth, 2025): 21 studies, 3,267 patients, 12
  countries. Door-to-balloon reduced by **19.11 min (95% CI −26.22 to −12.00; P<.01)**.
  Short-term mortality risk difference −0.03 (95% CI −0.07 to 0.01; P=.10) — i.e. *process*
  improvement is established, *mortality* benefit is not yet. Heterogeneity was high (I²=89%),
  with **healthcare setting (low- vs high-income) the most significant moderator** [20].
- **Pulsara feasibility study** (BMJ Open, 2022, regional Victoria, Australia): patients were
  off the ambulance stretcher faster for stroke (**11 vs 19 min**, p=0.0001) and STEMI
  (**14 vs 19 min**, p=0.0014); ED triage to emergency category rose to 92% (from 47%) for
  stroke [21].
- **Taiwan multicentre stroke study**: prenotification shortened median door-to-CT from
  **19 → 13 min** (p<0.001); prenotification sensitivity 78.3%, PPV 78.2% [22].
- **2026 AHA/ASA Guideline** (371,988 patients): EMS prenotification associated with
  door-to-imaging **26 vs 31 min**, door-to-needle **78 vs 80 min**, and on-time tPA **82.8% vs
  79.2%** [23].

On the systems side, IoT "smart ambulance" work (vital-sign telemetry, GPS, cloud hospital
databases, NodeMCU/Firebase-class prototypes) is abundant in IEEE and regional venues and
establishes engineering feasibility of the sensing/notification layer [34, 35], but is
predominantly **prototype-scale, non-randomised, and not clinically validated** — Tier B
evidence.

### 5.2 Synthesis table — what has been done vs. what is missing

| Study | Capacity live? | Capability match? | Dynamic routing? | Predicted (not current) capacity? | Notification? | Evaluated clinically? |
|---|:--:|:--:|:--:|:--:|:--:|:--:|
| Sprivulis 2005 [4] | ✓ | – | – | – | – | Observational |
| McLeod (REPAC) 2010 [3] | ✓ | Partial | – | – | – | Observational |
| Xu 2024 [1] | ✓ | ✓ (8-dim) | ✓ (distance) | **✓** | – | Simulation |
| Zeng 2021 (ASCE) [11] | – | ✓ (screening) | ✓ | – | – | Case study |
| EMVLight [13] | – | – | **✓✓** | – | – | Simulation |
| Almalki 2025 [16] | ✓ | **✓** | ✓ | – | – | Simulation |
| Tedesco 2024 [17] | ✓ | – | – | ✓ (throughput) | – | Simulation |
| Azarnoush 2026 [2] | **✗ (explicit gap)** | – | **✓✓** | – | – | Case study |
| Pulsara / STEMI apps [20,21] | – | – | – | – | **✓✓** | **✓ (meta-analysis)** |
| **This research** | **✓** | **✓** | **✓** | **✓** | **✓** | Targeted |

### 5.3 Findings established by prior work

1. Real-time capacity visibility **reduces diversion** [3, 4] but **does not by itself relieve
   crowding** [5].
2. Selection criteria that incorporate **ED state** (expected wait, resource saturation) beat
   **proximity** decisively in simulation — ~20% reduction in total process time [17, 18].
3. **Predicting** bed availability at the arrival horizon is feasible and materially changes the
   recommendation [1].
4. Joint routing + signal control yields large gains (**42.6%** EMV travel time) without the
   ~10% general-traffic penalty of naive preemption [13].
5. Pre-arrival notification reliably improves **process** times in time-critical pathways;
   **mortality** benefit remains unproven [20, 23].
6. ML in EMS is heavily skewed to **allocation**, with **route optimization comparatively
   under-studied** (n=5 of 164) [19a].

### 5.4 Gaps this research aims to fill

> **G1 — Fragmentation.** Capacity dashboards [3,5], routing optimizers [11,13,16], and
> notification apps [20,21] are three mature but **disconnected** literatures. No indexed study
> integrates all three into one decision loop. *This is the primary gap.*

> **G2 — Capacity-blind routing.** The most rigorous real-time routing work treats hospital
> capacity as out of scope by assumption. Azarnoush & Nemati state it verbatim: "resource
> limitations and service times were neglected… future works can incorporate the constraints on
> available ambulances and hospital capacities" [2].

> **G3 — Current-state vs. arrival-state.** Most destination systems read *now*. Only Xu et al.
> forecast the arrival horizon [1], and they did so without live traffic-aware ETA integration.

> **G4 — Simulation-only validation.** Even the strongest coordination evidence is DES or
> case-study based; the authors of [1] explicitly note "the need for real-time testing… not
> conducted due to resource and time limitations." The field has a **translation gap**, not a
> modelling gap.

> **G5 — No shared benchmark.** No common data model, API contract, scoring function, or
> scenario set exists, so results across [1], [11], [16], [17] are not comparable. The scoping
> review's call for standardisation [19a] is unmet for this task.

> **G6 — False-activation measurement.** Meta-analysis could not pool false-activation rates
> across studies due to definitional inconsistency [20]. Without this, notification systems
> cannot be tuned against alarm fatigue.

> **G7 — Equity is unmeasured.** Disparities in diversion impact are documented as a *harm* [8]
> but no proposed system evaluates destination assignment for distributional equity.

---

## 6) Research Questions and Hypotheses

### Research questions

- **RQ1 (Integration).** Can real-time hospital capacity, capability matching, and traffic-aware
  route optimisation be integrated into a single decision-support loop that returns a
  recommendation within an operationally acceptable latency (<5 s)?
- **RQ2 (Destination).** Does destination selection based on *predicted* bed availability and
  capability match at the estimated arrival time reduce time-to-provider compared with
  (a) nearest-hospital and (b) current-state-capacity selection?
- **RQ3 (Routing).** To what extent does traffic-aware / ML-predicted routing reduce transport
  time versus static shortest-path and versus a commercial traffic API, and how does ETA error
  propagate into destination-choice error?
- **RQ4 (Notification).** Does structured pre-arrival notification generated *by the platform*
  (rather than by a human, by radio) reduce simulated door-to-provider and ED preparation lag?
- **RQ5 (System externalities).** What is the joint effect on ED load balance across the
  network, diversion rate, ambulance turnaround/offload time, and general traffic delay?
- **RQ6 (Equity and safety).** Does the system redistribute access equitably across catchment
  geographies and demographics, and what is its false-activation rate and failure mode profile?

### Hypotheses

| ID | Hypothesis | Test |
|---|---|---|
| **H1** | Joint destination+route optimisation reduces mean **time-to-provider** by **≥10%** vs. nearest-hospital + static routing | DES over historical EMS data; paired comparison with 95% CIs |
| **H2** | Using **predicted** rather than **current** bed availability reduces the rate of **arrival-at-full-capacity** (proxy for diversion) by **≥15%** | Ablation: forecast vs. current-state arm |
| **H3** | Traffic-aware ML routing reduces mean **transport time** by **8–20%** vs. static distance routing, with MAE within ~3 min of observed travel time | Benchmark vs. Dijkstra/A\*, commercial API, historical average |
| **H4** | Platform-generated structured prenotification reduces **door-to-provider** time by **≥15%** | Comparison against documented prenotification effect sizes [20,22,23] |
| **H5** | The system improves **ED load balance** — the coefficient of variation of arrivals-to-beds across network hospitals falls | Baseline vs. intervention DES runs |
| **H6** | Diversion recommendations are **not** systematically biased against rural or lower-SES catchments; equity metrics are no worse than baseline | Stratified analysis by geography/deprivation index |
| **H0 (null)** | No difference between integrated system and best single-component baseline | Two-sided test, α=0.05 |

*Note on H3's target range:* it is deliberately conservative. Published estimates span
18–22% peak-hour gains from ML travel-time models in dense urban settings [36] up to 42.6% from
joint RL routing *with* signal control [13]. Without signal-control authority — the realistic
case for a first deployment — **8–20% is the defensible pre-registration range.**

---

## 7) Technology / Emerging Areas, and Research Design & Methods

### 7.1 Technology stack and emerging areas

| Layer | Technology | Rationale / evidence anchor |
|---|---|---|
| **Interoperability** | **HL7 FHIR R4/R5** (REST, JSON); FHIR **SANER** IG for bed-availability measures; CDC **NHSN Bed Capacity** project (78 data elements, ~15-min reporting cadence) | Standardised, machine-readable capacity reporting is now specified, so the platform need not invent a schema [26, 27] |
| **Capacity state** | Time-series / simulation forecaster of ED & ICU bed availability at 20/40/60-min horizons | Feasible at high accuracy [1] |
| **Patient telemetry** | IoT/IoMT vitals (HR, BP, SpO₂, temp, ECG), edge gateway (ESP32/NodeMCU class), MQTT over TLS | Engineering feasibility established in IEEE literature [34, 35] |
| **Geolocation** | GNSS + map matching; geofencing for arrival detection | Standard |
| **Routing engine** | Time-dependent shortest path over OpenStreetMap; compare **OSRM** (static), **Dijkstra/A\*** baseline, **commercial traffic API**, and **XGBoost/LGBM travel-time prediction** | Zhao & Vanberkel show ANN/Google Maps are *complementary* — ML for simulation & planning, API for live dispatch [25] |
| **Travel-time ML** | Gradient-boosted regression (XGBoost/LightGBM) + LSTM/GRU for spatio-temporal congestion; Burg-divergence or conformal uncertainty sets for ETA risk | Contextual uncertainty modelling is the current frontier [28] |
| **Decision engine** | Multi-criteria scoring → constrained assignment; **MILP/ILP** with semi-soft time windows solved by CPLEX/Gurobi/HiGHS, with heuristic fallback under latency budget | Zeng (ASCE) MIP formulation [11]; Azarnoush ILP [2] |
| **Traffic cooperation** | V2I / C-V2X signal preemption; RL-based green-wave (EMVLight-class) where the operator has signal authority | 42.6% EMV travel-time reduction demonstrated [13] |
| **Notification** | Push + in-app to a hospital web console; structured (not free-text) case card: chief complaint, vitals trend, ECG image, ETA, required capability | Meta-analysis: −19.11 min D2B [20]; Pulsara off-stretcher −8 min [21] |
| **Evaluation** | **Discrete-event simulation** (AnyLogic / SimPy / JaamSim) over real historical EMS + ED data | Accepted substitute where randomising real destinations is unethical [17, 18] |
| **Governance** | HIPAA/GDPR controls: encryption in transit & at rest, RBAC, data minimisation, audit logging; threat model for IoMT | Required; IoMT is historically weak here [29] |

### 7.2 Research design

A **four-phase, mixed-methods design**: *descriptive → constructive → experimental (simulation)
→ analytical*. This is the standard design for information-systems/health-services engineering
research (design-science / DSR), and it is appropriate here because the artefact and its
evaluation must be produced together — the question "does it work?" is not answerable before the
system exists.

```
Phase 1 (Months 1–3)    Phase 2 (Months 4–7)     Phase 3 (Months 8–11)    Phase 4 (Month 12)
Requirements &          Prototype build          DES experimentation      Analysis, equity,
evidence synthesis      ───────────────►         & benchmarking          security, write-up
─────────────────►     ────────────────►        ───────────────────►    ──────────────────►
 • PRISMA-scR review     • FHIR capacity svc     • Scenario generation   • Statistical analysis
 • Stakeholder           • Crew mobile client    • Baseline arms         • Equity stratification
   interviews            • Routing service       • Ablation studies      • Threat & privacy model
 • Criteria taxonomy     • Notification channel  • Sensitivity analysis  • Spec + open release
 • Data acquisition      • Integration & tests   • Latency profiling
```

### 7.3 Data collection

| Data | Source | Notes |
|---|---|---|
| EMS CAD records (timestamps: call, dispatch, en route, on-scene, transport, arrival, back-in-service) | Partner EMS agency / open municipal data | Core DES input; de-identified |
| Ambulance GPS traces | Partner agency AVL | For map matching and travel-time ground truth |
| ED operational data (arrivals, triage acuity, bed census, ICU/ventilator counts, LOS, throughput times) | Partner hospitals; FHIR endpoints where available | Best logic approximates real-time assignment via **throughput time of the last 10 patients** [17] |
| Hospital capability registry (trauma level, stroke/PCI designation, specialties) | Public registries / regional EMSA | Static-ish; refreshes on designation change |
| Road network + traffic | **OpenStreetMap**; traffic APIs (TomTom/Google/HERE); loop/probe data where available | Keep a static arm for controlled comparison |
| Weather & calendar | Open APIs | Documented driver of EMS response-time variance [31] |
| Qualitative | Semi-structured interviews: paramedics, dispatchers, ED charge nurses, EMS medical directors, traffic-ops staff | Requirement elicitation + acceptability; thematic analysis |

**Sampling / scale target:** ≥12 months of contiguous EMS records (ideally ≥50,000 transports)
and ≥4 hospitals, with at least one tertiary/capability-differentiated facility so that
capability-matching is actually exercised. If a real agency partnership is unavailable, use a
public EMS dataset plus a synthetic-but-calibrated hospital network — and declare this as a
limitation (§9).

### 7.4 Analysis procedures

**7.4.1 Destination model.** Score-and-constrain:

$$\text{Score}(h, p) = w_1 \hat{T}_p + w_2 \widehat{W}_h(t_0 + \hat{T}_p) + w_3 \widehat{O}_h(t_0 + \hat{T}_p) - w_4 Q_h + w_5 \Phi_h$$

subject to hard constraints $h \models C(\text{patient})$ (capability), $h \notin \text{diversion}$,
and $p \in P(\text{scene}, h)$. Terms: $\hat T_p$ predicted traversal time; $\widehat W_h$
predicted ED wait; $\widehat O_h$ predicted offload delay; $Q_h$ quality/appropriateness score
(cf. the eight-dimension assessment in [1]); $\Phi_h$ an **equity/load-balancing** penalty term
(this is a novel addition — no prior study in the corpus optimises for it). Weights are tuned
by grid/Bayesian search on the training period and **reported transparently**, because
undocumented weights are a reproducibility defect in this literature.

**7.4.2 Routing benchmark.** Four arms, evaluated MAE / RMSE / R² / MAPE and mean travel time:
(i) static distance Dijkstra/A\*; (ii) OSRM default; (iii) commercial traffic API; (iv) ML
(gradient-boosted + temporal features, and an LSTM/GRU variant). Train on a chronological split
(never random — temporal leakage inflates results), hold out the last 20% by time, and report
per-hour-of-day error to expose peak-period degradation.

**7.4.3 DES experiment.** Build the full process (dispatch → response → on-scene → transport → ED
triage → evaluation → disposition → departure) following the validated structure in [17, 18].

*Experimental arms:*
- **A0 Baseline:** nearest hospital + static shortest path (current default)
- **A1:** nearest hospital + traffic-aware routing
- **A2:** current-state capacity selection + static routing
- **A3:** predicted-capacity selection + traffic-aware routing (**full system**)
- **A4:** full system + equity term $\Phi_h$
- **A5:** full system + traffic-signal cooperation (upper bound, where authority exists)

*Ablations:* forecast vs. current-state (isolates G3); capability constraint on/off (isolates
capability value); notification on/off (isolates RQ4).

*Primary outcome:* **time-to-provider** (start of ambulance journey → start of clinical
evaluation in ED) — chosen because it is the criterion that performed best in prior DES work
[18] and because it is the metric that actually measures *patient-relevant* delay rather than
*ambulance-relevant* delay.

*Secondary outcomes:* transport time; arrival-at-full-capacity (diversion proxy); ED load
balance (CV of arrivals-to-beds); ambulance turnaround/offload time; door-to-CT / door-to-balloon
analogues for time-critical subcohorts; general-traffic delay (externality); false-activation
rate.

*Statistics:* ≥30 replications per arm with common random numbers for variance reduction;
report means with 95% confidence intervals; paired comparisons across arms; two-sided α=0.05;
report effect sizes, not just p-values. Pre-specify the analysis plan before running the
intervention arms to avoid garden-of-forking-paths.

**7.4.4 Qualitative.** Thematic analysis of interview transcripts (deductive coding against the
criteria taxonomy, inductive coding for emergent themes: trust, override behaviour, alarm
fatigue, liability). Sample to thematic saturation (~12–20 participants).

### 7.5 Why this methodology suits the problem

- **DES is the right instrument** because destination selection is a *queuing* problem — the
  cost of a choice depends on other actors' choices, and the system has feedback. Closed-form
  analysis is intractable; real-world experimentation is unethical. DES is also the established
  method in this exact literature [17, 18], so results are comparable.
- **Benchmarking against four routing arms** is necessary because the field's reported gains span
  8–42% and are frequently not comparable; a controlled multi-arm benchmark is the only way to
  attribute effect.
- **Ablation design** is essential to answer RQ2 vs. RQ3 separately — otherwise "the system
  works" is unfalsifiable and uninformative about *which* component earns the gain.
- **Design-science + stakeholder elicitation** addresses the adoption gap: prior work shows
  technically-sound dashboards under-deliver when criteria are subjective and actors are not
  engaged [5, 19].
- **Pre-specification and open release** respond directly to the standardisation gap (G5) [19a].

---

## 8) Significance of the Study

### 8.1 Academic contributions

1. **First integrated formulation** of the joint destination-and-route problem under live
   capacity, capability, and traffic constraints — closing the fragmentation gap (G1) between
   three mature but separate literatures.
2. **Arrival-horizon capacity forecasting coupled to traffic-aware ETA**, unifying [1] and
   [25] — the two have not previously been composed, and their interaction is where the
   decision actually lives (G3).
3. **An explicit equity/load-balancing term** in destination scoring — absent from all reviewed
   work, and a direct response to documented distributional harms of diversion (G7) [8].
4. **A reusable benchmark**: data model, API contract, scoring function, scenario set, and
   reference implementation, addressing the standardisation gap (G5) [19a].
5. **Quantified false-activation and failure-mode characterisation** of automated prenotification
   (G6) — an outcome prior meta-analysis could not pool.
6. **External-validity evidence** on the simulation-to-deployment gap (G4), by benchmarking
   simulated effect sizes against the *observed* clinical effect sizes from [3], [20], [23].

### 8.2 Practical implications

- **For ambulance crews:** a single recommendation (destination + turn-by-turn route) replacing
  radio consultation and guesswork, with the hospital pre-alerted before arrival.
- **For hospitals:** structured advance warning with an ETA, enabling pre-positioning of the
  stroke team, cath lab, trauma bay, or resuscitation room — the mechanism behind the
  −19.11 min D2B result [20].
- **For EMS system managers:** load balancing across the network, reducing both diversion and
  offload delay against a documented 30-minute APOT benchmark [24].
- **For traffic authorities:** an interface for cooperative signal priority; where deployed,
  joint routing + signal control yields ~42.6% EMV travel-time reduction *without* the ~10%
  general-traffic penalty of conventional preemption [13].
- **For policymakers:** an evidence base for mandating machine-readable capacity reporting
  (FHIR SANER / NHSN-style) as public infrastructure, and for equity criteria in EMS destination
  protocols.
- **For low-resource settings:** the meta-analysis found setting was the strongest moderator of
  benefit, with *larger* gains where baseline infrastructure is weaker [20] — so the largest
  marginal value may be in LMIC urban EMS systems (cf. Dhaka: 24–35% rush-hour improvement [32]).

### 8.3 Who benefits

Patients (faster definitive care) · EMS crews (less wall time, less decision burden) ·
ED clinicians (prepared arrivals) · hospital system managers (load balancing) ·
traffic agencies (cooperative priority) · payers and society (reduced LOS, reduced ambulance-
hours lost, better equity).

---

## 9) Scope and Limitations

### In scope

- Urban and peri-urban EMS with ≥4 participating hospitals and road-network routing.
- Ground ambulance transport (not HEMS/air ambulance, not mobile stroke units).
- **Adult** medical, trauma, stroke, and cardiac emergencies.
- Day-to-day operations (not mass-casualty incident triage, not disaster response — a distinct
  problem with its own literature [37]).
- Design, prototype, and **simulation-based** evaluation.
- Interoperability conformance to FHIR where hospital endpoints exist, with an adapter layer
  where they do not.

### Out of scope

- On-scene clinical care protocols and paramedic scope of practice.
- Ambulance *fleet* location/relocation and dynamic redeployment (a separate, well-populated
  research stream [30, 32]).
- Direct control of traffic signals (modelled as an upper-bound arm, not implemented).
- Paediatric and obstetric destination protocols (future extension).
- Prospective clinical deployment and patient-outcome trials.

### Anticipated limitations — and mitigations

| # | Limitation | Mitigation |
|---|---|---|
| L1 | **No prospective clinical validation**; results are simulation estimates | Benchmark simulated effects against observed effects in [3, 20, 23]; report a calibrated external-validity interval; design the prototype to full deployment readiness so a later stepped-wedge trial needs no rebuild |
| L2 | **Simulation fidelity** — DES cannot capture all real-world variability | Calibrate and validate against historical distributions; run sensitivity analysis on all fitted parameters; report which conclusions are robust to perturbation |
| L3 | **Data-quality dependence** — output quality bounded by input honesty/refresh rate; Dutch data show dashboards can reflect *perceived* rather than objective crowding [5] | Model input latency and error explicitly as a stress-test scenario; define objective, automatically-derived capacity measures rather than self-reported status |
| L4 | **Single-region generalisability** — network topology, hospital mix, and traffic culture are local | Release the scenario generator so others can re-run with local parameters; report topology sensitivity |
| L5 | **Assumption of hospital participation** — capacity disclosure is voluntary and uneven | Treat partial participation as an experimental condition; quantify the value of each additional participating hospital |
| L6 | **ETA uncertainty** is non-Gaussian and spatially correlated | Use contextual uncertainty sets / conformal intervals [28]; report decision robustness under ETA error, not just mean ETA |
| L7 | **Optimality vs. latency trade-off** — exact ILP may exceed the time budget | Benchmark solver latency against network scale; specify heuristic with bounded regret and a latency SLA |
| L8 | **Equity metrics require demographic data** that may not be available | Use geographic/deprivation-index proxies; state proxy validity limits explicitly |
| L9 | **Hawthorne / adoption effects** unmeasurable in simulation | Stakeholder evaluation for acceptability and override behaviour; note as a deployment risk |
| L10 | **Commercial API dependence** creates cost, rate-limit, and vendor-lock-in risk | Keep a self-hosted OSRM + local ML path; design the routing service as a swappable adapter |

---

## 10) Timeline and Key Milestones

A **12-month** programme (Q4 2026 → Q4 2027). Adjust for your academic calendar.

| Phase | Month | Activities | Milestone / deliverable | Deadline |
|---|---|---|---|---|
| **1 — Foundations** | M1 | Protocol finalisation; PROSPERO-style pre-registration; ethics/institutional approvals; data-sharing agreements initiated | **M1.1** Approved protocol + pre-registered analysis plan | End M1 |
| | M2 | PRISMA-scR literature review; data-model and criteria taxonomy; begin stakeholder recruitment | **M1.2** Review matrix + taxonomy v1 | End M2 |
| | M3 | Stakeholder interviews & thematic analysis; EMS/ED data acquisition and de-identification | **M1.3** Requirements document; **data in hand** ✅ *hard gate* | End M3 |
| **2 — Build** | M4 | FHIR-aligned capacity service; capability registry schema; adapter for non-FHIR hospitals | **M2.1** Capacity service ingesting live data | End M4 |
| | M5 | Travel-time ML pipeline; OSRM + commercial API adapters; routing service | **M2.2** Routing benchmark report v1 (MAE/RMSE/R²) | End M5 |
| | M6 | Destination decision engine: scoring + MILP + capability constraints + heuristic fallback | **M2.3** Decision engine passing unit & latency tests (<5 s) | End M6 |
| | M7 | Crew mobile client; hospital notification console; end-to-end integration test | **M2.4** End-to-end prototype demo ✅ *hard gate* | End M7 |
| **3 — Experiment** | M8 | DES model construction; calibration & validation against historical data | **M3.1** Validated DES model (trace-driven) | End M8 |
| | M9 | Baseline arms (A0–A2) and full-system arm (A3); replication runs | **M3.2** Primary outcome results, baseline vs. system | End M9 |
| | M10 | Ablations (forecast vs. current; capability on/off; notification on/off); equity arm A4; signal-cooperation upper bound A5 | **M3.3** Ablation table — component-wise attribution | End M10 |
| | M11 | Sensitivity & stress testing (input latency, ETA error, partial participation, demand surge) | **M3.4** Robustness report | End M11 |
| **4 — Synthesis** | M12 | Statistical synthesis; equity analysis; threat & privacy model; open specification release; paper drafting | **M4.1** Full draft paper + open artefact release | End M12 |

### Critical path and risk

```
M3 data gate ──► M7 prototype gate ──► M8 DES validation ──► M9 primary results
   (highest risk)     (medium risk)        (medium risk)        (low risk)
```

**The M3 data gate is the single largest schedule risk.** Data-sharing agreements with EMS
agencies and hospitals routinely take 3–6 months. Two mitigations: (a) begin agreements in
**M1, not M3**; (b) identify a *fallback* public EMS dataset in M1 so that M4–M7 can proceed
on synthetic-calibrated data if the partnership slips. Do not let the build phase start
un-gated — everything downstream depends on real data.

### Milestone summary

| Milestone | Month | Type |
|---|---|---|
| Pre-registered protocol | M1 | Deliverable |
| Literature review matrix | M2 | Deliverable |
| **Data acquisition complete** | M3 | **Gate** |
| Routing benchmark v1 | M5 | Deliverable |
| Decision engine within latency SLA | M6 | Deliverable |
| **Working end-to-end prototype** | M7 | **Gate** |
| Validated DES model | M8 | Deliverable |
| Primary outcome results | M9 | Result |
| Ablation / attribution results | M10 | Result |
| Robustness report | M11 | Deliverable |
| Draft paper + open artefact | M12 | **Final deliverable** |

---

## 11) Expected Results

### 11.1 Quantitative expectations

| Outcome | Expected result | Basis / anchor |
|---|---|---|
| **Time-to-provider** (primary) | **−10% to −20%** vs. nearest-hospital + static routing | ~20% GPT reduction from effectiveness-driven selection [17]; capacity dashboards + routing gains compound |
| **Transport time** | **−8% to −20%** vs. static routing | 18–22% peak-hour from ML travel-time models [36]; 42.6% only *with* signal control [13] |
| **Arrival-at-full-capacity / diversion proxy** | **−15% to −30%** | Arrival-horizon forecasting [1]; REPAC cut avoidance 4.4%→0.6% [3] |
| **ED load balance** | CV of arrivals-to-beds **−10% to −25%** | Load-balancing is the mechanism in [3, 17, 18] |
| **Ambulance offload / turnaround** | **−5% to −15%** | Offload delay partially determined by destination choice [24] |
| **Door-to-provider for time-critical cohorts** | **−10% to −20%** | Observed prenotification effects: −19.11 min D2B [20]; 26 vs 31 min door-to-imaging [23] |
| **ETA prediction accuracy** | MAE ≈ 2–4 min; R² ≈ 0.80–0.90 | R²=0.84 achieved on 287 real ambulance trips [36] |
| **Decision latency** | **<5 s** at city scale | Heuristic fallback guarantees the SLA where exact ILP is too slow |
| **False-activation rate** | **<20%**, with a documented definition | Currently unmeasurable across the literature [20] — a contribution regardless of the number |
| **General traffic delay** | No degradation; −5% to −10% where signal cooperation is available | Naive preemption *costs* ~10% [13]; joint optimisation recovers it |

### 11.2 Qualitative / artefactual expectations

- A validated **criteria taxonomy** for destination selection, ranked by stakeholders.
- A working **open prototype** (capacity service, routing service, decision engine, crew client,
  hospital console).
- An **open benchmark specification** — data model, API contract, scoring function, scenario
  generator — enabling replication and cross-study comparison (G5).
- A **security and privacy control profile** mapped to HIPAA/GDPR, with a documented threat model.
- An **equity impact assessment** of destination assignment by geography and deprivation.
- A quantified statement of **which component contributes what** (ablation), so implementers
  with limited budgets know what to build first.

### 11.3 Honest statement of uncertainty

> Prior work in this area consistently reports larger simulation effects than are later observed
> in deployment. The Dutch national dashboard experience — 75.6% adoption but only occasional
> benefit per 52.8% of users [5] — is the field's most sobering datapoint, and the Xu et al.
> authors' own admission that real-time testing was not conducted [1] is the second. **We
> therefore expect our point estimates to be optimistically biased, and pre-commit to reporting
> the simulation-to-reality calibration rather than the raw simulation number.** The most likely
> genuine finding is not "42% faster" but something closer to: *most of the achievable gain
> comes from arrival-horizon capacity forecasting and capability matching, and comparatively
> little from routing sophistication once a competent traffic-aware router is in place.*

If the ablation shows that, it is a valuable and publishable result — it tells implementers to
invest in hospital data feeds rather than in routing algorithms.

---

## 12) Conclusion

Emergency care is a race against time, and the race is currently run with an incomplete map.
Ambulance crews choose destinations by proximity because they cannot see hospital capacity,
capability, or crowding; they choose routes by distance because they cannot see traffic; and
hospitals learn about incoming patients by radio, late and lossily. The measurable cost of those
three blind spots is real: 1.7–7 minutes added per diversion, mortality increases of up to 35%
under prolonged diversion, a 5% increase in the odds of inpatient death during high-crowding
periods, and statewide mean offload times exceeding 40 minutes.

The evidence that the blind spots are *fixable* is equally real, and it comes from three
directions that have never been joined. Capacity dashboards have cut ambulance avoidance from
4.4% to 0.6%. Arrival-horizon bed forecasting predicts availability at 20/40/60 minutes with
near-perfect ED accuracy. Traffic-aware and learning-based routing — and, where signal authority
exists, joint routing with preemption — has cut emergency-vehicle travel time by up to 42.6%
while *improving* general traffic by 23.5%. And structured pre-arrival notification has shortened
door-to-balloon times by 19 minutes across a meta-analysis of 3,267 patients.

**This research's contribution is the conjunction.** No indexed study integrates live capacity,
capability matching, traffic-aware routing, arrival-horizon forecasting, and automated
prenotification into a single decision loop — and the most rigorous routing work in the field
explicitly names capacity constraints as its omitted future work.

The proposed programme — requirements elicitation, standards-aligned prototype construction,
multi-arm discrete-event simulation with pre-specified ablations, and an explicit equity term —
is designed to do three things the field has not: attribute gain to components rather than to
"the system", measure distributional consequences rather than only means, and release a
benchmark so that future results are comparable. We expect a 10–20% reduction in
time-to-provider, and we pre-commit to reporting the simulation-to-reality calibration honestly
rather than the headline number.

The problem matters because the minutes being lost are neurons, myocardium, and lives. It is
solvable now because capacity reporting is being standardised, traffic data is commoditised, and
the clinical value of coordination is already proven. What remains is to build the loop — and to
measure it rigorously enough that an EMS director can act on the result.

---

## 13) References

### Tier A — Peer-reviewed, indexed (safe to cite as primary evidence)

1. Xu Y-Y, Weng S-J, Huang P-W, et al. **The emergency medical service dispatch recommendation
   system using simulation based on bed availability.** *BMC Health Serv Res.* 2024;24:1513.
   doi:[10.1186/s12913-024-12006-8](https://doi.org/10.1186/s12913-024-12006-8).
   PMID: 39616393. https://pubmed.ncbi.nlm.nih.gov/39616393/

2. Azarnoush S, Nemati A. **Transportation 4.0 planning in emergency medical services
   considering real-time ambulance two-phase assignment-routing and mission change.**
   *Sci Rep.* 2026;16:26246. doi:[10.1038/s41598-026-56495-5](https://doi.org/10.1038/s41598-026-56495-5).
   PMID: 42265297; PMCID: PMC13493795. https://pubmed.ncbi.nlm.nih.gov/42265297/

3. McLeod B, Zaver F, Avery C, Martin DP, Wang D, Jessen K, Lang ES. **Matching capacity to
   demand: a regional dashboard reduces ambulance avoidance and improves accessibility of
   receiving hospitals.** *Acad Emerg Med.* 2010;17(12):1383–1389.
   doi:[10.1111/j.1553-2712.2010.00928.x](https://doi.org/10.1111/j.1553-2712.2010.00928.x).
   PMID: 21122023. https://pubmed.ncbi.nlm.nih.gov/21122023/

4. Sprivulis P, Gerrard B. **Internet-accessible emergency department workload information
   reduces ambulance diversion.** *Prehosp Emerg Care.* 2005;9(3):285–291.
   doi:[10.1080/10903120590962094](https://doi.org/10.1080/10903120590962094).
   PMID: 16147477. https://pubmed.ncbi.nlm.nih.gov/16147477/

5. Baan-Kooman ECM, Mol S, van der Linden MC, et al. **Emergency department crowding in the
   Netherlands; evaluation of a real-time ambulance diversion dashboard.** *Int J Emerg Med.*
   2025;18:18. doi:[10.1186/s12245-024-00784-1](https://doi.org/10.1186/s12245-024-00784-1).
   PMID: 39838286; PMCID: PMC11753112. https://pubmed.ncbi.nlm.nih.gov/39838286/

6. Felice J, Coughlin RF, Burns K, et al. **Effects of real-time EMS direction on optimizing EMS
   turnaround and load-balancing between neighboring hospital campuses.** *Prehosp Emerg Care.*
   2019;23(6):788–794. PMID: 30798628. https://pubmed.ncbi.nlm.nih.gov/30798628

7. **Centralized Ambulance Destination Determination: A Retrospective Data Analysis to Determine
   Impact on EMS System Distribution, Surge Events, and Diversion Status.** PMID: 34787556;
   PMCID: PMC8597692. https://pubmed.ncbi.nlm.nih.gov/34787556/

8. Ong JHM, Lim BJW, Zahrin MABM, Yong IJS, Tan LLL, Mao RHD, Ong MEH, Siddiqui FJ. **Ambulance
   diversion and its use as an ED overcrowding mitigation strategy: Does it work? A scoping
   review.** *Int J Emerg Med.* 2025;18:125.
   doi:[10.1186/s12245-025-00933-0](https://doi.org/10.1186/s12245-025-00933-0).
   PMID: 40629271; PMCID: PMC12239308. https://pmc.ncbi.nlm.nih.gov/articles/PMC12239308/

9. **Effect of Emergency Department Crowding on Outcomes of Admitted Patients.** *Ann Emerg Med.*
   2013. PMCID: PMC3690784. https://pmc.ncbi.nlm.nih.gov/articles/PMC3690784/

10. Shen Y-C, Hsia RY. **Ambulance diversion associated with reduced access to cardiac
    technology and increased one-year mortality.** *Health Aff (Millwood).*
    2015;34(8):1273–1280. doi:[10.1377/hlthaff.2014.1462](https://doi.org/10.1377/hlthaff.2014.1462)

11. Zeng Z, Yi W, Wang S, Qu X. **Emergency Vehicle Routing in Urban Road Networks with
    Multistakeholder Cooperation.** *J Transp Eng Part A: Systems.* 2021;147(10).
    doi:[10.1061/JTEPBS.0000577](https://doi.org/10.1061/JTEPBS.0000577). ASCE Library.
    https://ascelibrary.org/doi/10.1061/JTEPBS.0000577

12. Kohneh JN, Murray-Tuite P, Chantem T, Gerdes R. **A Connected Emergency Response System to
    Facilitate the Movement of Multiple Emergency Response Vehicles through Two-Way Roadways.**
    *J Transp Eng Part A: Systems.* 2024;150(5).
    doi:[10.1061/JTEPBS.TEENG-7973](https://doi.org/10.1061/JTEPBS.TEENG-7973).
    https://ascelibrary.org/doi/abs/10.1061/JTEPBS.TEENG-7973

13. Su H, et al. **EMVLight: A Decentralized Reinforcement Learning Framework for Efficient
    Passage of Emergency Vehicles.** arXiv:2206.13441.
    https://arxiv.org/abs/2206.13441

14. **Emergency Preemption Without Online Exploration: A Decision Transformer Approach.**
    arXiv:2603.22315. https://arxiv.org/abs/2603.22315

15. Mu H, et al. **Route-Based Signal Preemption Control of Emergency Vehicle.**
    https://www.researchgate.net/publication/324156993_Route-Based_Signal_Preemption_Control_of_Emergency_Vehicle

16. Almalki M, Aldhahri E, Aljojo N. **Ambulance routing optimization based on emergency medical
    service availability.** *Discov Artif Intell.* 2025;5:297.
    doi:[10.1007/s44163-025-00352-3](https://doi.org/10.1007/s44163-025-00352-3)

17. Tedesco D, Feletti G, et al. **Hospital selection decision in emergency medical services: a
    simulation-based assessment of assignment criteria.** *J Ind Prod Eng.* 2024;41(6).
    doi:[10.1080/21681015.2024.2348557](https://doi.org/10.1080/21681015.2024.2348557)

18. **Hospital Selection in Emergency Medical Services: A Discrete Event Simulation Approach to
    Test Different Policies.** ACM.
    doi:[10.1145/3587889.3587893](https://doi.org/10.1145/3587889.3587893)

19. Caicedo Rolón ÁJ, Rivera Cadavid L. **Hospital selection in emergency medical service
    systems: A literature review.** (2021)
    https://www.researchgate.net/publication/353303332_Hospital_selection_in_emergency_medical_service_systems_A_literature_review

19a. Alrawashdeh A, Alqahtani S, Alkhatib ZI, Kheirallah K, Melhem NY, Alwidyan M, Al-Dekah AM,
    Alshammari T, Nehme Z. **Applications and performance of machine learning algorithms in
    Emergency Medical Services: a scoping review.** *Prehosp Disaster Med.* 2024;39(5):368–378.
    doi:[10.1017/S1049023X24000414](https://doi.org/10.1017/S1049023X24000414).
    PMID: PMC11810483. https://pmc.ncbi.nlm.nih.gov/articles/PMC11810483/

20. Gibson W, Al Kindi D, Akl E, et al. **Impact of Smartphone Apps on Reperfusion Times and
    Clinical Outcomes in Acute ST-Segment Elevation Myocardial Infarction: Systematic Review and
    Meta-Analysis.** *JMIR Mhealth Uhealth.* 2025;13:e66605.
    doi:[10.2196/66605](https://doi.org/10.2196/66605). PMID: 40854299; PMCID: PMC12377873.
    https://pmc.ncbi.nlm.nih.gov/articles/PMC12377873

21. Bladin CF, Bagot KL, Vu M, Kim J, Bernard S, Smith K, et al. **Real-world, feasibility study
    to investigate the use of a multidisciplinary app (Pulsara) to improve prehospital
    communication and timelines for acute stroke/STEMI care.** *BMJ Open.* 2022;12(2).
    https://research.monash.edu/en/publications/

22. **Effect of prehospital notification on acute stroke care: a multicenter study.**
    *Scand J Trauma Resusc Emerg Med.* 2016;24:57.
    doi:[10.1186/s13049-016-0251-2](https://doi.org/10.1186/s13049-016-0251-2)

23. **2026 Guideline for the Early Management of Patients With Acute Ischemic Stroke: A Guideline
    From the American Heart Association/American Stroke Association.** *Stroke.*
    doi:[10.1161/STR.0000000000000513](https://doi.org/10.1161/STR.0000000000000513)

24. **Patterns in California Ambulance Patient Offload Times by Local Emergency Medical Services
    Agency.** *JAMA Netw Open.* 2024. (5,913,399 offloads; mean APOT-1 42.8 min)
    https://jamanetwork.com/journals/jamanetworkopen/fullarticle/2828075

25. Zhao Q, Vanberkel P. **A Comparison of Ambulance Travel Time Approximation: Using Google Maps
    and Machine Learning.** *Proc Int ISCRAM Conf.* 2025;22.
    https://ojs.iscram.org/index.php/Proceedings/article/view/178

26. **HL7 FHIR SANER Implementation Guide — Creating the Bed Availability Group.**
    https://build.fhir.org/ig/HL7/fhir-saner/measure_group_beds.html

27. **CDC NHSN Connectivity Initiative: Hospital Bed Capacity Project** (78 data elements;
    ~15-min automated reporting; FHIR-aligned).
    https://www.cdc.gov/nhsn/bed-capacity/Connectivity-Initiative_Hospital-Bed-Capacity-Project-Instruction-Book.pdf
    · HL7 US SAFR track: https://confluence.hl7.org/spaces/FHIR/pages/281283222/

28. **Selective Ambulance Dispatch Under Contextual Travel-Time Uncertainty (IDEAL).**
    arXiv:2605.23378. https://arxiv.org/html/2605.23378v1

29. **Transforming digital health using the internet of things for personalized interoperable and
    secure healthcare systems.** *Discov Health Syst.* 2026.
    doi:[10.1007/s44250-026-00359-2](https://doi.org/10.1007/s44250-026-00359-2)

30. **Optimization-Augmented Machine Learning for Vehicle Operations in Emergency Medical
    Services** (San Francisco 911 case study; up to 30% mean response-time reduction).
    arXiv:2503.11848. https://arxiv.org/html/2503.11848v1

31. **Understanding EMS response times: a machine-learning-based analysis** (>1M Stockholm EMS
    missions, 2017–2022). https://www.researchgate.net/publication/390143764

32. Boutilier JJ, Chan TCY. **Ambulance Emergency Response Optimization in Developing Countries**
    (Dhaka; robust optimization; 10–18% median, 24–35% rush-hour response-time reduction).
    https://utoronto.scholaris.ca/

33. **Machine learning-based forecasting of firemen ambulances' turnaround time in hospitals,
    considering the COVID-19 impact.** PMID: 34899108.
    https://pubmed.ncbi.nlm.nih.gov/34899108/

37. Shiri D, Akbari V, Salman S. **Online optimisation for ambulance routing in disaster response
    with partial or no information on victim conditions.** *Comput Oper Res.*
    2023;159:106314.

### Tier B — Emerging / preprint / regional venues (cite as preliminary; do not anchor claims)

34. **Smart Ambulance: A Comprehensive IoT and Cloud-Based System Integrating Fingerprint Sensor
    with Medical Sensors for Real-time Patient Vital Signs Monitoring.** *Int J Intell Syst Appl
    Eng.* 2024;12(2):555–567. https://ijisae.org/index.php/IJISAE/article/view/4299

35. **Smart Ambulance System using IoT.** IEEE Xplore document 10550283.
    https://ieeexplore.ieee.org/document/10550283 ·
    **Design and development of an IoT-based smart ambulance system with patient monitoring.**
    IEEE Xplore document 10059568. https://ieeexplore.ieee.org/document/10059568

36. **SWIFTAID: An AI-driven Ambulance Dispatch and Route Optimization Platform for Dense Urban
    Environments** (Bengaluru; XGBoost R²=0.84 on 287 real trips; 18–22% peak-hour response
    reduction). https://ijcope.org/wp-content/uploads/2026/04/SWIFTAID-...pdf
    *(regional venue, n=287 — use for plausibility only)*

### Additional useful context (not primary evidence)

- **GAO-23-105740, Intelligent Transportation Systems: Benefits Related to Traffic Congestion**
  (emergency vehicle preemption deployment status). https://www.gao.gov/assets/gao-23-105740.pdf
- **Rural ITS Toolkit — ES5 Emergency Vehicle Traffic Signal Preemption** (Savannah GA: 5–7 min
  EMS response-time reduction). https://ruralsafetycenter.org/wp-content/uploads/2022/08/ES5_Updated2022_508.pdf
- **Amorim M, Ferreira S, Couto A. Emergency Medical Service Response: Analyzing Vehicle
  Dispatching Rules.** *Transp Res Rec.* 2018;2672(32):10–21.
- **Haghani A, Tian Q, Hu H. Simulation Model for Real-Time Emergency Vehicle Dispatching and
  Routing.** *Transp Res Rec.* 2004;1882:176–183.

---

## 14) Appendix — Evidence Extraction Table

Lift this directly into your literature matrix.

| # | Source & year | Venue / DB | Design | Setting / N | Key intervention | **Key finding (extracted number)** | Gap it leaves |
|---|---|---|---|---|---|---|---|
| 1 | Xu 2024 | BMC HSR (PubMed) | Simulation + web crawler | Taiwan, 10 hospitals | Bed-availability forecaster + Google Maps routing + 8-dim hospital assessment | ED/ICU forecast accuracy at 20/40/60 min: **[100%, 88.7%] / [100%, 90.5%] / [100%, 92.1%]** | No real-time deployment test; routing is distance-only |
| 2 | Azarnoush 2026 | Sci Rep (PubMed) | ILP + CPLEX + scenarios | Amol, Iran: 25 street, 9 ambulance, 6 call, 5 hospital nodes | Two-phase (ambulance→patient, patient→hospital) real-time assignment-routing | Model re-solves correctly under new-call & breakdown scenarios | **Hospital capacity & service times omitted**; deterministic TTs |
| 3 | McLeod 2010 | Acad Emerg Med (PubMed) | Before/after observational | Calgary, 3 EDs, 3×6-month windows | REPAC dashboard, 120-s refresh, auto destination assignment | Favourable status **57.5%→64.1%→78.7%**; EMS avoidance **4.4%→1.8%→0.6%** (p<0.001) | No routing component; no capability matching |
| 4 | Sprivulis 2005 | Prehosp Emerg Care (PubMed) | Before/after | Perth, 8 EDs | Internet ED workload schematic + ATS allocation | Diversion reduced; flow shifted inner→outer metro | Complementary to, not a substitute for, inpatient flow fixes |
| 5 | Baan-Kooman 2025 | Int J Emerg Med (PubMed) | National survey | Netherlands, 62/82 EDs | Traffic-light diversion dashboard | **69.7%** crowding >3×/week; **52.8%**: only occasionally reduces inflow | Dashboard reflects *perceived* crowding; displaces rather than solves |
| 6 | Felice 2019 | Prehosp Emerg Care (PubMed) | Pre/post | 2 campuses, 1 health system | Nurse navigator + real-time capacity metrics | Improved EMS turnaround & intercampus transfers | Single system; labour-intensive (human in the loop) |
| 7 | CAD-D 2021 | PubMed/PMC | Retrospective | Regional EMS system | Centralised physician-paramedic destination determination | Improved ambulance distribution (CV) & surge; reduced diversion | Labour-intensive; not automated |
| 8 | Ong 2025 | Int J Emerg Med (PubMed) | Scoping review | Multiple | Ambulance diversion | +**1.7–7 min** transport; mortality **+up to 35%** if diversion >12 h | Evidence mixed; equity harms documented |
| 9 | Sun 2013 | Ann Emerg Med (PMC) | Retrospective cohort | California, 995,358 admissions | ED crowding (diversion-hours proxy) | High crowding: **+5% odds inpatient death (95% CI 2–8%)**, +0.8% LOS, +1% cost | Observational; proxy measure |
| 10 | Shen 2015 | Health Aff | Retrospective cohort | California AMI patients | Ambulance diversion exposure | Reduced access to cardiac technology; **↑1-year mortality** | — |
| 11 | Zeng 2021 | ASCE JTEP-A | MIP + case study | Shenzhen, China | Prehospital screening + lane preclearing + semi-soft time windows | Quantified value of multistakeholder participation | **Hospital choice / capacity not modelled** |
| 12 | Kohneh 2024 | ASCE JTEP-A | Bi-objective model + NSGA-II/PSO | Simulated two-way roadway | Connected-vehicle contraflow for ERVs | Faster than CPLEX; improved ERV travel time & safety | Not hospital-integrated |
| 13 | EMVLight 2022 | arXiv / ACM | MARL (MA2C) + dynamic Dijkstra | Synthetic grids, Manhattan, Hangzhou | Joint EMV routing + signal pre-emption | **−42.6%** EMV travel time; **−23.5%** all-vehicle travel time (baseline preemption **+10%** general delay) | No hospital-capacity dimension; simulation only |
| 14 | Decision Transformer 2026 | arXiv | Offline RL | 4×4 to 8×8 grids | Offline preemption policy | **−37.7%** vs. fixed-timing preemption; no unsafe online exploration | Synthetic grids; fixed routes |
| 15 | Mu 2018 | Math Probl Eng | Multi-objective programme | Simulation | Route-based signal preemption | EV delay ↓ and system throughput ↑ | Isolated intersections |
| 16 | Almalki 2025 | Discov Artif Intell | Two-stage algorithm | OSM road network | Time-based Dijkstra + service/capability verification | Joint proximity+capability selection; fallback to alternatives | Static capacity; no traffic dynamics |
| 17 | Tedesco 2024 | J Ind Prod Eng | DES | Milan EMS | Hospital-selection logics incl. ED state | Best logic: GPT **7.28 h → 6.08 h (~20%)** vs. proximity baseline | No routing; no notification |
| 18 | (ACM) 2023 | ACM | DES (AnyLogic) | EMS network | Assignment policies incl. Time-To-Provider | **Minimising TTP** best criterion; ↓ throughput & overcrowding | Simulation only |
| 19 | Caicedo Rolón 2021 | Literature review | Systematic review | EMS hospital-selection literature | Criteria frequency | Closeness **83.33%**, capacity **62.5%**, shortest queue **45.83%**; DES/queuing/MILP dominant | Calls for tech + actor participation |
| 19a | Alrawashdeh 2024 | Prehosp Disaster Med | PRISMA-ScR scoping review | **164 studies** (125 clinical, 39 operational) | ML in EMS | Operational tasks: allocation n=21, **route optimization n=5**; median AUC 96.1%, accuracy 90.0% | **Route optimization under-studied**; standardisation needed |
| 20 | Gibson 2025 | JMIR mHealth | Systematic review + meta-analysis | 21 studies, **3,267 patients**, 12 countries | Smartphone STEMI coordination apps | **D2B −19.11 min (95% CI −26.22 to −12.00, P<.01)**, I²=89%; mortality RD −0.03 (P=.10) | Mortality benefit unproven; false-activation not poolable |
| 21 | Bladin 2022 | BMJ Open | Quasi-experimental feasibility | Regional Victoria, AU; 604 stroke, 247 STEMI | Pulsara multidisciplinary app | Off-stretcher: stroke **11 vs 19 min** (p=0.0001); STEMI **14 vs 19 min** (p=0.0014); ED emergency triage 47%→92% | DTN/DTB similar; uptake variable |
| 22 | Hsieh 2016 | Scand J Trauma Resusc | Retrospective observational | Taipei, 9 hospitals, 928 patients | Prehospital stroke notification | Door-to-CT **13 vs 19 min** (p<0.001); sensitivity 78.3%, PPV 78.2% | DTN 63 vs 68 min (p=0.138, ns) |
| 23 | AHA/ASA 2026 | Stroke | Guideline (GWTG-Stroke) | **371,988** EMS-transported patients | EMS prenotification | Door-to-imaging **26 vs 31 min**; DTN **78 vs 80 min**; on-time tPA **82.8% vs 79.2%** | Observational basis |
| 24 | APOT 2024 | JAMA Netw Open | Retrospective cohort | California, **5,913,399** offloads, 34 agencies | Ambulance patient offload time | Mean APOT-1 **42.8 min** (SD 27.3); median 28.9 min; **16/34 agencies >30-min standard** | Hospital factors, not routing |
| 25 | Zhao 2025 | ISCRAM | Model comparison | Nova Scotia EMS | 3× ANN vs. Google Maps vs. constant-speed | ANN best for simulation/planning; Google Maps for live dispatch | Complementary, not competing |
| 26/27 | FHIR SANER / CDC NHSN | Standards | Implementation guide | — | Machine-readable bed capacity | 78 data elements; ~15-min automated reporting; FHIR-aligned | Real-world EHR exposure uneven |
| 28 | IDEAL 2026 | arXiv | Bilevel representation learning + robust dispatch | Dispatch records | Contextual travel-time uncertainty sets | Learns edge TTs from **unobserved routes**; provable convergence | Not yet deployed |
| 30 | Opt-Augmented ML 2025 | arXiv | Structured learning + simulation | San Francisco 911 data | Dispatch + redeployment policies | Mean response time **−up to 30%** vs. online benchmarks | Simulation |
| 32 | Boutilier & Chan | — | Robust optimization | Dhaka, Bangladesh | Location + routing under TT uncertainty | Median response **−10–18%** overall; **−24–35%** rush hour | LMIC-specific fleet mix |
| 33 | Turnaround ML 2021 | PubMed | Two-stage ML forecasting | 3 hospitals | Ambulance turnaround time | Accuracy **90.16%, 97.02%, 93.09%** | Single-country fire-EMS system |
| 36 | SWIFTAID 2026 | Regional | ML pipeline + field test | Bengaluru, **287 trips** | XGBoost travel-time prediction | **R²=0.84**; MAE 2.9 min; **18–22%** peak-hour response reduction | Small n; regional venue |
