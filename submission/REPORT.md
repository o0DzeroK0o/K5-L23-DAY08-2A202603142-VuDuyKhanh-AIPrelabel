# Báo cáo Day 08 — 2A202603142 - Vũ Duy Khanh

## 1. Chọn model và ngưỡng

Trên 8 ảnh pilot, tăng ngưỡng YOLO26m từ 0,25 lên 0,50 làm số box giảm từ 215 xuống 139. Riêng `img_007.jpg` chỉ còn 11 trong 26 box ban đầu, nhiều người và xe nhỏ ở xa bị mất box. Còn ở `img_002.jpg`, YOLOE tìm được chiếc tuk-tuk xanh với điểm 0,9265 nhưng cũng gọi nhầm xe bán tải là xe ba bánh với điểm 0,3658. Hai ca này giải thích vì sao dùng một model để gợi ý người và xe thông dụng, rồi dùng model có prompt riêng cho xe ba bánh. [Log các lượt r01–r03](model_log.csv)

| Lớp | Model | Ngưỡng dùng khi gộp | Kết quả trên gold ở ngưỡng đó |
|---|---|---|---|
| `person` | YOLO26m (`r06`) | 0,30 | 25/44 vật được ghép. |
| `bicycle` | YOLO26m (`r06`) | 0,30 | 3/6 vật được ghép. |
| `motorbike` | YOLO26m (`r06`) | 0,30 | 10/23 vật được ghép. |
| `car` | YOLO26m (`r06`) | 0,30 | 7/8 vật được ghép. |
| `bus` và `truck` | YOLO26m (`r06`) | 0,30 | Gold có 1 xe buýt và 0 xe tải, dưới mốc 5 vật để chọn ngưỡng riêng. |
| `three_wheeler` | YOLOE-26s (`r07`) | 0,15 | Recall 7/7 (100%), precision 7/9 (77,8%). Prompt gồm `tuk-tuk`, `cyclo` và `cargo tricycle`. |

Ngưỡng 0,15 là mốc cao nhất vẫn đạt mục tiêu recall 90% cho `three_wheeler` trong lượt quét. Với bốn lớp đủ mẫu của YOLO26m, recall tốt nhất lần lượt là 30/44, 5/6, 16/23 và 7/8. Cả bốn vẫn thấp hơn mục tiêu nên script giữ ngưỡng mặc định. Hai lượt chạy CPU ở ngưỡng 0,05 sinh 1.669 box YOLO26m và 125 box YOLOE trên 36 ảnh. Khi gộp với `--conf 0.3 --thr "three_wheeler=0.15" --iou 0.6 --split "batch,gold"`, box YOLOE được ưu tiên và còn 606 box trên 28 ảnh. [Kết quả quét](qc/threshold_sweep.csv) · [Bảng chọn ngưỡng](qc/threshold_sweep.md)

Đổi ngưỡng cũng có giới hạn. Ở `img_001.jpg`, YOLO26m gọi một xe bán tải là `truck` với điểm 0,8758 dù box chồng lên box `car` gán tay với IoU 0,983. Đổi ngưỡng chỉ làm box đó được giữ hoặc bị loại. Số đo gold cũng cần đọc dè dặt vì cả chọn ngưỡng lẫn đo đều dùng cùng 4 ảnh.

## 2. Các lỗi còn thấy sau khi so nhãn

Task `batch-review` #6 trên CVAT local nhận 24 ảnh và 532 box từ [tệp nhập](prelabel/pre_traps_import.coco.json). Bốn lượt chạy đối chiếu trên CPU gồm YOLO26s/m ở cỡ 960 và YOLOE-26s ở cỡ 768/960. Lượt đối chiếu sửa khung 47 box, bỏ 36 box và bổ sung 35 box. [Bản COCO xuất lại từ CVAT](labels/batch_reviewed.json) có 531 box trên đúng 24 ảnh. Các con số này mô tả lượt đối chiếu do Codex thực hiện bằng các model local.

| So với nhãn đầu vào | Kết quả ngoài vùng bẫy |
|---|---|
| Giữ (`accepted`) | 445/520 box máy = 85,6% |
| Sửa khung (`edited`) | 43/520 box máy = 8,3% |
| Bỏ (`deleted`) | 32/520 box máy = 6,2% |
| Bổ sung (`added`) | 30/518 box sau đối chiếu = 5,8% |

Script loại 25 box ở hai phía của phép so quanh vùng bẫy để lỗi được chèn vào không làm lệch số đo. Hai ca đổi lớp được đếm ở cả cột bỏ và cột thêm. Trong các lớp có nhiều box, xe máy cần sửa khung nhiều nhất (21/138 box máy thuộc lớp này). Chẳng hạn ở `img_005.jpg`, một box xe máy được chỉnh về vị trí mà hai lượt YOLO26 đồng thuận, với IoU 0,605 so với box nhập. Những con số này vẫn phụ thuộc vào nhãn sau đối chiếu, nên cần kiểm trên ảnh gốc để biết sửa nào đúng.

