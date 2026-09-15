# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyen Van Thanh Long - 2A202602284`
Clip: `clip_01` (clip chính), `clip_02` (warm-up)

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                           | Không gán                                           |
| ----------------------------- | --------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                         |
| van, minivan                  | xe đạp                                              |
| xe buýt, minibus              | **xe máy / mô tô**                                  |
| xe tải, xe đầu kéo            | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): chỉ giữ xe bốn bánh; không gán người đi bộ, xe máy hoặc xe đạp dù chúng xuất hiện cùng khung hình.

## 2. Luật ID — phần quan trọng nhất

| Tình huống                       | Luật của nhóm                                                                                | Vì sao |
| -------------------------------- | -------------------------------------------------------------------------------------------- | ------ |
| Xe bị che một phần rồi hiện lại  | giữ nguyên ID nếu bị che dưới 25 frame (khoảng 2 giây ở 12.5 fps)                             | giữ identity theo quỹ đạo liên tục |
| Xe bị che lâu hơn ngưỡng trên    | mở track mới khi không thể xác định chắc đó là cùng xe                                        | tránh nối nhầm identity |
| Xe rời khung hình rồi quay lại   | dùng track mới                                                                         | xe đã ra khỏi khung được xem là kết thúc track |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo hướng chuyển động, vị trí trước/sau và hình dáng; không đổi ID chỉ vì giao nhau | giảm ID switch khi crossing |

## 3. Luật bbox

| Tình huống                             | Luật của nhóm                                                                         |
| -------------------------------------- | ------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                  | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                         |
| Xe bị xe khác che một phần             | bbox ôm phần **nhìn thấy được**                                                       |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu từ frame đầu tiên xác định được là xe bốn bánh; nếu chưa chắc thì chờ frame kế tiếp |
| Xe đang đỗ, không di chuyển            | vẫn giữ track trong toàn bộ thời gian xe còn nhìn thấy trong khung                    |
| Keyframe đặt dày ở đâu                 | đặt dày khi xe rẽ, phanh, đổi scale, bị che, gần biên ảnh hoặc khi interpolation lệch; frame thẳng đều có thể đặt thưa |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01 / frame 79 / ID 5`
- Tình huống: bbox của xe nhỏ cạnh ID 4 có IoU chỉ khoảng 0.50 với reference, là điểm lệch interpolation rõ nhất trong evaluator.
- Quyết định: cần thêm keyframe quanh frame 79 sau khi kiểm tra lại ảnh; trạng thái pre-gold: `needs-review`, chưa rework.
- Lý do: xe vẫn nhìn thấy nhưng bbox phải ôm phần nhìn thấy, không để interpolation trôi theo xe lớn bên cạnh.

### Ca 2

- Clip / frame / ID: `clip_01 / frames 78-100 / ID 6`
- Tình huống: ID 6 đã có bbox ở vùng phải trước khoảng xuất hiện của track reference tương ứng.
- Quyết định: kiểm tra frame đầu tiên thực sự nhận ra xe; nếu xe chưa xác định được thì bấm outside cho đến frame bắt đầu đúng. Trạng thái: `needs-review`.
- Lý do: luật entry yêu cầu bắt đầu ở frame đầu tiên xác định được xe bốn bánh, không tạo bbox sớm.

### Ca 3

- Clip / frame / ID: `clip_01 / frames 149-151 / ID 4`
- Tình huống: ID 4 chạm và đi ra khỏi biên trái; evaluator phát hiện bbox vẫn còn trong 3 frame cuối của đoạn tham chiếu.
- Quyết định: đặt outside đúng frame xe rời khung, không đoán phần xe nằm ngoài ảnh; trạng thái: `needs-review`, chưa rework.
- Lý do: bbox ở biên chỉ ôm phần còn nhìn thấy và track phải kết thúc khi xe ra khỏi khung.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Phải ghi rõ frame bắt đầu/kết thúc của track khi xe nhỏ, bị che hoặc đi sát biên; không kéo bbox trước khi xe xác định được và phải dùng outside khi xe rời khung.
- Khi evaluator báo IoU thấp ở một frame giữa hai keyframe, phải xem ảnh gốc và thêm keyframe quanh điểm lệch; không sửa ID hoặc bbox chỉ dựa trên metric.
