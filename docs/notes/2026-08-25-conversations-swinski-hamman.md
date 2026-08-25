# Conversations: J.P. Swinski and Joe Hamman

**2026-08-25 · Responsible Gen-AI for NASA Earthdata, Seattle** — reported by Jim.

## J.P. Swinski (SlideRule) — the gap, validated

Asked about the gap directly. **He agreed.**

He described his own role as a developer whose job is to take the scientific ideas of
**Tyler Sutterley** and **David Shean** and deliver them either as:

1. **best-practice default options**, or
2. **a system that asks questions that lead to good choices**

Those are our two mechanisms, named independently and without prompting — defaults baked
into the access recipe, and the disagreement-driven interview. That is the strongest
external validation the design has had.

He confirmed SlideRule is a **technology push aimed at reducing the scientific insight
required from ICESat-2 experts, so that the data can be used more widely.**

### The tacit knowledge envelope

Jim proposed the generalisation and J.P. did not push back: **this pattern is general
across the NASA Earthdata ecosystem.** Every mission has experts holding a great deal of
tacit knowledge. Researchers *adjacent* to the mission — a lab over, a collaborator, a
workshop attendee — emerge as flexible users. Researchers *outside* that envelope are
much less able to get value from the data.

**Openness is not access.** The archive being public does nothing for someone outside
the envelope. So:

> The target population is everyone outside the envelope, and the measure of success is
> how far the envelope extends **without adding people to the mission team.**

SlideRule extends it for ICESat-2 by encoding expert judgement into defaults. We are
attempting the same move one layer up — across missions, at the point of *choosing*
rather than *processing*.

## Joe Hamman (Earthmover) — applications are the missing half

### Scientists document science, not applications

Dataset producers — often scientists under an employment relationship — are **very good
at providing the scientific information** about their data. That is the operand material
in standard metadata: resolution, uncertainty, provenance, algorithm.

They are **not verbose about applications.** *How a land manager could use this dataset*
is an application, and it is nobody's job to write it down.

This splits our fitness block cleanly in two, and we had been conflating them:

| | Producer-side | Application-side |
|---|---|---|
| What it says | what the data **is** and what it **can** do | what people **did** with it |
| Who knows it | the mission team | third parties, scattered |
| Where it lives | ATBDs, UMM-C, ISO fields | blog posts, papers, notebooks, forum threads |
| Authority | specification | precedent |

### The incentive nobody has named yet

Joe: data providers **want to know when their datasets have impact beyond mission
scope.** Datasets answering scientific questions inside the mission's own playground are
fine — but when they break out into industry, local government or NGO use, that is
**very interesting to NASA leadership.**

This is the adoption argument the project has been missing. **The applications record is
simultaneously the librarian's context and NASA's impact evidence.** One artifact, two
customers, and the second one holds the budget.

### Joe on ISO 19115: they are not inspired by it

Direct pushback on the ontology-slot argument. Earthmover is **not inspired by ISO 19115
or similar best-practice metadata structures** — they are "often making it up." Their
supported fields include an **applications field** capturing usage scenarios that go
beyond the science facts.

Examples he shared (Arraylake Marketplace — require login, contents not readable from
here; the team should open them):

- <https://app.earthmover.io/marketplace/6a19bcfe9aa6e97720a2fad2#applications> (ERA5)
- <https://app.earthmover.io/marketplace/69b303f4204c4306b35fbeff#applications>
- <https://app.earthmover.io/marketplace/69eb48e26835834c6a930dd4#use-cases>

**Taking that seriously against our earlier position.** Both can be true, and the
reconciliation is about *shape*, not legitimacy:

- ISO/UMM-C remains the right **citation** for producer-side facts — resolution, quality
  conventions, cautions. That argument stands, and it is what makes the fitness extension
  a small delta rather than a rival vocabulary.
- ISO is the wrong **shape** for applications. `MD_Usage.specificUsage` and UMM-C
  `Purpose` are single-valued, producer-authored, write-once fields describing *intended*
  use. Applications are append-only, multi-author, third-party, and discovered after the
  fact. That is a different data structure, and Joe's team built one because the standard
  did not offer it.

