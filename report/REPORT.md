# Báo cáo Lab: Self evolving Agentic

> Sao chép tệp này thành `report/REPORT.md` (đã làm ở Phần 0) và điền dần qua các Phần của lab. Xóa các dòng hướng dẫn dạng trích dẫn (bắt đầu bằng `>`). Văn phong kỹ thuật, ngắn gọn, mọi nhận định đi kèm số liệu hoặc bằng chứng. Trong buổi học: điền mục 1 đến 7 (bản nháp). Sau buổi học: hoàn thiện mục 8 đến 10.

## 1. Thông tin nhóm và cấu hình

| Họ tên | Mã sinh viên | Phần đóng góp |
|---|---|---|
| (điền) | (điền) | toàn bộ |

- Mô hình: `LAB_MODEL=deepseek:deepseek-flash`, nhiệt độ `LAB_TEMPERATURE=0`, `recursion_limit=60`.
- Deep Agents 0.7.21; chạy trong Docker (image `python:3.12-slim`) trên Windows; shell của tác tử là `/bin/sh`.
- Ngân sách: đã dùng khoảng 15 lần chạy tác vụ học (không tính 6 lần chạy bị loại trước khi vá cô lập) + 12 lần chạy tác vụ đánh giá.
- Commit của tag `freeze`: (điền hash sau khi tạo tag).

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
(dán bảng ở đây)
```

## 8. Phân tích

> Trả lời từng câu bằng số liệu từ mục 7 và bằng chứng từ vết. Kết quả âm hoặc không có khác biệt vẫn hợp lệ nếu được phân tích tốt.

1. So với `baseline`, điều kiện nào cải thiện điểm tác vụ **học**? Điều kiện nào cải thiện điểm tác vụ **đánh giá**? Có điều kiện nào cải thiện tác vụ học nhưng không cải thiện tác vụ đánh giá? Nếu có, đó là dấu hiệu gì?
2. Tách điểm thành check kỹ thuật và check quy ước (`rule_`). Skill do curator sinh giúp nhóm check nào? Check quy ước **mới** của tác vụ đánh giá có được skill giúp không, và vì sao?
3. Dựa vào vết và `skills_read`, giải thích một check mà skill giúp đạt và một check mà skill không giúp (skill chưa được đọc, đọc nhưng không làm theo, skill thiếu hoặc sai).
4. Chi phí: so sánh số token trung bình giữa các điều kiện. Điều kiện nào có hiệu quả tốt nhất theo điểm trên mỗi token? Đa tác tử có đáng chi phí trong thí nghiệm này không?
5. Có dấu hiệu rò rỉ dữ liệu hoặc quá khớp nào trong skill sinh ra không? Nhóm đã phòng tránh như thế nào?
6. Nhiễu: so sánh điểm tác vụ học của cùng bộ skill ở Phần 3.4 (đã sao lưu) và sau đóng băng. Chênh lệch bao nhiêu? Nó cho biết điều gì về độ tin cậy của các chênh lệch trong bảng ở mục 7?

## 9. Hạn chế và tính hợp lệ

> Nêu ít nhất 3 hạn chế và ảnh hưởng của từng hạn chế đến kết luận (ví dụ: chỉ 3 tác vụ mỗi vai trò, mỗi cấu hình chạy một lần, nhiễu của mô hình, tác vụ do giảng viên thiết kế sẵn quy ước, chỉ một mô hình).

1.
2.
3.

## 10. Kết luận

> Tối đa 5 câu. Chỉ khẳng định điều số liệu hỗ trợ. Nêu một đề xuất cải tiến tiếp theo.

## Phụ lục

- Lệnh đã chạy (theo thứ tự):
- Thử thách mở rộng (nếu có): hướng chọn, kết quả, nhận xét.
- Ghi chú khác:
