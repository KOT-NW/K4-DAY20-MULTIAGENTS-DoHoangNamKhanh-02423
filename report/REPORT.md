# Báo cáo Lab: Self evolving Agentic

> Sao chép tệp này thành `report/REPORT.md` (đã làm ở Phần 0) và điền dần qua các Phần của lab. Xóa các dòng hướng dẫn dạng trích dẫn (bắt đầu bằng `>`). Văn phong kỹ thuật, ngắn gọn, mọi nhận định đi kèm số liệu hoặc bằng chứng. Trong buổi học: điền mục 1 đến 7 (bản nháp). Sau buổi học: hoàn thiện mục 8 đến 10.

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| (điền) | (điền) | toàn bộ |

- Mô hình: `LAB_MODEL=deepseek:deepseek-flash`, nhiệt độ `LAB_TEMPERATURE=0`, `recursion_limit=60`.
- Deep Agents 0.7.21; chạy trong Docker (image `python:3.12-slim`) trên Windows; shell của tác tử là `/bin/sh`.
- Ngân sách: ~18 lần chạy tác vụ học + 9 lần chạy tác vụ đánh giá (gồm các lần bị loại do lỗi cô lập; xem mục 9).
- Commit của tag `freeze`: `fc50e94` (commit `hypotheses` trước đó: `1299d9b`).

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
| code-learn | rule_regression_tests | E | "RULE: add tests/test_regressions.py with one test function per bug you fixed (at least 3)" |
| code-learn | rule_changelog | E | "RULE: record each fix in CHANGELOG.md under the heading '## Unreleased' ..." |
| data-learn | rule_money_in_cents | E | "RULE: money values in answer.json are integer cents (1606.67 USD is written 160667)." |
| data-learn | rule_meta_block | E | "RULE: answer.json has an object `meta` = {"source": ..., "rows_in": ..., "rows_used": ...}" |
| data-learn | rule_clean_csv | E | "RULE: write workspace/clean.csv with the header order_id,timestamp_utc,region,amount_cents ..." |
| logs-learn | rule_service_names | E | "RULE: service names ... lower-case with '-' replaced by '_'" |
| logs-learn | rule_sorted_errors | E | "RULE: `errors` is sorted by service, then by timestamp_utc, ascending." |
| logs-learn | rule_schema_header | E | "RULE: the top-level object has "schema_version": 2 and "generated_by": "log-triage"." |

Nhận xét: **nhóm E (quy ước tổ chức) chiếm 8/9 check thất bại**; ngoại lệ là `tests_not_modified` (nhóm A - vi phạm chỉ dẫn rõ ràng). Bằng chứng phủ định cho nhóm A–D: check **kỹ thuật** đạt 17/18 (code-learn 6/7, data-learn 5/5, logs-learn 6/6) - tác tử xử lý dữ liệu/log/test đúng phần lớn; nút thắt là quy ước không có trong đề. `rule_type_hints` (từng fail khi shell bị hỏng) nay **đạt**, cho thấy việc chạy được test giúp ích. Skill quy trình tổng quát có thể phòng ngừa nhóm E, nhưng kết quả thực nghiệm cho thấy skill đã không làm được (xem mục 6, 8).

## 5. Điều kiện `subagents` (Phần 2.3)

- Các subagent đã định nghĩa (tên, vai trò, lý do thiết kế): `explorer` (chỉ đọc và báo cáo sự thật), `implementer` (thực hiện thay đổi, chạy test), `reviewer` (kiểm tra độc lập, không sửa). Ba vai tách biệt để cô lập ngữ cảnh và có bước kiểm chéo.
- `subagent_calls` ở từng tác vụ: `subagents` learn = code-learn 1, data-learn 0, logs-learn 1; `subagents` eval = code-eval 1, data-eval 3, logs-eval 1. Ở `baseline`, mọi tác vụ = 0 (tác tử chính tự làm).
- Thông tin khi giao việc: tác tử chính gọi `task` với subagent `reviewer`/`explorer` để kiểm chứng; subagent chạy stateless (chỉ thấy prompt được gửi). `trace.md` chỉ hiện lời gọi `task` và báo cáo cuối.
- Ảnh hưởng token/thời gian: `subagents` mean 425k token so với `baseline` 153k (~2.8×), thời gian dài hơn rõ rệt (code-learn 223s so với 53s).

## 6. Self-evolving: skill do curator sinh (Phần 3)

- Số lần chạy curator, số skill bị xóa và lý do: chạy curator **1 lần** (sau khi sửa cô lập), sinh **3 skill**, **không xóa** skill nào (cả ba đều hợp lệ, tổng quát, không lộ đáp án).

