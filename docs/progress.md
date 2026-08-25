# Progress Log

## [2026-08-25]
### What I did
- Read the 2026-08-24 architecture memo and reconciled `docs/` against it —
  `objective.md` rewritten from template placeholder, `decisions.md` and
  `progress.md` restored with real content.

### What I learned
- The repo was a scaffold only: `src/`, `data/` and `outputs/` hold nothing but
  README/gitkeep files. All project thinking so far lives in the memo artifact, not
  in git.
- The shared link on the artifact serves collaborators a **pinned earlier version**,
  not the current one. Anyone working from that link may be a revision behind.

### Next steps
- [ ] Assign an owner for D1 (region, question, persona) — it gates fixtures, gold
      set and demo script
- [ ] Test R1 on three datasets: can fitness knowledge be extracted from DAAC docs,
      or must it be hand-curated?
- [ ] Hand-write three Dataset Readiness Records so the librarian has something to
      develop against
- [ ] Write the readiness record JSON Schema, with `{value, source, confidence}`
      wherever a value exists

### Blockers
- D1 unassigned — fixtures, gold set and demo script can't start without it.

---

## [2026-08-24]
### What I did
- Workshop day at UW eScience Institute (Responsible Generative AI). Converged three
  separate proposals into one architecture with Keenan, Aimee, Sarah, Julia and Rajat.
- Drafted the architecture memo:
  https://claude.ai/code/artifact/37a5c8c4-ee37-4c90-a97b-f9cdaa7f9dd5
- Divided the work: technical services / readiness assessment (Aimee), reference desk
  / librarian agent (James), circulation / access recipe (Keenan).

### What I learned
- **The missing row.** NASA's *Which data tool is right for you?* (v2, July 2025)
  scores ten tools across nineteen rows, every one asking what the tool can do. None
  asks whether the data is fit for the asking. That absent row is the project.
- **The chart hands us three things** in the agency's own voice: the premise, a schema
  someone already designed, and an evaluation baseline the librarian can be scored
  against. Its tri-state encoding is worth borrowing for the compatibility report —
  the audience already reads it.
- **Only 1 of 10 tools is a clean "no programming skills required"** (CMR Search API
  is the yes, VEDA Dashboard partial). The tools that reach non-experts are the ones
  that ask least of them.
- **The shared record is the project.** Three skills spanning production, selection
  and use — fitness knowledge produced upstream and carried downstream, where today it
  lives in PDFs and in the heads of people who have used a product for a decade. That
  chain is the responsible-AI claim, and it's stronger than any one agent being clever.
- **Two skills, one record — never two records.** Reconciling documents at query time
  is work the librarian should never do.
- **Catalog beats documentation as a discoverability channel.** The open-web channel is
  popularity-biased and frozen at training cutoff; the catalog is current by
  construction and conformance is binary. The catalog is the fix a small DAAC can
  afford, and the one that stays true after the next model is trained.
- **CI is what makes a recipe trustworthy** rather than decorative. A recipe that
  passed an hour ago is a fact; a recipe in a PDF is a hope.
- **The interview and the qualified refusal are the contribution** — ordinary behaviour
  for a librarian, almost unknown in recommendation systems.

### Next steps
- [ ] Settle the seven open decisions — see [decisions.md](decisions.md)
- [ ] Test R1 (can fitness knowledge be extracted from DAAC docs?)

### Blockers
- None recorded.
