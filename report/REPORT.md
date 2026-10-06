# Báo cáo Lab: Self evolving Agentic

> Sao chép tệp này thành `report/REPORT.md` (đã làm ở Phần 0) và điền dần qua các Phần của lab. Xóa các dòng hướng dẫn dạng trích dẫn (bắt đầu bằng `>`). Văn phong kỹ thuật, ngắn gọn, mọi nhận định đi kèm số liệu hoặc bằng chứng. Trong buổi học: điền mục 1 đến 7 (bản nháp). Sau buổi học: hoàn thiện mục 8 đến 10.

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| (điền) | (điền) | toàn bộ |

- Mô hình: `LAB_MODEL=deepseek:deepseek-flash`, nhiệt độ `LAB_TEMPERATURE=0`, `recursion_limit=60`.
- Deep Agents 0.7.21; chạy trong Docker (image `python:3.12-slim`) trên Windows; shell của tác tử là `/bin/sh`.
- Ngân sách: 12 lần chạy tác vụ học hợp lệ (chưa tính 6 lần bị loại trước khi vá cô lập) + 9 lần chạy tác vụ đánh giá.
- Commit của tag `freeze`: `419b375` (commit `hypotheses` trước đó: `1299d9b`).

## 2. Giả thuyết (commit TRƯỚC tag `freeze`, Phần 4.0)

> Dự đoán điều kiện nào đạt điểm cao nhất trên **tác vụ đánh giá** và vì sao. Nêu căn cứ từ phân loại lỗi (mục 4) và từ tài liệu tham khảo. Điền cả ba dòng; `verify_freeze.py` kiểm tra điều này.

- H1 (subagents so với baseline): trên tác vụ đánh giá, `subagents` sẽ KHÔNG cao hơn `baseline`. Trên tác vụ học hai điều kiện hòa nhau (mean 0.63) nhưng đa tác tử tốn token hơn (mean 371k so với 215k, ~1.7×); cô lập ngữ cảnh khiến subagent chỉ thấy prompt được gửi nên dễ bỏ sót quy tắc.
- H2 (skills-auto so với baseline): trên tác vụ học, `skills-auto` cải thiện các check quy ước (`rule_*`) vì curator học trực tiếp từ trường `detail` (code-learn 7/10 so với 6/10; `skills_read=3`). Trên tác vụ đánh giá, mức cải thiện sẽ nhỏ hơn hoặc không còn vì eval thêm một quy ước mới chưa từng xuất hiện trong phản hồi học (nguy cơ overfitting theo SkillEvolBench).
- H3 (tác vụ học so với tác vụ đánh giá): điểm trung bình tác vụ đánh giá sẽ THẤP hơn tác vụ học ở cả ba điều kiện, vì tác vụ đánh giá dùng dữ liệu khác và thêm một quy ước `rule_` mới mà skill sinh từ tập học không bao phủ.

## 3. Làm quen Deep Agents (Phần 0.3)

1. Công cụ: tệp (`ls`, `read_file`, `write_file`, `edit_file`, `delete`, `glob`, `grep`), shell (`execute`) và giao việc (`task`). Công cụ chạy lệnh là `execute`.
2. Mô tả `task` nói subagent `general-purpose` "has access to all tools as the main agent" và chạy **stateless**: nó chỉ thấy prompt được gửi và trả về một báo cáo cuối, **không** kế thừa hội thoại của tác tử chính.
3. System prompt mặc định rỗng (`''`). Từ `task`: "Tell the agent whether to create content, analyze, or only research." Từ `execute`: "You MUST avoid using search commands like find and grep. Instead use the grep, glob tools to search."

## 4. Đường cơ sở và phân loại lỗi (Phần 2.2)

> Chỉ dùng tác vụ học. Mỗi dòng là một check thất bại.

