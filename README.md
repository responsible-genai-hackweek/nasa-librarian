# nasa-librarian

*A librarian agent for NASA Earth data.* The agent uses earthdata_mcp and earthaccess to help users find, access, and analyze NASA data more effectively. 

Started at the Responsible Gen-AI for NASA Earthdata hackweek, UW eScience Institute,
Seattle, August 2026. 

---

## Start here

**New to the project?** Read [docs/objective.md](docs/objective.md) — the gap, and what
we are building this week. Ten minutes.

**Coming to the working session?** Jump straight to
[This week's scope](docs/objective.md#this-weeks-scope) — six deliverables with owners,
and the cut list.

**The one thing blocking everything:** the region, question and persona are still
unassigned ([decisions.md](docs/decisions.md) D1). Fixtures, gold set and demo script all
descend from it.

---

## The gap in one line

NASA can tell you what data exists and how to download it. **Nothing tells you whether it
will answer your question.**

```
earthdata-mcp  ──▶   [ the choice ]   ──▶  earthaccess
"287 collections        unmade            "here are
 mention your topic"                       the bytes"
```

Both ends are built, hosted and free. The middle is empty, because the choice is a
*comparison* and nobody stores its two operands.

---

## Documentation

| File | Purpose |
|---|---|
| [docs/objective.md](docs/objective.md) | **Start here.** The gap, this week's scope, and how value gets validated |
| [docs/design.md](docs/design.md) | The full analysis — the ten fitness axes, the Dataset Readiness Record, the reference-desk design. Exceeds what week one builds, deliberately |
| [docs/decisions.md](docs/decisions.md) | Ten open decisions and two open risks, ordered by when they stop being free to answer |
| [docs/notes/](docs/notes/) | Day-by-day findings — talks, live probes against NASA's servers, prior-art assessments |
| [docs/progress.md](docs/progress.md) | Session log |

**Readable version** (a snapshot, not always current):
[Where the Catalog Stops](https://claude.ai/code/artifact/3f497cf9-1f21-46d3-937f-774c2291088f)

The original 2026-08-24 architecture memo is superseded; this repo is canonical.

## Status

Active — started 2026-08-24, target 2026-08-31.
