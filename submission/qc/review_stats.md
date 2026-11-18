# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 432; đối tượng sau duyệt: 433; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 357 | 357/432 = 82.6% | 432 box máy |
| edited (sửa) | 25 | 25/432 = 5.8% | 432 box máy |
| deleted (xoá) | 50 | 50/432 = 11.6% | 432 box máy |
| added (vẽ thêm) | 51 | 51/433 = 11.8% | 433 đối tượng sau duyệt |

Trong số deleted + added, **4 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 47.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 216 | 10 | 23 | 20 |
| bicycle | 5 | 0 | 4 | 3 |
| motorbike | 79 | 6 | 2 | 15 |
| car | 43 | 1 | 7 | 3 |
| bus | 2 | 0 | 1 | 2 |
| truck | 5 | 0 | 2 | 1 |
| three_wheeler | 7 | 8 | 11 | 7 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 60 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 36 | 24 | 24 | 60.0% (KTC 95%: 47–71%) | 60.0% (KTC 95%: 47–71%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 5/12 = 41.7% (KTC 95%: 19–68%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - extra: 3/3
  - wrong_class: 0/3
  - loose: 0/3
  - lọt: img_022.jpg (missing), img_010.jpg (wrong_class), img_005.jpg (loose), img_027.jpg (wrong_class), img_033.jpg (loose), img_035.jpg (wrong_class), img_013.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