| Tác vụ | Check thất bại | Nhóm lỗi (A-G) | Bằng chứng (trích ngắn từ `detail` hoặc vết) |
|---|---|---|---|
| code-learn | tests_not_modified | A | "the original files in tests/ must not be modified" (đề ghi rõ "Do not modify the existing files in tests/") |
| code-learn | rule_type_hints | E | "RULE: every public function ... has type annotations on all parameters and on the return value." |
| code-learn | rule_regression_tests | E | "RULE: add tests/test_regressions.py with one test function per bug you fixed (at least 3)" |
| code-learn | rule_changelog | E | "RULE: record each fix in CHANGELOG.md under the heading '## Unreleased' ..." |
| data-learn | rule_money_in_cents | E | "RULE: money values in answer.json are integer cents (1606.67 USD is written 160667)." |
| data-learn | rule_meta_block | E | "RULE: answer.json has an object `meta` = {"source": ..., "rows_in": ..., "rows_used": ...}" |
| data-learn | rule_clean_csv | E | "RULE: write workspace/clean.csv with the header order_id,timestamp_utc,region,amount_cents ..." |
| logs-learn | rule_service_names | E | "RULE: service names ... lower-case with '-' replaced by '_'" |
| logs-learn | rule_sorted_errors | E | "RULE: `errors` is sorted by service, then by timestamp_utc, ascending." |
| logs-learn | rule_schema_header | E | "RULE: the top-level object has "schema_version": 2 and "generated_by": "log-triage"." |

Nhận xét: **nhóm E (quy ước tổ chức) chiếm đa số tuyệt đối: 9/10 check thất bại**; ngoại lệ duy nhất là `tests_not_modified` (nhóm A - vi phạm chỉ dẫn rõ ràng). Bằng chứng phủ định cho các nhóm A–D: các check **kỹ thuật** (không bắt đầu bằng `rule_`) đạt 17/18 trên baseline (code-learn 6/7, data-learn 5/5, logs-learn 6/6) - tức tác tử vẫn xử lý dữ liệu bẩn/định dạng đúng phần lớn; nút thắt là các quy ước không có trong đề. Một skill quy trình tổng quát **có thể** phòng ngừa nhóm E vì `detail` phát biểu quy tắc; nhưng vì mỗi tác vụ có quy ước riêng, skill phải tổng quát hóa.

## 5. Điều kiện `subagents` (Phần 2.3)

- Các subagent đã định nghĩa (tên, vai trò, lý do thiết kế): `explorer` (chỉ đọc và báo cáo sự thật), `implementer` (thực hiện thay đổi, chạy test), `reviewer` (kiểm tra độc lập, không sửa). Ba vai tách biệt để cô lập ngữ cảnh và có bước kiểm chéo.
- `subagent_calls` ở từng tác vụ: `subagents` = code-learn 1, data-learn 1, logs-learn 1. Lưu ý: `baseline` cũng có 1 ở code-learn/data-learn do subagent `general-purpose` mặc định của Deep Agents (logs-learn baseline = 0).
- Thông tin khi giao việc: tác tử chính gọi `task` nhưng subagent chạy stateless (chỉ thấy prompt được gửi); `trace.md` chỉ hiện lời gọi `task` và báo cáo cuối, không thấy bước bên trong.
- Ảnh hưởng token/thời gian: `subagents` mean 371k token so với `baseline` 215k (~1.7×), thời gian dài hơn rõ rệt (code-learn 213s so với 61s).

## 6. Self-evolving: skill do curator sinh (Phần 3)

- Số lần chạy curator, số skill bị xóa và lý do: chạy curator **1 lần**, sinh **3 skill**, **không xóa** skill nào (cả ba đều hợp lệ, tổng quát, không lộ đáp án).

| Skill | Tổng quát hay riêng cho tác vụ học? | Đúng hay sai (nêu chỗ sai nếu có) | Độ dài, `description` và `skills_read` ở Phần 3.4 |
|---|---|---|---|
| verify-output-contract | Tổng quát: checklist đối chiếu mọi yêu cầu đầu ra (tên tệp, khóa, đơn vị, thứ tự, chuẩn hóa). | Đúng, khớp các `detail` `rule_*` (đơn vị cents, chuẩn hóa, sắp xếp). | 12 dòng; description nêu tình huống "output file/schema/format"; được đọc (`skills_read=3`). |
| deliver-all-artifacts | Tổng quát: liệt kê đủ mọi deliverable + áp chuẩn code (type hints, test mới, changelog). | Đúng, nhắm đúng nhóm E của code-learn. | 12 dòng; description nêu "multiple deliverables"; được đọc. |
| recover-from-tool-failure | Tổng quát: đổi công cụ khi lệnh lỗi, không lùng sục filesystem. | Đúng; thậm chí dạy tránh hành vi dò `/lab` gây lỗi quyền. | 10 dòng; description nêu "command/tool fails repeatedly"; được đọc. |

## 7. Kết quả so sánh (Phần 4.3, 4.4)

> Dán nội dung `report/table.md` và kết quả `python scripts/check_breakdown.py`. Nêu các lần chạy có `error` hoặc `skills_modified = true` (nếu có) và cách xử lý.

