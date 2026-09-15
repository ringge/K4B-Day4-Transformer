# v2 implementation plan — identifier routing and clarification

## Objective and handoff

Implement a focused v2 experiment that improves how the agent selects tools when identifiers are missing or the requested environment is ambiguous. Preserve the ticket-confirmation behavior established in v1.

This file is a plan, not evidence that v2 has been implemented or evaluated. The implementing agent should make the changes, perform local checks, and then tell the user to run `run_eval.py`. The user will run the live evaluation, as in v1. Do not launch a provider evaluation during implementation or invent a v2 result.

All paths below are relative to `starter_v0/` unless stated otherwise.

## 1. Read the evidence before editing

Read the repository-level `README.md`, `RULES.md`, `RUBRIC.md`, `SUBMISSION.md`, and any applicable `AGENTS.md`, then inspect:

- [v1 run](../runs/v1_B_base_openrouter_20260915T184934819325.json)
- [v0 run](../runs/v0_B_base_openrouter_20260915T183118119938.json)
- [Current prompt](system_prompt.md), [tool declarations](tools.yaml), [report](REPORT.md), and [version log](version_log.csv)
- `tools/__init__.py`, `tools/lookup_user/tool.py`, `tools/inspect_device/tool.py`, `tools/check_service_status/tool.py`, and `tools/clarify/tool.py`

The recorded v1 baseline is:

| Metric | v1 |
|---|---:|
| Cases passed | 24/30 |
| Case accuracy | 0.80 |
| Tool routing accuracy | 0.8667 |
| Argument accuracy | 0.80 |
| Multi-turn accuracy | 1.00 |
| Provider error cases | 0 |
| Measured cases | 30/30 |

Provider/model: `openrouter` / `openai/gpt-4o-mini`. H12, M05 and M09 changed from FAIL to PASS in v1; all 21 original passes were retained.

### Four target failures and their causes

| Case | Actual v1 behavior | Required behavior | Why the declaration needs improvement |
|---|---|---|---|
| H04_user_routing | Calls lookup_user(employee_id=EMP-1003), then inspect_device(asset_id=EMP-1003); inspection returns asset_not_found | Only lookup_user; its result includes assigned_assets | The model confuses employee identifiers with asset identifiers and treats an assigned-device listing as a diagnostic request |
| H10_missing_asset | Calls inspect_device(asset_id=laptop, check=network); returns asset_not_found | clarify(response_type=text), asking for the asset ID | A generic device noun is being treated as an actual identifier |
| H11_missing_employee | Calls lookup_user(employee_id=Sales); returns employee_not_found | clarify(response_type=text), asking for the employee ID | A department is being treated as an employee identifier |
| H19_ambiguous_environment | Calls check_service_status(service=email, environment=staging) for an ambiguous environment | clarify(response_type=choice, options=[production, staging]) | A plausible interpretation is being treated as a confirmed environment |

H19 illustrates why a successful tool response does not prove correct behavior: the service exists, but the user never selected that environment.

## 2. Hypothesis and scope

**Hypothesis:** explaining tool ownership, valid identifier sources, and when to call `clarify` in the tool declarations will fix these four cases while retaining v1's confirmation and multi-turn behavior.

Use a **tool-description-only experiment** first. The existing Python functions perform the requested lookup; the observed failures originate in tool selection and argument construction. Tool descriptions are the appropriate first place to explain these distinctions. Changing the system prompt at the same time would make it harder to assess the effect of the declarations.

Do not change names, parameter types, enums, defaults, required fields, tool registry, evaluator, provider settings, or fixed cases for this experiment. Avoid new regex constraints: they do not explain when to ask the user and may introduce schema compatibility issues. Keep diagnostic `check` guidance for the planned v3 experiment; H13 and H17 remain separate targets.

If inspection reveals an actual execution defect, document the evidence and treat it as a separate change rather than silently expanding this experiment. Do not add a prompt change merely to predict a better score before obtaining the v2 result.

## 3. Preserve the measured v1 artifacts

