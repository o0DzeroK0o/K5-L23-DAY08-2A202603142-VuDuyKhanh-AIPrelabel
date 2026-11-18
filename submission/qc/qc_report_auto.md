# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 110 đối tượng.
- Đã kiểm 5/5 ảnh mẫu.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 14/110 = 12.7% | 110 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 5/5 = 100.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 445/520 = 85.6% / 43/520 = 8.3% / 32/520 = 6.2% | 520 box máy |
| Máy: added | 30/518 = 5.8% | 518 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 9 | img_003.jpg, img_014.jpg, img_031.jpg |
| sai_vi_tri_kich_thuoc | 3 | img_003.jpg, img_031.jpg, img_035.jpg |
| sai_lop | 1 | img_003.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 5 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **100%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 100.0% > 2%
- lỗi hệ thống: 'thua' chiếm 9/14 lỗi

Cây quyết định: Hold (QC chưa đủ / guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
