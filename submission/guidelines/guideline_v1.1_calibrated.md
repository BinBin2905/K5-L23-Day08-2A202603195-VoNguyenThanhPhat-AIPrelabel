# Label contract — Phố châu Á — BỔ SUNG v1.1 — 03915 - Võ Nguyễn Thành Phát

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| #   | Câu hỏi                                                   | Quyết định                                                                                                                                     | Ảnh ví dụ   | `case_id` |
| --- | --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | --------- |
| 1   | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Xe có bàn đạp phụ trợ và kết cấu nhẹ tính là `bicycle`; xe máy điện có bệ để chân, vận tốc/công suất lớn không có bàn đạp tính là `motorbike`. | img_025.jpg | CAL-04    |
| 7   | Confidence cao có đủ để chấp nhận box máy không?          | Confidence chỉ dùng hỗ trợ xếp thứ tự duyệt; luôn phải kiểm tra lại lớp, vị trí box bằng mắt trước khi chấp nhận.                              | img_013.jpg | CAL-02    |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh         | Tình huống                                            | Máy làm gì (model nào)                  | Quyết định của bạn                                                        | Quy tắc mới (vào mục nào của v1.0)                                    |
| ----------- | ----------------------------------------------------- | --------------------------------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| img_013.jpg | YOLO26 gán nhầm tuk-tuk thành `car`                   | YOLO26m gán nhầm lớp phương tiện        | Gán lại đúng lớp `three_wheeler` theo đặc điểm xe và tách riêng người lái | Áp dụng mục 2 v1.0; xử lý lỗi gán nhầm lớp của detector               |
| img_018.jpg | Box bị lỏng biên còn lọt sau khi duyệt                | Pre-label sinh box rộng hơn vật thực tế | Kiểm tra kỹ cả 4 cạnh và chỉnh box ôm sát phần nhìn thấy                  | Áp dụng mục 4.1 v1.0; bổ sung bước rà soát 4 cạnh vào quy trình duyệt |
| img_025.jpg | YOLOE gán nhầm xe đạp điện thành xe máy (`motorbike`) | YOLOE-26s gán nhầm lớp phương tiện      | Chỉnh lại thành lớp `bicycle` dựa trên kết cấu xe có bàn đạp              | Áp dụng mục 2 v1.0; phân biệt rõ xe đạp điện và xe máy điện           |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi                                                                                                                                                       | Lý do (ca nào, `case_id`)      |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| v1.0      | Bản giảng viên                                                                                                                                                 | —[cite: 2]                     |
| v1.1      | Chốt câu hỏi mở về phân loại xe đạp điện, giá trị của conf, bổ sung quy tắc xử lý xe tuk-tuk bị nhận nhầm thành car và chuẩn hóa quy trình kiểm tra 4 cạnh box | CAL-01, CAL-02, CAL-03, CAL-04 |
