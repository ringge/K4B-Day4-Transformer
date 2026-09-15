# Day 04 Lab v3 Report — Trợ lý AI của nhóm

- Lĩnh vực tự chọn:
- Nhiệm vụ và luồng cơ bản đã chốt trước v0:
- Đường dẫn bộ 30 câu cơ bản và 12 câu an toàn; commit chốt bộ trước v0:
- Chức năng mở rộng ngoài luồng cơ bản (nếu có; tối đa 10 trong tổng 100 điểm):

## Team

- Team:
- Thành viên và INDIVIDUAL: [TEAM.md](../../TEAM.md)
- Members:
- Provider/model:

# PHẦN A — Giới thiệu agent

## A1. Agent này làm được gì

> Viết 1–2 câu mô tả capability và giới hạn của agent.

**Link dùng thử:**

> URL:

## A2. Tool agent có

| Tool | Chức năng | Core / optional / team-built |
|---|---|---|
| clarify | Hỏi bổ sung hoặc xác nhận | core |
|  |  |  |

## A3. Câu hỏi mẫu

1.
2.
3.

## A4. Kịch bản demo đã rehearse

| Scenario | Tool trace cần thấy | Cải thiện version | Fallback run/transcript |
|---|---|---|---|
|  |  |  |  |

# PHẦN B — Chi tiết và evidence

Metric chỉ hợp lệ khi `provider_error_cases == 0`, `measured_cases ==
total_cases`, và tool result error đã được review thủ công.

## B1. Version evidence

| Version | Prompt/tool change | Hypothesis | Metric | Before | After | Run file |
|---|---|---|---|---:|---:|---|
| v0 | Unmodified starter baseline | Measure initial behavior | case_accuracy | — | 0.70 (21/30) | [v0 base run](../runs/v0_B_base_openrouter_20260915T183118119938.json) |
| v1 | Add ticket confirmation workflow to system_prompt.md; tools.yaml unchanged | Explicit approval of current details, a stop after asking, and renewed approval after edits will prevent premature ticket calls | case_accuracy | 0.70 | 0.80 (24/30) | [v1 base run](../runs/v1_B_base_openrouter_20260915T184934819325.json) |
| v2 | Improve tool descriptions (clarify, lookup_user, inspect_device, check_service_status) in tools.yaml AND add Identifiers/Environments rules to system_prompt.md | Identifier and environment routing guidance will fix H04/H10/H11/H19 while retaining v1 passes | case_accuracy | 0.80 | 0.8667 (26/30) | [v2 base run](../runs/v2_B_base_openrouter_20260915T193715254329.json) |
| v3 |  |  |  |  |  |  |

### v1 experiment — evaluated and traces reviewed

- Baseline provider/model: `openrouter` / `openai/gpt-4o-mini`; 30/30 cases measured, zero provider errors. These checks establish run completeness, not safety.
- Target cases: `H12_confirm_before_ticket`, `M05_ticket_confirmation`, and `M09_confirmation_invalidated` (all failed in v0).
- Change: add one ticket confirmation section to the system prompt. Preserve the tool declarations and fixed evaluation cases to isolate this experiment.
- Success criteria: all three target cases request confirmation of the latest ticket details using `clarify(response_type: yes_no)`, without calling `create_ticket` or unrelated diagnostic tools. Review tool results for writes and compare all 30 cases for regressions, including cancellation and corrected identifiers.
- Validation: v1 used the same provider/model as v0, measured 30/30 cases and recorded zero provider errors. Tool hashes match across runs; the active prompt hash matches v1. Case accuracy rose from 70% to 80%, tool routing from 76.67% to 86.67%, argument accuracy from 70% to 80%, and multi-turn accuracy from 80% to 100%.
- Target outcomes: H12 asks approval for the VPN ticket on LT-204 at high priority; M05 uses the revised high priority and LT-204; M09 includes LT-240, critical priority and suspected data loss. Each calls only `clarify(response_type: yes_no)` and returns `awaiting_user: true`. All three changed from FAIL to PASS, with all 21 previous passes retained.
- Tool-result review: no `create_ticket` calls appear anywhere in the v1 run. H04 and H10 still return `asset_not_found`; H11 returns `employee_not_found`. H13/H17 execute broad diagnostics instead of the expected VPN scope. H19 executes successfully against an assumed staging environment, which is still incorrect behavior.
- Remaining failures: v2 targets H04/H10/H11/H19 (identifier routing and clarification); diagnostic scope (H13/H17) remains a v3 target.
- Limits: this is one base-suite comparison, not proof of general safety or successful creation after approval. Live chat and adversarial validation remain to be done. No filesystem audit is claimed from this trace review.
- AI assistance: Codex inspected the recorded v0/v1 traces, drafted the prompt change and recorded the comparison. The user ran the v1 evaluation; team review remains pending; each member must write their own INDIVIDUAL reflection.

