# Safe recovery of a completed Gemma 4 tool call

- Upstream issue: [ollama/ollama#17562](https://github.com/ollama/ollama/issues/17562), third reported bug.
- Affected code: `model/parsers/gemma4.go`.
- Regression tests: `model/parsers/gemma4_test.go`.

## Before

Input:

```text
<|tool_call>call:set_temp{value:42<tool_call|>
```

The model has emitted a closing marker, but its argument object is missing the final brace. Strict parsing fails; the tool call is discarded and the caller can receive no usable content.

## After

The parser retries with one final object brace only when it has seen `<tool_call|`. The arguments then parse as `{"value": 42}`.

A second regression test asserts that the same incomplete payload **without** `<tool_call|` remains rejected. That guard matters because the generation may have been cut off at a token limit. We should never turn a truncated tool call into a valid-looking one with missing arguments.

## Apply and test

From an Ollama checkout:

```bash
git apply /path/to/MLH-open-source-networking/fixes/ollama-gemma4-missing-brace.patch
go test ./model/parsers -run 'TestGemma4Parser_(RepairsMissingFinalObjectBraceWhenClosed|DoesNotRepairMissingBraceWhenTruncated)$' -count=1
go test ./model/parsers -count=1
```

The patch is based on the Ollama `main` source fetched on 25 October 2026. This environment could not clone Ollama to run Go tests, so tests remain unverified here; run them locally before submitting upstream.

## Why this is a reasonable intermediate contribution

It fixes an actual user-visible failure in model tool calling, introduces a conservative parsing rule, and tests both the successful repair and the truncation guard. The contribution is more substantial than changing a model-name threshold, but maintainers must still review it and may prefer another design.
