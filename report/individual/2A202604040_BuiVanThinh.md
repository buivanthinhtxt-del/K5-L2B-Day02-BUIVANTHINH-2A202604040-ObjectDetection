# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Bùi Văn Thịnh
- MSSV: 2A202604040
- Email: buivanthinhtxt@gmail.com
- Lớp: K5-L2B
- Link repo bài làm: https://github.com/buivanthinhtxt-del/K5-L2B-Day02-BUIVANTHINH-2A202604040-ObjectDetection
- Link Colab đã chạy: *(Cập nhật sau khi chạy notebook ở CP6)*

## 2. Schema và mini guideline

Chi tiết định nghĩa 4 lớp và quy tắc vẽ box được trình bày tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

- **Cách áp dụng schema (`0: car`, `1: truck`, `2: bus`, `3: van`):**
  - Phân loại theo **loại phương tiện**, không phụ thuộc trạng thái di chuyển hay đang đỗ.
  - Mỗi phương tiện hợp lệ vẽ 1 bounding box ôm sát phần thân xe nhìn thấy được; chạm đúng mép ảnh nếu xe bị cắt khung (truncated); không gán phương tiện ngoài 4 lớp (như mô tô, xe máy, xe đạp, người đi bộ).
- **Tóm tắt các ca mơ hồ đã quyết định:**
  1. `drive_001.jpg`: Xe SUV màu trắng ở làn phải bị cột đèn kim loại chắn dọc giữa thân -> vẽ 1 box `car` (ID 0) bao trọn phần thân xe nhìn thấy ở hai bên cột đèn; các xe con đỗ tĩnh sát lề trái vẫn gán `car`.
  2. `drive_008.jpg`: Giao lộ có xe tải ben đỏ (`truck` - ID 1), xe buýt khớp nối 2 khoang (`bus` - ID 2, gán 1 box cho toàn xe), 2 xe minivan thân hộp liền khối ở giữa ngã tư (`van` - ID 3), xe con bị che khuất một phần sau xe buýt (`car` - ID 0, vẽ phần nhìn thấy) và 2 người đi xe máy bên trái (**bỏ qua không gán** vì ngoài phạm vi 4 lớp).
  3. `drive_022.jpg`: Xe tải thùng kín (box truck) màu trắng bên phải có kích thước gần tương đương van nhưng có cabin đầu bằng tách biệt với thùng hàng vuông phía sau -> gán `truck` (ID 1).
  4. `drive_033.jpg`: Xe buýt đỏ ở góc dưới trái, xe buýt mép phải và sedan bạc góc dưới phải bị rìa ảnh cắt một phần -> vẽ box chạm sát biên ảnh vì vẫn nhận diện rõ lớp; bỏ qua các chấm xe quá nhỏ/mờ ở tít xa trên cầu vượt.

## 3. Annotation và self-QC

- Số file nhãn train: `50/50` file trong `dataset/labels/train/` (tổng cộng `1,010` bounding boxes: `car`: 837, `bus`: 97, `van`: 58, `truck`: 18; từ 4 đến 50 boxes/ảnh).
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` đạt kết quả `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ. Chờ ground truth để chấm chất lượng.` Không có lỗi định dạng hay box tràn biên.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_005`, `drive_008`, `drive_012`, `drive_018`, `drive_022`, `drive_027`, `drive_033`, `drive_038`, `drive_045`, `drive_050`
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Đã kiểm tra và tinh chỉnh các box bị cắt ở rìa ảnh (`drive_033`, `drive_038`) chạm đúng mép ảnh, bổ sung các xe con bị che một phần phía sau xe buýt (`drive_008`) và xe đỗ sát lề đường (`drive_001`), đồng thời rà soát kỹ phân lớp giữa `van` và `car`/`truck` (`drive_008`, `drive_022`).
- Commit khóa nhãn ban đầu: `49d2606` (`CP4: lock initial 50 train YOLO labels and mini guideline`)

## 4. Gold QC và rework

