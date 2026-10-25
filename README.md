# Gemma 4 parser bug fix — Ollama

This repository contains a proposed, test-backed contribution for an open Gemma 4 tool-calling bug in [Ollama issue #17562](https://github.com/ollama/ollama/issues/17562), sub-issue 3.

## Problem

Gemma 4 can emit a completed tool call with the closing marker `<tool_call|>` but omit the final `}` from its arguments object. Ollama's parser then fails JSON decoding and silently drops the tool call, so an agent may receive an empty response.

## Proposed fix

The patch only allows the missing-outer-brace repair after the parser has seen the model's explicit tool-call closing marker. If generation ends without that marker, the output may have been truncated; the parser deliberately does **not** repair that case. This avoids manufacturing a plausible-looking tool call from incomplete output.

## Contents

- `fixes/ollama-gemma4-missing-brace.patch`: source change and regression tests to apply to the upstream Ollama repository.
- `fixes/README.md`: issue analysis, rationale, and validation steps.

## Status

This is a proposed contribution staged in this repository. It has not been merged upstream, and the upstream Go tests must be run after applying the patch. It cannot guarantee acceptance into Ollama or a prize at the event.
