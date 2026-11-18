# Thông tin học viên

- Họ và tên: Vũ Gia Khang
- MSHV: 2A202604069
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: bayvai20kg
- Repo bài nộp (URL): https://github.com/bayvai20kg/K5-L23-Day08-2A202604069-VuGiaKhang-AI-Assisted-Labeling
- Commit chốt bài (hash): chốt sau commit cuối cùng
- Máy: Windows 11, GPU NVIDIA GeForce GTX 1650 Ti with Max-Q Design (4GB VRAM)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 386/432 = 89.4% / 15/432 = 3.5% / 31/432 = 7.2% (mẫu số: 432 box máy); added: 29/430 = 6.7% (mẫu số: 430 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 57.8% (37/64) / 74.0% (37/50)
- Bắt bẫy: 11/12 = 91.7%
- Thời gian: gán tay 7.5 phút/ảnh (30 phút / 4 ảnh gold); duyệt có pre-label 2.9 phút/ảnh (70 phút / 24 ảnh batch)
- Lỗi QC: 0.0% theo ảnh (0/7 ảnh), 0.0% theo đối tượng (0/124 đối tượng)
- **Quyết định:** Release — lý do một dòng: Tỉ lệ ảnh lỗi 0.0% <= 2%, tỉ lệ bắt bẫy 91.7% >= 80%, không có lỗi hệ thống và đã hoàn tất calibration guideline v1.1.
