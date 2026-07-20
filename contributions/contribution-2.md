# Contribution 2: [Docs: Convert ImageCache example into a testsuite test]

**Contribution Number:** 2

**Student:** Faisal  

**Issue:** https://github.com/AcademySoftwareFoundation/OpenImageIO/issues/3992

**Status:** Phase 3 Complete — Implemented & Locally Verified. Code written, committed locally (signed off), and verified end-to-end for both C++ and Python. Not yet pushed to fork / PR not yet opened (Phase 4).

---

## Why I Chose This Issue

OpenImageIO is a C++ image I/O library used across VFX/animation pipelines (backed by the Academy Software Foundation — Pixar, ILM, Sony, etc.) for reading/writing image data, which overlaps with the kind of image-processing code that feeds computer-vision/ML pipelines.

I'm interested in this because:

- It plays to my C++ background directly, and the task itself (converting a documented code example into a real compiled test) is a contained, well-understood unit of work rather than an open-ended feature.
- Issue #3992 is a standing "umbrella" task the maintainer (lgritz, the project's founder) explicitly keeps open for anyone to pick a piece of — he has stated in the issue that no single PR closes it, and that people should each grab one unclaimed doc example. That structure avoids the problem I ran into scoping candidates elsewhere: popular repos where 3-4 people all submit competing PRs for the same "good first issue" before you can finish yours. Here, everyone works on a different, non-overlapping example, so there's no race.
- The maintainer has a strong track record of fast, encouraging responses (same-day replies across the entire multi-year issue thread), and roughly a dozen other newcomers have successfully gone through this exact process and gotten PRs merged.

Left a comment on the issue claiming the ImageCache chapter example specifically, so no one else duplicates this piece.

---

## Understanding the Issue

### Problem Description

OpenImageIO's documentation (`src/doc/*.rst`) contains C++/Python code snippets embedded directly as plain text to illustrate API usage. These snippets are never compiled or executed, so some have drifted out of sync with the real API over time (or were never fully correct to begin with).

### Expected Behavior

