# Báo cáo Day 08 — 2A202603139 - Nguyễn Văn Hoa

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n | 0,30 (ngưỡng gộp) | Dùng làm nền theo các lượt thử `r01` và `r02`. `qc/threshold_sweep.md` không gợi ý ngưỡng riêng. |
| three_wheeler | YOLOE-26s; prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0,30 (ngưỡng gộp) | Lượt `r03` nhận ra tuk-tuk và xích lô; dùng để bổ sung lớp xe ba bánh. |

- Lệnh đã dùng: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM:** SAM 2.1 Tiny (`sam2.1_t.pt`), dùng bản đã cài trên máy để tạo mask rồi kiểm tra lại thay vì vẽ từ đầu (`r04`, mục 5).
- Lượt `r04` có một xe ba bánh bị gán thành `person` (`model_log.csv`). Đổi ngưỡng chỉ giữ hoặc bỏ box, không đổi lớp của box đó. Khi duyệt cần kiểm tra và sửa lớp theo ảnh gốc.
- Ví dụ điểm hiển thị cao nhưng lớp sai: ở `img_024.jpg`, box #164 trong `pre_traps.coco.json` có `conf=0,74` và lớp `three_wheeler`. Đây là bẫy đổi lớp từ `car`, không phải lỗi phân lớp gốc của model. `review_stats.md` ghi nhận bẫy này bị lọt.
- Quét ngưỡng trên 4 ảnh gold, không có tập đánh giá độc lập cho bước chọn ngưỡng. Các lớp ít mẫu không đủ căn cứ để đặt ngưỡng riêng (`qc/threshold_sweep.md`).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 216 | 92,7% | 233 box máy |
| edited | 5 | 2,1% | 233 box máy |
| deleted | 12 | 5,2% | 233 box máy |
| added | 13 | 5,6% | 234 đối tượng sau duyệt |

- Khi duyệt, mình thấy `person` và `motorbike` khó kiểm tra nhất ở những vùng đông, vì người và xe che khuất lẫn nhau. Ảnh minh họa: <<>>
- So với 4 ảnh gold, precision của máy là 24/35 = 68,6% (35 box dự đoán); recall là 24/36 = 66,7% (36 đối tượng gold được tính). Nguồn: `qc/review_stats.md`.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được 7/12 bẫy (58,3%, khoảng tin cậy 95%: 32–81%), dưới mốc 80%. Lọt 2 bẫy bỏ sót, 1 sai lớp, 1 box lỏng và 1 box thừa (`qc/review_stats.md`). Mình nghĩ ở những vùng đông, người và xe chồng lên nhau nên dễ nhìn sót khi duyệt.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay khoảng 20 phút cho 4 ảnh; duyệt khoảng 45–60 phút cho 24 ảnh. Hai lượt dùng hai tập ảnh khác nhau và không bấm giờ, nên không phải phép đo trực tiếp mức tiết kiệm thời gian.
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Phóng to vùng đông người và xe máy, rồi rà từng người và từng xe thay vì nhìn cả cụm một lượt.
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 Tiny | 3 (2 dương, 1 âm) | Chưa xác nhận người lái / hàng hóa | Mask dính ba người phía sau xe. | `tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 Tiny | 3 (2 dương, 1 âm) | Không thấy dính rõ người hoặc hàng hóa | Mask chủ yếu phủ một xe nhưng hơi lem xuống mặt đất. | `tuktuk_img004.jpg` |
| `img_017.jpg` | SAM 2.1 Tiny | 3 (2 dương, 1 âm) | Không thấy dính rõ người hoặc hàng hóa | Mask phủ một xe; nhìn chung tạm ổn. | `tuktuk_img017.jpg` |

Với ba điểm gợi ý mỗi ảnh, mình thấy bản Tiny cho kết quả khá tốt ở hai ảnh này. Ảnh `img_013.jpg` vẫn cho thấy cần kiểm tra người đứng sát xe.

- Đối chiếu guideline mục 4.2: mask cần bám phần nhìn thấy của vật, không lấy bóng đổ và không lấn sang vật khác. `img_004.jpg` hơi lem xuống mặt đất; `img_013.jpg` dính ba người phía sau xe. Chưa chốt bổ sung quy tắc ở bước này.
- *(Tuỳ chọn)* Không chạy đối chiếu ba ảnh này bằng Interactor trong CVAT; kết quả trên được tạo bằng `sam2_points.py`.

## 6. Calibration (Part 6)

- Đã ghi 1 ca trong `qc/calibration_decisions.csv`: `CAL-01`, loại phần mask dính ba người phía sau xe ở `img_013.jpg`, theo mục 4.2 của guideline v1.0. Còn hai ca cần chốt.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- `CAL-01` là lỗi mask của máy: mask lấn sang người bên cạnh, trong khi mục 4.2 của v1.0 đã quy định không lấy phần thuộc vật khác.

## 7. QC và quyết định (Part 6)

- Mẫu: 5/24 ảnh batch (0 ảnh `can_review`, 5 ảnh ngẫu nhiên, seed 42), gồm 50 đối tượng (`qc/qc_sample.csv`).

- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Validator kiểm cấu trúc, tập ảnh và hình học box; báo box `car` #174 ở `img_027.jpg` cao 17,1 px, dưới mốc 20 px. Nó không xác nhận được box ôm sát, đúng lớp hay ảnh còn vật bị bỏ sót.

**Quyết định tạm thời: Rework, chưa phát hành bộ nhãn.** Lượt duyệt bắt được 7/12 bẫy (58,3%), dưới mốc 80%. Chưa có kết quả QC chính thức và phần calibration chưa hoàn tất.
Việc cần làm tiếp: chốt các ca calibration còn thiếu, kiểm đủ 5 ảnh QC, rồi sửa lỗi đã xác nhận và kiểm lại trước khi phát hành.

## 8. Ghi chú cá nhân

- **AI hỗ trợ (Hermes):** giải thích các bước, chạy script SAM theo tọa độ mẫu, đối chiếu số liệu với `qc/review_stats.md` và soi sơ bộ ảnh mẫu QC. Mình duyệt nhãn trên CVAT và xem trực tiếp ảnh mask để nhận xét; các lỗi do AI gợi ý chưa được tính vào QC chính thức.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy: Windows 11, Ubuntu WSL, RTX 3050 Ti Laptop (4 GB VRAM), CVAT Docker v2.74.1. SAM trong CVAT từng lỗi do Docker không gắn được thư mục WSL; đã triển khai lại riêng function SAM.
- Tự chạy YOLO26 và YOLOE để sinh các dự đoán dùng cho quét ngưỡng; không lấy dự đoán thay thế từ `data/cache/`.
- Lượt `r01` có cảnh báo giới hạn thời gian NMS; vẫn xuất được 120 box trên 8 ảnh (`model_log.csv`).
- Lần sau mình sẽ phóng to vùng đông và rà từng người, từng xe thay vì nhìn cả cụm một lượt.
