# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Nguyễn An Thái

Công cụ gán nhãn đã dùng: CVAT (AnyLabeling, CVAT, SAM hoặc sửa trực tiếp file nhãn)

Sao chép file này thành `reports/REPORT.md` rồi điền vào các chỗ ĐIỀN. Mọi con số phải truy được
từ `reports/rounds_table.md`, `outputs/selection_round1.csv`, `outputs/metrics_round*.json` hoặc
`outputs/round*_diff.md`. Không coi nhãn test do mô hình tạo là chân lý tuyệt đối.

## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng
đệm ở giữa, thay vì chia ngẫu nhiên? Nếu chia ngẫu nhiên, số đo trên tập kiểm thử sẽ bị lệch theo
hướng nào, và vì sao?

Bài toán sử dụng bộ ảnh giao thông ban đêm trên cao tốc, với đối tượng cần phát hiện là xe.
Tập kiểm thử được cố định gồm 20 ảnh với 403 box tham chiếu, trong đó bỏ qua 14 box có chiều cao dưới 16 px. Việc đánh giá sử dụng IoU = 0.5; Precision, Recall và F1 được tính tại confidence threshold = 0.25.
Dữ liệu train ở vòng 1 gồm 12 ảnh được lựa chọn bằng chiến lược học chủ động. Sau quá trình review pre-label trên CVAT, 12 ảnh này có tổng cộng 232 bounding box được sử dụng để fine-tune.
Do dữ liệu được lấy từ video/camera cố định, các frame gần nhau có thể chứa cùng một nhóm phương tiện ở các vị trí rất tương tự. Vì vậy khi chia dữ liệu cần tránh để các frame quá gần nhau xuất hiện đồng thời ở train và test, vì điều này có thể làm kết quả đánh giá bị lạc quan hơn thực tế.
Tập test được giữ cố định giữa các vòng để có thể so sánh trực tiếp sự thay đổi của mô hình.

## 2. Mô hình khởi đầu lạnh (cold start)

Chép dòng vòng 0 từ `rounds_table.md`. Dựa vào `outputs/compare_round0.jpg`, cho biết mô hình khởi
đầu lạnh không khớp nhãn tham chiếu ở những loại xe nào. Độ phủ (recall) theo kích thước xe cho
thấy điều gì? Một trường hợp nào cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai?

Ở vòng 0, mô hình không sử dụng ảnh train bổ sung và không fine-tune. Kết quả trên cùng tập test 20 ảnh:

| Chỉ số | Round 0 |
|---|---:|
| AP50 | 0.7714 |
| Precision | 0.9249 |
| Recall | 0.4888 |
| F1 | 0.6396 |
| Recall - Small | 0.1818 |
| Recall - Medium | 0.5473 |
| Recall - Large | 0.5610 |

Mô hình có Precision khá cao nhưng Recall thấp hơn đáng kể, đặc biệt với các xe nhỏ. Recall của nhóm small chỉ đạt 0.1818, trong khi medium đạt 0.5473 và large đạt 0.5610.
Điều này cho thấy một vấn đề đáng chú ý của bài toán là các xe nhỏ hoặc ở xa trong ảnh ban đêm khó được phát hiện đầy đủ. Đây cũng là nhóm đối tượng phù hợp để đưa vào quá trình lựa chọn mẫu bằng active learning.
Tuy nhiên, cần lưu ý rằng các box tham chiếu của tập test được tạo từ model và chưa được kiểm tra thủ công hoàn toàn. Vì vậy một số trường hợp model bị tính là FN có thể liên quan đến sai sót của reference labels.

## 3. Chiến lược chọn mẫu

Giải thích bằng lời công thức `score = W_U·U + W_A·A + W_D·D` và vai trò của `MIN_GAP_S`.
Dẫn ba frame trong `reports/SELECTION.md` và một frame khác để chứng minh cách bạn cân nhắc
độ bất định, ảnh gần trùng và công gán nhãn. Điểm bất định có chứng minh ảnh đó sẽ cải thiện
mô hình không? Vì sao?

Ở vòng 1, chiến lược active learning sử dụng uncertainty để tìm các frame mà mô hình có độ không chắc chắn cao và có khả năng cung cấp thêm thông tin cho quá trình huấn luyện.
Các frame được lựa chọn gồm:

- `frame_0099.jpg`
- `frame_0107.jpg`
- `frame_0182.jpg`
- `frame_0187.jpg`
- `frame_0227.jpg`
- `frame_0270.jpg`
- `frame_0312.jpg`
- `frame_0326.jpg`
- `frame_0331.jpg`
- `frame_0369.jpg`
- `frame_0380.jpg`
- `frame_0392.jpg`