| Skill | Tổng quát hay riêng cho tác vụ học? | Đúng hay sai (nêu chỗ sai nếu có) | Độ dài, `description` và `skills_read` ở Phần 3.4 |
|---|---|---|---|
| output-contract-compliance | Tổng quát: checklist đối chiếu mọi yêu cầu đầu ra (đường dẫn, header, schema, thứ tự trường, đơn vị, metadata). | Đúng, khớp các `detail` `rule_*` (cents, chuẩn hóa, sắp xếp, meta). | 13 dòng; description "Use when a task specifies required output files, schemas..."; được đọc (`skills_read=3`). |
| protected-files-and-new-artifacts | Tổng quát: không sửa tệp bảo vệ; tạo artifact mới (test hồi quy, changelog) đúng nơi. | Đúng, nhắm đúng nhóm E của code-learn. | 12 dòng; description "Use when a task forbids changing certain files or requires adding new files..."; được đọc. |
| final-compliance-verification | Tổng quát: biến mọi quy tắc thành checklist pass/fail và tự kiểm trước khi kết thúc. | Đúng; hướng dẫn kiểm chứng, không gây hại. | 13 dòng; description "Use before declaring a task done..."; được đọc. |

## 7. Kết quả so sánh (Phần 4.3, 4.4)

> Dán nội dung `report/table.md` và kết quả `python scripts/check_breakdown.py`. Nêu các lần chạy có `error` hoặc `skills_modified = true` (nếu có) và cách xử lý.

```text
| Task | baseline | subagents | skills-auto |
|---|---|---|---|
| code-learn | 7/10 | 6/10 | 7/10 |
| data-learn | 5/8 | 5/8 | 5/8 |
| logs-learn | 6/9 | 6/9 | 6/9 |
| code-eval | 6/11 | 6/11 | 6/11 |
| data-eval | 5/9 | 5/9 | 5/9 |
| logs-eval | 6/10 | 6/10 | 6/10 |
| **Mean score - learning tasks** | 0.66 | 0.63 | 0.66 |
| **Mean score - evaluation tasks** | 0.57 | 0.57 | 0.57 |
| **Mean tokens per run** | 153,152 | 425,159 | 239,888 |
| **Runs that read a skill** | 0/6 | 0/6 | 6/6 |
```

```text
condition     role    technical  house rules  mean tokens  read a skill
baseline      eval     17/18         0/12         130,996      0/3
baseline      learn    17/18         1/9          175,308      0/3
subagents     eval     17/18         0/12         426,408      0/3
subagents     learn    17/18         0/9          423,911      0/3
skills-auto   eval     17/18         0/12         235,874      3/3
skills-auto   learn    17/18         1/9          243,902      3/3
```

Ghi chú: không lần chạy nào có `error`; `skills_modified=false` ở mọi lần `skills-auto`; `python scripts/verify_freeze.py` báo `OK` (6 runs). Check kỹ thuật đạt 17/18 ở mọi điều kiện/vai trò; khác biệt chỉ nằm ở check quy ước.

## 8. Phân tích

> Trả lời từng câu bằng số liệu từ mục 7 và bằng chứng từ vết. Kết quả âm hoặc không có khác biệt vẫn hợp lệ nếu được phân tích tốt.

1. So với `baseline`: **không điều kiện nào cải thiện**. Trên tác vụ **học**, `skills-auto` bằng baseline (mean 0.66), `subagents` thấp hơn (0.63; code-learn 6 so với 7). Trên tác vụ **đánh giá**, cả ba điều kiện bằng nhau (0.57). Không có điều kiện nào cải thiện học mà không cải thiện đánh giá, nên **không có dấu hiệu overfitting** - nhưng cũng không có lợi ích nào để quá khớp.
2. Tách kỹ thuật/quy ước (mục 7): check **kỹ thuật** = 17/18 ở mọi điều kiện/vai trò. House rules: baseline học 1/9, `skills-auto` học 1/9 → **skill không giúp thêm quy ước nào**. Trên eval, mọi điều kiện 0/12; các quy ước **mới** của eval (`rule_version_bump`, `rule_sorted_keys_format`, `rule_source_line`) không được skill giúp.
3. Skill **được đọc nhưng không hiệu quả**: `skills_read=3` ở mọi run `skills-auto`, nhưng `data-learn/rule_money_in_cents` vẫn fail dù `output-contract-compliance` ghi rõ "apply unit ... exactly at output time" - skill được đọc nhưng tác tử **không làm theo**. Không có check nào `skills-auto` vượt baseline (bằng chứng âm rõ ràng).
4. Chi phí: mean token baseline 153k, `subagents` 425k (~2.8×), `skills-auto` 240k. Hiệu quả điểm/token tốt nhất là `baseline`. Đa tác tử **không đáng** chi phí: cùng điểm eval (0.57) mà token gần gấp ba.
5. Rò rỉ/quá khớp: **không rò rỉ** - curator chỉ đọc run `role=="learn"`, `validate_skill` từ chối skill chứa `eval_markers()`, 3 skill không nêu id tác vụ/tên tệp riêng/đáp án. **Không quan sát được quá khớp** vì skill vốn không cải thiện gì.
6. Nhiễu: cùng bộ skill, tác vụ học ở Phần 3.4 (bản `results/skills-auto-dev`) là code 8/10, data 5/8, logs 6/9; sau đóng băng là code 7/10, data 5/8, logs 6/9. Chênh lệch code-learn **-1** (≈0.03 mean) với CÙNG skill → nhiễu. Vậy mọi chênh lệch ±1 check trong bảng mục 7 là nhiễu; kết luận hợp lệ là **các điều kiện tương đương trong sai số**.