Trên 4 ảnh gold, bỏ 24 vùng `ignore_region` khỏi 113 nhãn thì còn 89 vật để tính recall. Recipe ghép đúng 53 vật, có 15 dự đoán thừa và sót 36 vật. Precision là 53/68 (77,9%) và recall là 53/89 (59,6%). [Kết quả so nhãn và gold](qc/review_stats.md)

## 3. Bẫy và lỗi bỏ sót

Chấm sau khi đã xuất nhãn từ CVAT, lượt đối chiếu sửa được 11/12 bẫy (91,7%). Cụ thể, bắt 3/3 bẫy sai lớp, 3/3 bẫy box lỏng, 3/3 bẫy box thừa và 2/3 bẫy thiếu box. Bẫy còn lọt là `missing` ở `img_006.jpg`. [Kết quả chấm bẫy](qc/review_stats.md)

Điểm này cho thấy hai lượt dò bằng model vẫn có thể cùng bỏ qua một vật. Bước cần kiểm lại trên ảnh gốc là lượt tìm vật chưa có box khi tắt lớp nhãn. Bốn lượt máy có liên quan về model và trọng số, nên kết quả 11/12 chỉ mô tả lần đối chiếu này. Kết quả duyệt độc lập của học viên chưa có số đo.

## 4. Có nên dùng nhãn gợi ý?

Bốn ảnh gold hết 90 phút, tương đương 22,5 phút/ảnh. Bốn lượt đối chiếu trên batch mất tổng cộng 65,4 giây inference CPU cho 24 ảnh (2,73 giây/ảnh), chưa tính tải model, nhập/xuất CVAT và kiểm ảnh. Hiện chưa có thời gian bấm đồng hồ cho một lượt duyệt đủ 24 ảnh bằng mắt, nên chưa tính được tiết kiệm bao nhiêu phút hay bao nhiêu phần trăm. Batch và gold khác ảnh, và người gán gold đã quen dạng cảnh khi chuyển sang batch. [Nhật ký thời gian](time_log.csv)

Nhãn máy có thể làm điểm xuất phát để giảm số box phải vẽ từ đầu. Recall chỉ 53/89 trên gold, và việc giữ 445/520 box ngoài vùng bẫy vẫn đi cùng một bẫy bỏ sót. Cách kiểm đề xuất cho lượt duyệt bằng mắt sau:

1. Tắt lớp box, quét và đếm nhẩm người, xe máy, xe ba bánh trên đủ 24/24 ảnh trước khi bật lớp nhãn.
2. Đặt dấu hoàn thành cho từng bước tìm vật sót, bỏ box thừa, kiểm lớp và kiểm biên. Với ảnh trên 30 box, quét theo bốn phần ảnh.
3. Xuất bản duyệt và ghi thời gian trước khi chấm bẫy. Sau đó rà riêng các lỗi bỏ sót, kể cả khi đã đạt mốc ít nhất 10/12 bẫy.

## 5. Thử SAM với điểm dương và âm

SAM 2.1 Base (`sam2.1_b.pt`) đã được thử từ box và từ điểm trên CPU. Lượt từ box `r05` tạo 16 mask trên 2 ảnh ở 17,30 giây/ảnh. Lượt từ điểm dùng cùng model cho ba ảnh để dễ so sánh. Với riêng `img_013.jpg`, MobileSAM dùng cùng 3 điểm giúp thấy biên thay đổi giữa hai phiên bản. Các tỷ lệ dưới đây tính trên 698.368 pixel mỗi ảnh. [Log r05](model_log.csv)

| Ảnh | Model | Điểm `x,y` | Kết quả và chỗ cần xem lại |
|---|---|---|---|
| `img_004.jpg` | SAM 2.1 Base | +`620,480`, +`700,560`, −`465,450` | 93.190 px (13,3%), 1 polygon. Vùng tô vẫn chồng lên một phần box `person` cạnh xe. [Mask](masks/tuktuk_img004_sam2.jpg) |
| `img_013.jpg` | SAM 2.1 Base | +`400,480`, +`600,110`, −`720,300` | 317.255 px (45,4%), 1 polygon. Mask phủ rộng cả vùng box người lái, cần rà lại điểm âm. [Mask](masks/tuktuk_img013_sam2.jpg) |
| `img_017.jpg` | SAM 2.1 Base | +`560,420`, +`680,500`, −`645,355` | 84.532 px (12,1%), 1 polygon. Vùng tô giao với box của hai xe ba bánh chồng nhau, cần kiểm đường biên giữa hai xe. [Mask](masks/tuktuk_img017_sam2.jpg) |
| `img_013.jpg` | MobileSAM | +`400,480`, +`600,110`, −`720,300` | 245.541 px (35,2%), 2 polygon. Vùng tô giao box người lái ít hơn SAM 2.1, phần biên và các đoạn bị tách vẫn cần xem trên ảnh. [Mask](masks/tuktuk_img013_sam1.jpg) |

