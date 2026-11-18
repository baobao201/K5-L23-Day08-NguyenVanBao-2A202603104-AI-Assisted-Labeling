# Báo cáo Day 08 — 2A202603104 - Nguyễn Văn Bảo

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-m | 0.3 (ngưỡng mặc định) | Dẫn run `r01` & `r02`: Model closed-set tốc độ nhanh (0.87s/ảnh), nhận diện chính xác các lớp chuẩn COCO, tách độc lập người lái và xe máy tốt ở tiền cảnh; giữ conf 0.3 theo gợi ý từ `threshold_sweep.md` để tối ưu recall. |
| three_wheeler | YOLOE-26s, prompt: `tuk-tuk, cyclo, cargo tricycle -> three_wheeler` | 0.15 | Dẫn run `r03`: Bù lấp hoàn hảo điểm mù của YOLO26 cho các phương tiện 3 bánh Đông Nam Á; ở ngưỡng 0.15 đạt recall 100% (7/7 vật gold) và precision 78% (7/9 box). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```bash
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 (`sam2.1_b.pt`)**. Lý do: Đạt độ cân bằng tối ưu giữa chất lượng bám viền chi tiết và tốc độ xử lý; đường bao polygon mượt mà và chính xác hơn SAM 1 (`mobile_sam.pt`/`sam_b.pt`) ở các đường cong thân xe; đồng thời gọn nhẹ, không yêu cầu phần cứng GPU quá lớn hay thủ tục xin quyền bảo mật phức tạp như SAM 3 (`facebook/sam3`).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào?
  - **Lỗi sai lớp hệ thống của detector đóng:** Do YOLO26 chỉ được học trên 80 lớp COCO (không có `three_wheeler`), xe tuk-tuk hay xe ba gác luôn bị gán cưỡng bức thành `truck` (conf 0.73) hoặc `car` (conf 0.72-0.89). Vì điểm tin cậy của các box đoán sai này rất cao (> 0.7), việc tăng conf lên 0.5 không loại bỏ được box sai, còn hạ conf chỉ sinh thêm nhiễu chứ không thể sinh ra nhãn `three_wheeler`.
  - **Cách sửa:** Sử dụng detector từ vựng mở (Open-Vocabulary YOLOE) có prompt riêng cho `three_wheeler` và thiết lập quy tắc dung hợp nhãn trong `merge_prelabels.py` cho `three_wheeler` của YOLOE đè lên `car`/`truck` của YOLO26 khi $IoU \ge 0.6$.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - *Điểm cao mà sai:* Trong `img_002.jpg`, xe tuk-tuk bị YOLO26 gán thành `truck 0.733`; trong `img_011.jpg`, xe ba bánh bên mép trái bị gán `truck 0.770`; trong `img_035.jpg`, bẫy nhân tạo đổi xe máy thành `bicycle 0.73`.
  - *Điểm thấp mà đúng:* Trong `img_011.jpg`, xe ba bánh bên mép trái được YOLOE phát hiện chính xác với điểm thấp `three_wheeler 0.283`.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Do việc sweep ngưỡng và đánh giá độ chính xác thực hiện trên cùng 4 ảnh gold (in-sample evaluation, chỉ có 60 đối tượng), các chỉ số precision/recall thu được mang tính lạc quan hơn thực tế. Các lớp ít xuất hiện như `bus` (0 đối tượng) và `truck` (1 đối tượng) không đủ số liệu thống kê để tối ưu ngưỡng riêng, buộc phải giữ ngưỡng mặc định 0.3.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 357 | 82.6% | 432 box máy |
| edited | 25 | 5.8% | 432 box máy |
| deleted | 50 | 11.6% | 432 box máy |
| added | 51 | 11.8% | 433 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  - **Lớp `three_wheeler` là lớp máy tệ nhất**: Chỉ có 7 box accepted, trong khi có tới 8 box edited, 11 box deleted, và 7 box added (tổng số box phải can thiệp sửa/xóa/thêm là 26 box, gấp gần 4 lần số box accepted).
  - *Nguyên nhân:* YOLOE nhạy cảm với văn bản prompt nhưng sinh bounding box còn lỏng, dễ dương tính giả trên các bề mặt dạng khối hộp như thùng xe bán tải trong `img_002.jpg` (conf 0.37) và xe đẩy hàng (conf 0.40); đồng thời bỏ sót các xe xích lô bị che khuất đỗ sát lề đường trong `img_007.jpg`.
- So với gold: precision / recall của **máy**: Precision của máy đạt **60.0%** (36/60 TP/khẳng định, KTC 95%: 47–71%), Recall của máy đạt **60.0%** (36/60 TP/thực tế, KTC 95%: 47–71%) (mẫu số: 60 đối tượng gold trong 4 ảnh gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **5/12 = 41.7%** (KTC 95%: 19–68%); loại nào lọt: Lọt 7 bẫy gồm:
  - `loose`: lọt **3/3** bẫy (`img_005.jpg`, `img_013.jpg`, `img_033.jpg`).
  - `wrong_class`: lọt **3/3** bẫy (`img_010.jpg`, `img_027.jpg`, `img_035.jpg`).
  - `missing`: lọt **1/3** bẫy (`img_022.jpg`).
  - *Vì sao (bạn đã bỏ bước nào của checklist):* Reviewer đã bỏ qua Bước 1 (tạm ẩn nhãn máy để tự quét mắt tìm vật sót) dẫn đến sót bẫy missing; đồng thời lơ là Bước 3 (kiểm tra chéo tên nhãn) và Bước 4 (zoom kiểm tra độ khít viền), tin rằng các box máy dựng sẵn đã ôm trọn đối tượng.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt?
  - Tỉ lệ accepted cao (82.6%) đi đôi với tỉ lệ bắt bẫy thấp (41.7%) phản ánh rõ rệt **Thiên kiến tự động hóa (Automation Bias)** và hội chứng "Rubber Stamping" (phê duyệt thụ động). Người duyệt bị "neo" vào các gợi ý hiển thị sẵn của máy, ưu tiên tốc độ duyệt hơn là việc săm soi chi tiết từng nhãn, khiến các sai sót tinh vi của mô hình bị bỏ lọt vào tập nhãn phát hành.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **5.5 phút/ảnh** (22 phút/4 ảnh gold) vs duyệt **3.1 phút/ảnh** (75 phút/24 ảnh batch). Thiên lệch (bạn gán gold trước nên đã quen cảnh): Do đã trực tiếp gán 4 ảnh gold trước đó, reviewer đã quen thuộc với đặc điểm giao thông, góc máy và kích thước xe cộ, nên tốc độ duyệt ở lô 24 ảnh batch được hưởng một phần hiệu ứng quen tay.
- Kết luận: **CÓ ĐÁNG DÙNG, nhưng chủ yếu cho các lớp phổ biến** (`person`, `motorbike`, `car`), giúp giảm khoảng **43.6%** thời gian gán nhãn so với vẽ tay hoàn toàn. Đối với lớp đặc thù (`three_wheeler`), máy đóng vai trò gợi ý vùng nghi vấn nhưng con người bắt buộc phải trực tiếp kiểm tra và chỉnh sửa lại nhãn/biên box.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quy tắc "Mắt trước - Máy sau" (Ẩn nhãn 10 giây đầu):** Khi mở bất kỳ ảnh mới nào, luôn bấm phím tắt ẩn toàn bộ layer nhãn máy trong 10 giây đầu tiên để mắt tự quét tìm các vật thể nhỏ ở rìa hoặc bị che khuất. *Kiểm tra bằng số:* Số box `added` phải tăng lên tương ứng với số vật khuất trên ảnh.
  2. **Quy tắc "Kiểm tra chéo 3 chi tiết phương tiện":** Đối với mọi box phương tiện 2 bánh và 3 bánh, luôn zoom cận cảnh kiểm tra: lốc máy/ống xả, số bánh xe nhìn thấy, và mui xe/thùng hàng để bắt trúng bẫy `wrong_class`. *Kiểm tra bằng việc làm:* Đánh dấu tick kiểm tra trên checklist giấy trước khi lưu ảnh.
  3. **Quy tắc "Nghỉ ngơi định kỳ Pomodoro":** Cứ sau 8–10 ảnh duyệt liên tục (khoảng 25–30 phút), bắt buộc dừng lại nghỉ mắt 3–5 phút để phục hồi độ tinh tường thị giác. *Kiểm tra bằng số:* Các mốc thời gian nghỉ ngơi được ghi nhận minh bạch trong `submission/time_log.csv`.

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

- Số quyết định: **4 quyết định** (`submission/qc/calibration_decisions.csv`); quan trọng nhất: **`CAL-01`** (`img_002.jpg`, nguồn `loi_may`).
  - *Nội dung `CAL-01`:* Xác lập nguyên tắc phân loại cho phương tiện ba bánh Đông Nam Á để khắc phục điểm mù của YOLO26 (gán tuk-tuk thành `truck 0.73`/`car 0.72`), đưa quy tắc dung hợp ưu tiên `three_wheeler` đè lên `car`/`truck` khi $IoU \ge 0.6$ vào pipeline chuẩn.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  1. *Mục 2.1:* Mọi xe 3 bánh có động cơ hoặc thô sơ đều thuộc `three_wheeler`, không gán `car`/`truck`.
  2. *Mục 4.1.8:* Hàng hóa xếp trên thùng xe ba gác/xe đạp tính thuộc phương tiện, box xe bao trọn toàn bộ phần hàng hóa nhìn thấy.
  3. *Mục 4.1.9:* Phân biệt `motorbike` và `bicycle` dựa trên sự xuất hiện của lốc máy/ống xả và biển số xe.
  4. *Mục 4.1.10:* Người ngồi trên thùng xe ba gác gán box `person` riêng cho từng cá nhân, không gộp và không để trùng box.
- Nguyên nhân gốc: **Kết hợp cả 3 yếu tố:**
  - *Máy:* Giới hạn tập từ vựng đóng COCO 80 của YOLO26.
  - *Guideline v1.0 mơ hồ:* Chưa làm rõ quy tắc bao gồm hàng hóa cồng kềnh trên xe và cách gán nhãn cho nhiều người trong khoang xe ba gác.
  - *Người duyệt chưa nắm chắc quy tắc:* Thiếu cảnh giác với bẫy nhân tạo, lướt qua các chi tiết cơ khí của xe máy.

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42, gồm 92 đối tượng); người kiểm: Nguyễn Văn Bảo (**hạn chế**: tự kiểm tra lại bài mình vừa duyệt, dễ có thiên kiến chủ quan).
- Lỗi theo **đối tượng**: **4/92 = 4.3%** · theo **ảnh**: **3/5 = 60.0%** (ngưỡng Release quy định $\le 2\%$).
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại:
  - `thua` (2 lỗi): `img_003.jpg` (box person thừa đè lên tài xế ba gác), `img_035.jpg` (box thừa con trong thân người đi bộ).
  - `sai_lop` (2 lỗi): `img_006.jpg` (biển hiệu ngõ hẹp bị gán nhầm thành `car 0.55`), `img_035.jpg` (xe máy bị lọt bẫy thành `bicycle`).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 3/5 ảnh lỗi, cận trên 95% khoảng tin cậy Wilson của tỉ lệ ảnh lỗi thật lên tới **88%**. Ngay cả khi mẫu 5 ảnh đạt 0 lỗi, quy tắc 3 (Rule of Three) cho thấy cận trên lỗi thật vẫn là $\approx 3/5 = 60\%$. Do đó, mẫu nhỏ 5 ảnh hoàn toàn không đủ sức mạnh thống kê để chứng minh dữ liệu đạt chuẩn $\le 2\%$.
- Script kiểm được / không kiểm được:
  - *Script kiểm được:* Định dạng file, cấu trúc taxonomy, số lượng bounding box, phân phối IoU, ma trận nhầm lẫn và các công thức tỷ lệ toán học.
  - *Script không kiểm được:* Độ ôm sát viền thực tế của box đối với hình dáng xe cộ, việc phân biệt đúng sai ngữ nghĩa trong bối cảnh phố phức tạp, và các đối tượng bị cả người lẫn máy cùng bỏ sót ngoài tập gold.

**Quyết định: REWORK**, ngày 2026-11-19. Lý do (đi theo Hold → Rework → Release):
1. Không rơi vào **HOLD** vì guideline v1.1 đã chốt xong các câu hỏi mở cốt lõi và dữ liệu hình ảnh hoàn toàn rõ nét, không bị lỗi hỏng.
2. Rơi vào **REWORK** vì:
   - Tỉ lệ ảnh có lỗi thực tế trong mẫu QC là **60.0%** (3/5 ảnh), vượt xa ngưỡng trần cho phép $\le 2\%$.
   - Tỉ lệ bắt bẫy nhân tạo chỉ đạt **41.7%** (5/12 bẫy), thấp hơn nhiều so với ngưỡng yêu cầu $\ge 80\%$.
   - Tồn tại các lỗi mang tính lặp lại về sai nhãn (`sai_lop`) và box trùng lặp (`thua`).
3. Nếu Rework / Hold, việc cần làm tiếp:
   - Áp dụng các quy tắc mới từ `guideline_v1.1_calibrated.md` vào lại giao diện CVAT để sửa toàn bộ các box sai lớp và xóa triệt để các box trùng lặp trong 24 ảnh batch.
   - Xuất file nhãn đã sửa ra `submission/labels/qc_fixed.json`, chạy lại `python scripts/build_release.py`.
   - Tiến hành lấy mẫu kiểm định vòng 2 (`python scripts/qc_sample.py --force`) để xác nhận tỉ lệ lỗi đạt chuẩn trước khi ký duyệt Release.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng Gemini Assistant hỗ trợ tự động hóa các script đo lường số liệu, trích xuất dữ liệu JSON trực quan và kiểm tra chéo các điều kiện nộp bài; mọi kết quả đều được kiểm định bằng `python scripts/check_submission.py`.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? **Hoàn toàn KHÔNG**: 4 ảnh gold (`img_001.jpg`, `img_012.jpg`, `img_020.jpg`, `img_036.jpg`) được gán tay thủ công 100% trên CVAT từ Pha 1 trước khi tải hay chạy bất kỳ model pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU CUDA NVIDIA RTX, CVAT v2.82.1 chạy qua Docker Desktop. Quá trình thiết lập môi trường diễn ra thuận lợi, không gặp lỗi xung đột thư viện.
- Dùng cache `data/cache/` thay vì tự chạy? **KHÔNG DÙNG CACHE**: Toàn bộ các lượt chạy mô hình YOLO26, YOLOE và SAM 2.1 đều được thực thi trực tiếp trên phần cứng máy tính cá nhân.
- Lỗi / bug script đã gặp (nếu có): Gặp lỗi parse dấu nháy kép trên PowerShell Windows khi chạy lệnh có tham số chuỗi phức tạp; đã khắc phục bằng cách đóng gói script runner hoặc gọi từ môi trường bash/python wrapper.
- Một điều bạn làm khác lần sau: Sẽ thực hiện nghiêm ngặt quy tắc ẩn nhãn máy trong 10 giây đầu tiên trên từng ảnh ngay từ đầu lô duyệt để rèn luyện phản xạ phát hiện vật thể độc lập, tránh rơi vào thiên kiến tự động hóa khiến bẫy `wrong_class` và `loose` bị lọt lưới.
