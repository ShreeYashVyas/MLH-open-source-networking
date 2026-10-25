# Safe recovery for completed Gemma 4 tool calls

- Upstream report: [ollama/ollama#17562](https://github.com/ollama/ollama/issues/17562), third issue in the report.
- Code: `model/parsers/gemma4.go`.
- Regression coverage: `model/parsers/gemma4_test.go`.

## Bug

Gemma 4 sometimes emits `<tool_call>call:set_temp{value:42<tool_call|>`: all arguments are present, and the model emitted its closing marker, but the outer arguments object is missing its final `}`. The parser's generic JSON repair path only adds a brace if another repair has already changed the candidate, so this otherwise recoverable call is dropped.

## Change

The close-marker path passes an explicit option enabling the generic final-brace repair. The end-of-generation path, where there is no close marker, does not enable it. This is important because a generation cut off at the token limit may have incomplete values; blindly adding braces can make truncated calls look valid.

A separate test covers the same payload without the close marker and requires it to remain rejected.

## Apply and test

From an Ollama checkout:

```bash
git apply /path/to/MLH-open-source-networking/fixes/ollama-gemma4-missing-brace.patch
go test ./model/parsers -run 'TestGemma4Parser_(RepairsMissingFinalObjectBraceWhenClosed|DoesNotRepairMissingBraceWhenTruncated)$' -count=1
go test ./model/parsers -count=1
```

I attempted to obtain an upstream checkout here, but outbound DNS prevented cloning GitHub, so I could not execute the Go tests in this environment. Run the tests locally before opening an upstream pull request.

## Contribution status

This patch is committed in the contribution repository for review. It is **not** merged into `ollama/ollama`. Existing [PR #16075](https://github.com/ollama/ollama/pull/16075) also touches the parser state machine, so a future upstream submission should check its latest state and rebase/adapt as needed.
