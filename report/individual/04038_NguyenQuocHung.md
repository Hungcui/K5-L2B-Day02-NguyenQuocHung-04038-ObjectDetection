# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Quốc Hùng
- MSSV: 04038
- Email: hungnq.04038@vinuni.edu.vn
- Lớp: K5-L2B
- Link repo bài làm: https://github.com/Hungcui/K5-L2B-Day02-NguyenQuocHung-04038-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/111oMOabBidZRgH6F-82T-jStlQt9cty8?usp=sharing

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định:
1. `drive_001`: Xe bán tải di chuyển trong làn và các xe ở khu vực xa ngoài luồng giao thông chính. Quyết định: Gán xe bán tải là `car (0)` theo đúng quy chuẩn schema; không gán các xe nhỏ ngoài rìa luồng giám sát (ROI) để tránh false positive.
2. `drive_002`: Xe dạng minivan/van chuyển phát nhanh có kích thước nhỉnh hơn SUV. Quyết định: Gán `van (3)` do dáng xe hình hộp vuông vức, nóc cao, thân liền khối phục vụ vận tải hàng hóa/hành khách tầm trung.
3. `drive_005`: Xe khách loại nhỏ/minibus tại ngã tư. Quyết định: Gán `bus (2)` do chiều dài thân xe lớn và phục vụ vận chuyển hành khách công cộng, phân biệt với van cá nhân/gia đình.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 file YOLO (từ `drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` ĐẠT cấu trúc 50/50 file nhãn hợp lệ, không có lỗi định dạng hay box tràn biên.
- Danh sách 10 ảnh đã rà bằng mắt: `drive_001`, `drive_005`, `drive_010`, `drive_015`, `drive_020`, `drive_025`, `drive_030`, `drive_035`, `drive_040`, `drive_045`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện bị lệch thứ tự class giữa truck/bus/van trong cấu hình tool gán nhãn và một số xe ở rìa bị box hơi lỏng; đã điều chỉnh lại biên box ôm sát thân xe.
- Commit khóa nhãn ban đầu: `786fe1a` (hoặc `b3c6245`).

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 411 | 575 |
| FP | 414 | 30 |
| FN | 242 | 78 |
| Precision | 0.4982 | 0.9504 |
| Recall | 0.6294 | 0.8806 |
| F1 | 0.5562 | 0.9141 |
| mean_iou_of_matches | 0.8549 | 0.8609 |
| passed | false | true |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.
- **Phân tích nguyên nhân trước sửa:** Bản ban đầu đạt mean IoU rất cao (0.8549 chứng tỏ box vẽ rất chuẩn và khít), tuy nhiên F1 chỉ đạt 0.5562 do:
  1. Lệch Class ID giữa truck, bus và van (107 trường hợp lệch class ID do thứ tự class trong tool gán nhãn bị lệch vòng giữa 1: truck, 2: bus, 3: van).
  2. False Positive cao (414) do gán cả các xe nằm ngoài luồng giao thông chính/bãi xe phụ ở xa.
  3. False Negative (242) do một số xe bị che khuất một phần trong làn đường bị bỏ sót.
- **Quy trình rework:**
  1. Chuẩn hóa lại ID class về đúng schema 4 lớp (`0: car, 1: truck, 2: bus, 3: van`).
  2. Lọc bỏ các box vẽ ngoài khu vực luồng đường chính (ROI).
  3. Bổ sung các xe bị che khuất trong luồng di chuyển.
- **Kết quả sau rework:** F1 đạt **0.9141** và Mean IoU đạt **0.8609**, vượt ngưỡng yêu cầu và đạt mức **Xuất sắc** theo rubric.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab Runtime T4 GPU (Tesla T4 15GB), Python 3.11, PyTorch 2.5.1+cu124, Ultralytics 8.3+.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên mặc định (epochs=50, imgsz=640, batch=-1, seed=20260909).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (đạt tốt nhất ở epoch 34).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.5930 (59.30%) |
| Recall | 0.5434 (54.34%) |
| mAP50 | 0.5801 (58.01%) |
| mAP50–95 | 0.3591 (35.91%) |

*Hình ảnh đánh giá trên 10 ảnh eval xem tại thư mục: `assets/eval_predictions/visuals/` và biểu đồ tại `assets/yolo26_runs/vehicle_eval/`.*

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có recall thấp nhất là `truck` và `bus` (Recall ~ 0.40 - 0.45) do số lượng mẫu trong tập 50 ảnh train ít hơn rất nhiều so với `car` (hiện tượng class imbalance nghiêm trọng trong tập dữ liệu camera giao thông). Minh họa: quan sát ảnh `assets/eval_predictions/visuals/drive_055.jpg` nơi xe lớn/ít mẫu khó phát hiện hơn hoặc bị dự đoán với độ tin cậy thấp.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra độ bám sát (tightness) của bounding box. Ở ngưỡng IoU 0.50, model chỉ cần bắt trúng đại khái vị trí vật thể là được tính là True Positive. Tuy nhiên, nếu các cạnh viền box bị lỏng, dư mặt đường, bóng xe hoặc cắt lệch biên giới hạn thì ở các ngưỡng IoU khắt khe hơn (từ 0.55 đến 0.95), IoU không đáp ứng được ngưỡng cao khiến mAP50–95 bị tụt sâu.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa `car` (SUV/CUV cỡ lớn) và `van`, hoặc giữa `car` (pickup bán tải) và `truck`. Dữ kiện hỗ trợ là ma trận nhầm lẫn (confusion matrix trong `assets/yolo26_runs/vehicle_eval/confusion_matrix.png`) thể hiện tỉ lệ dự đoán nhầm giữa các cặp lớp này cao hơn hẳn các cặp khác do đặc trưng ngoại hình có sự giao thoa nếu guideline không phân định rành mạch.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train nhằm tránh hiện tượng rò rỉ dữ liệu (data leakage) và overfitting, bảo toàn tính khách quan khi đo lường khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy.
   - Vai trò validation trong notebook: 10 ảnh eval dùng làm thước đo độc lập sau mỗi epoch để theo dõi validation loss, tính toán mAP và lựa chọn checkpoint tốt nhất (`best.pt`) tránh lưu model bị overfit ở các epoch muộn.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khởi tạo công cụ | Lệch thứ tự ID class trong tool so với schema đề bài | Kiểm tra lại `data.yaml` và xuất lại bảng map class | Đã map chuẩn 4 lớp 0:car, 1:truck, 2:bus, 3:van |
| Gold QC | F1 bản đầu đạt 0.5562 do nhiều FP ngoài luồng đường chính | Phân tích JSON `per_image` và lọc bỏ các box ngoài ROI | F1 tăng lên 0.9141, đạt mức xuất sắc |
| Colab chạy YOLO | Cần GPU và kết nối repo tự động | Cấp quyền và chạy clone tự động từ repo GitHub | Chạy mượt mà 50 epoch, lưu outputs đầy đủ |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- Hoàn thành và định dạng YOLO (10đ): Đạt 50/50 file, đúng chuẩn 5 cột, normalized.
- Đầy đủ vật thể (20đ): Bao phủ tốt các xe trong luồng di chuyển.
- Độ bám sát và hình học box (20đ): Mean IoU đạt 0.8609 (Mức xuất sắc >= 0.80).
- Phân lớp nhất quán (15đ): Đã phân định rõ 4 lớp theo guideline.
- Mini guideline và ca mơ hồ (10đ): Đã hoàn thiện trong `GUIDELINE_MINI.md` với 3 ca chi tiết.
- Gold-set annotation QC (15đ): F1 = 0.9141 (Mức xuất sắc >= 0.90) kèm nhật ký rework chi tiết.
- Pipeline YOLO và phân tích (10đ): Chạy đầy đủ notebook trên Colab, giải thích rõ 4 câu hỏi.
- **Dự kiến điểm: 100/100.**

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng Google Antigravity AI để hỗ trợ kiểm tra phân tích lỗi so sánh với Gold set, tìm nguyên nhân lệch Class ID và gợi ý giải thích các câu hỏi chuyên sâu về IoU và Class Imbalance.
- Tự kiểm tra toàn bộ dữ liệu, đối chiếu trực tiếp các ảnh và tự chạy các script kiểm tra.
- Tham khảo tài liệu: Ultralytics YOLO Docs, UA-DETRAC dataset documentation.
