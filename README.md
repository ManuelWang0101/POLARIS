# POLARIS

**P**layer-conditional p**ol**icy estimation with **A**ugmented **R**elational and **I**ndividual **S**tate

A research project on how individual football players make decisions — and how those decisions change as the body tires.

🔗 **Build log:** [manuelwang0101.github.io](https://manuelwang0101.github.io)
📍 **Status:** in development · data reconnaissance complete · state representation in progress

---

## What POLARIS is

Most computational models of football decision-making answer a population question: *given this situation, what does a player typically do?* POLARIS asks a narrower and more personal one.

For an individual player *i*, POLARIS estimates a **conditional decision policy**

```
π_i(a | s)  —  the probability that player i takes action a, given the state s they are in
```

The novelty is in two parts of that expression that existing work tends to flatten:

1. **The player is named, not anonymous.** Identity is a first-class variable. Two players in the same situation are allowed to have different policies, and the model is built to recover those differences.
2. **The state includes the body.** Alongside the usual tactical context (where the ball is, the scoreline, the phase of play), the state *s* carries a measure of **within-match physical fatigue** derived from GPS data. Almost no public decision model conditions on this.

The wager is simple to state: a tired player decides differently from a fresh one, and a model blind to fatigue cannot see it.

---

## The gap it targets

| | Per-player identity | Physical / fatigue state |
|---|:---:|:---:|
| Typical decision models (e.g. EDMS, Ide et al. 2025) | shared / anonymous policy | not modelled |
| **POLARIS** | **individual policies π_i** | **within-match fatigue as state** |

EDMS and similar frameworks model a single shared policy over discretised actions and treat players as physically constant for ninety minutes. POLARIS keeps the discrete-action decision framing but adds individual identity and a fatigue axis on top of it. Related cohort-level physical-performance work (e.g. HPI'25) studies the body but not the decision; POLARIS connects the two.

---

## The data

POLARIS is built on the **J1 League 2024** season, combining two coordinated sources — a pairing that is rarely available together in public data.

- **Hudl StatsBomb event data** — every on-ball action, player-resolved and time-stamped, with pitch coordinates, pressure flags, play patterns, and a pre-computed action value (OBV). ~1.24M events across 376 of 380 matches.
- **Hudl Pro GPS physical data** — 14 physical metrics per player per match (total distance, high-intensity distance, sprint counts, accelerations, max speed, …), decomposed into **six 15-minute bins**. 378 matches, 581 players.

The two are joined through player- and match-level ID mappings (StatsBomb ↔ Wyscout).

### What the data does *not* contain — and why it matters

POLARIS is deliberate about its limits:

- **No continuous tracking (no 360).** For ~99% of events, only the ball-carrier's position is known. Full multi-player spatial state exists only on the ~9,900 shot freeze-frames (≈8 players each). **Any model here must work without continuous off-ball positions.**
- **Fatigue resolves to 15 minutes, not seconds.** The physical data answers "load entering the 46–60′ window," never "fatigue at 47′12″." This is the time-resolution ceiling for every fatigue-conditioned claim in the project, and the modelling is scoped to that register.

These constraints are not footnotes — they shape what POLARIS chooses to model.

---

## The state representation

A "decision moment" is decomposed into three layers, all derivable from the data above:

- **Event context** — ball location, play pattern, under-pressure flag, position, possession step, and the OBV value of the state.
- **Macro game-state** — minute, period, scoreline (accumulated from events), manpower (substitutions / red cards), match-week.
- **Physical state** — *the project's core contribution.* Fatigue is operationalised as **cumulative high-intensity load entering the current bin** (a lagged, backward-looking quantity, expressed relative to a player's own baseline), chosen specifically to avoid leakage and to separate genuine fatigue from tactical effort.

---

## Method (planned)

Recovering an individual fatigue–decision relationship is impossible from one player in one match — the signal is buried in the noise of a few dozen actions. POLARIS therefore pools information across hundreds of players and matches using **hierarchical Bayesian models** that borrow statistical strength between players who share a position, a role, or a fatigue regime, while still letting individual policies separate where the data support it.

---

## Roadmap

This repository's blog doubles as a public build log. Each step ships as a post.

- [x] **Step 1 — Data reconnaissance.** Inventory the three sources, confirm joins, document what is and isn't there. *(complete · [Post 1](https://manuelwang0101.github.io))*
- [ ] **Step 2 — State extractor.** Turn events + macro + physical into a state vector per decision moment. *(in progress)*
- [ ] **Step 3 — Action space & policy estimation.** Define the discrete action set and fit per-player policies.
- [ ] **Step 4 — Fatigue-conditioned analysis.** Test whether and how policies shift along the fatigue axis.
- [ ] **Step 5 — Write-up.**

---

## Repository status & layout

This repository is the home of the POLARIS project — its methods, roadmap, and (as it matures) the analysis code. A narrative **build log** runs in parallel as a blog: [manuelwang0101.github.io](https://manuelwang0101.github.io).

The analysis code (state extraction, policy estimation) is being migrated here from a research-cluster workspace; see the roadmap above for what exists today. Planned layout:

```
data/        data inventory & join specifications (no proprietary data committed)
src/         state extractor, action-space definitions, models
notebooks/   reconnaissance & exploratory analysis
docs/        methodology notes, build-log drafts
```

If you want to cite or follow the project, the blog is the most up-to-date surface.

## Data & attribution

POLARIS is an **independent research project**. It uses Hudl StatsBomb J1 League 2024 event data and Hudl Pro physical data under their respective terms; no proprietary data is redistributed in this repository. All analysis, framing, and conclusions are the author's own.

## Author

**Mufan Wang** — PhD researcher in biostatistics, working on causal inference and football analytics.
GitHub: [@ManuelWang0101](https://github.com/ManuelWang0101)
