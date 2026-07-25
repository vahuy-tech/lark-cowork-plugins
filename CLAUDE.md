# CLAUDE.md

Hướng dẫn cho Claude Code khi làm việc trong repo này.

## Second Brain (bắt buộc)

Người dùng (Huy) có một **"Second Brain"** — kho ghi chú/kiến thức cá nhân — lưu trên
**Lark Drive**, trong thư mục có token `BYscfRhCSlk7nAdaI0llWp3Xgge`
(workspace `hakicovn.sg.larksuite.com`).

Với **MỌI câu hỏi** mang tính tra cứu, ghi nhớ, hoặc hỏi về công việc / dự án / ghi chú
của người dùng: **TRƯỚC TIÊN hãy tự động tìm trong Second Brain** — không cần người dùng
chỉ định tài liệu cụ thể.

Cách tra cứu:
1. Dùng `lark_doc_search` để tìm doc theo từ khóa liên quan tới câu hỏi.
2. Dùng `lark_doc_fetch` để đọc nội dung doc phù hợp nhất.
3. (Nếu cần) liệt kê file trong thư mục brain qua `lark_api`
   `GET /open-apis/drive/v1/files` với `folder_token=BYscfRhCSlk7nAdaI0llWp3Xgge`.

Nguyên tắc trả lời:
- **Ưu tiên** trả lời dựa trên ghi chú của người dùng; **trích rõ tên/nguồn doc**.
- Nếu **không tìm thấy** trong Second Brain, nói rõ *"không có trong Second Brain"* rồi
  mới dùng kiến thức chung.
- Không đợi người dùng nhắc "tìm trong Lark" — **tự động tra cứu mỗi lần**.