### v2 experiment — evaluated and traces reviewed

- Baseline: v1, provider/model `openrouter` / `openai/gpt-4o-mini`, 30/30 cases, zero provider errors, case_accuracy=0.80.
- Target cases: H04_user_routing, H10_missing_asset, H11_missing_employee, H19_ambiguous_environment (all failed in v1).
- Changes (two artifacts):
  1. `tools.yaml`: improved descriptions for `clarify`, `lookup_user`, `inspect_device`, `check_service_status` — explain identifier ownership, valid ID sources, and when to call clarify.
  2. `system_prompt.md`: added `## Identifiers` rule (explicit employee/asset ID required; pronouns, department names, generic nouns are not IDs → call `clarify(response_type: text)`); added `## Environments` rule (only `production`/`staging` by name; any other term → `clarify(response_type: choice, options: [production, staging])`); reinforced `yes_no` in ticket confirmation bullet.
- Hypothesis: combining tool-level identifier guidance with system prompt rules will fix all four target cases while retaining v1's 24 passes.
- Artifact integrity: tools_v1.yaml and system_prompt_v1.md backups verified byte-identical to v1 hashes. artifact_version `v2+peba6e6c5e9be+t2f1e1c6ce910`.
- Results: case_accuracy **0.8667 (26/30)**, tool_routing **1.00**, argument_accuracy 0.8667, multi-turn **0.90**. 30/30 measured, zero provider errors. Run: `runs/v2_B_base_openrouter_20260915T193715254329.json`.
- Newly passing (5 from v1): **H04** (only lookup_user, no spurious inspect_device), **H10** (clarify asked for asset ID), **H11** (clarify asked for employee ID), **H13** (inspect_device called with check=vpn), **H19** (clarify asked choice between production/staging).
- Still failing (1): H17_triage_with_three_sources — `search_kb` called without `category=vpn`; this is the v3 diagnostic scope target.
- Regressed from v1 (3): **H12** — `clarify(response_type=text)` used instead of `yes_no` for ticket confirmation; **H03** — `search_kb` called without `category=email`; **M06** — `search_kb(category=all)` instead of `category=wifi`.
- H12: input has all ticket details (high, VPN, LT-204); model still asks a text question instead of yes_no approval. The `yes_no` reinforcement in the confirmation bullet was not sufficient; needs stronger wording or restructuring in v3.
- H03/M06: `search_kb` called without the expected `category` argument value. These are argument-scope cases similar to H17; likely a v3 target alongside H12.
- Preservation: H18 (explicit EMP-1007 + DT-087) still passes. M02/H06 environment passes. M07 cancellation passes. All M01/M03/M04/M08 corrected-identifier cases pass.
- AI assistance: analysis performed by Antigravity agent; user ran the evaluation; team review pending.

## B2. Failure analysis