| Chỉ số | Trước sửa (Bản v1) | Sau sửa (Bản v2) |
|---|---:|---:|
| TP | 463 | 442 |
| FP | 547 | 390 |
| FN | 190 | 211 |
| Precision | 45.84% (0.4584) | **53.12% (0.5312)** |
| Recall | 70.90% (0.7090) | 67.69% (0.6769) |
| F1 | 55.68% (0.5568) | **59.53% (0.5953)** |
| mean_iou_of_matches | 84.43% (0.8443) | **84.76% (0.8476)** |
| passed | False | False (F1 < 0.80) |

File kết quả chi tiết: [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

### Phân tích chi tiết vòng Rework:

1. **Hiệu quả vòng Rework (từ Bản v1 sang Bản v2):**
   - **Giảm mạnh False Positives (FP):** FP giảm từ 547 xuống **390** (giảm 157 box thừa) nhờ việc rà soát và loại bỏ các box ở quá xa hoặc các chấm mờ trên cầu vượt/đường chân trời (ví dụ tại `drive_002`, `drive_003`, `drive_030`).
   - **Tăng Precision và F1:** Precision tăng đáng kể từ `45.84%` lên **`53.12%`**, kéo theo F1 tăng từ `55.68%` lên **`59.53%`**.
   - **Độ khít hình học (mean IoU):** Duy trì ở mức rất cao và tiếp tục tăng nhẹ từ `84.43%` lên **`84.76%`**, đạt mức **Xuất sắc** ($\ge 80\%$) theo RUBRIC. Các bounding box bám rất khít vật thể thực tế.

2. **Ghi nhận về giới hạn trần của F1 do chênh lệch ID lớp với Gold:**
   - Dù đã tinh chỉnh box rất kỹ, chỉ số F1 dừng ở mức `~60%` do rào cản từ việc file `gold_train_labels` của BTC có sự gán nhầm lẫn lớp mang tính quy luật (`bus` $\leftrightarrow$ `van` $\leftrightarrow$ `truck`) so với định nghĩa trong `data.yaml` của chính bài lab.
   - Khi loại trừ yếu tố lệch nhãn này, tỷ lệ phát hiện thực tế (IoU cao) đạt trên `86%`. Học viên ghi nhận đầy đủ hiện tượng vào báo cáo, giữ nguyên tính trung thực của dữ liệu và tiếp tục sử dụng bộ nhãn v2 này để chạy pipeline YOLO trên Google Colab.

### Phân tích chi tiết lỗi và báo cáo hiện tượng Gold-set:

1. **Về độ khít và chất lượng box (mean IoU = 0.8443):**
   - Chỉ số `mean_iou_of_matches = 84.43%` vượt xa ngưỡng yêu cầu tối thiểu (`0.70`) và đạt mức **Xuất sắc** ($\ge 0.80$) theo RUBRIC. Các bounding box học viên vẽ bám rất sát hình học của phương tiện nhìn thấy.

2. **Phát hiện hiện tượng lệch class ID mang tính hệ thống trong Gold-set:**
   - Khi đối chiếu trực quan từng ảnh giữa nhãn học viên và tập `gold_train_labels` (cụ thể tại `drive_008`, `drive_030`), phát hiện các xe trong tập gold bị gán lệch nhãn so với schema quy định tại `data.yaml` và `README.md` (`0: car, 1: truck, 2: bus, 3: van`):
     - Xe buýt khớp nối 2 khoang lớn trong `drive_008` bị gold gán ID `3` (schema là `van`).
     - Xe tải ben chở vật liệu trong `drive_008` bị gold gán ID `2` (schema là `bus`).
     - Xe minivan trắng trong `drive_008` bị gold gán ID `1` (schema là `truck`).
   - Thống kê trên toàn bộ 50 ảnh cho thấy có sự nhầm lẫn chéo có tính quy luật:
     - Học viên gán `bus` $\rightarrow$ Gold ghi `van`: 54 box.
     - Học viên gán `van` $\rightarrow$ Gold ghi `truck`: 32 box.
     - Học viên gán `truck` $\rightarrow$ Gold ghi `bus`: 16 box.
   - Thử nghiệm kiểm chứng trên bản sao tạm (không sửa đổi dữ liệu gốc trong repo hay gold): Nếu ánh xạ lại thứ tự ID lớp này, Recall tăng từ `70.90%` lên **`86.37%`** và F1 tăng lên **`67.83%`**, chứng minh phần lớn lỗi sai lớp là do sự không đồng nhất về ID giữa file gold và schema của đề bài.

3. **Phân tích False Positives (FP = 547 box):**
   - 414 box là "box thừa" do học viên gán đầy đủ các xe con đang đỗ tĩnh bên đường hoặc xe trong dòng phương tiện dày đặc ở hậu cảnh (ví dụ `drive_030`, `drive_002`, `drive_003`), trong khi tập gold bỏ qua không gán các xe ở xa này.
   - 118 box do sai lệch class ID giữa gold và học viên như đã phân tích ở trên.
   - Chỉ có 15 box bị tính là FP do IoU lệch nằm trong khoảng $[0.10, 0.50)$.

4. **Phân tích False Negatives (FN = 190 box):**
   - 154 box FN thực chất là học viên đã bắt đúng xe và vị trí có IoU cao nhưng bị lệch class ID với gold nên thuật toán matching cùng lớp không ghép cặp được.
   - Chỉ có 36 box bị bỏ sót hoàn toàn (chiếm dưới 5% tổng số box của gold).

*Ghi chú rework:* Tuân thủ nghiêm ngặt [RULES.md](../../RULES.md) (không tự ý sửa file gold và không tự ý sửa nhãn học viên sai lệch bản chất để "ép điểm"), học viên giữ nguyên bộ nhãn đã khóa tại commit `49d2606` và gửi phản hồi, ảnh minh chứng cụ thể lên BTC/Coach để xin hướng dẫn chính thức trước khi thực hiện rework.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: *(Cập nhật sau khi chạy Colab/YOLO)*
- Các tham số thay đổi so với notebook nếu có và lý do: Không thay đổi (`epochs=50`, `imgsz=640`, `seed=20260909`, `model='yolo26n.pt'`).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

- Thư mục ảnh dự đoán trên tập eval: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/)
- Thư mục log & biểu đồ đánh giá: [assets/yolo26_runs/vehicle_annotation/](assets/yolo26_runs/vehicle_annotation/) và [assets/yolo26_runs/vehicle_eval/](assets/yolo26_runs/vehicle_eval/)

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - *(Cập nhật cụ thể từ kết quả chạy eval trên 10 ảnh `drive_051.jpg` – `drive_060.jpg`; thông thường các lớp thiểu số như `van`/`truck`/`bus` có số lượng mẫu trong 50 ảnh train ít hơn nhiều so với `car` nên recall thấp hơn).*
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - `mAP50` chỉ yêu cầu IoU $\ge 0.50$ để tính là phát hiện đúng, trong khi `mAP50–95` lấy trung bình ở các ngưỡng khắt khe hơn từ $0.50$ đến $0.95$ (bước nhảy $0.05$). Nếu `mAP50` cao mà `mAP50–95` thấp, nghĩa là mô hình định vị được đúng vùng có xe và đúng lớp nhưng **độ bám sát hình học (tightness) của bounding box chưa cao**: cần kiểm tra xem nhãn ground truth hoặc box dự đoán có bị lỏng (lấy dư nền/bóng đổ), bị cắt hụt thân xe, hay thiếu nhất quán ở các xe bị che khuất/cắt rìa ảnh hay không.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa `car` (SUV/MPV) với `van` (minivan), hoặc giữa `van` chở hàng với `truck` thùng kín nhỏ (box truck). Dữ kiện kiểm chứng nằm ở Confusion Matrix trong [assets/yolo26_runs/vehicle_eval/confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và các trường hợp biên trong [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được đưa 10 ảnh `eval` vào tập train để tránh rò rỉ dữ liệu (**data leakage**): nếu mô hình đã học trực tiếp trên chính các ảnh dùng để chấm điểm thì các chỉ số Precision/Recall/mAP sẽ bị thổi phồng do học thuộc lòng (overfitting) thay vì phản ánh khả năng tổng quát hóa trên ảnh chưa từng thấy.
   - Trong notebook này, 10 ảnh `eval` đóng vai trò là tập **validation** (`val: images/eval` trong `data.yaml`) được đánh giá sau mỗi epoch nhằm theo dõi quá trình hội tụ và chọn ra trọng số tốt nhất (`best.pt`), đồng thời dùng để đánh giá chất lượng cuối cùng của pipeline. Do đó, tập `eval` này cũng không phải là một tập test độc lập hoàn toàn với việc chọn model.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP1 — Phân biệt lớp | Khó phân biệt giữa MPV/SUV cỡ lớn (`car`), minivan (`van`) và xe tải thùng kín nhỏ (`truck`). | Đối chiếu kỹ bảng schema trong `README.md` và ghi quy tắc phân biệt theo cấu trúc cabin/thùng hàng vào `GUIDELINE_MINI.md`. | Đã thống nhất quy tắc nhất quán cho cả 50 ảnh. |
| CP2 — Gán nhãn CVAT | Các xe ở xa trên cầu vượt hoặc bị cột đèn/xe buýt che khuất một phần. | Áp dụng quy tắc chỉ vẽ box cho phần nhìn thấy khi xác định chắc chắn lớp; bỏ qua chấm xe quá mờ ở xa. | Đảm bảo box bám sát và đúng 4 lớp. |
| CP5 — Gold QC | Chỉ số F1 chưa đạt (0.5568) dù mean IoU đạt 84.43%, xuất hiện nhiều FP/FN do lệch class và box ở xa. | Phân tích ma trận nhầm lẫn giữa nhãn học viên và gold; vẽ box đối chiếu trực quan trên `drive_008`, `drive_030`; thử nghiệm mapping trên bản sao tạm để xác thực nguyên nhân. | Phát hiện sự không đồng nhất về ID lớp trong file gold so với `data.yaml` của BTC; đã soạn báo cáo phản hồi kèm minh chứng hình ảnh gửi tới Coach để xin hướng dẫn chính thức. |

## 7. Tự đánh giá theo rubric

Đối chiếu với [RUBRIC.md](../../RUBRIC.md):
- [x] **CP1 — Mini guideline và quyết định mơ hồ (10đ):** Đã hoàn thành đầy đủ định nghĩa 4 lớp, quy tắc vẽ box và 4 ca mơ hồ thực tế trong [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).
- [x] **CP2, CP3 & CP4 — Hoàn thành 50 file YOLO, đầy đủ vật thể, độ bám sát, phân lớp nhất quán và khóa nhãn (65đ):** Đã gán nhãn đủ 50/50 ảnh trên CVAT (1,010 boxes), vượt qua `tools/check_yolo_labels.py` và khóa nhãn tại commit `49d2606`.
- [x] **CP5 — Gold-set annotation QC (15đ):** Đã chạy đối chiếu với `gold_train_labels/`, lưu kết quả tại `outputs/annotation_evaluation.json`; mean IoU đạt 84.43% (mức Xuất sắc); phân tích khoa học nguyên nhân F1 chưa đạt (lệch ID class có hệ thống giữa gold và schema) và gửi báo cáo phản hồi tới BTC theo đúng quy trình học thuật.
- [ ] **CP6 — Pipeline YOLO và phân tích (10đ):** Chạy notebook sau khi nhận phản hồi từ BTC/Coach về Gold set hoặc thực hiện vòng rework tương ứng.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ sử dụng:** CVAT (`localhost:8080`) để gán nhãn thủ công 50 ảnh train; Google Colab + Ultralytics YOLO để huấn luyện/đánh giá model; trợ lý AI (Antigravity) hỗ trợ kiểm tra cấu trúc thư mục, định dạng báo cáo markdown và giải thích khái niệm IoU/mAP.
- **Phần tự thực hiện và kiểm tra:** Tự quan sát ảnh, quyết định các ca mơ hồ trong guideline, thực hiện gán nhãn và rà soát (Self-QC) bằng mắt trên tập ảnh train.
- **Tài liệu tham khảo:** Tài liệu hướng dẫn trong repo bài học (`README.md`, `GUIDE.md`, `RULES.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md`).

