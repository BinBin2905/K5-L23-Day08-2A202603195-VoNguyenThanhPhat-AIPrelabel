# Thông tin học viên

- Họ và tên: Võ Nguyễn Thành Phát
- MSHV: 03915
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: BinBin2905
- Repo bài nộp (URL): https://github.com/BinBin2905/K5-L23-Day08-2A202603195-VoNguyenThanhPhat-AIPrelabel
- Commit chốt bài (hash):0347d671725d6279424131996067ba73089d54c2
- Máy: Window11, GPU Nvidia
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 84.8% / 4.4% / 10.7% (442 / 23 / 56, mẫu số: 521 box máy); added: 7.9% (40, mẫu số: 505 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 59.5% / 78.6% (44 TP / 74 box máy; 44 TP / 56 đối tượng gold)
- Bắt bẫy: 11/12 = 91.7%
- Thời gian: gán tay 7.0 phút/ảnh (28 phút / 4 ảnh); duyệt có pre-label 1.2 phút/ảnh (29 phút / 24 ảnh)
- Lỗi QC: 12.5% theo ảnh (1/8), 0.8% theo đối tượng (1/129)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 12.5% vượt ngưỡng 2% do bẫy loose ở img_031 lọt cả lúc duyệt lẫn QC lần đầu (bắt bẫy 11/12 vẫn đạt).