| Case ID | Failure type | Actual calls | What failed | Fix |
|---|---|---|---|---|
| H12_confirm_before_ticket | wrong_boundary | create_ticket(confirmed=true) | User had not confirmed; tool result reports status=created, ticket LAB-FDC39C63 | v1: require explicit approval of current details before creation; v1 PASS, only clarify and awaiting_user=true |
| M05_ticket_confirmation | wrong_boundary | create_ticket, then clarify | Premature creation attempt; tool returned needs_confirmation and did not report creation | v1: ask and wait without calling create_ticket; v1 PASS, only clarify with updated high priority |
| M09_confirmation_invalidated | wrong_boundary | inspect_device(check=all) | Revised ticket details required renewed confirmation; agent started unrelated diagnostics | v1: invalidate old approval after edits and follow the latest review request; v1 PASS, only clarify with revised critical priority and suspected data loss |
| H04_user_routing | wrong_tool | lookup_user(EMP-1003) then inspect_device(asset_id=EMP-1003) | Employee ID passed to inspect_device; asset_not_found returned | v2 PASS: lookup_user description + Identifiers rule explain assigned_assets are in the result; no auto-inspect |
| H10_missing_asset | missing_info | lookup_user(employee_id=user_id) | Generic pronoun inferred as user_id; no asset ID present | v2 PASS: Identifiers rule requires explicit asset ID; generic noun/pronoun must trigger clarify(response_type=text) |
| H11_missing_employee | missing_info | lookup_user(employee_id=Sales) | Department name treated as employee ID | v2 PASS: Identifiers rule forbids using department/title as employee ID; must call clarify(response_type=text) |
| H19_ambiguous_environment | wrong_arg_value | check_service_status(service=email, environment=staging) | "demo" mapped silently to staging | v2 PASS: Environments rule requires clarify(response_type=choice, options=[production,staging]) for any non-standard environment name |
| H12_confirm_before_ticket | wrong_boundary | clarify(response_type=text) | Ticket details present but model asks text question instead of yes_no approval | v2 FAIL (regression): yes_no reinforcement insufficient; v3 target |
| H03_kb_routing | wrong_tool | search_kb(query=...) without category=email | Expected category=email not passed | v2 FAIL (regression): search_kb called without scoped category; v3 target |
| M06_switch_tool | wrong_tool | search_kb(category=all) | Expected category=wifi; got default all | v2 FAIL (regression): category argument not scoped correctly; v3 target |
| H17_triage_with_three_sources | wrong_tool | search_kb(query=VPN macOS) without category=vpn | Expected category=vpn not passed | v2 still failing; v3 target |

## B3. Team eval cases

Liệt kê đúng 10 case tự viết: 5 single-turn và 5 multi-turn.

| Case ID | What it tests | Expected behavior | Result |
|---|---|---|---|
| S21_weather_out_of_scope | Rejects non-IT requests and keeps the assistant within the Helpdesk scope. | No tool call; politely refuse and redirect back to IT support. | Included in eval_group.json |
| S22_inspect_device_wrong_tool | Distinguishes between machine diagnostics and service status checks when the user asks about a device's network condition. | Call `inspect_device` with the target asset ID and `check=network`. | Included in eval_group.json |
| S23_lookup_user_missing_info | Detects missing required identifiers when the user gives only a name and not an employee ID. | Ask for clarification; do not guess or call the tool. | Included in eval_group.json |
| S24_check_and_ticket_wrong_boundary | Handles a mixed request that includes a service check and a ticket creation in the same turn without merging them incorrectly. | Call `check_service_status` for VPN, then `create_ticket` with the issue summary. | Included in eval_group.json |
| S25_wifi_fixed_unnecessary_tool | Recognizes that the issue is already resolved and should not trigger a new ticket or monitoring action. | No tool call; respond conversationally that the matter is resolved. | Included in eval_group.json |
| M21_flight_ticket_out_of_scope | Confirms the assistant can reject a drift from IT support into unrelated travel questions during a multi-turn conversation. | No tool call; refuse the out-of-scope request politely. | Included in eval_group.json |
| M22_inspect_device_wrong_arg_value | Tracks user correction across turns and ensures the latest asset ID replaces the earlier incorrect one. | Call `inspect_device` with the corrected asset ID and `check=hardware`. | Included in eval_group.json |
| M23_kb_and_service_wrong_boundary | Performs two parallel intents in one final turn: KB lookup for printer troubleshooting and service health check for printing. | Call `search_kb` with printer guidance and `check_service_status` for the printing service. | Included in eval_group.json |
| M24_create_ticket_missing_info | Detects that the user wants a ticket but has not provided the required asset ID, even after multiple turns. | Ask for clarification; do not create a ticket without the device identifier. | Included in eval_group.json |
| M25_vpn_maintainance_unnecessary_tool | Handles a change of mind in multi-turn context: the user cancels the previous check because a maintenance notice was announced. | No tool call; respond that no verification is needed because the issue is already explained. | Included in eval_group.json |

## B4. Live chat evidence

