# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 233; đối tượng sau duyệt: 234; ngưỡng accepted: IoU >= 0.9
- Đã **loại 21 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 216 | 216/233 = 92.7% | 233 box máy |
| edited (sửa) | 5 | 5/233 = 2.1% | 233 box máy |
| deleted (xoá) | 12 | 12/233 = 5.2% | 233 box máy |
| added (vẽ thêm) | 13 | 13/234 = 5.6% | 234 đối tượng sau duyệt |

Trong số deleted + added, **2 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 11.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 107 | 2 | 4 | 7 |
| bicycle | 3 | 0 | 0 | 0 |
| motorbike | 61 | 1 | 5 | 3 |
| car | 23 | 2 | 2 | 0 |
| bus | 2 | 0 | 0 | 1 |
| truck | 2 | 0 | 0 | 0 |
| three_wheeler | 18 | 0 | 1 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 36 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 24 | 11 | 12 | 68.6% (KTC 95%: 52–81%) | 66.7% (KTC 95%: 50–80%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - missing: 1/3
  - loose: 2/3
  - extra: 2/3
  - lọt: img_024.jpg (wrong_class), img_018.jpg (missing), img_029.jpg (extra), img_028.jpg (missing), img_006.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
