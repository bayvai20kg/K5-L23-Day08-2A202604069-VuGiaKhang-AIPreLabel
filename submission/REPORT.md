# Báo cáo Day 08 — 2A202604069 - Vu Gia Khang

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / car / bus / truck | YOLO26m | 0.30 (mặc định) | `r01`, `r02`: YOLO26m nhận diện tốt 80 lớp COCO, ở mức 0.30 cân bằng tốt giữa recall và precision; bicycle/bus/truck có < 5 vật trong gold nên giữ mặc định theo quy tắc thống kê. |
| motorbike | YOLO26m | 0.10 | `threshold_sweep.md`: Ở ngưỡng 0.10, motorbike đạt recall 92.3% (12/13 vật gold), đảm bảo không sót phương tiện phổ biến nhất trên phố châu Á. |
| three_wheeler | YOLOE-26s | 0.15 | `r03`, `threshold_sweep.md`: YOLOE prompt `tuk-tuk, cyclo, cargo tricycle` đạt recall 100% (7/7 vật gold) ở ngưỡng 0.15, bổ sung cho điểm mù của YOLO26. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 base (`sam2.1_b.pt`)**. Lý do: SAM 2.1 cân bằng xuất sắc giữa chất lượng bám biên mask và tốc độ thực thi; qua so sánh trực quan ở `img_013` giữa SAM 1 (`mobile_sam.pt`) và SAM 2.1, SAM 2.1 xử lý vùng giao cắt giữa mui xe và thân xe mượt hơn hẳn, ít bị răng cưa; dung lượng 162 MB nhẹ hơn SAM 3 rất nhiều và không đòi hỏi quyền truy cập gated model trên HuggingFace.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi nhận nhầm bản chất lớp giữa các họ model (YOLO26 là closed-set không có nhãn `three_wheeler` nên ép gán xe ba bánh thành `car` hoặc `truck` với conf cao > 0.77 ở `img_011`). Đổi ngưỡng không giải quyết được vì nếu tăng ngưỡng thì mất box, hạ ngưỡng vẫn sai nhãn. Cách sửa: Phải kết hợp model open-vocabulary (YOLOE) với text prompt riêng cho xe ba bánh, rồi dùng `merge_prelabels.py` với NMS IoU 0.6 cho phép `three_wheeler` của YOLOE ghi đè lên `car`/`truck` của YOLO26.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - *Điểm cao mà sai:* Tại `img_011.jpg`, chiếc xe tuk-tuk bên mép trái bị YOLO26 gán nhầm là `truck` với điểm tin cậy rất cao (conf 0.77); tại `img_013.jpg`, tuk-tuk bị gán thành `car` (conf 0.42).
  - *Điểm thấp mà đúng:* Tại `img_011.jpg`, xe máy ở hậu cảnh xa có điểm tin cậy chỉ 0.35 nhưng thực tế là xe máy hợp lệ; tại `img_032.jpg`, tuk-tuk ở xa có conf 0.27 nhưng vẫn bắt đúng vị trí.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Quá trình quét ngưỡng được thực hiện trên chính 4 ảnh gold (in-sample) với tổng cộng chỉ 50 đối tượng. Do cỡ mẫu quá nhỏ, các lớp hiếm (bus, truck, bicycle có 0-1 vật) không thể suy rộng thống kê, và điểm precision/recall đo được trên gold cao hơn thực tế khi triển khai trên toàn bộ tập dữ liệu mở rộng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 386 | 89.4% | 432 box máy |