Each significant doc example should be moved into a real, compiling, running test in the `testsuite/docs-examples-cpp/` (and `-python/`) directories, with the documentation `.rst` file including that code by reference (via Sphinx's `literalinclude` directive) instead of hosting a copy of the text. This way the docs and the actual tested code can never go out of sync again.

### Current Behavior

`src/doc/imagecache.rst` still contains a raw, untested code fragment demonstrating `ImageCache::create()`, `get_pixels()`, `get_imagespec()`, `get_tile()`/`tile_pixels()`/`release_tile()`, and `ImageCache::destroy()`. It has not yet been converted — confirmed by checking that the doc file has no `literalinclude` directive, and that no PR or issue comment across the multi-year thread mentions "ImageCache" as a claimed or completed section (I checked all ~25 merged PRs that touch ImageCache in this repo; none of them are documentation-to-test conversions).

There is already a placeholder file waiting for this specific conversion: `testsuite/docs-examples-cpp/src/docs-examples-imagecache.cpp` contains a stub `example1()` function with a comment: "Example code fragment from the docs goes here."

### Affected Components

- `src/doc/imagecache.rst` — doc source, raw snippet to be replaced with a `literalinclude` reference
- `testsuite/docs-examples-cpp/src/docs-examples-imagecache.cpp` — stub file to fill in with real, compiling code
- `testsuite/docs-examples-cpp/ref/` — reference output this test's run will be compared against (new reference file likely needed)
- `testsuite/docs-examples-cpp/run.py` — test runner script, may need the new test's output file added to its `outputs` list

---

## Reproduction Process

### Environment Setup

Completed. Forked the repo, cloned locally, and built with CMake + Homebrew dependencies on macOS (Apple Silicon):

```
git clone https://github.com/FaisalXL/OpenImageIO.git
cd OpenImageIO
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release ...
cmake --build build -j
```

**Errors hit and how I fixed them:**

1. **`ctest` reported "No tests were found!!!"** — I was running `ctest` from the repo root instead of from inside `build/`. Fix: `cd build && ctest`, or `ctest --test-dir build` from the root.
2. **Every single test failed with `env: python: No such file or directory`** — root-caused this by reading `src/cmake/testing.cmake`: every test is registered as `add_test(NAME ... COMMAND ${Python3_EXECUTABLE} testsuite/runtest.py ...)`. Because my build was configured with `USE_PYTHON=0`, `find_package(Python3)` never ran, so `Python3_EXECUTABLE` was empty — CMake fell back to invoking `runtest.py` directly via its `#!/usr/bin/env python` shebang, which fails on macOS since only `python3` (not `python`) exists on `PATH`. Fix: reconfigure with the interpreter pinned explicitly, without having to flip `USE_PYTHON` back on:
   ```
   cmake -DPython3_EXECUTABLE=$(command -v python3) build
   ```
   This fixed all 135 tests going from 100% erroring to running for real.
3. After the fix, established a real baseline: **120/135 tests pass (89%)**. The 15 failures are pre-existing and unrelated to my issue — confirmed via `git status` that no source files are modified (clean checkout except build artifacts):
   - `docs-examples-cpp` — the test I'm actually working on. Fails because 5 of its 8 "chapters" (including `imagecache`, mine) are still unimplemented stub files (43 lines each, just a placeholder `example1()`). This is the expected starting state for this umbrella issue.
   - `cmake-consumer`, `igrep`, `oiiotool`, `oiiotool-copy`, `oiiotool-subimage`, `oiiotool-text` — likely reference-image/FreeType-version mismatches (spotted a `ref/out-freetype2.7.tif` vs. default reference pattern elsewhere in the log) and local-install-path quirks, unrelated to ImageCache.
   - `texture-levels-stochaniso(.batch)`, `texture-levels-stochmip(.batch)`, `texture-udim(.batch)` — known-flaky stochastic texture-sampling tests, sensitive to SIMD/thread timing on Apple Silicon.

   This 15-failure baseline is documented so that after implementing my fix, I can confirm I haven't introduced any *new* failures beyond this known set.

- OpenImageIO requires a CLA (individual or corporate) to be signed via EasyCLA before a PR can be merged — need to complete this before opening a PR.
- Commits must be signed off per the DCO (`git commit -s`).
- The project has an AI-assistance disclosure policy requiring an `Assisted-by: TOOL/MODEL` line in commits/PRs for any AI-assisted work.

### Steps to Reproduce

1. Open `src/doc/imagecache.rst` — the ImageCache code fragment appears as inline text (not a `literalinclude`).
2. Open `testsuite/docs-examples-cpp/src/docs-examples-imagecache.cpp` — contains only the stub `example1()` template, not the real example code.
3. Search the `#3992` issue thread and OpenImageIO's merged-PR history for "imagecache" — no existing claim or completed conversion found.

### Reproduction Evidence

- Issue: https://github.com/AcademySoftwareFoundation/OpenImageIO/issues/3992
- Claiming comment posted: https://github.com/AcademySoftwareFoundation/OpenImageIO/issues/3992#issuecomment-4889912173 (Jul 6, 2026)
- Working branch (in my fork): https://github.com/FaisalXL/OpenImageIO/tree/fix-issue-3992-imagecache-docs-example
- Conversion process reference: https://github.com/OpenImageIO/oiio/wiki/Converting-documentation-examples-to-tests

---

## Solution Approach

### Analysis

This isn't a bug — it's a gap in an ongoing, intentionally-parallelizable documentation task. The fix pattern is already well-established by ~15 prior merged PRs against other chapters (imagebuf, imagebufalgo, imageinput, imageoutput); ImageCache just hasn't had anyone pick it up yet.

### Proposed Solution

Move the ImageCache doc example's code into `docs-examples-imagecache.cpp`'s stub function, wrap it with `BEGIN-imagecache-*` / `END-imagecache-*` marker comments, replace the raw snippet in `imagecache.rst` with a `literalinclude` pointing at those markers, and verify the test builds/runs/passes.

### Implementation Plan

1. Write a working version of the `ImageCache::create` / `attribute` / `get_pixels` / `get_imagespec` / `get_tile` / `release_tile` / `destroy` example into `example1()` in the stub `.cpp` file, using a real small test image instead of the placeholder `"file1.jpg"` / `"file2.exr"` filenames.
2. Wrap the relevant code with `// BEGIN-imagecache-example1` / `// END-imagecache-example1` markers.
3. Call `example1()` from `main()` (already stubbed in).
4. Edit `src/doc/imagecache.rst`: delete the raw code block, replace with a `.. literalinclude::` directive referencing the new file and markers.
5. Build OpenImageIO locally and run `make test TEST=docs-examples` (or the equivalent `ctest` invocation) until the test compiles and passes.
6. Add any new reference output to `testsuite/docs-examples-cpp/ref/` and register new output filenames in `run.py` if the example produces an image or text output.
7. Sign the CLA, open a PR referencing #3992, following conventional-commit prefixes (`docs: ...`) and DCO sign-off.

**Status: not yet started** — currently at the "claimed the issue" stage.

---

## Testing Strategy

### Unit Tests

- [x] Test case 1: `docs-examples-imagecache` test target builds without errors (both C++ and Python)
- [x] Test case 2: Test runs and produces output matching a checked-in reference — ran the real `example1()` against `tahoe.tif`, got `resolution is 512x384` from both the C++ executable and the Python script, and updated `ref/out.txt` (+ `out-arm.txt`, `out-linuxarm.txt`, in both `docs-examples-cpp/` and `docs-examples-python/`) to include the new line at its exact real position, confirmed by diffing a fresh full 8-chapter run against the patched reference
- [x] Test case 3: `imagecache.rst` uses the exact `literalinclude`/`tabs::` structure already used by the converted `imageoutput.rst`/`imagebufalgo.rst` chapters (spot-checked by pattern match against those files, not an actual Sphinx build)

### Integration Tests

- [x] Ran all 8 `docs-examples-cpp` chapter executables and all 8 `docs-examples-python` chapter scripts together (replicating `run.py`'s prep + execution order) and confirmed the *text* output matches the patched reference exactly, chapter-for-chapter. (The full `ctest` target still fails overall — 5 of 8 chapters are still unconverted stubs from other contributors' unclaimed pieces, which is expected for this umbrella issue and pre-dates my change.)

### Manual Testing

Built the full library locally (CMake + Homebrew deps, including installing `opencolorio` which this checkout needed but didn't have) and ran both the C++ and Python examples directly against a real test image (`testsuite/common/tahoe-small.tif`). Both printed the correct `resolution is 512x384`.

**Real bugs/API drift found and fixed along the way** (this is exactly what the umbrella issue is meant to catch):
- `ImageCache::create()`/`destroy()` used to take/return a raw pointer; the current API uses `std::shared_ptr<ImageCache>`. Doc's old snippet was already stale here.
- `ustring`'s `const char*` constructor is `explicit` now — string literals like `"tahoe.tif"` no longer implicitly convert where the API wants a `ustring`. Fixed by constructing `ustring filename("tahoe.tif")` explicitly.
- `ImageCache::get_pixels(ustring, int, int, ROI, span<T>&, ...)` — the templated `span<T>` overload has an apparent bug in the current header: it builds a correct `image_span<T>` internally but then calls `as_image_span_writable_bytes()` on the *original* `span<T>` argument instead of the `image_span` it just built, which doesn't compile. Worked around by calling the lower-level `image_span<T>` overload directly instead of the `span<T>` convenience wrapper — may be worth flagging to the maintainer separately.
- Python bindings (`src/python/py_imagecache.cpp`) don't expose `get_tile()`/`tile_pixels()`/`release_tile()` at all, so the Python example is necessarily narrower than the C++ one — documented directly in the `.rst`.

---

## Implementation Notes

### Week 1 Progress

Selected OpenImageIO issue #3992 after evaluating and ruling out several other candidate repos/issues (documented the comparison separately) for being either stale-labeled, oversaturated with competing duplicate PRs, or gated behind maintainer pre-assignment. Read the issue thread in full, confirmed the ImageCache chapter example was unclaimed by checking every comment in the thread plus OpenImageIO's merged PR history. Posted a comment on the issue claiming the ImageCache example specifically. Identified the exact files that need to change (`src/doc/imagecache.rst` and the pre-existing stub `docs-examples-imagecache.cpp`) and read the maintainer's wiki write-up of the expected conversion process.

### Week 2 Progress

Forked the repo, cloned it locally, and got the full CMake build compiling on macOS. Hit and resolved a real build/test-infra bug along the way (see Environment Setup above — a `Python3_EXECUTABLE` detection gap that made every single `ctest` test fail with `env: python: No such file or directory`; root-caused it by reading through `src/cmake/testing.cmake`, then fixed it with an explicit `-DPython3_EXECUTABLE` reconfigure). Ran the full test suite and established a baseline of 120/135 passing (89%), with the 15 failures documented as pre-existing and unrelated to my change. Created and pushed my working branch (`fix-issue-3992-imagecache-docs-example`) to my fork.

**Next steps:** write the real `ImageCache` example code into the `example1()` stub, wrap it in `BEGIN`/`END` marker comments, update `src/doc/imagecache.rst` to use `literalinclude`, get `docs-examples-cpp` passing, then open a draft PR.

**Status: implementation (Phase 3) not yet started** — currently at "environment ready, branch created, plan written."

### Week 3 Progress

Implemented both the C++ and Python versions of the `example1()` fix (the doc's original snippet only ever showed C++, but the established pattern from merged reference PRs — e.g. #5197 for ImageOutput — covers both `docs-examples-cpp/` and `docs-examples-python/`, so did both). Built the full library locally, hit and fixed real compile errors caused by API drift (see Manual Testing above — stale `create()`/`destroy()` signature, `ustring` no longer implicitly constructible from a string literal, and what looks like an actual bug in the current `get_pixels(span<T>)` template). Verified both examples run correctly end-to-end against a real test image. Replicated the full 8-chapter test run (matching `run.py`'s prep/execution order) for both testsuites and patched all 6 affected reference files (`ref/out.txt`, `out-arm.txt`, `out-linuxarm.txt` under both `docs-examples-cpp/` and `docs-examples-python/`) with the new line at its verified real position. Committed locally with DCO sign-off and the required `Assisted-by:` line, folded the follow-up fixes into that same commit since nothing had been pushed yet.

**Next steps:** push the branch to the fork and open the PR against `main` (maintainer confirmed in a 2023 comment on a similar doc-conversion PR that `main`/`master` is the right target, no intermediate branch needed).

**Status: implementation (Phase 3) complete, locally verified.** Ready to push — holding for explicit go-ahead before touching the real GitHub fork.

---

## Pull Request

**PR Link:** *(not yet opened)*

**PR Description:** *(pending)*

**Maintainer Feedback:** *(pending)*

**Status:** Not yet opened — issue claimed, implementation in progress

---

## Learnings & Reflections

*(To be filled in as implementation progresses.)*

---

## Resources Used

- [Issue #3992](https://github.com/AcademySoftwareFoundation/OpenImageIO/issues/3992) — original issue and multi-year thread of prior contributors' examples
- [Converting documentation examples to tests (wiki)](https://github.com/OpenImageIO/oiio/wiki/Converting-documentation-examples-to-tests) — maintainer's step-by-step guide for this exact task
- [OpenImageIO CONTRIBUTING.md](https://github.com/AcademySoftwareFoundation/OpenImageIO/blob/main/CONTRIBUTING.md) — CLA, DCO, commit style, AI-disclosure policy
- Example prior conversions for reference: [PR #4016](https://github.com/AcademySoftwareFoundation/OpenImageIO/pull/4016) (imagebufalgo), [PR #5197](https://github.com/AcademySoftwareFoundation/OpenImageIO/pull/5197) (imageoutput)
