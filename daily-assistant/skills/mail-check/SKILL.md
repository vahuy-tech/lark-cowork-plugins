---
name: mail-check
description: Review công việc Lark Mail 3 ngày gần nhất cho BOD/CEO — đọc từng mail, lọc noise, đánh giá mail nào cần sếp quan tâm và đề xuất hướng phản hồi/xử lý. Triggers "check mail", "kiểm tra mail", "check mail 3 ngày", "review mail công việc", "mail gần đây có gì".
version: 1.0.0
last_updated: 2026-09-25
---

# mail-check

Bản review nhanh hộp thư Lark cho người ra quyết định (Mr. Huy / BOD).
Mục tiêu KHÔNG phải dọn inbox (đó là `inbox-zero`), mà là: **đọc mail
công việc 3 ngày qua → lọc bỏ noise → chỉ ra mail nào cần sếp quan tâm
→ gợi ý hướng phản hồi/xử lý**, kết lại bằng danh sách ưu tiên.

## Khi kích hoạt
- "check mail", "kiểm tra mail", "check mail 3 ngày", "review mail công việc".
- Mặc định cửa sổ = **3 ngày gần nhất** (hôm nay − 3 ngày, giờ Asia/Ho_Chi_Minh, UTC+7).
  Nếu người dùng nói số ngày khác ("check mail 5 ngày") thì theo con số đó.

## Công cụ
Dùng MCP `mcp__lark__lark_api` (passthrough, `as: "user"`) — đi qua MCP nên
KHÔNG bị chặn egress như tải file trực tiếp. Draft dùng
`mcp__lark__lark_mail_draft_create`. (Bản CLI tương đương: `mail +triage`,
`mail +messages`, `mail +draft-create`.)

## Workflow
1. **Liệt kê INBOX** (mới → cũ), phân trang tới khi phủ đủ cửa sổ ngày:
   `GET /open-apis/mail/v1/user_mailboxes/me/messages?folder_id=INBOX&page_size=20`
   (`page_size` tối đa 20). Lấy `page_token` để sang trang.
2. **Lấy header** từng message (song song nhiều call một lượt):
   `GET .../messages/{id}` với `jq`:
   `.data.message | {s:.subject, f:.head_from.name, a:.head_from.mail_address, d:.internal_date}`.
   `internal_date` là epoch ms → so với ngưỡng cửa sổ để cắt.
3. **Lọc noise** (không cần sếp đọc): MB Bank thông báo giao dịch tự động,
   Grab (báo cáo doanh số/quảng cáo), newsletter (Brands Vietnam, VietJet,
   Google, Genesia, Techcombank QC, Traveloka), AWING booking-change tự động,
   LnD auto-registration, iPOS policy. GIỮ mọi mail `@hakico.vn` + đối tác/
   luật/ngân hàng có hành động (MB Bank "chờ phê duyệt" ĐANG treo là ngoại lệ
   phải giữ).
4. **Đọc nội dung** khi cần (thread nóng, cần đánh giá): lấy `body_plain_text`
   rồi giải mã — **body là base64URL**, decode đúng:
   `.data.message.body_plain_text | gsub("\\s";"") | gsub("-";"+") | gsub("_";"/") | @base64d`
   (KHÔNG dùng `@base64d` thẳng — sẽ lỗi ở ký tự `-`/`_`). Reply thường
   trích cả thread bên dưới → đọc 1 mail nắm được cả mạch.
5. **File/link đính kèm:**
   - File đính kèm (.xlsx/.pdf) tải qua `.../attachments/download_url` → host
     `internal-api-drive-stream-sg.larksuite.com` **bị policy egress chặn (403)**.
     Không tải được → nhờ người dùng **upload file vào chat** nếu cần soi số liệu.
   - **Link Lark trong thân mail thì ĐỌC ĐƯỢC qua MCP** (không bị chặn): trích href
     từ `body_html` (cùng cách decode base64url), rồi:
     Doc: `GET /open-apis/docx/v1/documents/{token}/raw_content` (`jq .data.content`).
     Sheet: `wiki/{token}` → `GET /open-apis/wiki/v2/spaces/get_node?token=..&obj_type=wiki`
     lấy `obj_token` → `sheets/v3/.../sheets/query` lấy sheet_id →
     `GET /open-apis/sheets/v2/spreadsheets/{token}/values/{sheetId}!A1:..`.
6. **Đánh giá & phân nhóm** theo mức ưu tiên cho người ra quyết định:
   - 🔴 **Cần CEO quyết / gấp** (tranh chấp, pháp lý, nhân sự chủ chốt, tiền, sự cố).
   - 🟡 **Chờ CEO duyệt** (đề xuất/kế hoạch/ngân sách chờ phê duyệt).
   - 🟢 **Team tự lo / FYI** (báo cáo, recap, thông báo, đào tạo).
   Không xếp mail của VIP/đối tác/đầu mối quan trọng vào noise.
7. **Mỗi mail cần quan tâm** ghi 3 phần gọn: *(a) là gì · (b) vì sao cần sếp ·
   (c) hướng phản hồi/xử lý đề xuất*.
8. **Kết**: danh sách **ưu tiên xử hôm nay** + đề nghị soạn draft cho mục sếp chọn.

## Hard rules
- **Read-only mặc định.** Không xóa/gắn nhãn/di chuyển mail.
- **Không tự gửi.** Chỉ tạo *draft* (`lark_mail_draft_create`); `lark_mail_send`
  chỉ khi sếp xác nhận rõ. Ký tên theo người dùng (mặc định "Huy").
- Trả lời **tiếng Việt, ngắn – scannable** (bảng + gạch đầu dòng), không đổ nguyên văn mail.
- Lỗi auth → báo `lark-cli auth login`, dừng sạch.
- Ngày "hôm nay" lấy từ context phiên, không đoán.

## Output
Một block review gọn: nhóm 🔴/🟡/🟢 + nhận định 1 dòng + danh sách ưu tiên.
Không phải bức tường mail.