| edited | 15 | 3.5% | 432 box máy |
| deleted | 31 | 7.2% | 432 box máy |
| added | 29 | 6.7% | 430 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` là tệ nhất: máy sinh ra nhiều box thừa và box lỏng (deleted = 11 box, edited = 5 box trên tổng số 19 box). Ví dụ tại `img_011.jpg`, YOLOE sinh 1 box kéo dài toàn bộ mép trái màn hình; tại `img_002.jpg`, YOLOE sinh các box lỏng bao trùm cả tài xế và vượt quá thân xe. Ngoài ra lớp `car` cũng bị xóa 6 box do nhận nhầm xe ba gác thành ô tô.
- So với gold: precision / recall của **máy**: Precision đạt 57.8% (37/64 box máy khớp đúng), Recall đạt 74.0% (37/50 đối tượng gold được máy tìm thấy, mẫu số: 50 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: `11/12 = 91.7%` (vượt ngưỡng yêu cầu $\ge 80\%$). Loại lọt: 1 bẫy `missing` ở `img_027.jpg`. Lý do lọt: Ở ảnh `img_027.jpg`, chiếc xe máy ở xa bị che khuất một phần trong khung cảnh nhiều xe cộ. Do mắt bị "neo nhãn" vào các box máy đã vẽ sẵn ở tiền cảnh, người duyệt đã bỏ qua Bước 0 (quét ảnh tổng thể trước khi hiện nhãn máy) nên không nhận ra vật thể bị bỏ sót.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Nếu `accepted` quá cao (> 95%) trong khi tỉ lệ bắt bẫy thấp (< 80%), điều đó chứng minh người duyệt bị hội chứng "làm biếng nhận thức" (automation bias), thụ động chấp nhận mọi gợi ý của AI mà không thực sự kiểm tra kỹ. Ngược lại, nếu bắt bẫy cao mà accepted vừa phải (~85-90%) chứng tỏ người duyệt giữ được tư duy phản biện độc lập, tuân thủ đúng 4 bước checklist để rà soát lỗi của máy.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 7.5 phút/ảnh (30 phút / 4 ảnh gold) vs duyệt có pre-label 2.9 phút/ảnh (70 phút / 24 ảnh batch). Tiết kiệm được ~61.3% thời gian trên mỗi ảnh. Thiên lệch: Do 4 ảnh gold được gán tay từ đầu nên người duyệt đã quen mắt với các loại phương tiện và bối cảnh phố xá, giúp tốc độ review ở batch sau nhanh hơn một phần nhờ hiệu ứng học tập (learning curve).
- Kết luận: **Rất đáng dùng**, nhưng không áp dụng đồng đều cho mọi lớp. Với các lớp phổ biến (`person`, `motorbike`, `car`), AI tiết kiệm hơn 60% công sức. Tuy nhiên với lớp đặc thù (`three_wheeler`), AI chỉ đóng vai trò gợi ý sơ bộ và con người bắt buộc phải chỉnh sửa lại box và phân loại lớp.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Luôn thực hiện Bước 0: Ẩn 100% layer nhãn máy, tự nhẩm đếm số lượng người và phương tiện trong 15 giây đầu tiên ở mọi ảnh trước khi bật nhãn máy lên kiểm tra.
  2. Với các ảnh có mật độ dày đặc (> 20 vật thể như `img_005`, `img_018`, `img_028`), bắt buộc chia ảnh thành 4 góc phần tư để duyệt tuần tự, tránh bị cuốn mắt vào khu vực trung tâm.
  3. Tuyệt đối không tin tưởng điểm tin cậy cao (`conf > 0.8`): kiểm tra kỹ viền bounding box của tất cả các xe ba bánh và người đi bộ để phát hiện lỗi lỏng box và lỗi gán sai lớp `car`/`three_wheeler`.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Không gồm người lái (đã loại bỏ bằng 1 click âm); mask ôm khít thân xe | Ban đầu 1 click dương tạo mask nhỏ ở huy hiệu, thêm click dương thứ 2 thì phủ cả xe dính người lái; cần thêm 1 click âm vào ngực tài xế để tách người ra | `submission/masks/tuktuk_img013_sam2.jpg` |
| `img_013.jpg` | SAM 1 (`mobile_sam.pt`) | 3 click (2 pos, 1 neg) | Tách được người lái nhưng viền mui xe bị răng cưa | So sánh cùng điểm click với SAM 2.1: SAM 1 có đường biên thô hơn, vùng viền bánh xe bị lẹm nhẹ vào nền | `submission/masks/tuktuk_img013_sam1.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Mask ôm khít phần thân xe tuk-tuk, tách biệt khỏi cột đèn và xe bên cạnh | Vùng mui sau bị bóng râm che khuất cần đặt click dương thứ 2 vào chính giữa vùng tối mới nhận đủ | `submission/masks/tuktuk_img004_sam2.jpg` |
| `img_017.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Mask ôm sát xe ba gác, tách được người đi bộ phía sau | Cần 1 click âm vào chân người đi bộ đứng sát thành xe để tránh mask tràn sang người | `submission/masks/tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc "hai vật chạm nhau: mỗi vật một polygon không chồng lấn" rất cần thiết khi dùng SAM vì SAM có xu hướng gom các vật thể tiếp xúc gần nhau thành một mảng mask chung. Thiếu sót của SAM là không tự hiểu được khái niệm "người lái là đối tượng độc lập", đòi hỏi con người phải can thiệp bằng các click âm (negative points).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Hoạt động trực quan tương tự script offline (click chuột trái thêm vùng, chuột phải trừ vùng), tuy nhiên khi mạng chậm hoặc ảnh nặng thì độ trễ phản hồi giữa client và server Docker cao hơn so với chạy script CLI cục bộ.

## 6. Calibration (Part 6)

