# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603104 - Nguyễn Văn Bảo

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Câu 3: Hàng nhô ra: xe ba gác / xe đạp chở chuối, hoa quả, nông sản nhô rộng ra ngoài khung xe: box xe có gồm hàng không? | **Có gồm hàng hóa**: Hàng hóa xếp trên thùng xe hoặc giá đèo hàng gắn liền trong lúc phương tiện lưu thông được tính gộp vào bounding box của phương tiện (`three_wheeler`, `bicycle`), không tách nhãn riêng cho hàng hóa. | `img_003.jpg` | CAL-02 |
| 2 | Câu 1: Xe đạp điện / xe máy điện: xếp vào `bicycle` hay `motorbike`? | **Theo động cơ và hình thái**: Phương tiện có khung sườn xe máy, lốc máy/động cơ điện gắn bánh sau, có biển số xe máy được tính là `motorbike`. Chỉ gán `bicycle` cho xe đạp thô sơ hoặc xe trợ lực có bàn đạp thanh mảnh. | `img_035.jpg` | CAL-03 |
| 3 | Câu 5: Kính chắn gió / mui bạt che của tuk-tuk nhìn xuyên thấu: thuộc xe hay người? | **Thuộc xe**: Kính chắn gió, mui bạt và khung nan sắt tính vào box/mask của xe; người ngồi sau kính nhìn thấy $\ge 20\%$ diện mạo vẫn gán box `person` độc lập bên trong. | `img_013.jpg` | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_002.jpg` | Phương tiện ba bánh (tuk-tuk, xe ba gác chở hàng) chạy giữa phố | YOLO26 gọi nhầm thành `truck 0.73` hoặc `car 0.72` do giới hạn COCO 80 lớp | Luôn gán `three_wheeler`; dùng YOLOE để bù lớp và áp dụng merge policy ưu tiên `three_wheeler` đè lên `car`/`truck` khi $IoU \ge 0.6$. | **Mục 2.1**: Mọi xe 3 bánh có động cơ hoặc thô sơ đều thuộc `three_wheeler`, không gán `car`/`truck`. |
| `img_003.jpg` | Nhiều người (người lớn và trẻ em) ngồi chung trong khoang hở xe ba gác | YOLO26 sinh 2 box `person` trùng lặp (`id=2` và `id=24`) và dính người vào xe | Bóc tách độc lập từng cá nhân nếu nhìn thấy $\ge 20\%$ cơ thể; xoá bỏ triệt để box trùng lặp (`thua`). | **Mục 4.1.10**: Người ngồi trên xe ba gác/thùng xe gán box `person` riêng cho từng cá nhân, không gộp và không để trùng box. |
| `img_035.jpg` | Xe máy dựng trước cửa hiệu tạp hóa bị gán sai nhãn | Bẫy nhân tạo gán thành `bicycle 0.73` (lọt lưới reviewer do chủ quan tin máy) | Bắt buộc kiểm tra chi tiết lốc máy, bô xe và biển số; nếu là xe máy phải sửa về `motorbike`, không chấp nhận nhãn sai của máy. | **Mục 4.1.9**: Kiểm tra kỹ phương tiện 2 bánh dựng sát cửa hàng, xe có khung sườn xe máy/ống xả phải gán `motorbike`. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | 1. Bổ sung Mục 2.1: Phân định ranh giới phương tiện 3 bánh (`three_wheeler`) và closed-set detector.<br>2. Bổ sung Mục 4.1.8: Quy chuẩn bao trọn hàng hóa trên thùng xe (`three_wheeler`, `bicycle`).<br>3. Bổ sung Mục 4.1.9: Quy tắc nhận diện `motorbike` tránh bẫy nhầm sang `bicycle`.<br>4. Bổ sung Mục 4.1.10: Quy tắc bóc tách độc lập nhiều người ngồi trong khoang hở xe ba gác. | CAL-01 (`img_002.jpg`),<br>CAL-02 (`img_003.jpg`),<br>CAL-03 (`img_035.jpg`),<br>CAL-04 (`img_003.jpg`) |
