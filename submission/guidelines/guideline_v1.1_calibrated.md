# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604069 - Vu Gia Khang

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao) và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Phân loại theo cơ cấu truyền động: Có bàn đạp trợ lực gán `bicycle`; không có bàn đạp (chạy thuần tay ga/kiểu dáng xe tay ga điện) gán `motorbike`. | `img_009`, `img_018` | `CAL-02` |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Cụm $\ge 5$ xe máy đỗ chen chúc bị che khuất $> 80\%$ diện tích và khó xác định ranh giới thì gom thành 1 `ignore_region` bao cụm; các xe ở rìa ngoài còn thấy $\ge 20\%$ vẫn vẽ box riêng. | `img_005`, `img_028` | `CAL-03` |
| 3 | Hàng nhô ra: xe đạp / ba gác chở hàng cồng kềnh nhô xa: box có gồm hàng không? | Hàng hóa gắn cố định trên giá xe hoặc trong giỏ xe thuộc box xe. Nếu hàng hóa nhô ra $> 20\%$ kích thước xe (cột tre, thanh sắt dài) thì không bao trùm phần nhô xa để tránh làm méo tỷ lệ bounding box của phương tiện. | `img_011`, `img_032` | `CAL-01` |
| 4 | Người ngồi trong xe ba bánh có mui che (chỉ thấy thấp thoáng): ngưỡng còn thấy $\ge 20\%$ có hợp lý không? | Giữ nguyên ngưỡng $\ge 20\%$, nhưng bổ sung tiêu chí nhận diện: phải nhìn thấy rõ ít nhất phần đầu/vai hoặc nửa thân trên qua cửa xe thì mới gán `person`; chỉ thấy bóng mờ sau rèm/kính tối màu thì không gán. | `img_013`, `img_004` | `CAL-01` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013`, `img_004` | Xe tuk-tuk có mui chở người ở giữa ảnh | YOLO26 gán nhầm thành `car`/`truck` (conf 0.42); SAM phủ mask xe dính cả người lái | Sửa nhãn sang `three_wheeler`; tách riêng box và mask của `person` điều khiển | Mục 4.1.4 & 4.2.4: Xe ba bánh có mui luôn là `three_wheeler`; người lái phải là polygon/box độc lập |
| `img_027` | Xe máy nhỏ ở hậu cảnh xa bị detector bỏ sót | Detector bỏ sót xe máy (bẫy missing motorbike); người duyệt bỏ qua vì bị neo nhãn | Thêm box `motorbike` [702.6, 369.6]; siết quy trình quét 4 góc phần tư trước khi bật nhãn | Mục 3 & Quy trình review: Bắt buộc ẩn nhãn máy và quét ảnh theo góc phần tư đối với ảnh đông vật thể |
| `img_028` | Người đi bộ trong đám đông bị detector vẽ box lệch/lỏng (loose trap) | Máy vẽ box rộng thừa nền đất phía dưới chân (IoU 0.52 so với vật thật) | Kéo cạnh box khít lại đúng đỉnh đầu và chân của người đi bộ | Mục 4.1.1: Bounding box phải ôm sát các điểm cực trị (đầu, chân, tay) của người, không để thừa khoảng trống nền $> 5px$ |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | 1. Chốt phân loại xe điện theo bàn đạp (`CAL-02`).<br>2. Quy định cụm xe máy dày đặc che khuất $>80\%$ dùng `ignore_region` (`CAL-03`).<br>3. Bổ sung quy tắc tách `person` khỏi `three_wheeler` có mui và sửa lỗi phân loại xe ba bánh của YOLO (`CAL-01`).<br>4. Thêm quy định quét 4 góc phần tư tránh lỗi bỏ sót vật thể ở xa (`CAL-04`). | Ca thực tế từ Part 2, Part 4 và Part 5 (`CAL-01`, `CAL-02`, `CAL-03`, `CAL-04`) |