```text
| Task | baseline | subagents | skills-auto |
|---|---|---|---|
| code-learn | 6/10 | 6/10 | 8/10 |
| data-learn | 5/8 | 5/8 | 5/8 |
| logs-learn | 6/9 | 6/9 | 6/9 |
| code-eval | 7/11 | 6/11 | 8/11 |
| data-eval | 5/9 | 5/9 | 5/9 |
| logs-eval | 6/10 | 6/10 | 6/10 |
| **Mean score - learning tasks** | 0.63 | 0.63 | 0.70 |
| **Mean score - evaluation tasks** | 0.60 | 0.57 | 0.63 |
| **Mean tokens per run** | 190,475 | 355,879 | 210,323 |
| **Runs that read a skill** | 0/6 | 0/6 | 6/6 |
```

```text
condition     role    technical  house rules  mean tokens  read a skill
baseline      eval     17/18         1/12         165,989      0/3
baseline      learn    17/18         0/9          214,961      0/3
subagents     eval     17/18         0/12         340,266      0/3
subagents     learn    17/18         0/9          371,492      0/3
skills-auto   eval     17/18         2/12         203,112      3/3
skills-auto   learn    17/18         2/9          217,534      3/3
```

Ghi chú: không lần chạy nào có `error`; `skills_modified=false` ở mọi lần `skills-auto`; `python scripts/verify_freeze.py` báo `OK` (6 runs). Check kỹ thuật đạt 17/18 ở mọi điều kiện/vai trò; khác biệt chỉ nằm ở check quy ước.

## 8. Phân tích

> Trả lời từng câu bằng số liệu từ mục 7 và bằng chứng từ vết. Kết quả âm hoặc không có khác biệt vẫn hợp lệ nếu được phân tích tốt.

1. So với `baseline`: trên tác vụ **học**, chỉ `skills-auto` cải thiện (mean 0.70 so với 0.63), `subagents` hòa (0.63). Trên tác vụ **đánh giá**, `skills-auto` cải thiện (0.63 so với 0.60), `subagents` giảm (0.57). Phần cải thiện của `skills-auto` xuất hiện ở cả code-learn (6→8) và code-eval (7→8); data/logs phẳng ở cả hai. Vì vậy **không có** điều kiện cải thiện học mà không cải thiện đánh giá - không có dấu hiệu overfitting mạnh (nhưng mức eval chỉ +1 check, xem câu 6).
2. Tách kỹ thuật/quy ước (mục 7): check **kỹ thuật** giữ nguyên 17/18 ở mọi điều kiện; khác biệt nằm ở check **quy ước**. Baseline học 0/9 → `skills-auto` 2/9 (`rule_type_hints`, `rule_regression_tests`). Trên eval, baseline 1/12 → `skills-auto` 2/12. Các quy ước **mới** của eval (`rule_version_bump`, `rule_sorted_keys_format`, `rule_source_line`) **không** được skill giúp (vẫn fail) vì skill sinh từ tập học không chứa chúng.
3. Một check skill **giúp đạt**: `code-learn/rule_regression_tests` (baseline fail → skills-auto pass; chuyển sang `code-eval` cũng pass). Vết `skills-auto` cho thấy đọc đủ 3 skill (`skills_read=3`) và làm theo `deliver-all-artifacts` (tạo `tests/test_regressions.py`). Một check skill **không giúp**: `data-learn/rule_money_in_cents` vẫn fail dù đã đọc `verify-output-contract` - skill được đọc nhưng **không làm theo** (không đổi sang integer cents).
4. Chi phí: mean token baseline 190k, `subagents` 356k (~1.9×), `skills-auto` 210k. Hiệu quả điểm/token tốt nhất thuộc `skills-auto` (0.70 học / 0.63 eval với 210k). Đa tác tử **không đáng** chi phí trong thí nghiệm này: điểm eval thấp hơn baseline (0.57 so với 0.60) mà token gần gấp đôi.
5. Rò rỉ/quá khớp: **không rò rỉ** - curator chỉ đọc run `role=="learn"`, `validate_skill` từ chối skill chứa `eval_markers()`, và 3 skill không nêu id tác vụ/tên tệp riêng/đáp án. Quá khớp: **một phần** - skill giúp học (+2 check code) và chỉ chuyển một phần sang eval (+1 check code); quy ước mới của eval không được giúp.
6. Nhiễu: cùng bộ skill, tác vụ học ở Phần 3.4 (bản sao `results/skills-auto-dev`) là code 7/10, data 5/8, logs 6/9; sau đóng băng là code 8/10, data 5/8, logs 6/9. Chênh lệch duy nhất là code-learn **+1** (≈0.03 mean) với CÙNG skill → nhiễu. Do đó chênh lệch ±1 check trong bảng mục 7 không đáng tin; chỉ chênh lệch ≥2 check (code-learn 6→8) mới có ý nghĩa.