## 9. Hạn chế và tính hợp lệ

> Nêu ít nhất 3 hạn chế và ảnh hưởng của từng hạn chế đến kết luận (ví dụ: chỉ 3 tác vụ mỗi vai trò, mỗi cấu hình chạy một lần, nhiễu của mô hình, tác vụ do giảng viên thiết kế sẵn quy ước, chỉ một mô hình).

1. **Số tác vụ nhỏ (3 mỗi vai trò):** mỗi check là một điểm rời rạc; chênh lệch 1 check (≈3-4% điểm) nằm trong nhiễu, nên chỉ kết luận chắc cho chênh lệch lớn (code-learn 6→8).
2. **Mỗi cấu hình chạy một lần** (trừ learn của skills-auto có bản Phần 3.4): không đo được phương sai; ước lượng nhiễu chỉ từ một cặp lặp lại, nên độ tin cậy của bảng bị giới hạn.
3. **Chỉ một mô hình** (`deepseek-flash`, nhiệt độ 0): kết quả không tổng quát cho mô hình khác; mô hình mạnh có thể đã ghi nhớ quy ước.
4. **Quy ước do giảng viên thiết kế sẵn:** "house rules" có thể đoán/memorize, làm điểm phụ thuộc vào việc mô hình có đọc được tài liệu ẩn hay không.
5. **Tính hợp lệ phụ thuộc cô lập (hai lỗi harness đã gặp):** (a) bản đầu shell đọc được `tasks/*/check.py` (điểm 9/9 giả) → vá bằng chạy shell user `nobody` + chmod `ROOT` 700; (b) bản vá đầu thiếu `/usr/sbin` trong `PATH` nên mọi lệnh shell chết (`runuser: not found`), làm hỏng 21 lần chạy → sửa (dùng đường dẫn tuyệt đối của `runuser`) rồi chạy lại toàn bộ. Nếu không phát hiện và vá cả hai, kết luận sẽ sai.

## 10. Kết luận

> Tối đa 5 câu. Chỉ khẳng định điều số liệu hỗ trợ. Nêu một đề xuất cải tiến tiếp theo.

Với harness đã sửa, **không điều kiện nào vượt baseline**: `skills-auto` bằng baseline (0.66 học / 0.57 eval), `subagents` thấp hơn trên học (0.63) với token gần gấp ba. Đa tác tử **không đáng chi phí** trong thí nghiệm này. Self-evolving **không cải thiện**: curator sinh 3 skill hợp lệ và tác tử đọc cả 3 (`skills_read=3`) nhưng không sửa thêm được check quy ước nào - skill được đọc mà không được làm theo. Kết quả **không ủng hộ H1/H2** (kỳ vọng cải thiện) nhưng **ủng hộ H3** (eval ≤ học) và khớp cảnh báo của SkillsBench/SkillEvolBench rằng skill do mô hình tự sinh trung bình không có lợi. Đề xuất: lặp ≥3 lần để tách nhiễu, cho subagent dùng skill (6d), và red-team curator (6c).

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
- Ghi chú khác: **Hai lỗi cô lập đã gặp.** (a) `LocalShellBackend` không giam shell (`virtual_mode` vô hiệu với shell): agent `ls -R /lab`, `cat tasks/*/check.py` → điểm giả 9/9. Vá bằng cách chạy shell user `nobody` (`runuser -u nobody -p`) + `chmod 700` repo trong lúc chạy, sandbox `chmod 777`. (b) Bản vá đầu thiếu `/usr/sbin` trong `PATH` nên `runuser: not found` → shell chết ở cả 21 lần chạy; sửa bằng đường dẫn tuyệt đối `runuser` rồi chạy lại toàn bộ. Tag `freeze` cuối cùng = `fc50e94`.
