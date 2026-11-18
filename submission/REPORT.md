# Báo cáo Day 08 — 2A202603195 - Võ Nguyễn Thành Phát

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp                                              | Model + cỡ                         | Ngưỡng                                           | Vì sao (`run_id` trong `model_log.csv`)                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------------------------ | ---------------------------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| person / bicycle / motorbike / car / bus / truck | YOLO26m                            | Mặc định 0.3 cho cả 6 lớp (`threshold_sweep.md`) | r01 (conf 0.25): 215 box/8 ảnh pilot, nhận diện tốt person, motorbike, car. r02 (conf 0.5): còn 139 box nhưng vẫn không thấy tuk-tuk, nên tăng conf chỉ bớt nhiễu. Quét ngưỡng: person tối đa 87% recall (23 vật), motorbike 82% (11 vật), car 70% (10 vật), đều không đạt 90% ở ngưỡng ≤ 0.5. bicycle (4 vật), bus (1), truck (0) dưới 5 vật nên không chọn ngưỡng riêng. |
| three_wheeler                                    | YOLOE-26s, prompt: `three_wheeler` | 0.15                                             | r04: bắt được tuk-tuk (8 box/8 ảnh pilot, 0.44 s/ảnh) mà YOLO26 không có. Quét ngưỡng: recall 100%, precision 78% trên 7 vật gold.                                                                                                                                                                                                                                         |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
- **Phiên bản SAM bạn chọn** (SAM 2.1 ) và lý do: chất lượng tốt có thể tối ưu, tốc độ vừa đủ, RAM chiếm không quá nhiều, quyền truy cập không bị giới hạn
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 không có khái niệm `three_wheeler`, nên gọi tuk-tuk thành `motorbike`/`car` và gọi xích lô thành xe đạp (r01); tăng conf lên 0.5 (r02) vẫn không thấy tuk-tuk. Cách sửa: dùng YOLOE với prompt `three_wheeler`, đặt file YOLOE trước trong `merge_prelabels.py` để nó thắng box `car` trùng, và sửa tay ca còn sót (CAL-01, `img_013.jpg`). YOLOE vẫn chưa bắt hết xích lô (r04)
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: ngưỡng chọn trên chính 4 ảnh gold (56 đối tượng) nên precision/recall là in-sample, lạc quan. Chỉ three_wheeler có ngưỡng riêng, dựa trên 7 vật gold. 6 lớp còn lại không đủ căn cứ (không đạt 90% recall hoặc dưới 5 vật)

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số  | Tỉ lệ | Mẫu số                  |
| ---------- | --- | ----- | ----------------------- |
| accepted   | 442 | 84.8% | 521 box máy             |
| edited     | 23  | 4.4%  | 521 box máy             |
| deleted    | 56  | 10.7% | 521 box máy             |
| added      | 40  | 7.9%  | 505 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler`.\*\* Trong 42 box máy của lớp này (5+11+26), chỉ 5 được giữ nguyên (11.9%), 11 phải sửa (26.2%) và 26 bị xoá (61.9%). Để so sánh: person accepted 256/270 = 94.8%, car 50/54 = 92.6%, motorbike 122/139 = 87.8%. truck xoá 5/6 nhưng quá ít mẫu để kết luận. Ảnh dẫn chứng: `img_013.jpg` (tuk-tuk bị gán `car`, CAL-01)
- So với gold: precision / recall của **máy** máy precision 59.5% (KTC 95%: 48–70%, 44 TP/74 box máy), recall 78.6% (KTC 95%: 66–87%, 44 TP/56 đối tượng gold). Số này lạc quan vì in-sample trên 4 ảnh. Precision trên gold (59.5%) thấp hơn nhiều so với tỉ lệ accepted trên batch (84.8%) (mẫu số: đối tượng gold)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 11/12 = 91.7% (KTC 95%: 65–99%; theo loại: loose 2/3, extra 3/3, missing 3/3, wrong_class 3/3). Lọt: `img_031.jpg` (loose, box `bicycle` bị nới rộng). Bẫy loose là lỗi của bước 3 (soi 4 cạnh), nên nhiều khả năng bạn đã nhìn nhãn và lớp mà chưa soi cạnh.
- Accepted 84.8% và bắt bẫy 91.7% đều cao, cho thấy lần duyệt batch không phải bấm qua loa. Nhưng 12 bẫy cho KTC rộng. Hơn nữa, lúc QC `img_031.jpg` bị ghi `khong` dù bẫy ở ảnh này đã lọt (xem mục 7). Cùng một điểm mù xuất hiện hai lần: box lỏng ở cạnh.

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gán tay 28 phút/4 ảnh = 7.0 phút/ảnh (khoảng 30 giây/đối tượng, 56 đối tượng gold); duyệt 29 phút/24 ảnh = 1.2 phút/ảnh (khoảng 3.4 giây/đối tượng, 505 đối tượng). Nhanh khoảng 5.8 lần theo ảnh. Thiên lệch: gán gold trước nên đã quen cảnh phố châu Á, và gold chỉ 4 ảnh nên số phút/ảnh thô. Tốc độ cũng không miễn phí: lọt 1 bẫy loose và một lỗi QC.
- Kết luận: có, cho person, car, motorbike (accepted 87.8–94.8%); không nên tin pre-label cho `three_wheeler` (accepted 11.9%, deleted 61.9%), nên vẽ/sửa tay lớp này. Căn cứ: AI error profile theo lớp (mục 2) và tốc độ ở trên.
- **Ba quy tắc giảm thiên kiến** :
  1. Ẩn hết nhãn máy, quét và đếm vật trước khi bật nhãn (bước 1 checklist). Kiểm bằng bắt bẫy ≥ 10/12.
  2. Mỗi box phải soi đủ 4 cạnh, kể cả box conf cao (CAL-02, CAL-03). Kiểm bằng bẫy loose bắt 3/3 và 0 lỗi `sai_vi_tri_kich_thuoc` trong mẫu QC.
  3. Lớp `three_wheeler` không chấp nhận theo conf: đối chiếu từng box với xe thật. Kiểm bằng tỉ lệ box three_wheeler phải xoá/sửa sau duyệt giảm so với 88.1% (37/42) lần này.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh     | SAM (phiên bản) | Số click       | Mask gồm người lái / hàng hoá? | Lỗi quan sát                    | File trong `masks/` |
| ------- | --------------- | -------------- | ------------------------------ | ------------------------------- | ------------------- |
| img_013 | sam2            | 3(2 pos 1 neg) | còn dính một phần người lái    | chưa tách hết đầu và chân       | tuktuk_img013_sam2  |
| img_004 | sam2            | 3(2 pos 1 neg) | mask đúng phần xe tuktuk       | mask hơi thừa 1 chút            | tuktuk_img013_sam2  |
| img_017 | sam2            | 3(2 pos 1 neg) | mask đúng phần xe tuktuk       | mask hơi thiếu 1 chút ở trần xe | tuktuk_img013_sam2  |

- Đối chiếu guideline mục 4.2: theo v1.0 mục 4.2. Quy tắc giúp: mask xe chỉ ôm khung xe, mái che và bánh, người lái tách riêng bằng điểm `--neg`; 1 điểm âm làm img_004 và img_017 sạch người lái, nhưng ở img_013 chưa đủ vì đầu và chân người lái vẫn dính. Quy tắc còn thiếu: không nói rõ cần bao nhiêu điểm âm hoặc khi nào dùng `--box` cho ca người lái che sát thân xe; không nói mái/trần xe có tính vào mask không khi khung mái hở (img_017 bị thiếu trần).
- _(Tuỳ chọn)_ Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: không thử

## 6. Calibration (Part 6)

- Số quyết định: 5 (`calibration_decisions.csv`). Quan trọng nhất: CAL-01 (YOLO26 không có lớp three_wheeler nên bỏ sót hoặc gọi nhầm tuk-tuk, đổi ngưỡng không sửa được) và CAL-05 (`img_031.jpg`, bẫy loose lọt vì tôi chưa soi đủ 4 cạnh).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu hỏi mở 1 (xe đạp điện: có bàn đạp thì bicycle, có bệ để chân không bàn đạp thì motorbike); xe 3 bánh gồm tuk-tuk và xích lô là three_wheeler, người lái vẽ riêng; mỗi xe một box; kiểm đủ 4 cạnh mọi box.
- Nguyên nhân gốc: CAL-01, CAL-02, CAL-03 là máy; CAL-04 là guideline v1.0 chưa nói rõ; CAL-05 là tôi duyệt chưa đủ kỹ.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh `can_review` + 5 ngẫu nhiên, seed 42), 129 đối tượng (`qc_sample.csv`); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 1/129 = 0.8% · theo **ảnh**: 1/8 = 12.5% (ngưỡng Release ≤ 2%).
- Top lỗi: `sai_vi_tri_kich_thuoc` 1 (`img_031.jpg`, box `bicycle` lỏng, bẫy loose lọt). Các loại còn lại (bỏ sót, thừa, sai lớp, sai thuộc tính, sai mask): 0 trong mẫu.
- **Giới hạn mẫu nhỏ:** nếu 0 lỗi/8 ảnh thì cận trên Wilson 95% là 32% (`qc_report_auto.md`); với 1/8 cận trên khoảng 47% . Mẫu 8 ảnh chỉ phát hiện lỗi lớn, không chứng minh được lô dưới 2%.
- Script kiểm được/không kiểm được: script chỉ đếm theo phiếu bằng mắt. Nó không kiểm được độ ôm sát box, không biết người kiểm có cùng bỏ sót với máy hay không, và không biết phiếu có làm theo checklist thật hay không. Chính ca `img_031` cho thấy: phiếu ghi `khong` thì script gợi ý RELEASE dù bẫy loose đã lọt.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (Hold → Rework → Release): không Hold vì không còn câu hỏi mở nào ở v1.0 mục 6 và ảnh không hỏng. Rework vì ảnh lỗi 12.5% > 2% (bẫy loose ở `img_031` lọt cả duyệt lẫn QC lần đầu). Bắt bẫy 11/12 = 91.7% đạt ngưỡng nên không phải lý do.
Việc cần làm tiếp: sửa box `bicycle` ở `img_031.jpg` trên CVAT, xuất `qc_fixed.json`, chạy lại `build_release.py`, rút mẫu QC lớn hơn có soi 4 cạnh, chạy lại `qc_report.py`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng AI lọc đọc kết quả của các script sau đó tổng hợp lại và viết REPORT
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? không / có, như sau: Làm tay
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Window 11, GPU Nvidia/ CVAT 2.81; không có vướng mắt
- Dùng cache `data/cache/` thay vì tự chạy? không / có, vì : không dùng, tự chạy local
- Lỗi / bug script đã gặp (nếu có):
- Một điều bạn làm khác lần sau:
