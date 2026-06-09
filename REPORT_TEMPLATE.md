# CSC4005 Lab 7 Report – Compression: KD + Quantization Trade-offs

## 1. Thông tin

| STT | Họ tên | Mã sinh viên | Lớp |
| :-: | ------ | ------------ | --- |
|  1  | Nguyễn Nam Cường    | 1671040005          | KHMT 16-01 |
|  2  | Nguyễn Trung Thành    | 1671040025          | KHMT 16-01 |
|  3  | Triệu Quốc Anh    | 1671040002          | KHMT 16-01 |

- Link GitHub repo:
- Kỹ thuật chọn: Quantization 
- Link W&B nếu dùng KD:
- Link model nếu không commit trực tiếp:

## 2. Mô tả baseline model

| Nội dung                   | Giá trị                                                                 |
| -------------------------- | ----------------------------------------------------------------------- |
| Bài toán                   | Smart Campus Scene Classification                                       |
| Dataset                    | MIT Indoor Scenes 67 subset (5 classes)                                 |
| Số lớp                     | 5                                                                       |
| Baseline model             | Vision Transformer (ViT-B/16)                                           |
| Baseline format            | PyTorch / ONNX                                                          |
| Baseline checkpoint / ONNX | `checkpoints/teacher_vit_best_model.pt` / `models/vit_smartcampus.onnx` |
| Baseline model size        | 327.40 MB                                                               |

## 3. Kỹ thuật nén đã chọn

### Quantization

| Thông tin | Giá trị |
|---|---|
| Loại quantization | Dynamic |
| Input model | models\vit_smartcampus.onnx |
| Output model | models\vit_smartcampus_dynamic_int8.onnx |
| Dạng dữ liệu sau nén | INT8 |
| Công cụ | onnxruntime.quantization |

Mô tả ngắn:

```text
Dynamic Quantization chuyển trọng số mô hình từ FP32 sang INT8 trong quá trình suy luận nhằm giảm kích thước mô hình và tăng tốc inference trên CPU mà không cần huấn luyện lại mô hình
```

## 4. Kết quả đánh giá

| Model | Accuracy | Macro-F1 | Model size (MB) |
|---|---:|---:|---:|
| Baseline ViT | 98% | 97.4% | 327.4 |
| Quantized Int8 | 97.5% | 96.5% | 84.4 |

### Nhận xét

- Accuracy giảm từ **98.0%** xuống **97.4%**, tương đương giảm **0.6 điểm phần trăm**.
- Macro-F1 giảm từ **97.5%** xuống **96.5%**, tương đương giảm **1.0 điểm phần trăm**.
- Mức suy giảm về độ chính xác là khá nhỏ so với lợi ích đạt được từ việc lượng tử hóa mô hình.
- Kích thước mô hình giảm mạnh từ **327.4 MB** xuống **84.4 MB** (giảm khoảng **74.2%**), giúp tiết kiệm đáng kể bộ nhớ lưu trữ và tài nguyên hệ thống.
- Kết quả cho thấy Dynamic INT8 Quantization vẫn duy trì được chất lượng phân loại ở mức cao trong khi cải thiện đáng kể khả năng triển khai thực tế.
- Với bài toán Smart Campus Scene Classification, mức giảm Accuracy và Macro-F1 này hoàn toàn có thể chấp nhận được vì mô hình sau nén vẫn đạt hiệu năng trên **96%** Macro-F1 và **97%** Accuracy.
- Do đó, mô hình Quantized INT8 mang lại trade-off tốt giữa độ chính xác, tốc độ suy luận và kích thước mô hình, phù hợp cho việc triển khai trên CPU hoặc các thiết bị có tài nguyên hạn chế.

## 5. Kết quả benchmark

| Model | ONNX Path | Batch Size | Mean Latency (ms) | Median Latency (ms) | P95 Latency (ms) | Throughput (img/s) | Model Size (MB) |
|---------|---------|---------:|---------:|---------:|---------:|---------:|---------:|
| Baseline ONNX | `models/vit_smartcampus.onnx` | 1 | 199.89 | 193.37 | 256.44 | 5.00 | 327.36 |
| Baseline ONNX | `models/vit_smartcampus.onnx` | 4 | 860.42 | 869.95 | 992.24 | 4.65 | 327.36 |
| Baseline ONNX | `models/vit_smartcampus.onnx` | 8 | 1622.73 | 1588.05 | 1844.37 | 4.93 | 327.36 |
| Quantized INT8 | `models/vit_smartcampus_dynamic_int8.onnx` | 1 | 145.99 | 140.66 | 225.04 | 6.85 | 84.42 |
| Quantized INT8 | `models/vit_smartcampus_dynamic_int8.onnx` | 4 | 572.62 | 578.20 | 660.66 | 6.99 | 84.42 |
| Quantized INT8 | `models/vit_smartcampus_dynamic_int8.onnx` | 8 | 1246.54 | 1230.45 | 1400.97 | 6.42 | 84.42 |

## 6. Bảng trade-off

## Trade-off Analysis

| Model | Accuracy | Macro-F1 | Mean Latency @ BS=1 (ms) | Throughput @ BS=1 (img/s) | Size (MB) | Nhận xét |
|---|---:|---:|---:|---:|---:|---|
| Baseline | 0.912 | 0.908 | 199.89 | 5.00 | 327.36 | Độ chính xác cao nhất nhưng yêu cầu nhiều bộ nhớ và thời gian suy luận hơn. |
| Quantized INT8 | 0.905 | 0.901 | 145.99 | 6.85 | 84.42 | Accuracy giảm nhẹ (~0.7 điểm %) nhưng kích thước giảm 74.2%, latency giảm 27.0% và throughput tăng 36.9%, mang lại trade-off rất tốt cho triển khai thực tế. |

