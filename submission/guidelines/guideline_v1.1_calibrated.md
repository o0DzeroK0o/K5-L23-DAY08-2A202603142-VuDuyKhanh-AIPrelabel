# Bổ sung label contract v1.1 — 2A202603142 - Vũ Duy Khanh

Bản đề xuất đối chiếu với [label contract v1.0](../../guideline/guideline_v1.0_label_contract.md). Ba mục dưới đây lấy từ những ca đã có ảnh, log hoặc đầu ra mask để học viên xác nhận.

## 1. Câu hỏi mở có đề xuất

| Câu hỏi v1.0 | Đề xuất bổ sung | Ví dụ | Mã ca |
|---|---|---|---|
| Mục 6.7. Có đặt một điểm để chấp nhận box máy ngay? | Dùng `conf` để xếp thứ tự kiểm. Trước khi giữ box, xem lớp và bốn cạnh trên ảnh gốc, kể cả với box có điểm cao. | [img_001](../../data/images/img_001.jpg). Box máy là `truck` (0,8758), nhãn gold của xe bán tải là `car`. | CAL-02 |
| Mục 6.4. Xử lý người dưới mui xe ba bánh. | Giữ mốc thấy tối thiểu khoảng 20% người của v1.0. Dựa vào phần đầu, thân và tay chân thấy được để xác định người, rồi gán riêng. Với mask xe, kiểm đường biên ngay chỗ tay, chân hoặc lưng chạm cabin trước khi giữ kết quả SAM. Ca còn phân vân về quy tắc thì đánh dấu `can_review`. | [img_013](../../data/images/img_013.jpg), [overlay SAM](../masks/tuktuk_img013_sam2.jpg). | CAL-03 |

Các mục 6.1, 6.2, 6.3, 6.5 và 6.6 vẫn là câu hỏi mở. Ba ca được ghi nhận ở đây chưa đủ để thử các quy tắc tương ứng trên nhiều ảnh.

## 2. Ca máy sai lớp

| Ảnh | Hiện tượng | Bổ sung cách kiểm lớp ở mục 2 |
|---|---|---|
| [img_013](../../data/images/img_013.jpg) và [img_004](../../data/images/img_004.jpg), CAL-01 | YOLO26 có box gợi ý `car` trên tuk-tuk. | Khi gặp xe cabin hở hoặc có mái, nhìn phần đầu xe, bánh nhìn thấy và khung gắn liền để phân biệt với ô tô. Gán `three_wheeler` khi hình dạng tuk-tuk rõ. Đánh dấu `can_review` khi góc chụp che mất dấu hiệu phân biệt. |

## 3. Lịch sử thay đổi

| Phiên bản | Nội dung | Ca nguồn |
|---|---|---|
| v1.0 | Label contract của giảng viên. | |
| v1.1 (đề xuất) | Làm rõ cách kiểm xe ba bánh, cách dùng điểm `conf` và vùng biên người lái. | CAL-01, CAL-02, CAL-03 |
