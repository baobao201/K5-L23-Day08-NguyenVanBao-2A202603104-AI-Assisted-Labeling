# Thông tin học viên

- Họ và tên: Nguyễn Văn Bảo
- MSHV: 2A202603104
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: baobao201
- Repo bài nộp (URL): https://github.com/baobao201/K5-L23-Day08-NguyenVanBao-2A202603104-AI-Assisted-Labeling
- Commit chốt bài (hash): 13bc162
- Máy: Windows GPU(CUDA)
- CVAT (Docker): v2.82.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 357/432 = 82.6% / 25/432 = 5.8% / 50/432 = 11.6% (mẫu số: 432 box máy); added: 51/433 = 11.8% (mẫu số: 433 đối tượng)
- So với gold: precision / recall của **máy** 36/60 = 60.0% / 36/60 = 60.0% (mẫu số: 60 đối tượng gold)
- Bắt bẫy: 5/12 = 41.7% (mẫu số: 12 bẫy)
- Thời gian: gán tay 5.5 phút/ảnh (22 phút / 4 ảnh gold); duyệt có pre-label 3.1 phút/ảnh (75 phút / 24 ảnh batch)
- Lỗi QC: 60.0% theo ảnh (3/5 ảnh), 4.3% theo đối tượng (4/92 đối tượng)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh có lỗi trong mẫu QC (60.0% > 2%) và tỉ lệ bắt bẫy nhân tạo (41.7% < 80%) chưa đạt chuẩn release, cần rà soát sửa nhãn theo guideline v1.1.