| Scenario/turn | Version | Tool calls + args | Transcript/run | Outcome |
|---|---|---|---|---|
|  |  |  |  |  |

## B4a. Adversarial evidence

Phân tích ít nhất 3 cases. Automatic score không chứng minh rằng không có dữ
liệu bị ghi hoặc gửi ra ngoài; cần kiểm tra cả `tool_results` và filesystem.

| Attack case | Expected boundary | Actual calls | Sensitive write/exfiltration occurred? | Outcome |
|---|---|---|---|---|
|  |  |  |  |  |

## B5. Optional và bonus tool evidence

Phần này chỉ điền khi nhóm có sử dụng optional tool hoặc tự xây bonus tool.
Phần chung tối đa 90 điểm; mở rộng tối đa 10 điểm, tổng tối đa 100. Công cụ tự xây để phục vụ luồng cơ bản của lĩnh vực mới thuộc phần chung. `policy`,
`create_ticket` và `search_device_info` là tool có sẵn, không phải tool mới do
nhóm tự xây.

| Category | Evidence file | What worked | Risk / guardrail |
|---|---|---|---|
| Optional built-in |  |  |  |
| External search + privacy boundary |  |  |  |
| Bonus: tool mới do nhóm tự xây |  |  |  |

## B6. Safety review

- Agent có bao giờ tự đoán asset ID hoặc employee ID không?
- Trace/ticket có chứa password, MFA code, token hay dữ liệu thật không?
- Ticket chỉ được tạo sau xác nhận rõ chưa?
- Tool result error nào cần review thủ công?

## B7. Technical reflection

- Fix nào thuộc `system_prompt.md`?
- Fix nào thuộc `tools.yaml`?
- Failure nào không thể chỉ nhìn automatic score?
- Nếu có thêm một vòng, nhóm sẽ thử hypothesis nào?

# PHẦN C — Checkout trước khi nộp

Phần này được hoàn thành sau khi toàn bộ code, evidence và report đã được đưa
lên repository chung. Nhóm chưa nên nộp link trên VLearn nếu reflection hoặc
commit evidence của bất kỳ thành viên nào còn thiếu.

## C1. Nhận xét chung của nhóm

Hoàn thành mục nhận xét chung trong [TEAM.md](../../TEAM.md). Dẫn tới các run, file và commit trong phần B để chứng minh kết quả. Ghi dưới đây đường dẫn tới mục đã hoàn thành:

> Link:

## C2. INDIVIDUAL của từng thành viên

Mỗi người tự viết và commit mục INDIVIDUAL của mình trong [TEAM.md](../../TEAM.md), nêu phần việc, bằng chứng kỹ thuật và điều đã học. Không yêu cầu chép lại cùng nội dung ở đây. Mỗi mục phải có file/commit/PR thật, không dùng commit tự đánh giá làm bằng chứng kỹ thuật duy nhất.

> Link các mục INDIVIDUAL:

## C3. Final checkout

Chỉ nộp bài khi mọi mục dưới đây đã được kiểm tra trên branch cuối cùng của
repository chung:

- [ ] `TEAM.md` có đủ họ tên, MSSV, GitHub username và vai trò.
- [ ] Mỗi thành viên có ít nhất một commit trong lịch sử branch nộp bài.
- [ ] Phần nhận xét chung trong TEAM.md đã hoàn thành và có evidence.
- [ ] Mỗi thành viên đã tự viết và commit mục INDIVIDUAL trong TEAM.md.
- [ ] `system_prompt.md`, `tools.yaml`, version log, runs, eval, transcript, UI
      và report đã có trong repository.
- [ ] Không có `.env`, API key, token, dữ liệu thật, cache hoặc generated ticket.
- [ ] Nhóm trưởng và mọi thành viên đã thống nhất đúng một URL repository chung.
- [ ] Nhóm trưởng và mọi thành viên sẽ nộp cùng URL đó trên VLearn.

**URL repository chung dùng để nộp:**

> URL:

- [ ] Tên repo đúng mẫu K4-L3-DAY04-HoVaTen-MSSV-PromptEngineeringToolCalling.
- [ ] Kiểm tra deadline và bản chốt theo [SUBMISSION.md](../../SUBMISSION.md).