Mục 4.2 của guideline yêu cầu tách người và xe theo biên nhìn thấy. Ở `img_013.jpg`, ước lượng từ ảnh overlay cho thấy vùng tô giao khoảng 79% diện tích box người lái với SAM 2.1 và khoảng 31% với MobileSAM. Box chữ nhật cũng chứa nền và các phần xe nhìn xuyên qua, nên phép so này chỉ khoanh vùng cần kiểm. Số pixel người hoặc hàng hóa dính vào mask chưa được xác định. [Quy tắc gán mask](../guideline/guideline_v1.0_label_contract.md)

## 6. Calibration

[Bảng calibration](qc/calibration_decisions.csv) ghi ba ca có ảnh đối chiếu. `CAL-01` lấy lỗi YOLO26 gọi tuk-tuk là ô tô ở `img_013.jpg`. Vì v1.0 đã có lớp xe ba bánh, phần bổ sung chỉ cụ thể hóa các dấu hiệu cần nhìn khi box máy ghi `car` hoặc `truck`. `CAL-02` xử lý câu hỏi về việc đặt một điểm tối thiểu để giữ box mà không sửa. Ví dụ xe bán tải `img_001.jpg` cho thấy điểm 0,8758 vẫn đi kèm sai lớp. `CAL-03` cụ thể hóa cách tách người và xe ở vùng cabin khi làm mask. Các mục này là bản đề xuất để học viên xác nhận và được ghi trong [phần bổ sung v1.1](guidelines/guideline_v1.1_calibrated.md).

## 7. QC và trạng thái bộ nhãn

Bản ứng viên có 24 ảnh và 531 box. Với seed 42, script rút 5 ảnh ngẫu nhiên gồm `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg` và `img_035.jpg`. Năm ảnh có 110 đối tượng. Không có ảnh được đánh dấu `can_review` trong bản xuất đang dùng. [Mẫu QC](qc/qc_sample.csv) · [Tóm tắt bộ ứng viên](release/release_summary.csv)

Đã mở đủ 5 ảnh gốc và so với ảnh overlay có mã box. Phiếu ghi **14 lỗi trên 110 đối tượng trong mẫu (12,7%)** theo cách tính của script. Trong đó có 13 box có ID và một người bị bỏ sót, chưa có ID. Cả **5/5 ảnh** có ít nhất một lỗi. [Phiếu QC](qc/qc_review.csv)

Box thừa chiếm 9/14 lỗi. Rõ nhất là ở `img_014.jpg`, hai đầu quang gánh và quầy gỗ bị gán thành xe ba bánh. Ở `img_031.jpg`, cùng một chiếc xe đạp có hai box và box lớn cắt mất một phần bánh sau. Tổng cộng có 3 lỗi sai biên, 1 lỗi sai lớp ở xe thùng trong `img_003.jpg` và 1 lỗi bỏ sót người áo sáng bên hàng xe máy trong `img_006.jpg`.

**Trạng thái kỹ thuật hiện tại là Rework (18/11/2026).** Tỷ lệ 5/5 ảnh lỗi vượt ngưỡng 2% và box thừa lặp lại trên nhiều ảnh. Cận trên Wilson 95% là 100% với mẫu này. Năm ảnh đủ để tìm lỗi cần sửa, còn tỷ lệ 5/5 chỉ mô tả mẫu đã xem. Khi sửa, ưu tiên bỏ box thừa trên các vật giống xe ba bánh, ghép các phần của cùng một xe về một box và rà người bị che một phần. [Kết quả QC](qc/qc_report_auto.md)

## 8. Ghi chú khi làm bài

- Gold xuất từ CVAT gồm 4 ảnh và 113 nhãn, trong đó có 24 vùng bỏ qua và 8 box đánh dấu `can_review`. Thời điểm tệp gold trên máy sớm hơn log lượt YOLO26 pilot. Việc có nhìn pre-label trước đó hay không còn cần học viên tự xác nhận.
- Codex hỗ trợ kiểm lại CP3, chạy đối chiếu box và SAM trên CPU, chuẩn bị bản xuất ở CVAT local.
- Khi chạy thử QC, phiếu bỏ trống có thể làm script gợi ý Release. Đã sửa `qc_report.py` để trả Hold với mẫu chưa kiểm đủ và chưa tính tỷ lệ lỗi hoặc cận Wilson trong trường hợp đó.
- Ở `img_006.jpg`, khi mở ảnh gốc mới thấy hai người áo sáng ở sát nhau cạnh dãy xe. Box #76 chỉ bao người bên phải. Ca này cho thấy cần quét riêng vùng người bị xe che, ngay cả khi overlay nhìn qua đã có box gần đó.
