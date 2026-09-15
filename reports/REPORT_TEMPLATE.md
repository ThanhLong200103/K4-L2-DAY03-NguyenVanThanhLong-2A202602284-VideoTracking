# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyen Van Thanh Long - 2A202602284`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục                               | Giá trị                                                               |
| --------------------------------- | --------------------------------------------------------------------- |
| Công cụ                           | CVAT / khác: `CVAT (theo quy trình lab)`                              |
| Thời gian gán `clip_02` (warm-up) | `20p` phút                                                            |
| Thời gian gán `clip_01`           | `40p` phút                                                            |
| Số track đã vẽ trong `clip_01`    | `8`                                                                   |
| Số keyframe trung bình mỗi track  | `Chưa có dữ liệu keyframe; 78.75 bbox/track theo 630 bbox và 8 track` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che hoặc xuất hiện lại: cần giữ ID nhất quán qua đoạn occlusion; xử lý bằng cách tua frame trước/sau và đối chiếu chuyển động.
2. Các xe chồng lấn/cắt nhau: giữ ID theo quỹ đạo liên tục, không đổi ID chỉ vì bbox gần nhau trong một frame.
3. Bbox ở đầu/cuối vùng xuất hiện và vùng biên: kiểm tra frame kế cận để tránh tạo bbox sớm, muộn hoặc lệch vị trí.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                             | Giá trị                                                            |
| ---------------------------------------------------- | ------------------------------------------------------------------ |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `23caeeec93afc28497d9114387e3cb18db203016d2b7788f25909de9bfcd4705` |
| Thời điểm khóa                                       | `2026-09-15T07:11:47.027878+00:00`                                 |
| Số row / frame / track trước khi mở reference        | `630 row / 190 frame / 8 track`                                    |

|                                |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ------------------------------ | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| Bản pre-gold                   | 0.743 | 0.730 | 0.758 | 0.833 | 0.953 | 0.901 | 0.810 |  57 |   0 |    0 |
| Sau rework / bản nhãn hiện tại | 0.743 | 0.730 | 0.758 | 0.833 | 0.953 | 0.901 | 0.810 |  57 |   0 |    0 |

> Lưu ý: hàng “Sau rework / bản nhãn hiện tại” lấy trực tiếp từ `outputs/eval_vs_gold.json` (630 row, 190 frame, 8 track). Đây là kết quả chấm hiện tại với gold, không phải phép so sánh trước/sau rework vì chưa có snapshot pre-gold.

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** theo `outputs/eval_pre_gold.json`

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi              | Frame   | ID  | Đã sửa thế nào                                                                |
| --------------------- | ------- | --- | ----------------------------------------------------------------------------- |
| Bbox treo / entry sớm | 78-100  | 6   | Kiểm tra lại frame bắt đầu và outside trong CVAT; chưa rework                 |
| Bbox treo / entry sớm | 61-78   | 5   | Kiểm tra lại frame bắt đầu và outside trong CVAT; chưa rework                 |
| Bbox treo / exit muộn | 149-151 | 4   | Kiểm tra lại outside ở frame xe rời khung; chưa rework                        |
| Bbox trôi             | 79      | 5   | Thêm keyframe quanh frame 79 nếu kiểm tra hình ảnh xác nhận lệch; chưa rework |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                                | Giá trị                                          |
| ---------------------------------- | ------------------------------------------------ |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13`        |
| weights / hai tracker              | `yolo26n.pt / bytetrack.yaml; botsort-reid.yaml` |
| conf / IoU / imgsz / classes       | `0.25 / 0.70 / 960 / [2, 5, 7]`                  |
| device                             | `cpu`                                            |

| So sánh                   |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| bạn vs gold               | 0.743 | 0.730 | 0.758 | 0.833 | 0.953 | 0.901 | 0.810 |  57 |   0 |    0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 |  88 |  54 |    2 |
| BoT-SORT + ReID vs gold   | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.859 |  91 |  26 |    2 |
| ReID vs bạn               | 0.683 | 0.623 | 0.752 | 0.839 | 0.863 | 0.725 | 0.817 |  90 |  82 |    1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của bạn cao hơn IDF1 (0.901 so với 0.953 là thấp hơn, vì vậy phát biểu đúng là MOTA thấp hơn IDF1). IDF1 phản ánh độ nhất quán identity trên các match; MOTA gộp FN, FP và IDSW theo số detection. Trong kết quả này annotation có 57 FP nhưng không có FN/IDSW, nên IDF1 vẫn cao và MOTA bị giảm bởi FP. MOTA không phạt nặng lỗi ID vì IDSW chỉ là một thành phần cộng thêm, còn nhiều lỗi association không nhất thiết được tính thành IDSW.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với ByteTrack, ReID tăng HOTA 0.709 -> 0.764, DetA 0.649 -> 0.711, AssA 0.776 -> 0.820 và IDF1 0.875 -> 0.900; MOTA tăng 0.749 -> 0.792. IDSW giữ nguyên ở 2. Một sequence đáng chú ý là gt ID 5: ByteTrack bị fragment tại frame 94 (23 -> 32), còn ReID bị fragment tại frame 87 (17 -> 18); vì vậy treatment tốt hơn ở một đoạn nhưng không loại bỏ hoàn toàn lỗi identity. Đây là system comparison, không cô lập causal effect của ReID vì hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

ReID có DetA 0.711, cao hơn ByteTrack 0.649; FN giảm 54 -> 26 nhưng FP tăng nhẹ 88 -> 91. Vì FN giảm rõ và vị trí tốt hơn (MOTP 0.823 -> 0.859), phần cải thiện chính liên quan đến detector/khả năng duy trì detection; phần lỗi IDSW và fragmentation còn lại là association. ReID chưa hoàn hảo vì vẫn có 2 IDSW và ghost tracks.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 113, gt ID 6: annotation giữ một track liên tục theo ID 6, trong khi ReID chuyển từ track 24 sang 31 và có bbox IoU 0.566. Frame 114-115 tiếp tục cùng vùng xe với track 31. So với gold, đây là lỗi association của ReID; không nên sửa annotation chỉ vì model khác ID.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame 104, ID 6 là điểm cần xem lại annotation: ReID có bbox track 24 và gold cũng ghi nhận vùng xe này, trong khi annotation thủ công không bao phủ đầy đủ đoạn đó. Evidence này cho thấy nên kiểm tra lại occlusion/visibility ở frame 104 trước khi kết luận model sai. Tuy nhiên chưa có ảnh review hoặc pre-gold log để xác định có sửa annotation hay không.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Ghi quy tắc cụ thể cho ngưỡng occlusion, xe ở biên frame, xe đứng yên và cách xử lý bbox rất nhỏ. Tạo checklist tua ba lượt trước khi khóa nhãn, lưu manifest SHA-256 trước khi xem gold, và bắt buộc lưu review partner cùng log sửa theo frame/ID. Cuối cùng chạy evaluator và lưu cả bốn JSON/model output trước khi viết kết luận.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