## 7. Phân tích

### 1. Mô hình sau nén nhỏ hơn bao nhiêu phần trăm?

Kích thước mô hình giảm từ **327.4 MB** xuống **84.4 MB**.
Mô hình sau khi lượng tử hóa nhỏ hơn khoảng **74.2%** so với mô hình ban đầu.

### 2. Latency giảm hay tăng?

Kết quả benchmark cho thấy độ trễ suy luận giảm ở tất cả các batch size. Với batch size = 1, mean latency giảm từ **199.89 ms** xuống **145.99 ms**, tương đương giảm khoảng **27.0%**. Điều này cho thấy mô hình lượng tử hóa có khả năng suy luận nhanh hơn trên CPU.

### 3. Throughput thay đổi thế nào?

Throughput tăng đáng kể sau khi áp dụng Dynamic Quantization. Với batch size = 1, throughput tăng từ **5.00 img/s** lên **6.85 img/s**, tương đương tăng khoảng **36.9%**. Kết quả này cho thấy hệ thống có thể xử lý được nhiều ảnh hơn trong cùng một khoảng thời gian.

### 4. Accuracy/F1 giảm nhiều không?

Accuracy giảm từ **98.0%** xuống **97.4%**, tương đương giảm **0.6 điểm phần trăm**. Macro-F1 giảm từ **97.5%** xuống **96.5%**, tương đương giảm **1.0 điểm phần trăm**. Mức suy giảm này là khá nhỏ so với lợi ích thu được về tốc độ suy luận và kích thước mô hình.

### 5. Nếu triển khai trên CPU hoặc edge device, bạn có chọn compressed model không?

Có. Mô hình Quantized INT8 là lựa chọn phù hợp để triển khai trên CPU hoặc các thiết bị edge vì kích thước nhỏ hơn đáng kể, yêu cầu ít bộ nhớ hơn và có tốc độ suy luận nhanh hơn trong khi độ chính xác vẫn được duy trì ở mức cao.

### 6. Nếu không chọn, lý do là gì?

Trong những trường hợp yêu cầu độ chính xác cao nhất và tài nguyên phần cứng không phải là vấn đề, mô hình baseline có thể được ưu tiên do đạt Accuracy và Macro-F1 cao hơn một chút. Tuy nhiên, sự khác biệt này là không đáng kể đối với hầu hết các bài toán thực tế.

---

## 8. Khi nào chọn KD, khi nào chọn Quantization?

### Khi nào Quantization phù hợp?

Quantization phù hợp khi cần giảm kích thước mô hình và tăng tốc độ suy luận mà không muốn huấn luyện lại mô hình. Đây là lựa chọn hiệu quả khi triển khai trên CPU hoặc các thiết bị có tài nguyên hạn chế. Ngoài ra, quantization thường dễ áp dụng và có chi phí triển khai thấp.

### Khi nào KD phù hợp?

Knowledge Distillation (KD) phù hợp khi cần xây dựng một mô hình nhỏ hơn nhưng vẫn duy trì độ chính xác gần với mô hình lớn. Phương pháp này yêu cầu quá trình huấn luyện student model nên tốn nhiều thời gian và tài nguyên hơn, nhưng thường cho kết quả tốt hơn về mặt cân bằng giữa kích thước và độ chính xác.

### Nếu được làm lại, bạn sẽ chọn kỹ thuật nào cho hệ thống Smart Campus?

Đối với hệ thống Smart Campus, tôi sẽ ưu tiên lựa chọn **Dynamic Quantization** vì phương pháp này đơn giản, dễ triển khai và mang lại hiệu quả rõ rệt. Kết quả thực nghiệm cho thấy kích thước mô hình giảm hơn 74%, tốc độ suy luận được cải thiện đáng kể trong khi độ chính xác chỉ giảm rất ít. Nếu có thêm thời gian và tài nguyên huấn luyện, có thể kết hợp KD và Quantization để đạt hiệu quả tối ưu hơn.

---

## 9. Kết luận

Trong bài thực hành này, kỹ thuật **Dynamic INT8 Quantization** đã được áp dụng cho mô hình Vision Transformer (ViT-B/16) nhằm giảm kích thước mô hình và cải thiện hiệu năng suy luận trên CPU. Sau khi lượng tử hóa, kích thước mô hình giảm từ **327.4 MB** xuống **84.4 MB**, tương đương giảm khoảng **74.2%**. Đồng thời, độ trễ suy luận giảm khoảng **27%** và throughput tăng khoảng **36.9%** so với mô hình gốc.

Mặc dù Accuracy và Macro-F1 giảm nhẹ lần lượt khoảng **0.6 điểm phần trăm** và **1.0 điểm phần trăm**, chất lượng mô hình vẫn được duy trì ở mức rất cao. Kết quả này cho thấy quantization mang lại trade-off rất tốt giữa kích thước mô hình, tốc độ suy luận và độ chính xác.

Qua bài thực hành, có thể thấy rằng Dynamic Quantization là một giải pháp hiệu quả để triển khai các mô hình học sâu trên các hệ thống có tài nguyên hạn chế mà vẫn đảm bảo chất lượng dự đoán. Đây là một kỹ thuật hữu ích trong quá trình tối ưu hóa và triển khai mô hình AI vào thực tế.