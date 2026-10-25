# Gemma 4 parser bug fix — Ollama

This repository stages a proposed, test-backed contribution for a real Gemma 4 tool-calling bug in [Ollama issue #17562](https://github.com/ollama/ollama/issues/17562), third reported bug.

## Problem

Gemma 4 can emit a completed tool call with the closing marker `<tool_call|>` but omit the final `}` from the argument object. Ollama's parser then fails JSON decoding and drops the tool call; an agent can receive an empty response.

## Proposed fix

The patch adds a generic missing-final-object-brace repair only when the parser has observed the model's explicit tool-call closing marker. If generation ends without that marker, the payload may be truncated; the new generic fallback does not repair it. Existing schema-guided recovery for raw terminal string values is left unchanged.

## Files

- `fixes/ollama-gemma4-missing-brace.patch`: changes to the upstream source and regression tests.
- `fixes/README.md`: issue analysis and commands for applying/testing the patch.

## Review note

Upstream PR [#16075](https://github.com/ollama/ollama/pull/16075) is open and touches the same parser area, so this proposal may need to be rebased or adapted if that work lands. This patch is not merged upstream, and its Go tests must be run after applying it. No code change can guarantee an event prize.
