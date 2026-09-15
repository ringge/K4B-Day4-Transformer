# TEAM — Day04, K4-L3B

**Làm nhóm.** Mỗi người tự viết và commit phần INDIVIDUAL của mình.

## Thông tin bài nộp

- Tên nhóm: Transformer
- Người đại diện / MSSV: Trần Kim Phương - 2A202602565
- Tên repo: `K4B-Day4-Transformer`
- URL repo, nhánh nộp, commit chốt: 
    + URL repo: https://github.com/ringge/K4B-Day4-Transformer
    + nhánh: `main`
    + commit chốt: 
- Deadline áp dụng và link thông báo đổi hạn nếu có:

## Thành viên

| Họ và tên | MSSV | GitHub | Vai trò và công việc | File/commit/PR |
|---|---|---|---|---|
| Trần Kim Phương | 2A202602565 | [ringge](https://github.com/ringge/) | Team lead – Điều phối nhóm, tổng hợp kết quả và hoàn thiện báo cáo | [Điền file/commit/PR] |
| Trần Gia Thành | 2A202602626 | [gtee2004](https://github.com/gtee2004/) | Teammate – Thực hiện phần việc được phân công, chạy kiểm thử và cập nhật kết quả | [Điền file/commit/PR] |
| Nguyễn Minh Thái | 2A202602726 | [aoi36](https://github.com/aoi36) | Teammate – Thực hiện phần việc được phân công và đóng góp vào quá trình đánh giá agent | [Điền file/commit/PR] |

## Nhận xét chung

- **Kết quả và bằng chứng:** Trên bộ 30 ca cơ bản, tỷ lệ đạt tăng từ 70% ở [v0 (21/30)](starter_v0/runs/v0_B_base_openrouter_20260915T183118119938.json), lên 80% ở v1, 86,67% ở v2 và 100% ở [v3 (30/30)](starter_v0/runs/v3_B_base_openrouter_20260915T200944073522.json). Ở v3, độ chính xác chọn công cụ, tham số và hội thoại nhiều lượt trên bộ cơ bản đều đạt 100%. Với [bộ đánh giá do nhóm tự viết](starter_v0/runs/v3_B_group_openrouter_20260915T201828323453.json), agent đạt 7/10 ca (70%), trong đó hội thoại nhiều lượt đạt 4/5 ca (80%). Các lượt chạy trên đều đo đủ số ca và không có lỗi từ nhà cung cấp. Chi tiết thay đổi và kiểm tra hội thoại thực tế được lưu trong [REPORT.md](starter_v0/artifacts/REPORT.md) và [transcript v3](starter_v0/transcripts/v3_openrouter_20260915T202732739240.transcript.json).
- **Thay đổi hiệu quả nhất:** Vòng sửa v3 kết hợp làm rõ quy tắc trong [system_prompt.md](starter_v0/artifacts/system_prompt.md) với mô tả công cụ trong [tools.yaml](starter_v0/artifacts/tools.yaml), giúp sửa cả 4 ca còn lỗi ở v2 và giữ nguyên 26 ca đã đạt. Cụ thể, agent dùng nội dung sự cố đã có để hỏi xác nhận tạo ticket bằng `clarify(response_type=yes_no)`, đồng thời truyền `category` đúng chủ đề khi gọi `search_kb`. Quy tắc xác nhận trước khi tạo ticket được bổ sung từ v1 cũng là cải tiến quan trọng để tránh thực hiện thao tác ghi khi người dùng chưa duyệt chi tiết.
- **Giới hạn còn lại:** Bộ đánh giá của nhóm vẫn có 3 ca mang kết quả `passed: false`: `S23_lookup_user_missing_info`, `S24_check_and_ticket_wrong_boundary` và `M24_create_ticket_missing_info`. S23 và M24 bị chấm lỗi vì gọi `clarify` trong khi đáp án yêu cầu `no_tool`; riêng M24 kỳ vọng hỏi mã tài sản nhưng agent hỏi xác nhận ticket, trong khi `asset_id` là tham số tùy chọn của `create_ticket`. S24 chỉ gọi `check_service_status`, thiếu `create_ticket` so với đáp án, nhưng yêu cầu tạo ngay của đáp án chưa phù hợp với quy tắc phải xác nhận trước trong prompt. Vì vậy, nhóm cần rà soát sự thống nhất giữa đáp án, schema công cụ và quy tắc nghiệp vụ, rồi chạy lại để phân biệt lỗi của agent với điểm chưa phù hợp của bộ kiểm thử. Ngoài ra, S22 và M22 được chấm đạt về công cụ/tham số nhưng kết quả công cụ trả về `asset_not_found`; điểm tự động chưa chứng minh yêu cầu đã được giải quyết thành công. Kết quả 100% trên bộ cơ bản cũng chưa đủ để khẳng định agent xử lý tốt mọi tình huống mới.
- **Cách phân công và tích hợp:** Trần Kim Phương điều phối, tổng hợp kết quả và thực hiện vòng cải tiến prompt/công cụ v3 (commit `ac5da4e`); Trần Gia Thành chạy kiểm thử, bổ sung kết quả v3 cho bộ an toàn và bộ của nhóm (commit `2181933`); Nguyễn Minh Thái cập nhật báo cáo và bằng chứng hội thoại thực tế (commit `96f58e6`, `a30310e`). Các phần được tích hợp trên nhánh `main` của repo chung; báo cáo liên kết tới các tệp kết quả và transcript để đối chiếu nhận xét với hành vi thực tế của agent.

## INDIVIDUAL

Sao chép mục này cho từng thành viên.

### Họ và tên — MSSV

- Phần việc và file/commit/PR:
- Quyết định, khó khăn và cách xử lý:
- Điều đã học:
- AI/công cụ đã dùng và cách kiểm tra:
- Thời điểm đã tự nộp URL repo chung trên VLearn:
