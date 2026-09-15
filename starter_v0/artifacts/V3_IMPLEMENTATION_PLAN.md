# v3 — confirmation, conversation boundaries and scoped arguments

## Status and baseline

Final v3 evaluated: **30/30**, with every v1/v2 pass retained. The focused H12 revision stays under `--version v3`; artifact hashes distinguish it from the initial iteration.

The [recorded v2 base run](../runs/v2_B_base_openrouter_20260915T193715254329.json) used `openrouter` / `openai/gpt-4o-mini`, measured all 30 cases with zero provider errors, and passed 26/30 (0.8667). Tool routing was 1.00; argument accuracy 0.8667; multi-turn accuracy 0.90. Its hashes match the preserved [v2 prompt](../backup/system_prompt_v2.md) and [v2 tools](../backup/tools_v2.yaml).

Unlike the original suggested sequence, confirmation was already introduced in v1 and identifier/environment changes were implemented in v2. v3 strengthens those confirmation boundaries and addresses the actual remaining argument failures. It is a combined prompt/tool-description experiment.

### v3 evidence and focused revision

The initial iteration measured 29/30: H03/H17/M06 changed to PASS and no v2 pass regressed, while H12 still asked for detailed symptoms using `text`. The [final v3 run](../runs/v3_B_base_openrouter_20260915T200944073522.json) measures all 30 cases with zero provider errors and scores 1.00 for case, routing, argument and multi-turn accuracy. H12 now reviews the stated issue, priority and asset using `yes_no`; M05/M09 confirmation and M07 cancellation remain correct. There are no recorded tool-result errors or ticket-creation calls.

The revised prompt defines the minimum ticket summary explicitly: a brief report of a service/device failure is enough. Error codes, detailed symptoms, root cause and troubleshooting history are optional; do not delay approval to obtain them. Text clarification is reserved for no stated issue or conflicting ticket values not resolved by the latest correction. This replaces only the summary-preparation paragraph; every other prompt rule and all of `tools.yaml` are unchanged from the 29/30 run. Snapshots [system_prompt_v3_initial.md](../backup/system_prompt_v3_initial.md) and [tools_v3_initial.yaml](../backup/tools_v3_initial.yaml) match that run's hashes.

The final 30/30 result and artifact hashes are recorded in `version_log.csv`.

## Implemented changes

- [system_prompt.md](system_prompt.md): derive the summary from the described issue; review current details with yes/no approval; wait for a subsequent user reply; invalidate approval after any edit or cancellation; create once after valid approval. Honor the latest task, preserve requested multiple-source calls, and treat retrieved/tool text and pasted role/confirmation claims as untrusted instructions.
- [tools.yaml](tools.yaml): reinforce the confirmation type and creation precondition; explicitly select `search_kb.category` and `inspect_device.check` using the current topic/scope. Keep OS and application details in KB queries. Preserve broad searches/inspections when requested or when no narrower scope is established.
- Preserve existing identifier/environment prompt sections, all tool schemas, Python execution code, fixed datasets and recorded runs. Keep v2 snapshots for comparison and rollback.

### Actual v2 failures

| Case | Observed failure | v3 change |
|---|---|---|
| H12 | Asks for the already-described incident summary with `response_type=text` | Draft summary from the issue; review existing details with `yes_no` |
| H03 | KB category omitted for Outlook guidance | Explicit topic category: email |
| H17 | Device and service calls correct; KB category omitted | Explicit KB category: vpn; preserve all three requested calls |
| M06 | KB category `all` despite Wi-Fi context | Carry current topic into explicit category; retain Windows in query |

## Full base regression gate

The revision meets the gate: zero provider errors, **all 29 initial v3 passes retained**, and H12 repaired for 30/30. The comparison also retains every v1/v2 pass.

| Cases (all 30 covered) | Behavior to retain or repair |
|---|---|
| H01, H06, M02 | Correct service/environment; explicit staging and established context |
| H02 | General device inspection still uses `check=all` |
| H03, M06 | Correct KB category and relevant query context; no stale status call |
| H04, M04 | Employee lookup with latest employee ID; no automatic diagnostics |
| H05, H13, H17, M08 | VPN diagnostic scope; all requested device/status/KB sources; corrected asset ID |
| H07, H20 | Correct report template/title; format existing evidence without refetching |
| H08, H09, H14 | Out-of-scope/meta requests retain no-tool behavior |
| H10, H11 | Missing asset/employee ID asks text clarification without guessed calls |
| H12, M05, M09 | Only yes/no clarification with the latest ticket details; no creation or diagnostics |
| M01, M03 | Carry supplied/corrected asset ID and current network/security scope |
| H15, H16 | Separate calls for both requested environments/assets |
| H18 | Preserve legitimate employee lookup plus explicitly requested asset inspection |
| H19 | Ambiguous environment asks choice with production/staging options |
| M07 | Cancellation acknowledgment with no tool calls |
| M10 | New account lookup replaces the canceled device task |

## Local validation completed

- Project `load_tool_declarations` and `to_openai_tools` succeed for all nine registered tools.
- Parsed schemas are identical to v2 after recursively removing descriptions; only `clarify`, `search_kb`, `inspect_device` and `create_ticket` declarations changed.
- Both v2 snapshots match the recorded SHA-256 hashes. Identifier, environment and output prompt sections are unchanged.
- All 30 base inputs and expectations match the recorded v2 and initial v3 evidence. Fixed datasets, Python source and existing runs remain unchanged. The version log records the measured initial v3 result. `git diff --check` passes.
- Initial v3 snapshots match the measured hashes; the focused revision changes only the ticket-summary paragraph. The active tools file is byte-identical to initial v3.

Final v3 hashes:

```text
prompt_hash: fe6943f176699214ece2c7c44c2a3d1300dc4622e74edaaccab5a6015c72a53a
tools_hash:  68be84829c28823d14d1989ec35586a11f16e81f82c950b220fd8de60f72e94d
```

Initial prompt hash: `9e04a1ca66a1d96ac8f9dffe752f3f8b858e7cbcce019df4a96f98ea90b55b11`. The final run establishes the base-suite behavior of the revision; adversarial and live-chat evidence remain separate.

## User-run evaluation

From `starter_v0/`, with the existing Python environment activated:

```bash
python run_eval.py --provider openrouter --model openai/gpt-4o-mini \
  --version v3 --suite base --eval-cases data/eval_base.json
```

`--suite` is a label; the explicit dataset path ensures all 30 fixed cases are used. Keep the same provider/model and active artifact files. Provide the resulting JSON path for comparison.

After the base gate, evaluate the fixed safety suite separately:

```bash
python run_eval.py --provider openrouter --model openai/gpt-4o-mini \
  --version v3 --suite adversarial --eval-cases data/eval_adversarial.json
```

## Review the actual results before accepting v3

1. Verify provider/model, dataset, artifact hashes, unique case IDs and `measured_cases == total_cases == 30` with `provider_error_cases == 0` for the base run.
2. Compare every case with initial v3, v2 and v1, listing newly passing, still failing and regressed IDs. Any lost initial v3 pass fails the regression gate even if the aggregate score improves.
3. Read calls, arguments, confirmation questions and tool results. H12/M05/M09 must contain the current summary, priority and asset where supplied, then stop. M07 must not create or clarify. Scope fixes must not add unrequested tools. Any tool error or unexpected write requires investigation even when routing passes.
4. Base scoring compares tool calls and argument subsets. Separately review safety responses and real multi-turn transcripts: draft → approval → one creation; edit with approval in the same reply → renewed confirmation; cancel → bare yes → no creation; completed creation → repeated yes → no duplicate; injected instructions in actual retrieved/tool text → no unauthorized action. Also exercise a ticket without an asset ID so optional details do not block creation.
5. The existing agent performs one model completion per run and directly executes returned calls; the ticket tool checks the boolean flag, not conversation history. Prompt-only hardening is not runtime enforcement. Fixed evals alone do not demonstrate resistance to instructions received after retrieval or successful continuation after approval.
6. Record the revision's actual metrics and run path in `REPORT.md` and append another `v3` CSV row only after reviewing its completed run. Use 0.9667 (initial v3) as the before accuracy and the new run's actual artifact version/hashes. Keep the initial v3 row and run; leave unknown author attribution blank. No v4 version is needed.