## The idea: a growing applications record

From the discussion between Aimee, Joe and Jim. Use frontier models to **enrich an
applications field as a growing record inside a dataset's metadata.** When a land manager
uses SMAP for something useful, their blog post gets captured into the SMAP metadata
layer. Over time these accumulate into context the librarian can draw on.

**This is a strong idea and it is explicitly not this week.** Recorded so it survives.

### Why it is strong

- It attacks the authoring bottleneck from the opposite side. Instead of extracting
  fitness knowledge from ATBDs (hard — that is R1), harvest it from **use in the wild**.
- It is self-reinforcing: more use → more applications → better guidance → more use.
- It serves the incentive above, which is how it gets funded.
- **A hook already exists.** The NASA MCP's `get_citations` links papers to collections
  by DOI, in both directions. That is a partial, already-populated applications corpus
  sitting in CMR today.

### Three risks that must be designed around

1. **It re-introduces popularity bias — the exact thing our two-channel argument said the
   catalog avoids.** Applications accumulate for well-used datasets; a mission launched
   last year has none. An applications-weighted librarian would prefer the famous product
   again. This is the sharpest critique and it must be answered before the field is
   allowed to influence ranking.
2. **Applications record what was done, not whether it was appropriate.** If fifty land
   managers used IMERG at county scale, the applications record *endorses the error*.
   Popularity of a use is not validity of a use. An applications corpus can encode and
   amplify a collective mistake.
3. **It industrialises the untrusted-content leg of the trifecta.** Harvesting blog posts
   into a metadata layer that an agent reads is exactly Joe's own warning from slide 32 —
   *a poisoned README becomes an instruction* — at scale, and with an economic incentive
   for people to get their content into it.

### The design answer: two fields, not one

Keep them structurally separate, because they have **opposite epistemics**:

| Field | Source | Authority | Role in the desk |
|---|---|---|---|
| `validated_uses[]` | producer or expert, vouched | specification | **may support a recommendation** |
| `observed_uses[]` | harvested from the wild, with citation | precedent | **may supply context only** |

And the rule that keeps risk 2 contained:

> `observed_uses[]` may never override a fitness axis. If the record says 9 km cannot
> resolve a field, fifty blog posts doing it anyway do not make it fit — they make it a
> documented, widespread error, which is *also* worth telling the user.

That last clause may be the most valuable thing the applications record produces:
**detecting widespread misuse.** A dataset whose observed uses systematically violate its
own fitness envelope is a finding NASA would want, and only a system holding both halves
can see it.

## Actions

- Corrected the objective's audience — it said "health researchers," which was wrong.
  Now Earth science researchers, land and resource managers, and users outside a
  mission's expert envelope.
- Added the tacit-knowledge-envelope framing to `objective.md`, crediting J.P.
- **Not this week.** The applications record is a phase-two direction. Do not let it into
  the week's scope.
- Ask Joe for read access to the marketplace examples, or a schema dump of the
  applications field.
- Cheap follow-up whenever we get there: `get_citations` already gives a free
  `observed_uses[]` seed for any collection.

---

## The Earthmover applications field, examined

Jim pasted the ERA5 marketplace page. The shape is now clear.

### The schema Joe is actually using

`Applications` is a top-level section containing `Use Cases`, a list of ~10 entries.
Every entry has the same three-slot form:

```
<domain>: <task> using <specific variables>
```

Examples, verbatim:

> **Renewable energy assessment:** Wind resource analysis using 10 m and 100 m wind
> components; solar resource evaluation using radiation variables
>
> **Agriculture and food security:** Soil moisture, multi-layer soil temperature, and
> precipitation analysis for crop modeling and yield prediction
>
> **Air quality modeling:** Boundary layer height and meteorological inputs for pollution
> dispersion models

**The load-bearing detail: each use case names the variables it uses.** It is not "good
for agriculture" — it is *these variables, for this task*. That binds application to
variable, which is machine-actionable and better than anything in UMM-C.

### It is a third epistemic category we had not named

Our earlier design had two fields. Joe's is neither:

