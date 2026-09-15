# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Solo / Nguyễn Đình Đại - 2A202602107`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `...` |
| Thời gian gán `clip_02` (warm-up) | 15 phút |
| Thời gian gán `clip_01` | 26 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 21 |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

### Tình huống 1
- Clip / frame / ID: clip02 - frame: 1 ID: 1
- Tình huống: xe bị cắt bởi rìa ảnh, chỉ còn ~1/3 thân xe nhìn thấy, xe đang di chuyển ra ngoài frame
- Quyết định: tiếp tục track; vẽ bbox ôm đúng phần 1/3 nhìn thấy, cạnh bbox chạm rìa ảnh — không đoán phần ngoài; dừng track ở frame cuối cùng còn nhìn thấy bất kỳ phần nào của xe
- Lý do: còn nhìn thấy xe → vẫn đủ điều kiện annotate; dừng sớm làm trajectory bị cắt ngắn, sai metric

### Tình huống 2
- Clip / frame / ID: clip 02 / frame 14 / ID 4
- Tình huống: xe mới bắt đầu vào frame, bị truncated, chỉ nhìn thấy đèn xe + 1 bánh trước (đúng 2 đặc điểm)
- Quyết định: chờ thêm 2-3 frame để xác nhận xe vẫn còn trong frame; 
- Lý do: chưa đủ đặc điểm để xác định xe vẫn còn trong frame, và chưa đủ đặc điểm nhận dạng để xác định object là vehicle

### Tình huống 3
- Clip / frame / ID: clip01 / frame77 / ID 5
- Tình huống: xe bị che hoàn toàn từ đầu clip, chưa từng được annotate; frame 77 là lần đầu tiên nhìn thấy và nhận dạng được là xe bốn bánh
- Quyết định: gán ID mới, bắt đầu track từ frame 77
- Lý do: xe chưa có ID nào trước đó → không có track cũ để giữ; đây tương đương "xe vừa xuất hiện lần đầu", áp dụng luật bbox mục 3 — không áp dụng luật occlusion 25 frame vì không có track cũ để so sánh

## 2. Tự kiểm và kiểm chéo
 SOLO
Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1:ID không bị stwich, không bị nhấp nháy hay biến mất
- Lượt 2: có những frame xe lần đầu xuất hiện nhưng tại frame đó chưa thực sự xác định được nó là vehicle
- Lượt 3: có những frame các xa key frame thì bbox chưa sát với xe


## 3. Chấm với gold — trước và sau rework

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Lần chấm đầu | 0.821 | 0.8038 | 0.8403 | 0.8841 | 0.9685 | 0.9354 | 0.8698 | 33 | 4 | 0 |
| Sau rework | 0.821 | 0.8038 | 0.8403 | 0.8841 | 0.9685 | 0.9354 | 0.8698 | 33 | 4 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): có

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| tracking quá sớm(khi chưa thể xác định object là vehicle) | 51-53 | 4 | đợi ID qua 2 frame sau rồi mới bắt đầu track |
| BBOX TREO — bbox có trước khi track tham chiếu xuất hiện | 85-100 | 6 | bấm outside đúng frame xe rời khung (16 frame thừa) |
| BBOX THỪA — bbox còn sau khi track tham chiếu đã rời khung | 149-151 | 4 | bấm outside đúng frame xe rời khung (3 frame thừa) |
| BBOX TREO — bbox có trước khi track tham chiếu xuất hiện | 51-53 | 4 | bấm outside đúng frame xe rời khung (3 frame thừa) |
| BBOX THỪA — bbox còn sau khi track tham chiếu đã rời khung | 169-171 | 8 | bấm outside đúng frame xe rời khung (3 frame thừa) |
| BBOX TRÔI — IoU thấp (0.54), bbox lệch khỏi vật thể | 55 | 4 | thêm keyframe quanh frame 55 để kéo bbox về đúng vị trí xe |
| BBOX TRÔI — IoU thấp (0.56), bbox lệch khỏi vật thể | 96 | 5 | thêm keyframe quanh frame 96 để kéo bbox về đúng vị trí xe |
| BBOX TRÔI — IoU thấp (0.57), bbox lệch khỏi vật thể | 103 | 6 | thêm keyframe quanh frame 103 để kéo bbox về đúng vị trí xe |
| BBOX TRÔI — IoU thấp (0.59), bbox lệch khỏi vật thể | 104 | 6 | thêm keyframe quanh frame 104 để kéo bbox về đúng vị trí xe |
| BBOX TRÔI — IoU thấp (0.60), bbox lệch khỏi vật thể | 102 | 6 | thêm keyframe quanh frame 102 để kéo bbox về đúng vị trí xe |


## 4. Kết quả model và so sánh ba chiều

Cấu hình: model `yolo26n`, tracker `bytetrack`, conf `0.25`, imgsz `960`

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.821 | 0.804 | 0.840 | 0.884 | 0.969 | 0.935 | 0.870 | 33 | 4 | 0 |
| model vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| model vs bạn | 0.717 | 0.649 | 0.796 | 0.878 | 0.845 | 0.698 | 0.862 | 92 | 87 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi  thấp hơn IDF1 . MOTA cao mà IDF1 thấp cho thấy model phát hiện và bám vị trí vật thể tốt, nhưng giữ ID qua các frame kém. MOTA không phạt nặng lỗi ID vì mỗi lần ID stwich chỉ được cộng như 1 lỗi và ID switch chỉ là một thành phần cộng trong tổng lỗi
**2. DetA và AssA của model lệch nhau bao nhiêu? Cái nào kéo HOTA xuống — model không tìm ra xe, hay tìm ra rồi nhưng đánh mất ID?**

