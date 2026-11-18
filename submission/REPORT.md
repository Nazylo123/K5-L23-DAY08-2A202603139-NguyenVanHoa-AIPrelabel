# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | <<YOLO26m>> | <<theo lớp, từ threshold_sweep>> | <<>> |
| three_wheeler | <<YOLOE-26s, prompt: ...>> | <<>> | <<>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<>>

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

- Bắt được 7/12 bẫy (58,3%), dưới mốc 80%. Lọt 2 bẫy bỏ sót, 1 sai lớp, 1 box lỏng và 1 box thừa (`qc/review_stats.md`). Mình nghĩ ở những vùng đông, người và xe chồng lên nhau nên dễ nhìn sót khi duyệt.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay khoảng 20 phút cho 4 ảnh; duyệt khoảng 45–60 phút cho 24 ảnh. Cả hai là ước lượng, không phải thời gian đo. Thiên lệch (gán gold trước nên có thể đã quen cảnh): <<>>
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

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Hermes hỗ trợ giải thích các bước, chạy lệnh kiểm tra, chạy script SAM theo tọa độ mẫu và ghi lại số liệu, nhận xét của mình. Số liệu review được đối chiếu với `qc/review_stats.md`; mình duyệt nhãn trên CVAT và xem trực tiếp ảnh mask để nhận xét.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
