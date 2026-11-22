# Ember Site Generator

[![make.fun](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/pranatakripto95/ember-site-generator/rank.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/pranatakripto95/ember-site-generator/reward.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own makefun-site 1.0.0, built for the make.fun
[site-generator-speed-01-run-20261122](https://make.fun/c/site-generator-speed-01-run-20261122)
bounty. `solve.js` answers every case exactly as makefun-site 1.0.0 does —
the same output tree (each page's `index.html`, the index and tag pages,
`build-report.txt` and each static file), byte for byte — only faster.

## How it is faster

The reference walks its Markdown one character at a time, lexes every body
into token objects, re-slugifies the same titles and tags over and over, and
renders templates by string concatenation through a scope chain that allocates
a scope object per loop iteration. This build instead:

- renders Markdown in a single pass straight to HTML, toc and excerpt, with
  no token objects and no regex per line;
- jumps between inline constructs with native `indexOf` scans instead of
  stepping character by character, keeping plain runs as slices;
- escapes HTML with a fast path that returns the input untouched when
  nothing needs escaping;
- slugifies with an ASCII fast path and a per-case cache (titles, tags and
  headings repeat constantly across a site);
- counts words with a manual scan instead of allocating a regex match array
  per body;
- renders templates into an output array joined once, with loop scopes
  mutated in place instead of allocated per iteration;
- computes each source's card and page objects once instead of rebuilding
  them on every render.

On the kit's practice set it is byte-identical to the reference and about
3.2x faster in `solve()` CPU time (measured beside the baseline on the same
machine with Node 22).

## Layout

- `solve.js` — the generator: `run(input)` returns makefun-site's answer
  for `{ site, templates, files }`.
- `main.ts` — the driver the harness runs: reads every case from
  `/input/cases.json`, answers each with `run`, writes
  `/output/answers.json`.
- `package.json` — `{"type": "module"}`, so `solve.js` loads as an
  ES module.

Dependency-free, Node 22, no network, no WebAssembly, no native addons.
