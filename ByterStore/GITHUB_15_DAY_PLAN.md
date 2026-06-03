# ByterStore — 15-Day GitHub Upload Plan

**Repository:** [https://github.com/phoenix00612/ByterStore](https://github.com/phoenix00612/ByterStore)  
**Start date:** June 1, 2026  
**End date:** June 15, 2026  
**Goal:** Publish ByterStore in steady, reviewable daily commits — one focused theme per day, green contribution graph, portfolio-ready repo.

---

## How to use this plan

Each day has:

1. **Focus** — what you ship that day  
2. **Files** — what to add or change  
3. **Commit message** — copy-paste ready  
4. **Commands** — exact git steps  
5. **Done when** — checklist before moving on  

Run all commands from the **repo root** (`ByterStore/` parent — where `.git` lives):

```bash
cd /Users/vaibhav/ByterStore
```

**Rules for the 15 days**

- One main commit per day (small fix commits same day are OK).  
- Never commit `.DS_Store`, `.cph/`, `data/`, build artifacts, or secrets.  
- Write commit messages in present tense, one line summary + optional body.  
- Push every day: `git push origin main`  
- Update the checklist at the bottom of this file as you go.

---

## Current baseline (before Day 1)

Already on GitHub (`main`):

- Core engine: `segment`, `segment_mgr`, `storage_engine`, `bloomfilter`, `thread_pool`  
- Headers: Robin Hood map, hash, utils, search index (header-only)  
- `main.cpp` (Crow HTTP server), `db.cpp`, `Makefile`, `config/db.conf`

Still local / not on GitHub yet:

- Root `README.md`, `Proj.md`, `.gitignore`  
- `DB/README.md`, `DB/.gitignore`  
- This plan file  

The schedule below turns the rest into **15 intentional days of progress**, not one giant dump.

---

## Day 1 — June 1, 2026 · Project identity & roadmap

**Focus:** First impression — README, vision doc, gitignore, and this plan.

| Action | Path |
|--------|------|
| Add | `ByterStore/README.md` |
| Add | `ByterStore/Proj.md` |
| Add | `ByterStore/.gitignore` |
| Add | `ByterStore/DB/.gitignore` |
| Add | `ByterStore/GITHUB_15_DAY_PLAN.md` |

**Commit message:**

```
docs: add README, project vision, gitignore, and 15-day upload plan
```

**Commands:**

```bash
git add ByterStore/README.md ByterStore/Proj.md ByterStore/.gitignore \
        ByterStore/DB/.gitignore ByterStore/GITHUB_15_DAY_PLAN.md
git commit -m "$(cat <<'EOF'
docs: add README, project vision, gitignore, and 15-day upload plan

EOF
)"
git push origin main
```

**Done when:**

- [ ] README renders on GitHub with features + API table  
- [ ] `.gitignore` excludes `data/`, `build/`, `.DS_Store`  
- [ ] Plan file visible on repo  

---

## Day 2 — June 2 · Architecture documentation

**Focus:** Explain how the engine works (for recruiters and contributors).

| Action | Path |
|--------|------|
| Add | `ByterStore/docs/ARCHITECTURE.md` |
| Add | `ByterStore/docs/DATA_FORMAT.md` |

**Content to cover:**

- Layer diagram: HTTP → StorageEngine → SegmentMgr → Segment  
- On-disk record layout (header, key, value, CRC, tombstone flag)  
- `.kv` / `.idx` / `.bf` file roles  
- FNV-1a + Robin Hood + Bloom filter flow  

**Commit message:**

```
docs: document storage architecture and on-disk record format
```

**Done when:**

- [ ] Architecture doc includes a simple ASCII or Mermaid diagram  
- [ ] Record byte layout is documented  

---

## Day 3 — June 3 · Build & install guide

**Focus:** Anyone can clone and build on macOS/Linux.

| Action | Path |
|--------|------|
| Add | `ByterStore/docs/BUILD.md` |
| Add | `ByterStore/scripts/install_deps.sh` (Crow, fmt, nlohmann-json hints) |
| Edit | `ByterStore/DB/src/Makefile` — rename target `byterstore` to match README |

**Commit message:**

```
build: add BUILD guide, deps script, and align binary name with README
```

**Done when:**

- [ ] `make` in `DB/src` produces `byterstore` (or documented alias)  
- [ ] BUILD.md lists exact `apt` / `brew` packages  

---

## Day 4 — June 4 · Configuration wiring

**Focus:** Make `db.conf` actually drive runtime behavior.

| Action | Path |
|--------|------|
| Edit | `ByterStore/DB/src/config.cpp` — pass bloom + thread settings |
| Edit | `ByterStore/DB/src/segment.cpp` — use config bloom size/hashes, not hardcoded `8*1024, 4` |
| Edit | `ByterStore/DB/include/kv/segment.hpp` — constructor accepts bloom params |
| Edit | `ByterStore/DB/src/segment_mgr.cpp` — pass config through |

**Commit message:**

```
fix: wire bloom filter and segment settings from db.conf
```

**Done when:**

- [ ] Changing `bloom_bits_kb` in config changes `.bf` file size  
- [ ] No hardcoded `8*1024, 4` in `segment.cpp`  

---

## Day 5 — June 5 · Segment rotation fix

**Focus:** Correct segment rollover when files reach `segment_size_mb`.

| Action | Path |
|--------|------|
| Edit | `ByterStore/DB/src/segment_mgr.cpp` — rotate on **file size**, not last record offset |
| Edit | `ByterStore/DB/include/kv/segment.hpp` — add `size()` or `fileSize()` helper |
| Add | `ByterStore/docs/SEGMENTS.md` — explain rotation policy |

**Commit message:**

```
fix: rotate segments based on file size instead of record offset
```

**Done when:**

- [ ] Appending until ~64 MB creates `segment_2.kv`  
- [ ] Lookups still work across closed + current segments  

---

## Day 6 — June 6 · REST API alignment

**Focus:** HTTP routes match README; cleaner JSON errors.

| Action | Path |
|--------|------|
| Edit | `ByterStore/DB/src/main.cpp` — add `POST /{model}/{key}` route from README |
| Edit | `ByterStore/README.md` — document `?search=` query param (already in code) |
| Add | `ByterStore/docs/API.md` — full endpoint reference with curl examples |

**Commit message:**

```
feat: align REST routes with README and add API reference
```

**Done when:**

- [ ] `POST /users/alice` with JSON body works  
- [ ] API.md has curl one-liners for every route  

---

## Day 7 — June 7 · Concurrency & thread pool

**Focus:** Use the existing `ThreadPool` for background index/bloom saves.

| Action | Path |
|--------|------|
| Edit | `ByterStore/DB/include/kv/storage_engine.hpp` — optional `ThreadPool` member |
| Edit | `ByterStore/DB/src/storage_engine.cpp` — enqueue `saveIndex`/`saveBloom` after append |
| Edit | `ByterStore/DB/src/main.cpp` — construct pool from `config.thread_pool_sz` |

**Commit message:**

```
feat: integrate thread pool for async segment index persistence
```

**Done when:**

- [ ] `thread_pool_size` in config controls worker count  
- [ ] Writes still correct under concurrent GET/POST  

---

## Day 8 — June 8 · Search index over HTTP

**Focus:** Expose inverted-index search from `search_index.hpp`.

| Action | Path |
|--------|------|
| Edit | `ByterStore/DB/src/main.cpp` — `GET /{model}/search?q=iphone` |
| Add | `ByterStore/docs/SEARCH.md` — tokenization, AND queries, limitations |

**Commit message:**

```
feat: add HTTP search endpoint backed by SearchIndex
```

**Done when:**

- [ ] Indexing happens on POST  
- [ ] Search returns matching document IDs or keys  

---

## Day 9 — June 9 · Developer CLI entry point

**Focus:** Fix or replace broken `db.cpp` benchmark.

| Action | Path |
|--------|------|
| Edit | `ByterStore/DB/src/db.cpp` — remove missing `user_model` / `product_model` includes |
| Add | `ByterStore/DB/src/cli.cpp` — simple `put/get/delete/list` without HTTP |
| Edit | `ByterStore/DB/src/Makefile` — target `cli` separate from server |

**Commit message:**

```
feat: add standalone CLI for local testing without Crow
```

**Done when:**

- [ ] `./cli put key val` and `./cli get key` work  
- [ ] `db.cpp` compiles or is removed from build  

---

## Day 10 — June 10 · Tests

**Focus:** Minimal automated tests for core engine.

| Action | Path |
|--------|------|
| Add | `ByterStore/DB/tests/test_bloom.cpp` |
| Add | `ByterStore/DB/tests/test_robin_hood.cpp` |
| Add | `ByterStore/DB/tests/test_storage.cpp` |
| Edit | `ByterStore/DB/src/Makefile` — `make test` target |

**Commit message:**

```
test: add unit tests for bloom filter, hash map, and storage engine
```

**Done when:**

- [ ] `make test` passes locally  
- [ ] Tests cover put/get/erase/tombstone  

---

## Day 11 — June 11 · CI pipeline

**Focus:** GitHub Actions build on every push.

| Action | Path |
|--------|------|
| Add | `.github/workflows/build.yml` — install deps, compile, run tests |

**Commit message:**

```
ci: add GitHub Actions workflow for build and tests
```

**Done when:**

- [ ] Green check on `main` in Actions tab  
- [ ] Workflow uses `g++-11` or `clang++` with C++17  

---

## Day 12 — June 12 · Docker & quick run

**Focus:** One-command demo for reviewers.

| Action | Path |
|--------|------|
| Add | `ByterStore/Dockerfile` |
| Add | `ByterStore/docker-compose.yml` |
| Edit | `ByterStore/README.md` — Docker section |

**Commit message:**

```
chore: add Docker and docker-compose for one-command demo
```

**Done when:**

- [ ] `docker compose up` serves port 8008  
- [ ] Data volume mounts to `./data`  

---

## Day 13 — June 13 · Examples & sample data

**Focus:** Show real usage patterns.

| Action | Path |
|--------|------|
| Add | `ByterStore/examples/users.json` |
| Add | `ByterStore/examples/products.json` |
| Add | `ByterStore/examples/load_examples.sh` — POST sample data via curl |
| Add | `ByterStore/docs/EXAMPLES.md` |

**Commit message:**

```
docs: add sample JSON models and curl load script
```

**Done when:**

- [ ] Script loads users + products into running server  
- [ ] EXAMPLES.md walks through GET/POST/DELETE  

---

## Day 14 — June 14 · Hardening & cleanup

**Focus:** Production-minded polish.

| Action | Path |
|--------|------|
| Edit | `storage_engine.cpp` — lock `get_all()` with shared lock or document race |
| Edit | `main.cpp` — consistent JSON error bodies `{ "error": "..." }` |
| Add | `LICENSE` (MIT recommended) |
| Add | `CONTRIBUTING.md` |
| Remove | dead code, stray `.DS_Store` from history if any |

**Commit message:**

```
chore: harden concurrency, standardize errors, add LICENSE and CONTRIBUTING
```

**Done when:**

- [ ] No `.DS_Store` in tracked files  
- [ ] LICENSE visible on GitHub repo page  

---

## Day 15 — June 15 · v1.0.0 release

**Focus:** Ship version 1.0 — tag, release notes, polished README.

| Action | Path |
|--------|------|
| Edit | `ByterStore/README.md` — badges (build status), quickstart, screenshots optional |
| Add | `ByterStore/CHANGELOG.md` — summarize all 15 days |
| Tag | `v1.0.0` on GitHub |

**Commit message:**

```
release: v1.0.0 — document changelog and finalize README for public release
```

**Commands:**

```bash
git tag -a v1.0.0 -m "ByterStore v1.0.0 — segmented KV store with REST API"
git push origin main
git push origin v1.0.0
```

Then on GitHub: **Releases → Draft new release →** pick tag `v1.0.0`, paste CHANGELOG.

**Done when:**

- [ ] GitHub Release published  
- [ ] README badges show passing CI  
- [ ] All Day 1–15 checkboxes below checked  

---

## Daily checklist (track progress)

| Day | Date | Focus | Pushed |
|-----|------|-------|--------|
| 1 | Jun 1 | README, vision, gitignore, plan | ☐ |
| 2 | Jun 2 | Architecture docs | ☐ |
| 3 | Jun 3 | Build guide & Makefile | ☐ |
| 4 | Jun 4 | Config wiring | ☐ |
| 5 | Jun 5 | Segment rotation fix | ☐ |
| 6 | Jun 6 | REST API alignment | ☐ |
| 7 | Jun 7 | Thread pool integration | ☐ |
| 8 | Jun 8 | Search HTTP endpoint | ☐ |
| 9 | Jun 9 | CLI tool | ☐ |
| 10 | Jun 10 | Unit tests | ☐ |
| 11 | Jun 11 | GitHub Actions CI | ☐ |
| 12 | Jun 12 | Docker | ☐ |
| 13 | Jun 13 | Examples & scripts | ☐ |
| 14 | Jun 14 | LICENSE, hardening | ☐ |
| 15 | Jun 15 | v1.0.0 release | ☐ |

---

## Optional: branch-per-day workflow

If you prefer PRs instead of direct commits to `main`:

```bash
git checkout -b day-04-config-wiring
# ... work ...
git push -u origin day-04-config-wiring
gh pr create --title "Day 4: wire bloom config from db.conf" --body "Part of 15-day upload plan."
```

Merge each PR before starting the next day.

---

## Tips for a strong GitHub profile

1. **Pin ByterStore** on your GitHub profile after Day 6.  
2. **Link the repo** in README to your LinkedIn / portfolio.  
3. **Tweet/post** one line per day: “Day N: shipped X for ByterStore.”  
4. Keep commits **small and readable** — recruiters diff your history.  
5. Reply to your own Issues (“Future: compaction”, “Future: replication”) to show roadmap thinking.

---

*Generated for ByterStore · Update checkboxes as you complete each day.*
