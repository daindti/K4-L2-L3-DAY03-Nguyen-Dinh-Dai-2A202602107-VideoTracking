# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: SOLO/Nguyễn Đình Đại
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | vì nếu trên 25 frame thì thời gian đã quá lâu để có thể chắc chắn là cùng 1 object |
| Xe bị che lâu hơn ngưỡng trên | gán **ID mới** (track mới) | thời gian che quá dài → tracker mất khả năng dự đoán vị trí, không thể chắc chắn đây là cùng 1 xe; gán nhầm còn sai hơn gán mới |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | khi object rời khỏi khung hình, tracker không còn thông tin về object đó → không thể chắc chắn nó là cùng 1 object khi quay lại |
| Hai xe cắt nhau / chồng lên nhau | mỗi xe giữ nguyên ID, nếu chồng một phần thì vẽ bbox riêng cho phần nhìn thấy của từng xe; nếu xe bị khuất hoàn toàn thì áp dụng luật 25 frame như occlusion bình thường | hoán đổi ID làm sai toàn bộ trajectory; xe vẫn là cùng 1 vật thể dù bị che bởi xe khác |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: bbox ≥ 20×20 px và nhìn rõ ít nhất 2 đặc điểm (thân/bánh/kính/đèn) và xuất hiện ≥ 2 frame liên tiếp |
| Xe đang đỗ, không di chuyển | vẫn phải có bbox trên mọi frame nhìn thấy |
| Keyframe đặt dày ở đâu | keyframe cần đặt dày ở chỗ mà object di chuyển nhanh, thay đổi hướng đột ngột |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: clip02 - frame: 1 ID: 1
- Tình huống: xe bị cắt bởi rìa ảnh, chỉ còn ~1/3 thân xe nhìn thấy, xe đang di chuyển ra ngoài frame
- Quyết định: tiếp tục track; vẽ bbox ôm đúng phần 1/3 nhìn thấy, cạnh bbox chạm rìa ảnh — không đoán phần ngoài; dừng track ở frame cuối cùng còn nhìn thấy bất kỳ phần nào của xe
- Lý do: còn nhìn thấy xe → vẫn đủ điều kiện annotate; dừng sớm làm trajectory bị cắt ngắn, sai metric

### Ca 2
- Clip / frame / ID: clip 02 / frame 14 / ID 4
- Tình huống: xe mới bắt đầu vào frame, bị truncated, chỉ nhìn thấy đèn xe + 1 bánh trước (đúng 2 đặc điểm)
- Quyết định: chờ thêm 2-3 frame để xác nhận xe vẫn còn trong frame; 
- Lý do: chưa đủ đặc điểm để xác định xe vẫn còn trong frame, và chưa đủ đặc điểm nhận dạng để xác định object là vehicle

### Ca 3
- Clip / frame / ID: clip01 / frame77 / ID 5
- Tình huống: xe bị che hoàn toàn từ đầu clip, chưa từng được annotate; frame 77 là lần đầu tiên nhìn thấy và nhận dạng được là xe bốn bánh
- Quyết định: gán ID mới, bắt đầu track từ frame 77
- Lý do: xe chưa có ID nào trước đó → không có track cũ để giữ; đây tương đương "xe vừa xuất hiện lần đầu", áp dụng luật bbox mục 3 — không áp dụng luật occlusion 25 frame vì không có track cũ để so sánh

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Mục 3 — luật bắt đầu track còn mơ hồ: mục 3 ghi "xuất hiện ≥ 2 frame liên tiếp" nhưng không nói rõ phải bắt đầu track từ frame nào trong 2 frame đó. Thực tế tôi bắt đầu track từ frame thứ 2 hoặc thứ 3 thay vì frame đầu tiên → sinh ghost bbox ở frame 51–53 (ID 4) và frame 85–100 (ID 6). Sửa lại: "bắt đầu track ngay tại frame đầu tiên nhìn thấy đủ điều kiện, không chờ thêm frame để xác nhận."

- Mục 3 — thiếu luật kết thúc track: không có quy tắc rõ về thời điểm bấm `outside`. Thực tế tôi để bbox tồn tại thêm 3 frame sau khi xe đã rời khung — lỗi ở frame 149–151 (ID 4) và frame 169–171 (ID 8). Thêm luật: "bấm `outside` ngay tại frame cuối cùng còn thấy bất kỳ pixel nào của xe; không để bbox tồn tại ở frame tiếp theo khi xe đã ra ngoài."

- Mục 3 — thiếu luật mật độ keyframe: mục 3 chỉ ghi "đặt dày ở chỗ di chuyển nhanh" nhưng không có ngưỡng cụ thể. Thực tế bbox trôi ở frame 102–104 (ID 6) và frame 55 (ID 4), frame 96 (ID 5) đều nằm giữa hai keyframe cách xa nhau. Thêm luật: "nếu hai keyframe liên tiếp cách nhau hơn 15 frame, kiểm tra IoU tại frame giữa; nếu IoU < 0.65 thì thêm keyframe tại đó."
