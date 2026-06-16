# Contribution 1: [Adapter: llama.cpp server Model Provider]

**Contribution Number:** 1

**Student:** Faisal  

**Issue:** https://github.com/orthogonalhq/nous-core/issues/318

**Status:** Phase 2 Complete

---

## Why I Chose This Issue

This issue requires implementing a model provider adapter so the nous-core agent framework can communicate with a locally hosted llama.cpp server. 

I'm interested in this because:

- I've dabbled with TypeScript before, so this provides a great environment to practically apply and build on those skills.

- The codebase area is tightly scoped (and also well structured!) making it a manageable candidate for my first pull request.

Left a comment on the issue introducing myself!

---

## Understanding the Issue

### Problem Description

The nous-core provider system has implementations for Anthropic, OpenAI, and Ollama, but no adapter for llama.cpp server. Users running llama.cpp locally have no way to connect it to the framework.

### Expected Behavior

A LlamaCppProvider class should exist that implements the IModelProvider interface, allowing the framework to send inference requests to a locally running llama.cpp server (default http://localhost:8080) without requiring an API key.

### Current Behavior

No `LlamaCppProvider` exists. The providers directory only contains `anthropic/`, `ollama/`, and `openai/`. There is no export for a llama.cpp provider in `index.ts`.

### Affected Components

- `self/subcortex/providers/src/providers/` — where the new provider directory will live
- `self/subcortex/providers/src/index.ts` — needs a new export
- `self/subcortex/providers/src/protocols/openai-api/provider.ts` — the shared ChatCompletionsProvider primitive this implementation will reuse
- `self/subcortex/providers/src/__tests__/` — where the unit test will be added

---

## Reproduction Process

### Environment Setup

The environment setup is laid out in a really straight forward manner by the maintainer.

```
pnpm install
pnpm build
pnpm test
```

- The project uses pnpm, not npm. Running npm install fails with platform errors because npm misreads pnpm-specific .npmrc config keys (shamefully-hoist, node-linker, etc.). Fix: use pnpm install.
- The maintainer indicated not to target main or dev. The correct base branch for this adapter work is feat/contributor-friendly-inference-provider-surface. I fetched it from upstream and created my working branch off it.
- After pnpm install && pnpm build && pnpm test, all existing tests pass.

### Steps to Reproduce

1. Clone the repo and check out feat/contributor-friendly-inference-provider-surface
2. Run ls self/subcortex/providers/src/providers/ — output shows anthropic/, ollama/, openai/ only
3. Open self/subcortex/providers/src/index.ts — no LlamaCppProvider export exists
4. Cross-reference with the issue acceptance criteria — invoke, stream, getConfig, and an index export are all required and all missing
### Reproduction Evidence

- Branch: [https://github.com/FaisalXL/nous-core/tree/feat/llama-cpp-provider](https://github.com/FaisalXL/nous-core/tree/feat/llama-cpp-provider)
- My findings: The gap is straightforward. The provider directory and implementation file simply do not exist. Because llama.cpp uses the same OpenAI-compatible wire format as ChatCompletionsProvider, the implementation can reuse that shared primitive rather than being built from scratch. The main difference from ChatCompletionsProvider is that llama.cpp requires no API key and targets a local endpoint.

---

## Solution Approach

### Analysis

The root cause is a gap in coverage: the provider system was built to be extensible (each provider lives in its own directory under providers/ and is registered via index.ts), but no one has yet implemented the llama.cpp entry. There is no bug, it just doesn't have this adapter yet.

The maintainer confirmed in issue comments that this should be implemented as a config entry against the shared ChatCompletionsProvider primitive (the OpenAI-compat protocol layer), not as a fully custom provider like Ollama.

### Proposed Solution

Create a LlamaCppProvider that wraps ChatCompletionsProvider with llama.cpp-specific defaults: local endpoint `(http://localhost:8080)`, no API key required, and a provider definition that marks it as local and auth-optional. Export it from `index.ts` and cover it with unit tests.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The IModelProvider interface requires three methods: invoke, stream, and getConfig. All three are already fully implemented in ChatCompletionsProvider for the OpenAI-compat wire format, which llama.cpp server also speaks. The only missing piece is a llama.cpp-specific entry point with the right defaults and no auth enforcement.

**Match:** ChatCompletionsProvider in protocols/openai-api/provider.ts is the direct model. It handles `/v1/chat/completions`. `OLLAMA_PROVIDER_DEFINITION` in `providers/ollama/implementation.ts` is the model for the provider definition shape `(isLocal: true, auth.required: false)`

**Plan:** [Step-by-step implementation plan]
1. Create self/subcortex/providers/src/providers/llama-cpp/implementation.ts — define `LLAMA_CPP_PROVIDER_DEFINITION` and export LlamaCppProvider, which extends or wraps ChatCompletionsProvider with default endpoint http://localhost:8080 and no API key requirement
2. Add `export { LlamaCppProvider } from './providers/llama-cpp/implementation.js'` to `self/subcortex/providers/src/index.ts`
3. Add `self/subcortex/providers/src/__tests__/llama-cpp-provider.test.ts` with unit tests covering invoke, stream, and getConfig, and confirming no error is thrown when no API key is provided

**Implement:** [https://github.com/FaisalXL/nous-core/tree/feat/llama-cpp-provider](https://github.com/FaisalXL/nous-core/tree/feat/llama-cpp-provider)

**Review:** Before submitting: run pnpm typecheck, pnpm lint, pnpm test. PR targets feat/contributor-friendly-inference-provider-surface. Commit message follows conventional commits format: feat: add LlamaCppProvider adapter for llama.cpp server

**Evaluate:**  Tests confirm: invoke returns a valid `ModelResponse`, stream yields `ModelStreamChunk` objects, `getConfig` returns the constructor config, and no `PROVIDER_AUTH_FAILED` error is thrown when instantiated without an API key.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: invoke sends a POST to /v1/chat/completions and returns ModelResponse
- [ ] Test case 2: stream yields chunks and a final done: true chunk
- [ ] Test case 3: getConfig returns the config passed to the constructor
- [ ] Test case 4: Instantiation succeeds with no API key (unlike ChatCompletionsProvider)
- [ ] Test case 5: 401/429/non-ok responses map to the correct NousError codes


### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