Before making edits:

1. Read `prompt_hash` and `tools_hash` from the v1 run.
2. Compute SHA-256 hashes of the current `artifacts/system_prompt.md` and `artifacts/tools.yaml`. Confirm both match that run. If either differs, inspect the diff and preserve the user's newer work; do not mislabel it as v1.
3. Save byte-for-byte backups as `artifacts/system_prompt_v1.md` and `artifacts/tools_v1.yaml`. If a backup already exists, verify its content instead of overwriting a different file.
4. Preserve `artifacts/system_prompt_v0.md` and both recorded runs.

The tools hash was identical in v0 and v1, so `tools_v1.yaml` also preserves the declarations used for v0. Keep the active `system_prompt.md` byte-identical to v1 throughout this experiment.

## 4. Files to adjust

| File | Planned change |
|---|---|
| `artifacts/tools.yaml` | Improve descriptions for clarify, lookup_user, inspect_device and check_service_status, including the relevant parameter descriptions |
| `artifacts/system_prompt_v1.md` | Add verified backup before editing |
| `artifacts/tools_v1.yaml` | Add verified backup before editing |
| `artifacts/REPORT.md` | Add v2 hypothesis, target failures, changes and validation criteria; mark results pending |
| `artifacts/version_log.csv` | Add the actual v2 record only after the user supplies a completed run |

### A. lookup_user and employee_id

Explain that this tool returns the directory/account record **and assigned asset IDs** for a known employee ID. A request to list an employee's assigned devices is already served by this tool; it does not automatically require device diagnostics.

The identifier must come from explicit conversation context or a trusted structured internal result. Do not turn a name, department, job title, or device description into an employee ID. When the required identifier is absent or ambiguous, call `clarify` with a text question and wait.

Do not prohibit all lookup-plus-inspection combinations: if the user independently requests diagnostics for an identified device, both tools may be appropriate. H18 currently passes and must remain supported.

### B. inspect_device and asset_id

Explain that this tool inspects an identified company asset. It requires the actual asset ID, not an employee ID, manufacturer/model name, or generic device noun. Use a known identifier from the conversation or an appropriate trusted structured result; do not fabricate one or substitute a sample ID.

When the asset ID is missing or ambiguous, ask for it using `clarify(response_type=text)` before attempting inspection. Listing assigned devices and inspecting their condition are distinct requests.

Keep the `check` parameter's description, enum and default unchanged for v2 so diagnostic scope remains an independent v3 experiment.

### C. check_service_status and environment

Explain these three situations explicitly:

1. A supported environment is specified or established in the conversation: use it, honoring the latest correction.
2. No environment is specified or established: retain the documented `production` default.
3. The user explicitly names an ambiguous or unsupported environment: ask which supported environment they mean using `clarify(response_type=choice, options=[production, staging])`. Do not silently map it to a supported value or apply the default over the ambiguity.

Use general wording rather than copying the H19 sentence or creating a case-specific mapping. Preserve valid environment carryover in M02 and explicit staging selection in H06.

### D. clarify and its parameters

Explain that clarification is needed before dependent calls when required information is missing or ambiguous:

- `text`: request an absent identifier.
- `choice`: resolve ambiguity among supported values; supply the actual choices in `options`.
- `yes_no`: request approval of the current ticket details, consistent with the v1 prompt.

The question should identify exactly what is missing or needs confirmation. After asking, wait for the user's reply instead of issuing the dependent lookup, diagnostic or write with guessed inputs.

Do not make clarification mandatory for every request. Valid identifiers and established context should continue to enable direct tool calls.

### Writing constraints

- Keep the existing Vietnamese description style and concise wording; use YAML block scalars if descriptions become difficult to quote safely.
- Case IDs and observed examples belong in the report, not in the prompt or tool descriptions.
- Preserve all nine tools and their implementation-compatible schemas.
- Do not read or print `.env` contents. Use the user's existing environment for the later run.

## 5. Record the prepared experiment

In `REPORT.md`:

1. Fill the v2 row with `tools.yaml`, the hypothesis above, and baseline `case_accuracy=0.80`.
2. Mark the after metric and run path as **Pending evaluation**.
3. Add the four v1 failures to the failure analysis, clearly distinguishing observed v1 behavior from proposed v2 fixes.
4. Record that the v1 prompt and tool schema structure are preserved.
5. Credit AI assistance accurately; leave individual reflections to the team members.

Keep v0/v1 evidence intact. Do not populate an invented v2 score, timestamp, run filename, author or successful outcome in the CSV.

## 6. Local checks before handing back to the user

Use the existing Python environment to perform read-only checks without provider calls:

- Load `tools.yaml` with the project's `load_tool_declarations` and convert it with `to_openai_tools`.
- Verify all nine names are unique and match `TOOL_FUNCTIONS` in `tools/__init__.py`.
- Compare the parsed declarations with `tools_v1.yaml`: after recursively removing `description` fields from both structures, they should be identical. This verifies that only descriptions changed.
- Verify only the four intended tools have changed descriptions; specifically check `inspect_device.check` is unchanged.
- Verify the active prompt still matches the v1 `prompt_hash`, both backups match their recorded hashes, and active `tools.yaml` now has a different hash.
- Review `git diff` and run `git diff --check`; confirm no fixed evaluation case, run JSON or unrelated source file changed.

These checks establish artifact integrity, not improved model behavior. Do not add a test suite just to check this reversible description edit. If dependencies are unavailable, report which local checks remain pending rather than claiming they passed.

## 7. Required user handoff after implementation

Once the edits and local checks are finished, tell the user:

> v2 is ready for evaluation. I updated tool descriptions for identifier routing and clarification, preserved the v1 prompt, saved the v1 backups, and documented the pending experiment. Please run the following command from `starter_v0/` with your existing Python environment activated.

```bash
python run_eval.py --provider openrouter --model openai/gpt-4o-mini \
  --version v2 --suite base --eval-cases data/eval_base.json
```

Then explain that the user should provide the generated v2 JSON path for review, and that the agent will compare it with v1 before preparing v3. Summarize the local checks actually completed. Do not claim that target cases are fixed until the run has been reviewed.

`--suite` is only a label; the explicit `--eval-cases data/eval_base.json` ensures the same 30 cases are evaluated. The run must load the active files, not their backup copies.

## 8. Follow-up when the user supplies the v2 run

1. Confirm provider/model, dataset and artifact hashes correspond to this experiment. Verify `provider_error_cases == 0` and `measured_cases == total_cases == 30` before using the run as scored evidence.
2. Compare all case IDs with v1 and report newly passing, still failing and regressed cases.
3. Inspect each target's calls, arguments and results:
   - H04: only the expected directory lookup; no employee ID passed to device inspection.
   - H10/H11: a relevant text clarification and no guessed lookup/inspection.
   - H19: a choice question with production/staging, with no assumed status lookup.
4. Check preservation of H12/M05/M09 confirmation, M07 cancellation, H18 legitimate combined tools, M01/M03/M04/M08 corrected or supplied identifiers, and H06/M02 environment selection/carryover.
5. Review tool errors across all cases. Separate routing success from successful execution and do not infer an absence of filesystem writes solely from an automatic score.
6. Report H13/H17 independently. They are not targeted by v2, but any incidental improvement or regression must still be recorded.
7. Update the report and append a v2 CSV record using the actual run's `artifact_version`, `prompt_hash`, `tools_hash`, relative run path and observed metrics. Use 0.80 as the before case accuracy. Leave unknown author attribution for the user/team to fill.
8. Decide the next experiment from the evidence. If clarification problems persist, explain that outcome honestly before considering a short prompt rule in a subsequent measured revision.

**Acceptance target:** the four target failures pass and all 24 v1 passes are retained. Fixing exactly those four would yield 28/30, or approximately 93.33%; this is a conditional target, not a prediction or a recorded result. Full safety and successful ticket creation after confirmation still require separate adversarial and live-chat evidence.