Model vs gold: DetA=0.649, AssA=0.776 — chênh nhau 0.127. DetA thấp hơn AssA  model không tìm ra xe là nguyên nhân  kéo HOTA xuống. FN=54 tức là  model bỏ sót 54 detection nghĩa là với những frame xe đã có mặt, model không phát hiện ra. AssA=0.776 vẫn tốt, cho thấy những xe nào model phát hiện được thì ID được giữ khá tốt.

**3. Một chỗ bạn đúng và model sai (frame, ID, vì sao):**

Frame 77–100, ID 5: xe này bị che hoàn toàn trong phần đầu clip, bắt đầu xuất hiện từ frame 77. Tôi nhận ra và bắt đầu track từ frame 77 đúng theo luật. Model bỏ sót hoàn toàn hoặc phát hiện trễ hơn — có thể do xe còn nhỏ/bị khuất một phần khi bắt đầu vào frame, confidence không đủ ngưỡng.

**4. Một chỗ model đúng và bạn sai (frame, ID, vì sao):**

Frame 111: model phát hiện 3 bbox trong khi tôi chỉ có 2 — model bắt được một xe mà tôi bỏ sót. Khi nhìn lại frame này, có một xe đang di chuyển ở vùng rìa ảnh mà tôi chưa tạo track vì chưa chắc chắn đó là vehicle. Model (yolo26n + bytetrack) phát hiện nó đúng vì detector chạy frame-by-frame độc lập và confidence vẫn đủ ngưỡng 0.25 ở frame đó. Đây là điểm model đúng — tôi sai do áp dụng ngưỡng "chờ xác nhận" quá thận trọng, dẫn đến FN tăng. Về phía ghost bbox: diagnostics ghost_pred_tracks chỉ ra tôi vẽ bbox trước khi xe tham chiếu xuất hiện (track 4 frame 51–53, track 6 frame 85–100) và sau khi xe đã rời khung (track 4 frame 149–151, track 8 frame 169–171) — những lỗi này model không mắc vì YOLO dừng detection ngay khi không còn xe trong ảnh.

**5. Trong ba loại bất đồng giữa bạn và model, loại nào nhiều nhất? Nó nói gì về clip này?**

Loại nhiều nhất: **tôi có bbox, model không** (model vs bạn FN=87 — tức là 87 frame-bbox model bỏ sót trong khi tôi đã gán). Ngược chiều thì model có bbox mà tôi không có ít hơn nhiều (FP=92 trong đó phần lớn là track ID10 tồn tại 42 frame mà không khớp xe nào trong gold — nhiều khả năng là vật thể tĩnh bị model nhầm thành xe). ID switch chỉ có 3, không đáng kể. Điều này phản ánh đặc điểm của clip_01: nhiều xe bị che khuất hoặc mất dấu giữa chừng — track gold 8 model chỉ phủ được 61% quãng đời, track gold 6 phủ 75%, track gold 5 phủ 77%. YOLO mất confidence khi xe bị che và không theo dõi được liên tục như người gán nhãn tay dùng ngữ cảnh chuyển động. Đặc biệt model còn tách một xe thành nhiều ID (track gold 4, 5, 7, 8 đều bị split) — tracker mất dấu rồi tạo ID mới thay vì nối lại track cũ.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

**Sửa trong `GUIDELINE_MINI.md`:**
- Bổ sung quy tắc rõ hơn về frame bắt đầu/kết thúc track: thay vì "chờ xác nhận 2-3 frame" phải ghi cụ thể tiêu chí — "bắt đầu track ngay từ frame đầu tiên nhìn thấy ít nhất 2 đặc điểm nhận dạng xe (đèn, bánh, kính), dù xe còn bị truncated". Tình huống frame 51-53 ID 4 và frame 85-100 ID 6 đều xảy ra do áp dụng tiêu chí này không nhất quán.
- Thêm mục cảnh báo về bbox trôi: "Sau mỗi keyframe cách nhau >15 frame, bắt buộc kiểm tra IoU tại frame giữa. Nếu IoU < 0.65 thì thêm keyframe". Lỗi bbox trôi ở ID 6 (frame 102-104) xuất phát từ 3 frame liên tiếp trong một đoạn thiếu keyframe.
- Thêm checklist trước khi submit: (1) tua toàn bộ clip và kiểm tra outside đúng frame; (2) chọn từng track và kiểm tra frame đầu/cuối bằng cách nhảy đến frame đó xem có xe không.

Lượt kiểm cuối sẽ tập trung riêng vào frame đầu và frame cuối của từng track — đây là nguồn gốc của hầu hết ghost bbox. Thay vì xem toàn clip liên tục, sẽ dùng bộ lọc track trong CVAT để nhảy thẳng đến frame đầu/cuối của từng ID.
Với đoạn xe tăng tốc hoặc rẽ cua (thường là nguyên nhân IoU tụt), sẽ đặt keyframe dày hơn (mỗi 5-8 frame thay vì 10-15 frame) trong vùng chuyển động phức tạp.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `GUIDELINE_MINI.md` đã điền
- [x    ] `outputs/eval_vs_gold.json`
- [x] `outputs/model_clip_01.txt`
- [x] `outputs/eval_model_vs_gold.json`, `outputs/eval_model_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
