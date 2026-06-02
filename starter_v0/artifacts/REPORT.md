# Day 04 Lab v2 Report — Research Agent

## Team

- Team: Zone6 Team 3
- Members: Lê Thiên Khang 2A202600726, Nguyễn Thụy Như Quỳnh 2A202600557.
- Provider/model: openrouter / openai/gpt-4o-mini.

## Final Metrics

- Final version: v3
- Final artifact_version: v3+p5bb531a3fcb9+tc3175e01710f
- Best base run file: v3_B_base_openrouter_20260602T153132204653.json
- Base case accuracy: 1.00
- Base tool routing accuracy: 1.00
- Base argument accuracy: 1.00
- Group eval run file: v3_B_group_openrouter_20260602T153211845926.json
- Group eval accuracy: 1.00
- Chat transcript file: v3_openrouter_20260602T152630066981.transcript.json

## Version Evidence

Fill from `artifacts/version_log.csv` and `runs/*.json`.

| Version | Changed Artifact | Hypothesis | Metric Before | Metric After | Run File |
|---|---|---|---:|---:|---|
| v0 | baseline | Initial prompt and tools configuration | 0.00 | 0.75 | v0_B_base_openrouter_20260602T150703968910.json |
| v1 | system_prompt.md | Addressing out-of-scope tasks and clarify tool boundaries | 0.75 | 0.85 | v1_B_base_openrouter_20260602T151058773184.json |
| v2 | system_prompt.md | Optimizing routing, query simplification, and multi-turn parameter consistency | 0.85 | 0.95 | v2_B_base_openrouter_20260602T151312416650.json |
| v3 | tools.yaml; tools/__init__.py; system_prompt.md | Integrating new weather tool, location mapping, and switching boundaries | 0.95 | 1.00 | v3_B_base_openrouter_20260602T153132204653.json |

## Failure Analysis

Use actual failures from `results[*].result.failures`.

| Case ID | Failure Type | Actual Tool Calls | What Failed | Fix |
|---|---|---|---|---|
| R10_missing_handle | missing_info | None (called no tools) | Guessed handle or returned general text | Instructed to use clarify with response_type=text |
| R12_confirm_before_send | wrong_boundary | send (direct execution) | Executed send tool without confirming first | Instructed to call clarify with response_type=yes_no |
| M06_switch_tool | wrong_tool | lookup + social_search (parallel) | Kept old tool in history and called both | Instructed to completely abandon previous tool on switch |
| G08_multiturn_switch_weather_to_tweets | wrong_tool | social_search (instead of timeline) | Routed to social search on transition from weather | Standardized timeline routing rule for specific names |

## Team Eval Cases

List at least 5 cases added to `data/eval_group.json`.

| Case ID | What It Tests | Expected Tool/Behavior | Result |
|---|---|---|---|
| G01_weather_hanoi | Weather lookup for Vietnamese city mapping to English | weather(location="Hanoi") | PASS |
| G02_weather_missing_location | Missing location on weather query triggers clarify | clarify(response_type="text") | PASS |
| G03_out_of_scope_cooking | Out of scope recipe instruction refusal | no_tool (behavior: refuse) | PASS |
| G04_confirm_send_telegram | Telegram send request triggers confirmation check | clarify(response_type="yes_no") | PASS |
| G08_multiturn_switch_weather_to_tweets | Switching from weather search to Sam Altman's timeline | timeline(screenname="sama") | PASS |

## Live Chat Evidence

Use `transcripts/*.transcript.json`.

| Turn | User Request | Tool Calls | Version Evidence | Outcome |
|---|---|---|---|---|
| 1 | Thời tiết Hà Nội hôm nay ra sao? | weather(location="Hanoi") | v3 | Retrieved weather info: "nắng nhẹ với nhiệt độ khoảng 35°C" |
| 2 | À nhầm, cho mình xem thời tiết ở Sài Gòn nhé | weather(location="Ho Chi Minh City") | v3 | Retrieved weather info: "mưa rào với nhiệt độ khoảng 31°C" |
| 3 | Đăng thông tin thời tiết này lên Telegram giúp mình | clarify(response_type="yes_no", question="...") | v3 | Prompted user for confirmation: "Bạn có muốn đăng bản tin thời tiết này lên Telegram không?" |
| 4 | Có, gửi đi | send(confirmed=true, text="...") | v3 | Called send tool with confirmed=true; caught expected environment error since keys are placeholder |

## Bonus Evidence

Only fill if your team did bonus.

| Bonus | Evidence File | What Worked | Risk / Guardrail |
|---|---|---|---|
| send (Telegram) | tools/send/tool.py | Requires yes_no confirmation and confirmed=True parameter | Direct writes block without confirmation |
| arXiv/company policy | tools/policy/tool.py; tools/papers/tool.py | arXiv academic lookup and internal policies indexing | Guardrailed via domain specificity |
| UI | frontend/src/components/chat/Sidebar.tsx | Web interface with chat history, logs and trace | None |

## Reflection

- Which fixes belonged in `system_prompt.md`?
  Fixes related to semantic mapping (e.g. mapping "Hà Nội" to "Hanoi"), strict multi-turn routing constraints (e.g. discarding previous tool upon explicit user switch), and output boundaries (e.g. write confirmation) belong in the system prompt.
- Which fixes belonged in `tools.yaml`?
  Accurate types, descriptive parameter descriptions (e.g. location format), and required lists belong in `tools.yaml` to ensure the schema is correctly interpreted by the LLM.
- Which failure needed manual review instead of automatic grading?
  Natural language output variations (e.g. how the weather report is summarized or the exact phrasing of clarification questions) need manual review to assess the conversational quality, whereas tool routing is perfectly suited for auto-grading.
- What would you improve next?
  We would add actual Telegram Bot tokens for live message sending testing and extend the weather tool to support a 5-day forecast.
