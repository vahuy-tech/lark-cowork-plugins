# Giao Việc — Task Delegation Plugin

Giao việc chuyên nghiệp trên Lark — từ phân tích yêu cầu đến theo dõi kết quả.

## Skills

| Skill | Mô tả | Trigger |
|-------|--------|---------|
| **giao-viec** | Giao việc cho người khác — resolve người nhận, tạo task, gửi card thông báo | "giao việc cho...", "assign ... to ...", "nhờ ... làm ..." |
| **theo-doi-viec** | Theo dõi tiến độ task đã giao — overdue, chưa bắt đầu, sắp tới hạn | "việc đã giao", "check tiến độ", "ai chưa xong" |
| **giao-viec-tu-meeting** | Trích action items từ meeting → giao task hàng loạt | "giao việc từ meeting", "action items từ cuộc họp" |

## Quy trình giao việc (P1→P4→P2)

1. **Resolve người nhận** (P1) — `lark_contact_search` → `open_id`
2. **Tạo task** (P2) — `lark_task_create(dry_run)` → xem trước → confirm → commit
3. **Thông báo** (P4) — gửi interactive card cho người nhận qua IM
4. **Ghi nhận** — log vào Base nếu có delegation tracker

## Lark tools sử dụng

- `lark_contact_search` — tra cứu người nhận
- `lark_task_create` — tạo task (có dry_run)
- `lark_task_my` — xem task của mình
- `lark_im_card_send` — gửi card thông báo
- `lark_im_send` — gửi tin nhắn
- `lark_minutes_search` — tìm meeting minutes
- `lark_calendar_agenda` — xem lịch
- `lark_base_search` / `lark_base_record_upsert` — delegation tracker
- `lark_api` — task list (việc đã giao cho người khác)
