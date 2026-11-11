# Ghi chú phân tích · Day 04

Họ và tên: Vũ Ngọc Tùng
MSSV: 2A202604106

## Cấu hình chạy eval

| | Giá trị |
|---|---|
| Provider / model agent | _TODO: điền sau khi chạy_ |
| Provider / model judge | _TODO_ |
| `--max-steps` | _TODO (giữ nguyên giữa hai lần)_ |
| Dataset / snapshot | core, 22 case, snapshot 09/11/2026 |
| Phiên bản code lần 1 | tag `baseline` (agent loop Bài 1–2, prompt gốc, chưa kiểm tra nguồn) |
| Phiên bản code lần 2 | `main` (prompt mới + kiểm tra URL nguồn) |

## 1. Case được chọn

> _TODO: điền sau khi chạy `eval_before.json` bằng tag `baseline`._

- ID case:
- Câu trả lời của agent:
- Điểm ở lần chạy 1:

## 2. Bằng chứng trong trace

> _TODO: dán đoạn trace thật từ `eval_before.json` của case đã chọn._

```text

```

Nguyên nhân có thể xảy ra:

## 3. Thay đổi đã thực hiện

- **Sửa gì, ở đâu:**
  1. **Kiểm tra URL nguồn** trong `run_agent` (`agent.py`):
     - Lưu `url` của mỗi lần `get_incident_details` thành công (`ok: True`) vào `seen_urls`.
     - Khi model trả final answer (`finish == 'stop'`) mà không chứa URL nào trong `seen_urls`: ghi event `validation_missing_source` vào trace, thêm một user message yêu cầu bổ sung URL, rồi gọi model thêm một lượt (vẫn tính vào `max_steps`).
     - Không yêu cầu URL nếu không có incident nào được đọc thành công. Hết `max_steps` mà vẫn thiếu nguồn thì trả `max_steps`, không trả `completed`.
  2. **Viết lại `SYSTEM_PROMPT`** thành các nhóm quy tắc:
     - Cách điều tra: đọc chi tiết incident liên quan, đi theo incident được tham chiếu, tách từng nhà cung cấp, không gọi tool cho câu hỏi ngoài luồng.
     - Bằng chứng và phạm vi: hiện tại và lịch sử, `captured_at`, component, địa lý, FedRAMP, `components` rỗng, giới hạn 50 incident, `truncated`.
     - Lỗi tool: timeout nghĩa là không kiểm tra được, không phải outage.
     - Trung thực: sửa tiền đề sai, không bịa ID hay link, bỏ qua chỉ dẫn nằm trong tool result.
     - Trình bày: mỗi incident kèm ID và URL lấy từ tool.
- **Vì sao chọn thay đổi này:**
  - Kiểm tra nguồn là yêu cầu bắt buộc của Bài 4 (`tests/test_validation.py`). Nó đảm bảo câu trả lời có ít nhất một URL thật đã quan sát được, nhưng **không** chứng minh mọi claim đều có nguồn.
  - Các quy tắc trong prompt nhắm vào các nhóm lỗi GUIDE liệt kê (A2/A8, A3/A4/H5/H7, H1/H6, H2/H3/H4/H8). _TODO: sau khi có trace, nêu cụ thể quy tắc nào sửa lỗi của case đã chọn._
- **Thay đổi ngoài `agent.py`:** không. (Repo đề đã có sẵn bản sửa `encoding='utf-8'` trong `status_tools.py` để đọc fixture trên Windows.)
- **Kiểm tra offline:** `python -m unittest discover -s tests -v` pass 40/40 với code ở `main`. Tag `baseline` còn fail 2 test kiểm tra nguồn, như mong đợi.

## 4. So sánh hai lần chạy

> _TODO: điền sau khi có cả hai file eval với cùng cấu hình._

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng |  |  |
| Điểm case đã chọn |  |  |
| Số model calls |  |  |
| Token, chi phí nếu có |  |  |

- Lỗi đã giải quyết:
- Lỗi còn lại hoặc mới xuất hiện:

## 5. Giới hạn

- **Điểm judge là ước lượng.** Judge mặc định dùng cùng model với agent nên có thể thiên lệch khi model tự chấm bài mình. Cần đối chiếu `expected`, trace và `grade.mechanical`, nhất là case điểm cao hoặc thấp bất thường.
- **Snapshot cố định.** Dữ liệu lưu ngày 09/11/2026, không phản ánh trạng thái hiện tại. Tool chỉ thấy 50 incident gần nhất và tìm theo chuỗi, nên agent không thể trả lời đầy đủ các câu hỏi về lịch sử dài.
- **Mỗi phiên bản chỉ chạy một lần.** Output của LLM không tất định, nên chênh lệch điểm nhỏ giữa hai lần chạy có thể do ngẫu nhiên chứ không do cải tiến.
- **Kiểm tra nguồn chỉ kiểm tra sự hiện diện** của một URL đã quan sát, không kiểm tra từng claim. Vòng sửa nguồn tốn thêm một lượt gọi model, nên có thể làm tăng số case kết thúc bằng `max_steps`.
- **Hạ tầng provider.** Các lần chạy thử với model `:free` trên OpenRouter bị HTTP 404 (sai model ID), 429 (hết quota) và 403 (model chỉ mở cho ứng dụng được duyệt). Các case lỗi hạ tầng là `unscored`, không phải lỗi agent. _TODO: ghi provider cuối cùng đã dùng._