- Số quyết định: 4 quyết định (`submission/qc/calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-01`): Chốt quy tắc phân định xe ba bánh có mui (`three_wheeler`) và bắt buộc tách riêng đối tượng người điều khiển (`person`), giải quyết triệt để lỗi lan truyền khi YOLO26 nhận nhầm thành ô tô.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bổ sung tiêu chí phân loại xe điện theo bàn đạp (`CAL-02`); quy định ngưỡng gom cụm $\ge 5$ xe máy đỗ chen chúc che khuất $>80\%$ vào `ignore_region` (`CAL-03`); siết quy trình quét 4 góc phần tư đối với ảnh đông đúc (`CAL-04`).
- Nguyên nhân gốc: Phần lớn xuất phát từ **guideline v1.0 còn chỗ mơ hồ** (chưa quy định cụ thể về xe điện, cụm xe máy đỗ gần) kết hợp với **lỗi cố hữu của detector** (closed-set mù lớp đặc thù địa phương).

## 7. QC và quyết định (Part 6)

- Mẫu: 7 ảnh (2 ảnh có ca khó/`can_review` + 5 ảnh ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: tự kiểm tra lại nhãn của chính mình nên có thể tồn tại điểm mù nhận thức chủ quan).
- Lỗi theo **đối tượng**: 0/124 đối tượng = 0.0% · theo **ảnh**: 0/7 ảnh = 0.0% (đạt ngưỡng Release $\le 2\%$).
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: Trong mẫu kiểm định 7 ảnh không ghi nhận lỗi hệ thống nào; các trường hợp sai lệch nhỏ trước đó ở Part 4 (`img_005` box ma, `img_028` box lỏng) đều đã được khắc phục hoàn toàn trong đợt review hiệu chỉnh.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 0 ảnh lỗi trên mẫu 7 ảnh, cận trên khoảng tin cậy 95% Wilson của tỉ lệ ảnh lỗi thật trong tổng thể có thể lên tới **35%**. Do đó, về mặt thống kê toán học, cỡ mẫu 7 ảnh chưa đủ mạnh để khẳng định chắc chắn 100% rằng toàn bộ dataset đạt sai số dưới 2%, đây là giới hạn thực tế cần nêu rõ.
- Script kiểm được / không kiểm được: Script kiểm tra tự động được định dạng COCO, kích thước bounding box hợp lệ ($>20\text{px}$), sự có mặt của polygon mask. Script **không** tự đánh giá được liệu bounding box có ôm khít vật thể thực tế trong ảnh hay không và không phát hiện được nếu cả người lẫn máy cùng bỏ sót một vật thể ngoài thực địa.

**Quyết định: Release**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release):
1. Không có vướng mắc guideline chưa chốt và không có ảnh hỏng $\rightarrow$ Không cần **Hold**.
2. Tỉ lệ ảnh lỗi trong mẫu QC là 0.0% ($\le 2\%$), không có lỗi hệ thống lặp lại, tỉ lệ bắt bẫy đạt 91.7% ($\ge 80\%$) $\rightarrow$ Không cần **Rework**.
3. Toàn bộ 24 ảnh batch đã được review hoàn chỉnh, đáp ứng đầy đủ tiêu chuẩn nghiệm thu $\rightarrow$ Đạt điều kiện **Release**.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Dùng trợ lý Antigravity để hỗ trợ chạy các script CLI, kiểm tra cú pháp COCO JSON và kiểm tra các điều kiện nghiệm thu của bộ test `check_submission.py`. Toàn bộ dữ liệu được xác thực lại bằng các script độc lập của ban tổ chức.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không bị nhiễm. 4 ảnh gold được gán tay thủ công từ Part 1 hoàn toàn độc lập trước khi chạy bất kỳ mô hình pre-label nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA GeForce GTX 1650 Ti with Max-Q Design (4GB VRAM), Python 3.11, CVAT chạy trên Docker Desktop v2.81.0. Ban đầu gặp khó khăn do PyTorch mặc định là bản CPU, sau đó đã khắc phục bằng cách chạy mô hình với các tham số tối ưu.
- Dùng cache `data/cache/` thay vì tự chạy? Không dùng cache. Toàn bộ các lần chạy YOLO26, YOLOE, SAM và quét ngưỡng đều được thực thi thật trên máy.
- Lỗi / bug script đã gặp (nếu có): Quá trình cài đặt ban đầu gặp hiện tượng file lock trên Windows khi mở đồng thời CSV trong trình đọc bảng tính; đã khắc phục bằng cách đóng ứng dụng trước khi ghi đè file.
- Một điều bạn làm khác lần sau: Sẽ luôn chia ảnh thành 4 góc phần tư để quét thủ công trước khi hiển thị nhãn máy ngay từ những ảnh đầu tiên để không bỏ sót các vật thể nhỏ ở cự ly xa.
