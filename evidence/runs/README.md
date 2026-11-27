# Run Archives

This folder holds one ZIP per review, 87 in total: the 84 completed reviews of the paper and
the three Codex runs that a provider safeguard stopped (Scans 76 to 78). Scans 1 to 60 form
Phase 1, Scans 61 to 87 Phase 2. Appendix G of the paper lists every review with its date,
result, duration, tool calls, tokens, and the share of code it saw.

## What Each Archive Contains

| File | Content |
|:---|:---|
| `input.txt` | The review prompt, the same in Scans 1 to 84 and a variant of it in Scans 85 to 87 |
| `events.jsonl` | The visible event stream of the client: every tool call, its printed output, and the notes the agent wrote between calls |
| `final.txt` | The final answer |
| `source-before.json`, `source-after.json` | SHA-256 manifests of every source file before and after the review |
| `trace-audit.json` | An automatic audit of every tool call for reads outside the source, network use, or Git use |
| `record.json` | Client, model, timestamps, result, the hash of every file, the redaction counts, and a short note |

The prompt of Scans 1 to 84 has the SHA-256 hash
`80ed67f9595954e604cb34bf5f6ee2b6416cc792ea1975b9e257736957f00cae`, as stated in Appendix C.
Scans 85 to 87 test three prompt variants, also given in Appendix C:

| Scan | Variant | What it asks for | SHA-256 |
|:---|:---|:---|:---|
| 85 | `minimal` | Search for vulnerabilities, with the same limits and output format | `cf9995d591afa4511d59bb686c3580a9eeb5a10f73e71804858f5ea964e8e190` |
| 86 | `coverage` | The minimal prompt, plus not to leave a single file unaudited | `ee9584ec8275aca7f5337cb15d99def2d41b5742f3be65361754c737e8575986` |
| 87 | `plan` | The coverage prompt, plus to decide a review plan first and follow it to the end | `f4b9137e1f871a273326001d4e85892d36652a3fb8e2eacadbb68c4e464f2fdf` |

The source manifests identify every reviewed file by its hash, so the reviewed code can be
matched against the pinned releases in Appendix D.

## Which Scan Is Which

| Scans | Model | What was reviewed |
|:---|:---|:---|
| 1 to 18, 34 to 45 | GPT-6.1 Sol in Codex | Phase 1, Gitea and Jenkins |
| 19 to 33, 46 to 60 | Opus 5.5 in Claude Code | Phase 1, Gitea and Jenkins |
| 61 to 63 | GPT-6.1 Sol in Codex | HIPC-1 issue search |
| 64 to 66 | GPT-6.1 Sol in Codex | HIPC-2 workflow health, light build |
| 67 to 69 | Opus 5.5 in Claude Code | HIPC-1 issue search |
| 70 to 72 | Opus 5.5 in Claude Code | HIPC-2 workflow health, light build |
| 73 to 75 | Opus 5.5 in Claude Code | HIPC-3 workflow health, full HIPC build |
| 76 to 78 | GPT-6.2 Astra, GPT-6.2 Sol, and GPT-6.1 Sol in Codex | HIPC-3, stopped by the provider safeguard |
| 79 to 81 | GPT-6 Sol in Codex with Daybreak | HIPC-1 issue search |
| 82 to 84 | GPT-6 Sol in Codex with Daybreak | HIPC-3 workflow health, full HIPC build |
| 85 to 87 | Opus 5.5 in Claude Code | HIPC-3 workflow health, full HIPC build, three prompt variants |

In Scans 58 and 70 the provider switched Opus 5.5 to Opus 4.8 midway. As in the paper, both
reviews are counted with Opus 5.5. The `record.json` of each archive names the configured and
the observed models.

The archive names encode the condition of each review. In Phase 1, `baseline` and `challenge` are the two
versions of one backdoor, `combined` is a source tree with all six backdoors of a project, and
`control` is the unchanged project. In Phase 2, `rq2` marks the phase, `search` the issue search
backdoor, `health` the light build of the workflow health backdoor, and `healthv2` its full HIPC
build. There, `combined` means the HIPC version of the source, and `minimal`, `coverage`, and
`plan` name the prompt variant of Scans 85 to 87.

## The Index

[`index.csv`](index.csv) has one row per archive:

| Column | Meaning |
|:---|:---|
| `scan` | Scan number as used in the paper |
| `condition` | Condition of the review, as in the archive name |
| `attempt_id` | Run ID |
| `target_result` | Result for the planted backdoor. `found` and `complete` both mean found in full, and `not_scored` marks a control review without a planted backdoor. The three stopped runs are recorded as `missed` but are not part of the detection counts in the paper. For the combined reviews, Appendix G gives the result for each backdoor. |
| `archive` | File name of the ZIP |
| `archive_sha256` | SHA-256 hash of the ZIP |
| `raw_events_sha256` | SHA-256 hash of the unmasked event stream, which stays outside this repository |
| `published_events_sha256` | SHA-256 hash of the published `events.jsonl` |
| `redactions` | Number of masked values in the archive |

## Masking

The event streams are publication copies. Session IDs, token-like strings in printed source
code, and local paths are masked, and in two Claude Code reviews so is the account e-mail
address that the client included in a context summary. `record.json` lists the original and
the published hash of every file and the redaction counts. All numbers in the paper were
computed from these published copies.

Not included: the constructed backdoors as patches or branches, the unmasked raw records,
login and local setup files, and the analysis scripts.

## Checking the Archives

With a POSIX shell:

```sh
cd evidence/runs
tail -n +2 index.csv | awk -F, '{print $6 "  " $5}' | sha256sum -c
```

With PowerShell:

```powershell
Import-Csv evidence/runs/index.csv | ForEach-Object {
  $hash = (Get-FileHash "evidence/runs/$($_.archive)" -Algorithm SHA256).Hash
  if ($hash -ne $_.archive_sha256) { "MISMATCH $($_.archive)" }
}
```

The PowerShell check prints nothing when every archive matches.
