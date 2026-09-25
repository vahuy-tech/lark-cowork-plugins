---
description: Rà soát tất cả approval Lark trong 3 ngày gần nhất cho Mr. Huy (Chạm/Hakico)
---

# /check_ap — Rà soát approval 3 ngày

Khi người dùng gõ `/check_ap`, `check ap`, `check approval`, hoặc "rà soát approval" (không kèm số ngày cụ thể), thực hiện quy trình dưới đây. Nếu người dùng nêu số ngày khác (vd "2 ngày", "4 ngày"), dùng số đó thay cho 3.

## Bối cảnh cố định
- **Người duyệt (Sếp):** Võ Anh Huy — `open_id = ou_715fd1862fbfac91534dbb2c8933372c`
- **Ngưỡng tự đề xuất duyệt:** dưới **10.000.000đ** (`< 10tr`). Từ 10tr trở lên: chỉ liệt kê, để anh tự quyết.
- **Múi giờ:** +07. Tính cửa sổ [00:00 ngày (hôm nay − 2), 23:59:59 hôm nay] cho "3 ngày"; timestamps là **epoch mili-giây (string)**.
- **Công cụ:** `mcp__lark__lark_api` (nạp qua ToolSearch `select:mcp__lark__lark_api` nếu chưa có). Đọc dùng `as:"bot"`.

## Mã approval (approval_code) đã biết
| Loại | code |
|--|--|
| ĐỀ XUẤT THANH TOÁN | 3F2F164F-2B23-4B54-8FFF-DA36FAFB2035 |
| Đề xuất thanh toán (có in PDF) | 79B17EBB-1D22-4796-8797-54346F5A9981 |
| OFFICE - PHIẾU ĐỀ XUẤT MUA HÀNG | FD59CD4D-CF99-4996-988C-2C4B83B856D8 |
| OFFICE - PHIẾU ĐỀ XUẤT MUA HÀNG - MỚI | 94E22403-413C-4330-89DC-04E4ACAE1F38 |
| DUYỆT CÔNG VIỆC | 02FB6E76-0F52-4430-AEB8-E8F05AD9BA8E |
| HKC-ĐƠN XIN NGHỈ PHÉP | D6FE67A8-33F9-4925-9B67-5AD9D5B48173 |
| HKC-ĐƠN XIN NGHỈ PHÉP - Copy | 8D1BB77F-2882-40E9-8E20-9234438A8678 |
| HKC - ĐƠN XIN NGHỈ VIỆC | C495645A-1B04-4AA8-AF0E-551FE2F5286F |
| YÊU CẦU TUYỂN DỤNG | C6E88E9A-E1E0-4DA9-B4CB-DF323DDE8386 |

## Các bước

1. **Tính cửa sổ 3 ngày** (epoch ms, +07) bằng Bash/python.

2. **Đếm theo loại (toàn bộ, mọi trạng thái):** GET `/open-apis/approval/v4/instances` với mỗi `approval_code` + `start_time`/`end_time` + `page_size:100`. Cũng lấy danh sách mã hiện có bằng `tasks/search` (PENDING) rồi `unique_by(.code)` để bắt loại mới/đổi tên.

3. **Xác định phiếu ĐANG CHỜ chính anh:** POST `/open-apis/approval/v4/tasks/search` với body `{"task_status":"PENDING","user_id":"ou_715fd1862fbfac91534dbb2c8933372c"}`, params `{"page_size":100,"user_id_type":"open_id"}`; lọc `.instance.serial_id` bắt đầu bằng các ngày trong cửa sổ. Lấy `approval.code`, `instance.code`, `task.task_id`, `approval.name`, `serial`.

4. **Lấy chi tiết từng phiếu chờ anh** (GET instance, dùng `jq`):
   - Thanh toán/Mua hàng: số tiền = field `"Số tiền cần thanh toán"`; mua hàng lấy tổng `fieldList.ext[] type=="formula"`. Kèm `"Tên hàng hóa/hạng mục"` / `"Lý do đề xuất"`.
   - Duyệt công việc: `"Họ và tên"`, `"Phòng ban"`, `"Nội dung"`, `"Mô tả"`.
   - Nghỉ phép: field `"Leave widgets"` (name/start/end/reason).

5. **Trình bày** theo nhóm:
   - 🔴 Tiền chờ anh — **< 10tr** (bảng: serial, số tiền, nội dung) và **≥ 10tr** (để anh tự xem).
   - 🟣 Không liên quan tiền chờ anh (duyệt công việc / nghỉ phép / tuyển dụng) — kèm **đánh giá ngắn** (nên duyệt / cần xem gì).
   - 🟡 Tóm tắt phần đã qua anh (ở Kế toán) + phần chờ bộ phận khác + đã huỷ/từ chối.
   - Cảnh báo phiếu bất thường: số tiền lệch (pay ≠ tổng bảng kê), form nhập lỗi, phiếu trùng/nộp lại, phiếu tồn lâu.

6. **Kết thúc bằng đề nghị hành động:** hỏi anh có duyệt nhóm **< 10tr** (và các việc nhỏ/cấp thiết) không. **Chỉ duyệt sau khi anh xác nhận** (trừ khi anh đã nói "duyệt hết"/"duyệt luôn").

## Khi duyệt
POST `/open-apis/approval/v4/tasks/approve`, params `{"user_id_type":"open_id"}`, body:
`{"approval_code":"<code>","instance_code":"<inst>","task_id":"<task>","user_id":"ou_715fd1862fbfac91534dbb2c8933372c","comment":"Duyet"}`
- Dùng ĐÚNG `approval_code` của từng phiếu (thanh toán vs mua hàng vs nghỉ phép khác nhau).
- Lấy `task_id` mới nhất từ `tasks/search` ngay trước khi duyệt.
- Lỗi `1390001 "process has ended"` = phiếu đã qua bước của anh rồi (thường đã duyệt) → kiểm tra instance status, coi như xong.
- Lỗi `60009 "no operation permission"` = kiểm tra lại cặp instance_code/task_id; nếu vẫn lỗi, báo anh duyệt tay.
