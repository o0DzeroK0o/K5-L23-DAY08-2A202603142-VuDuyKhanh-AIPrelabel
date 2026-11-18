# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 520; đối tượng sau duyệt: 518; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 445 | 445/520 = 85.6% | 520 box máy |
| edited (sửa) | 43 | 43/520 = 8.3% | 520 box máy |
| deleted (xoá) | 32 | 32/520 = 6.2% | 520 box máy |
| added (vẽ thêm) | 30 | 30/518 = 5.8% | 518 đối tượng sau duyệt |

Trong số deleted + added, **2 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 28.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 239 | 16 | 13 | 10 |
| bicycle | 6 | 1 | 1 | 3 |
| motorbike | 110 | 21 | 7 | 10 |
| car | 50 | 0 | 4 | 3 |
| bus | 3 | 0 | 0 | 0 |
| truck | 4 | 0 | 2 | 1 |
| three_wheeler | 33 | 5 | 5 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 89 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 53 | 15 | 36 | 77.9% (KTC 95%: 67–86%) | 59.6% (KTC 95%: 49–69%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - loose: 3/3
  - extra: 3/3
  - missing: 2/3
  - lọt: img_006.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
