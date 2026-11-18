# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 432; đối tượng sau duyệt: 430; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 386 | 386/432 = 89.4% | 432 box máy |
| edited (sửa) | 15 | 15/432 = 3.5% | 432 box máy |
| deleted (xoá) | 31 | 31/432 = 7.2% | 432 box máy |
| added (vẽ thêm) | 29 | 29/430 = 6.7% | 430 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 19.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 241 | 5 | 4 | 5 |
| bicycle | 4 | 2 | 3 | 2 |
| motorbike | 85 | 2 | 2 | 9 |
| car | 43 | 1 | 6 | 5 |
| bus | 2 | 0 | 1 | 2 |
| truck | 3 | 0 | 4 | 2 |
| three_wheeler | 8 | 5 | 11 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 50 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 37 | 27 | 13 | 57.8% (KTC 95%: 46–69%) | 74.0% (KTC 95%: 60–84%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - loose: 3/3
  - wrong_class: 3/3
  - extra: 3/3
  - lọt: img_027.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