Một số frame tiêu biểu:
- `frame_0182.jpg`: có nhiều xe với kích thước và mức độ rõ nét khác nhau, trong đó có các xe ở xa và các trường hợp khó quan sát.
- `frame_0369.jpg`: chứa nhiều xe ở các vị trí khác nhau, có nhiều đối tượng nhỏ và khó phân biệt trong điều kiện ánh sáng ban đêm.
- `frame_0380.jpg`: tiếp tục cung cấp các trường hợp xe có kích thước nhỏ và các vùng có độ tương phản thấp.
Các frame gần nhau về thời gian cần được hạn chế lựa chọn đồng thời vì camera cố định khiến nội dung giữa các frame có thể rất giống nhau. Mục tiêu của bước này là ưu tiên các mẫu có thông tin mới thay vì chỉ lấy nhiều frame gần như trùng lặp.
Quan trọng là việc một frame có uncertainty cao chỉ cho thấy frame đó có khả năng hữu ích cho việc review/annotation; nó không đảm bảo rằng việc đưa frame đó vào train sẽ làm AP50 tăng.

## 4. Các vòng học chủ động (active learning)

Chép bảng từ `rounds_table.md`. Với mỗi vòng, trình bày:

- mức độ bạn đã sửa nhãn gợi ý (số box giữ nguyên, chỉnh sửa, xoá, thêm mới, lấy từ
  `outputs/round*_diff.md`);
- AP50 thay đổi bao nhiêu so với khởi đầu lạnh và so với vòng trước;
- nhóm xe nào tốt lên hoặc xấu đi theo số đo trên cùng tập test.

Dựa vào các ảnh `compare_round*.jpg`, chỉ ra một ca kết quả đổi sau fine-tune (tốt hơn hoặc xấu
đi), cùng lý do có thể kiểm. Dùng `BLIND_SCAN.md`, `REVIEW_LOG.csv` và `round1_diff.md` phân biệt
quan sát độc lập, lỗi pre-label đã sửa và kết quả mô hình sau train. Mô tả một ca khó theo guideline.

### 4.1. Kết quả review pre-label

Trong vòng 1, model ban đầu đề xuất 169 box trên 12 ảnh. Sau khi review và chỉnh sửa trên CVAT, số box cuối cùng tăng lên 232.

| Action | Số lượng |
|---|---:|
| Accepted | 143 |
| Edited | 12 |
| Deleted | 14 |
| Added | 77 |
| Tổng box sau khi sửa | 232 |
| Accept rate | 85% |

Việc có 77 box được thêm vào cho thấy pre-label của model đã bỏ sót khá nhiều đối tượng trên các frame được lựa chọn. Đồng thời có 14 box bị xóa và 12 box được chỉnh sửa, cho thấy pre-label không chỉ có FN mà còn có FP và các bounding box cần hiệu chỉnh.

Các frame có số lượng box được thêm vào nhiều gồm:

| Frame | Model đề xuất | Sau khi sửa | Deleted | Added |
|---|---:|---:|---:|---:|
| frame_0099.jpg | 13 | 22 | 1 | 10 |
| frame_0107.jpg | 13 | 20 | 3 | 10 |
| frame_0182.jpg | 13 | 20 | 1 | 8 |
| frame_0187.jpg | 14 | 21 | 1 | 8 |
| frame_0270.jpg | 13 | 18 | 0 | 5 |
| frame_0369.jpg | 14 | 24 | 0 | 10 |

Những thay đổi này được thực hiện trước khi dùng dữ liệu để fine-tune model.

### 4.2. So sánh kết quả giữa các vòng

Tập test được giữ cố định gồm 20 ảnh và 403 box tham chiếu.

| Vòng | Model | Ảnh train | Box train | AP50 | Δ AP50 so cold start | P | R | F1 | R small | R medium | R large |
|---:|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | YOLOv8n cold start | 0 | 0 | 0.7714 | — | 0.9249 | 0.4888 | 0.6396 | 0.1818 | 0.5473 | 0.5610 |
| 1 | YOLOv8n fine-tune | 12 | 232 | 0.3823 | -0.3891 | 1.0000 | 0.0248 | 0.0484 | 0.0000 | 0.0135 | 0.1463 |

Sau vòng fine-tune, AP50 giảm từ 0.7714 xuống 0.3823, tương ứng giảm 0.3891 điểm.

Precision tăng từ 0.9249 lên 1.0000, nhưng Recall giảm rất mạnh từ 0.4888 xuống 0.0248. F1 cũng giảm từ 0.6396 xuống 0.0484.

Theo kích thước đối tượng:

- Recall small giảm từ 0.1818 xuống 0.0000.
- Recall medium giảm từ 0.5473 xuống 0.0135.
- Recall large giảm từ 0.5610 xuống 0.1463.

Như vậy model vòng 1 chỉ phát hiện được một số rất ít đối tượng nhưng các prediction được tạo ra trong kết quả đánh giá đều có Precision cao. Vấn đề chính của vòng 1 là Recall bị suy giảm nghiêm trọng.

### 4.3. Quan sát từ ảnh so sánh

Ảnh so sánh giữa reference, cold-start và round 1 cho thấy sự khác biệt rõ giữa hai model.

Ở các frame được minh họa, cold-start vẫn phát hiện được nhiều xe, mặc dù còn bỏ sót một số đối tượng. Ví dụ:

- `frame_0050`: cold-start có TP = 11, FP = 2, FN = 7.
- `frame_0150`: cold-start có TP = 10, FP = 2, FN = 10.
- `frame_0250`: cold-start có TP = 6, FP = 2, FN = 9.
- `frame_0350`: cold-start có TP = 9, FP = 2, FN = 14.

Trong khi đó, model round 1 trên các frame minh họa chỉ đạt:

- `frame_0050`: TP = 1, FP = 0, FN = 17.
- `frame_0150`: TP = 1, FP = 0, FN = 19.
- `frame_0250`: TP = 1, FP = 0, FN = 14.
- `frame_0350`: TP = 1, FP = 0, FN = 22.

Quan sát này phù hợp với kết quả tổng thể: round 1 tạo ra ít false positive nhưng bỏ sót phần lớn các xe có trong reference.

Một điểm cần phân biệt là:

1. `BLIND_SCAN.md` phản ánh những gì người gán nhãn quan sát trực tiếp từ ảnh.
2. `REVIEW_LOG.csv` và `round1_diff.md` phản ánh những gì đã được sửa trong pre-label trước khi train.
3. `metrics_round1.json` và ảnh compare phản ánh hành vi của model sau khi fine-tune.

Ba nguồn này không nên được trộn lẫn khi giải thích nguyên nhân hoặc đánh giá kết quả.

## 5. Kết luận và giới hạn

Kết quả vòng này so với cold start ra sao? Vì sao bạn dừng hoặc tiếp tục? Đề xuất hai ca còn yếu
hoặc bất định cho vòng sau, kèm chi phí rà nhãn và nguy cơ ảnh gần trùng. Tập kiểm thử chỉ 20 ảnh,
có luật bỏ qua xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được rà thủ công; các giới hạn đó
ảnh hưởng thế nào đến kết luận? Nếu AP50 giảm, bạn sẽ kiểm tra điều gì trước khi train thêm?

Vòng 1 đã hoàn thành quy trình active learning cơ bản:

1. Chọn 12 frame bằng uncertainty.
2. Sử dụng pre-label từ model.
3. Review và sửa annotation trên CVAT.
4. Tăng dữ liệu train lên 232 bounding boxes.
5. Fine-tune YOLOv8n trong 50 epochs với image size 960.
6. Đánh giá lại trên cùng tập test 20 ảnh.

Kết quả vòng 1 chưa cho thấy sự cải thiện so với cold start. AP50 giảm từ 0.7714 xuống 0.3823. Đặc biệt, Recall giảm rất mạnh từ 0.4888 xuống 0.0248.

Một nguyên nhân cần xem xét là sự khác biệt lớn giữa dữ liệu train chỉ gồm 12 frame và phân bố của tập test. Ngoài ra, dữ liệu được lấy từ các frame ban đêm và có nhiều xe nhỏ, xa hoặc khó quan sát. Việc fine-tune trên một số lượng frame hạn chế có thể khiến model học chưa tốt phân bố của toàn bộ tập test.

Một giới hạn khác là reference labels của tập test chưa được manually reviewed hoàn toàn. Vì vậy kết quả AP50 và các FN cần được diễn giải thận trọng.

Đặc biệt, việc vòng 1 giảm mạnh Recall cho thấy cần kiểm tra lại:

- chất lượng và độ nhất quán của annotation sau khi review;
- sự tương đồng giữa label train và reference label;
- cách chuẩn bị dataset trước khi fine-tune;
- phân bố kích thước object trong 12 ảnh train;
- cấu hình training và cách model được sử dụng sau fine-tune;
- các trường hợp model không tạo prediction trên ảnh test.

Do đó, kết quả vòng 1 nên được xem là một vòng thử nghiệm để phát hiện vấn đề trong pipeline, thay vì chỉ dựa vào AP50 để kết luận về hiệu quả của active learning.