| | Source | Evidence | Example |
|---|---|---|---|
| `validated_uses[]` | producer/expert, vouched | a validation study | "validated for regional SM assimilation" |
| **`suggested_uses[]`** | **curator, editorial** | **none — plausibility** | **Joe's field** |
| `observed_uses[]` | third party, in the wild | a citation | "used in this paper / blog post" |

Joe's field carries the **curator's authority without evidence**. That is not a flaw —
it is onboarding copy, and it is genuinely useful. But it is a distinct trust level and
must be modelled as one.

### The finding: the ERA5 page demonstrates our gap, in the best commercial example available

> **Agriculture and food security:** Soil moisture, multi-layer soil temperature, and
> precipitation analysis for crop modeling and yield prediction

ERA5 is **31 km (0.25°)** — roughly 961 km² per pixel. That entry is entirely correct for
*regional and national* food-security work, which is a real and important application.
It is entirely wrong for a farmer's 40 ha field, which is 0.4 km² — **one two-thousandth
of a pixel.**

The page never connects the two. Resolution is stated in *Data Characteristics*; the use
cases are three sections away and carry **no scale qualifier**. A land manager reading
"agriculture and food security… crop modeling and yield prediction" has been given a true
statement that will lead them somewhere false.

This is fair rather than a gotcha — Joe said outright they are "making it up," and adding
an applications field at all is the right direction, well beyond standard metadata. But
it shows the shape of the gap precisely:

> **An application says what you can do. Fitness says at what scale you can do it.
> Neither is sufficient alone**, and an application without a scale qualifier is exactly
> the trap our project exists to catch.

### What the page has, and what it is missing

**Present and worth stealing:**

- **"Query patterns that work well"** — fitness for *access pattern*, bound to the dual
  chunking (spatial "pancake" 1×721×1440 vs temporal 8736×12×12). Not just "it is
  chunked" but *which query each layout serves*. That is our `cloud_ready` layer made
  actionable, and it is the best example of it I have seen.
- **Example code per use case.** The wind-energy snippet maps to the renewable-energy use
  case. Applications and recipes are cross-linked — that is Keenan's layer bound to Joe's.
- **A verification tool** checking the data against the original ECMWF CDS. Adjacent to
  our CI-verified-recipe idea, applied to the data rather than the snippet.
- Explicit latency and update cadence; thorough provenance with DOIs and a cutover date;
  a changelog.

**Absent — and it is our whole list:**

- No resolution-to-question guidance. `0.25°` is stated but never connected to a use.
- **No cautions, no discouraged uses.** Nothing says what ERA5 is bad for.
- No uncertainty, though reanalysis has well-known biases.
- No minimum meaningful area.
- QC flags exist (`status/` group, per-hour flags) but no convention for using them.

The most sophisticated commercial dataset page available is excellent on *what it is*,
*how to reach it*, and *what it is for* — and silent on *when not to use it*.

### The small proposal this suggests

Joe's entry format needs one more slot:

```
  now:  <domain>: <task> using <variables>
  ours: <domain>: <task> using <variables>  at <scale>  ·  not for <scale>
```

Concretely:

> Agriculture and food security: soil moisture, multi-layer soil temperature and
> precipitation for crop modelling and yield prediction **at regional to national scale
> (≥ 100 km). Not for field or county scale — one pixel is ~961 km².**

That is a small, well-motivated delta to a field that already exists in a shipping
product, which is a far better proposal target than a greenfield vocabulary. **Take it to
Joe** — he owns the field, he is in the building, and the addition is one clause per
entry.

### Consequences for our design

- Model **three** use categories, not two: `validated_uses[]`, `suggested_uses[]`,
  `observed_uses[]`. Different authority, different role in the desk's reasoning.
- **Bind uses to variables**, as Joe does. A use case that names its variables can be
  checked against what the user needs; one that names only a domain cannot.
- **Every use entry carries a scale qualifier**, or it is not admissible to the desk.
  This is the rule that keeps a suggested use from becoming a recommendation.
- Steal "query patterns that work well" as the presentation form for `cloud_ready`.
