# Thông tin học viên

- Họ và tên — Vũ Duy Khanh
- MSHV — 2A202603142
- Lớp — 2A
- Tài khoản GitHub — `o0DzeroK0o`
- [Repo bài nộp](https://github.com/o0DzeroK0o/K5-L23-DAY08-2A202603142-VuDuyKhanh-AIPrelabel)
- Commit chốt bài — 
- Máy — Windows, Python 3.12.10 và PyTorch CPU
- CVAT local — 2.83.0, task `batch-review` #6, dùng nhập và xuất COCO 1.0.

## Kết quả tóm tắt

- Gold — 4 ảnh, 89 vật tính điểm và 24 vùng bỏ qua. Thời gian 90 phút, trung bình 22,5 phút/ảnh.
- Lượt đối chiếu bằng model local — Giữ 445/520, sửa khung 43/520, bỏ 32/520 box máy. Bổ sung 30/518 box sau đối chiếu. Bản xuất CVAT có 531 box trên 24 ảnh.
- So với gold — Precision 53/68 (77,9%), recall 53/89 (59,6%).
- Bắt bẫy — 11/12 (91,7%) lọt một bẫy thiếu box tại `img_006.jpg`.
- Thời gian duyệt bằng mắt — 
- QC — Codex đã kiểm trực quan đủ 5 ảnh. Ghi 14 lỗi trên 110 đối tượng (12,7%) theo cách tính của script, 5/5 ảnh có lỗi (100%).
- Trạng thái — **Rework** vì tỷ lệ ảnh lỗi vượt ngưỡng 2% và có 9/14 lỗi là box thừa. [Báo cáo](submission/REPORT.md)
