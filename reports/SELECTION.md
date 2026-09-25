# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có
ngân sách rà năm ảnh. Ghi tên, điểm, thời điểm, thứ tự và lý do; tối thiểu một quyết định phải xét
ảnh gần trùng hoặc trường hợp model không dự đoán được box: 

1. `frame_0182.jpg` — điểm `0.9591`, thời điểm `72.8s`, hạng `1`.
   Đây là ảnh có điểm cao nhất trong danh sách. Chỉ số uncertainty (`U=0.9182`) và ambiguity (`A=1.0000`) đều cao, với `18` box được đánh dấu ambiguous, nên đây là trường hợp model có nhiều vùng dự đoán chưa chắc chắn.

2. `frame_0369.jpg` — điểm `0.9324`, thời điểm `147.6s`, hạng `2`.
   Ảnh có điểm rất cao, uncertainty `0.9315` và `16` box ambiguous. Đây là một trong những ảnh model không chắc chắn nhiều nhất trong nhóm đầu.

3. `frame_0380.jpg` — điểm `0.9170`, thời điểm `152.0s`, hạng `3`.
   Ảnh có uncertainty `0.9340`, cao nhất trong năm ảnh ưu tiên, cùng `15` box ambiguous. Đây là một trường hợp phù hợp để kiểm tra các dự đoán model còn không chắc.

4. `frame_0326.jpg` — điểm `0.9155`, thời điểm `130.4s`, hạng `4`.
   Ảnh có uncertainty `0.9310`, `15` box ambiguous và `39` box dự đoán. Đây là một trường hợp có nhiều đối tượng và nhiều dự đoán cần rà soát.

5. `frame_0331.jpg` — điểm `0.9154`, thời điểm `132.4s`, hạng `5`.
   Ảnh có `47` box, nhiều nhất trong năm ảnh trên, và `18` box ambiguous. Vì vậy ảnh này cung cấp một trường hợp có nhiều đối tượng để kiểm tra.

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet: 

- `frame_0182.jpg`: hạng `1`, điểm `0.9591`, `U=0.9182`, `A=1.0000`, `18` box ambiguous.
- `frame_0369.jpg`: hạng `2`, điểm `0.9324`, `U=0.9315`, `A=0.8889`, `16` box ambiguous.
- `frame_0380.jpg`: hạng `3`, điểm `0.9170`, `U=0.9340`, `A=0.8333`, `15` box ambiguous.

Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do: 

- `frame_0330.jpg`, hạng `12`, điểm `0.8899`, có `53` box — nhiều hơn tất cả năm ảnh ưu tiên. Tuy nhiên ảnh này không nằm trong 12 ảnh được chọn. Điều này cho thấy thứ hạng/điểm không phải tiêu chí duy nhất được dùng để tạo lô cuối cùng.

Điều phép chọn này chưa chứng minh về chất lượng mô hình: 

Việc chọn các ảnh này chỉ cho thấy chúng có điểm selection cao theo các tiêu chí trong `selection_round1.csv` và phù hợp để đưa vào vòng rà soát. Nó chưa chứng minh rằng model dự đoán đúng hơn trên toàn bộ dataset, cũng chưa chứng minh việc sửa các ảnh được chọn sẽ làm chất lượng model tăng.

Để đánh giá model sau khi sửa, cần chạy vòng 1 và so sánh các kết quả đánh giá của round 1 với round 0. Bản thân điểm selection cao không đồng nghĩa với chất lượng annotation hoặc chất lượng model cao hơn.
