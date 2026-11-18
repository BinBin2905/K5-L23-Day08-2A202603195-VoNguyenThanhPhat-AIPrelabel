# Số đo review (seg)

- Ảnh tính: 8 (batch); box máy: 78; đối tượng sau duyệt: 72; ngưỡng accepted: IoU >= 0.85


## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 53 | 53/78 = 67.9% | 78 box máy |
| edited (sửa) | 6 | 6/78 = 7.7% | 78 box máy |
| deleted (xoá) | 19 | 19/78 = 24.4% | 78 box máy |
| added (vẽ thêm) | 13 | 13/72 = 18.1% | 72 đối tượng sau duyệt |

Trong số deleted + added, **4 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 9.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 28 | 4 | 4 | 3 |
| bicycle | 2 | 0 | 2 | 1 |
| motorbike | 5 | 1 | 1 | 3 |
| car | 7 | 0 | 0 | 0 |
| bus | 0 | 0 | 0 | 1 |
| truck | 1 | 0 | 1 | 2 |
| three_wheeler | 10 | 1 | 11 | 3 |

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
