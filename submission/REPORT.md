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
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Không (xe rỗng đỗ bên đường; click âm loại bỏ thành công người đứng cạnh xe) | Ranh giới bám mui xe và kính chắn gió rất chuẩn; phần gầm xe và bóng râm chân bánh trước bị lẹm nhẹ ra mặt đường gạch (~5-10px). | `tuktuk_img004_sam2.jpg` |
| `img_013.jpg` | SAM 1 (`sam_b.pt`) | 3 click (2 pos, 1 neg) | Có gồm người lái: dù có 1 click âm lên áo, mask vẫn nuốt trọn khuôn mặt, tai và bàn chân người lái; dính một phần cánh tay hành khách sau. | Ranh giới bị rách viền, đốm hạt quanh cụm đồng hồ trước; nan sắt trang trí bị đục lỗ lẹm sang nền; biên mask thô hơn SAM 2.1. | `tuktuk_img013_sam1.jpg` |
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Vẫn bị dính người lái (mặt, chân, phần lưng); 1 click âm không đủ bóc tách người lái ra khỏi khung xe theo guideline v1.0. | Viền thân xe bám mượt hơn SAM 1, nhưng xuất hiện hiện tượng rách lốm đốm (pepper noise) trên áo người lái do click âm gây nhiễu biên. | `tuktuk_img013_sam2.jpg` |
| `img_017.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Không (xe đỗ không có người lái; không bị dính người đi bộ qua đường ở cạnh xe) | Click âm đặt ở đỉnh mui (nhằm loại bạt quán ăn phía sau) tạo ra vết khuyết/lỗ rách sâu trên vải mui xe tuk-tuk. Bù lại, bánh xe và gầm bám khít mặt đường nhựa. | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? 
  - **Quy tắc giúp ích:** Quy tắc 4.2.2 về lỗ thủng (<5% diện tích không cần khoét lỗ) giúp xử lý các khe nan sắt trang trí trên thành xe tuk-tuk mà không phải mất công bấm thêm hàng chục click âm; quy tắc 4.2.3 (kính xe thuộc vật, bóng đổ không thuộc vật) giúp định hướng click chuẩn xác vào kính chắn gió và loại bỏ bóng chân xe.
  - **Quy tắc còn thiếu:** Guideline v1.0 chưa có quy định cụ thể về việc bóc tách polygon giữa người lái/hành khách và phương tiện hở buồng lái (tuk-tuk, xích lô, ba gác). Do SAM có xu hướng gom toàn bộ cụm vật thể ("một click, ba mask hợp lệ"), nếu không có quy định số điểm click âm tối thiểu hoặc chấp nhận mask xe bao trùm người lái (khi đã có box `person` riêng), người gán nhãn sẽ mất rất nhiều thời gian tỉa mask thủ công.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Đã thử nghiệm công cụ AI Tools (Interactor SAM) trên giao diện CVAT (click chuột trái tạo điểm dương, click chuột phải tạo điểm âm). Khác biệt lớn nhất so với script offline (`sam2_points.py`): CVAT Interactor hiển thị mask tức thời sau mỗi click, cho phép tương tác trực quan linh hoạt và gán trực tiếp nhãn đúng (`three_wheeler`) ngay tại giao diện; trong khi script offline phải căn tọa độ pixel trước (`--pos "x,y"`), mất nhiều lần thử nghiệm lại tọa độ nhưng chạy độc lập không phụ thuộc hạ tầng Docker Nuclio.

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

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