## 9. Hạn chế và tính hợp lệ

> Nêu ít nhất 3 hạn chế và ảnh hưởng của từng hạn chế đến kết luận (ví dụ: chỉ 3 tác vụ mỗi vai trò, mỗi cấu hình chạy một lần, nhiễu của mô hình, tác vụ do giảng viên thiết kế sẵn quy ước, chỉ một mô hình).

1. **Số tác vụ nhỏ (3 mỗi vai trò):** mỗi check là một điểm rời rạc; chênh lệch 1 check (≈3-4% điểm) nằm trong nhiễu, nên chỉ kết luận chắc cho chênh lệch lớn (code-learn 6→8).
2. **Mỗi cấu hình chạy một lần** (trừ learn của skills-auto có bản Phần 3.4): không đo được phương sai; ước lượng nhiễu chỉ từ một cặp lặp lại, nên độ tin cậy của bảng bị giới hạn.
3. **Chỉ một mô hình** (`deepseek-flash`, nhiệt độ 0): kết quả không tổng quát cho mô hình khác; mô hình mạnh có thể đã ghi nhớ quy ước.
4. **Quy ước do giảng viên thiết kế sẵn:** "house rules" có thể đoán/memorize, làm điểm phụ thuộc vào việc mô hình có đọc được tài liệu ẩn hay không.
5. **Tính hợp lệ phụ thuộc cô lập:** bản chạy đầu tiên bị vô hiệu vì shell của tác tử đọc được `tasks/*/check.py` (điểm 9/9 giả). Sau khi vá (chạy shell bằng `nobody`, chmod `ROOT` 700), baseline giảm còn đúng mức có lỗi. Nếu không phát hiện và vá, toàn bộ kết luận sẽ sai.

## 10. Kết luận

> Tối đa 5 câu. Chỉ khẳng định điều số liệu hỗ trợ. Nêu một đề xuất cải tiến tiếp theo.

Đa tác tử **không giúp**: `subagents` hòa baseline trên tác vụ học và thấp hơn trên eval (0.57 so với 0.60) nhưng tốn token gần gấp đôi. Self-evolving có hiệu quả **hạn chế nhưng thật**: `skills-auto` nâng điểm học (0.63→0.70) và eval (0.60→0.63), sửa được 2 quy ước code và chuyển 1 sang eval, nhưng không giúp data/logs hay quy ước mới của eval. Kết quả phù hợp H1 và H3, và phần lớn H2. Đề xuất: lặp mỗi cấu hình ≥3 lần để tách nhiễu, cho subagent dùng skill (hướng 6d), red-team curator (6c) và dùng sandbox cô lập thật.

## Phụ lục

- Lệnh đã chạy (theo thứ tự):
  1. `pytest` (29 passed); `python scripts/tour.py`
  2. `python -m lab.runner --condition baseline --tasks data-learn code-learn logs-learn`
  3. `python -m lab.runner --condition subagents --tasks learn`
  4. `python -m lab.curator`
  5. `python -m lab.runner --condition skills-auto --tasks learn`
  6. `git commit -m hypotheses`; `git commit --allow-empty -m "freeze skills"`; `git tag freeze`
  7. `python -m lab.runner --condition baseline --tasks eval`
  8. `python -m lab.runner --condition subagents --tasks eval`
  9. `python -m lab.runner --condition skills-auto --tasks all`
  10. `python -m lab.compare > report/table.md`; `python scripts/verify_freeze.py`; `python scripts/check_breakdown.py`
- Thử thách mở rộng (nếu có): chưa chọn; phát hiện bảo mật dưới đây là chất liệu trực tiếp cho hướng 6c (red team).
- Ghi chú khác: **Lỗ hổng cô lập.** `LocalShellBackend` không giam shell (`virtual_mode` vô hiệu với shell); agent đã `ls -R /lab`, `cat tasks/*/check.py` và đạt điểm giả 9/9. Đã vá trong `make_backend`/`run_task`: shell chạy bằng user `nobody` (`runuser -u nobody -p`) và `chmod 700` thư mục gốc repo trong lúc chạy, sandbox `chmod 777`. Sau vá, baseline trở về mức có lỗi đúng như thiết kế.
