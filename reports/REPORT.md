# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Trọng Minh Đức`
Ngày: `15-09-2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT, Rectangle → Track, export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | `10` phút |
| Thời gian gán `clip_01` | `34` phút |
| Số track đã vẽ trong `clip_01` | 8 (ID 1–8), 570 bbox trên 190 frame |
| Số keyframe trung bình mỗi track | `13` |

Ba tình huống khó nhất khi gán clip này, và tôi xử lý thế nào:

1. **Xe đỗ đứng yên (clip_01, MOT frame 1–15, ID 3).** Bbox gần như bất động suốt
   15 frame nên validator cảnh báo "quên bấm Outside?". Tôi giữ nguyên bbox: Outside
   chỉ dùng khi xe **vắng mặt**, còn xe đỗ vẫn nhìn thấy được nên theo rule "mọi xe
   bốn bánh nhìn thấy được đều phải có bbox" thì bắt buộc phải có box.
2. **Xe rời khung chỉ trong 2 frame (clip_02, MOT frame 1–2, ID 3).** Chỉ thấy phần
   đuôi xe ở rìa trái, bbox chạm `bb_left = 0` và width co từ 18.6 → 11.5 px. Tôi
   giữ track ngắn này thay vì xóa, rồi bật Outside tại frame 3 — áp rule "bbox chạm
   rìa ảnh, không đoán phần ngoài ảnh" + "Outside tại frame đầu object vắng mặt".
3. **Không biết lúc nào được phép mở track cho xe mới ló ra (clip_01, ID 5).** Xe bị
   xe buýt che gần hết, chỉ lộ vài pixel. Tôi mở track từ frame 75; chấm với gold cho
   thấy quyết định này sớm 5 frame và sinh ra FP — xem mục 3.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- **Lượt 1 (identity/timeline):** tua toàn bộ 190 ảnh `outputs/vis_clip_01/` chỉ nhìn
  số ID. Không thấy ID switch, fragmentation hay reuse ID. Kết quả chấm sau đó xác
  nhận: IDSW = 0 ở cả bản pre-gold lẫn bản cuối.
- **Lượt 2 (entry/exit):** đi frame đầu/cuối của cả 8 track, không có box treo sau khi
  xe rời khung. Ba cảnh báo của `check_mot_labels.py` (ID 3 clip_01; ID 1, 2, 3
  clip_02) đều được kiểm bằng mắt và kết luận `not-a-defect`.
- **Lượt 3 (geometry/interpolation):** soi midpoint giữa các keyframe xa nhau, mắt
  thường không thấy lệch. **Nhưng chấm định lượng lại thấy**: còn drift ở MOT frame
  80–83 (ID 5) và 107–109 (ID 6) với IoU 0.546–0.574. Đây là bài học lớn nhất của
  lượt QC: lệch ở mức IoU ~0.55 vẫn "trông có vẻ đúng" trên màn hình.

Kiểm chéo với: **Nguyễn Đăng Vĩ Anh** (Pair ID 01). Chi tiết ở `reports/review_partner.md`.
Số lỗi tôi tìm được trong bản của bạn ấy: **0 finding**. Số lỗi bạn ấy tìm được trong bản
của tôi: **0 finding** (3 mục nghi ngờ đều được hai bên thống nhất đóng `not-a-defect`).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Hai người không quyết khác nhau ca nào. Luật còn thiếu là **ngưỡng "đủ rõ để mở một
track"**: `GUIDELINE_MINI.md` mục 3 chỉ ghi "bắt đầu track từ frame đầu tiên xác định
được là xe bốn bánh" mà không có tiêu chí định lượng (bao nhiêu pixel? bao nhiêu phần
trăm thân xe?). Chính khoảng trống này tạo ra lỗi entry sớm 5 frame ở ID 5.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `1da5e1ab30b0191342eed4e1e9678ac62e2c85ca14afaf6c185cfd651fa3a2c3` |
| Thời điểm khóa | `2026-09-15T04:28:22Z` (11:28:22 giờ Việt Nam) |
| Số row / frame / track trước khi mở reference | 575 row / 190 frame / 8 track (ID 1–8) |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.8336 | 0.8187 | 0.8515 | 0.8746 | 0.9808 | 0.9616 | 0.8606 | 12 | 10 | 0 |
| Sau rework | 0.8379 | 0.8239 | 0.8550 | 0.8749 | 0.9816 | 0.9634 | 0.8618 | 9 | 12 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** — cả hai bản đều qua.

Sau khi đọc danh sách lỗi, tôi đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa — mở track sớm hơn thời điểm xe thật sự nhận diện được | MOT 75–78 (CVAT 74–77) | 5 | Lùi điểm bắt đầu track từ frame 75 sang frame 80. Kết quả: FP giảm 12 → 9 |
| Bbox mất — lỗi tự gây ra trong lúc sửa track 5 | MOT 75 (CVAT 74) | 4 | Xe buýt bị mất box đúng 1 frame giữa đoạn đang chạy liên tục (74 → 76). `check_mot_labels.py` bắt được ngay ở lần chạy lại; đã vẽ lại bbox frame 75 |
| Bbox trôi giữa hai keyframe | MOT 80–83 và 107–109 | 5 và 6 | **Chưa sửa** — IoU 0.546–0.574, vẫn trên ngưỡng khớp 0.5 và cả ba cổng đã đạt; tôi chọn ưu tiên thời gian cho phần model. Xem mục 5 câu 5 |

Ghi chú: FN tăng nhẹ 10 → 12 vì bốn frame bbox thừa bị gỡ bỏ, trong đó gold có tính
xe ở hai frame cuối. Đổi lại FP giảm 3 và cả HOTA, DetA, AssA, IDF1, MOTA, MOTP đều
tăng, nên rework là cải thiện thực chất chứ không phải đánh đổi ngang bằng.

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.14.0 / 8.4.145 / 2.14.0+cpu / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` (car, bus, truck) |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8379 | 0.8239 | 0.8550 | 0.8749 | 0.9816 | 0.9634 | 0.8618 | 9 | 12 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7890 | 0.7371 | 0.8449 | 0.8867 | 0.9139 | 0.8211 | 0.8744 | 84 | 16 | 2 |

Cả hai model sinh 16 track cho 8 track tham chiếu — gấp đôi số thực thể có thật.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Bản của tôi có MOTA 0.9634 **thấp hơn** IDF1 0.9816. Lý do là hai chỉ số đếm khác
nhau: MOTA = 1 − (FP + FN + IDSW)/GT = 1 − (9 + 12 + 0)/573, tức mọi bbox lệch đều bị
trừ; còn IDF1 chỉ quan tâm gán đúng danh tính, và vì IDSW = 0 nên nó gần như không bị
trừ gì. Khoảng cách nhỏ này đến từ lỗi **hình học**, không phải lỗi identity.

MOTA không phạt nặng lỗi ID vì nó tính mỗi ID switch là **một sự kiện tại đúng một
frame**. Một cú switch ở frame 20 của track dài 100 frame chỉ tốn 1/573 điểm MOTA,
trong khi IDF1 coi cả 80 frame sau đó là gán sai danh tính. Vì vậy MOTA cao mà IDF1
thấp nghĩa là: bbox vẽ đúng chỗ, phát hiện đủ xe, nhưng **nhãn ID bị hoán đổi** — đúng
kiểu lỗi mà bài hôm nay quan tâm nhất. Đó là lý do phải đọc IDF1 và IDSW trước MOTA.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

| | IDF1 | AssA | IDSW |
| --- | ---: | ---: | ---: |
| ByteTrack control | 0.8746 | 0.7761 | 2 |
| BoT-SORT + ReID | 0.9001 | 0.8204 | **2** |

Treatment **tốt hơn** ở IDF1 (+0.0255) và AssA (+0.0443), nhưng **số ID switch không
đổi: 2 vs 2**. Đây là điểm quan trọng — nếu chỉ nhìn IDF1 sẽ tưởng ReID đã sửa được
lỗi identity, thực tế nó chỉ đổi chỗ lỗi:

- ByteTrack switch tại **frame 59 (track tham chiếu 4)** và **frame 94 (track 5)**.
- ReID switch tại **frame 87 (track 5)** và **frame 113 (track 6)**.

Frame sequence cho thấy treatment tốt hơn: **track tham chiếu 4** (xe buýt, dài 95
frame). ByteTrack cắt nó thành hai pred track (15 giữ 90 frame, 14 giữ 2 frame) và
switch tại frame 59. ReID giữ track 4 liền mạch — không nằm trong danh sách
`fragmented_gt_tracks` lẫn `id_switches`. Xe buýt to, hoạ tiết vàng-xanh rất đặc
trưng, nên đặc trưng ngoại hình giúp nối lại đúng khi IoU tụt.

Frame sequence cho thấy treatment **tệ hơn**: **track 6 tại frame 113**. ByteTrack
không switch ở đây, ReID thì có (pred 24 → 31). Trước đó box đã xấu dần: IoU 0.522 ở
frame 104, rồi 0.566 / 0.575 / 0.600 ở frame 113–115 — xe đang vào cua nên cả dự đoán
chuyển động lẫn ngoại hình (góc nhìn xe đổi) đều kém tin cậy.

**Cảnh báo về nhân quả:** không được kết luận "ReID gây ra toàn bộ chênh lệch này".
ByteTrack và BoT-SORT khác nhau ở nhiều thứ ngoài ReID (mô hình Kalman, bù chuyển động
camera, luật khớp hai giai đoạn). Bằng chứng rõ nhất nằm ngay trong bảng: **LocA tăng
0.8463 → 0.8721**. LocA chỉ đo độ khít của bbox, hoàn toàn không liên quan đến việc
khớp ngoại hình — nếu chỉ có ReID khác nhau thì LocA phải gần như đứng yên. Vậy đây là
so sánh **hai hệ thống**, không phải thí nghiệm cô lập một biến.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng 0.6487 → 0.7110. Phân tách ra thì gần như toàn bộ mức tăng đến từ **recall**:
FN giảm mạnh 54 → 26 (−28 bbox), trong khi FP gần như giữ nguyên, thậm chí tăng nhẹ
88 → 91 (+3).

Lỗi còn lại chủ yếu là **detector**, không phải association. Ba căn cứ:

1. AssA (0.8204) đã **cao hơn** DetA (0.7110) — phần nối ID đang làm tốt hơn phần tìm xe.
2. FP = 91 giờ là nguồn lỗi lớn nhất, gấp 3.5 lần FN.
3. `ghost_pred_tracks` cho thấy các track model tự bịa ra mà không khớp track tham
   chiếu nào: pred 7 kéo dài **frame 16–116 (43 frame)**, pred 27 frame 106–121, pred
   38 frame 158–178. Đáng chú ý là ByteTrack cũng có đúng ghost tương tự ở cùng vị trí
   (pred 10, frame 17–116; pred 41, frame 106–121). **Hai tracker khác nhau bịa ra cùng
   một vật thể ở cùng khoảng frame** — đó là dấu hiệu chắc chắn lỗi nằm ở tầng detector
   YOLO26n dùng chung, chứ không phải ở tầng khớp ID.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

**Frame 113, track tham chiếu 6.** ReID đổi ID (pred 24 → 31) và cắt track 6 thành hai
mảnh; bản của tôi giữ đúng một ID xuyên suốt, IDSW = 0.

Vì sao tôi đúng: đây là chiếc xe đang vào cua. Khi xe quay, hai tín hiệu mà ReID dựa
vào đều suy yếu cùng lúc — dự đoán Kalman lệch vì quỹ đạo cong chứ không thẳng, và
vector đặc trưng ngoại hình đổi vì model đang nhìn một góc khác của cùng chiếc xe. Mắt
người không gặp vấn đề này: tôi thấy liên tục chiếc xe đó rẽ qua từng frame nên biết
chắc vẫn là một xe. Diễn biến IoU của ReID xác nhận: 0.522 (frame 104) → 0.566 → 0.575
→ 0.600 (frame 113–115), tức chất lượng bbox đã xấu dần trước khi identity đứt hẳn.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

**Frame 107–109, track 6** — đúng chỗ tôi đã quyết định **không sửa** ở mục 3.

Bản của tôi còn drift ở đây (IoU 0.550 / 0.562 / 0.574) và tôi cho rằng chấp nhận được
vì cổng đã qua. Nhưng ReID cũng gục ở **đúng vùng đó**: box lỏng từ frame 104 và mất
identity tại frame 113. Hai hệ thống độc lập — một con người, một model, không hề biết
kết quả của nhau — cùng hỏng trong cùng cửa sổ ~10 frame. Đó không còn là chuyện tay
tôi trượt, mà là bằng chứng khúc này của clip **thật sự khó**: xe vào cua nên bbox
thẳng trục không ôm sát được, và nội suy tuyến tính giữa hai keyframe không mô tả nổi
quỹ đạo cong.

Nếu làm lại, đây là chỗ đầu tiên tôi sửa: thêm 2–3 keyframe trong khoảng frame 105–112
thay vì để CVAT nội suy cả đoạn. Đây cũng là lý do tôi không dùng "model cũng sai ở đó"
để bào chữa cho việc bỏ qua — ngược lại, model sai ở đó chính là tín hiệu báo tôi nên
sửa.

## 6. Nếu phải gán thêm 10 clip nữa

Ba thứ tôi sẽ đổi, và cả ba đều xuất phát từ một lỗi cụ thể của buổi hôm nay chứ
không phải từ cảm giác chung chung.

**1. Viết rõ ngưỡng "đủ rõ để mở một track" vào `GUIDELINE_MINI.md`.**
Luật mới của tôi: **chỉ mở track khi đã nhìn ra được hình khối của một chiếc xe bốn
bánh, tức khoảng một nửa thân xe trở lên đã lộ ra** — vài pixel ló sau vật che thì
chưa tính. Tôi chọn tiêu chí theo hình khối chứ không theo số pixel cố định, vì xe ở
xa trong khung hình vốn đã nhỏ sẵn, đặt ngưỡng kiểu "bbox phải ≥ 20×20 px" sẽ khiến
tôi bỏ sót những xe ở cuối đường dù nhìn rõ mười mươi. Đây chính là chỗ tôi sai ở
`clip_01` ID 5: mở track từ frame 75 khi xe mới ló vài pixel sau xe buýt, trong khi
gold đợi tới frame 80 — bốn bbox ở giữa thành FP. Nếu có luật này từ đầu, tôi đã
không cần vòng rework đó.

**2. Bắt drift bằng cách tua từng frame ở khúc cua, thay vì chỉ kiểm midpoint.**
Bài học đắt nhất hôm nay là ba lượt QC bằng mắt của tôi đều báo PASS, nhưng chấm với
gold vẫn lòi ra 5 điểm drift ở IoU 0.546–0.574 (`clip_01` ID 5 frame 80–83, ID 6
frame 107–109). Ở mức lệch đó bbox vẫn "trông như đúng" trên màn hình nên kiểm
midpoint kiểu nhảy cóc không thể bắt được. Đáng chú ý là **cả 5 điểm drift đều rơi
vào lúc xe vào cua** — đúng chỗ nội suy tuyến tính của CVAT mô tả sai quỹ đạo nhất.
Vậy nên với 10 clip sau, tôi sẽ đánh dấu trước những đoạn xe đổi hướng rồi **tua chậm
từng frame qua đúng các đoạn đó**, còn đoạn xe chạy thẳng đều thì vẫn kiểm midpoint
như cũ. Đây là cách khoanh vùng công sức vào nơi rủi ro thật, vì nếu tua từng frame
toàn bộ 190 frame × 10 clip thì không đủ thời gian.

**3. Thêm hai bước chốt giữa "sửa" và "export".**
Lúc sửa entry của ID 5, tôi vô tình xoá mất bbox của ID 4 (xe buýt) ở frame 75 — một
chiếc xe đang chạy liên tục bỗng đứt đúng một frame. Mắt tôi không phát hiện ra, chỉ
`check_mot_labels.py` bắt được khi tôi chạy lại. Điều đáng sợ là nếu hôm đó tôi
export xong nộp luôn thì lỗi này đã lọt. Từ giờ tôi sẽ làm cả hai: **(a) sửa xong một
track thì soát ngay các track lân cận trong cùng khoảng frame**, vì thao tác trong
CVAT dễ tác động nhầm sang object đang được chọn; và **(b) chạy validator sau mỗi lần
export, coi đó là điều kiện bắt buộc trước khi tuyên bố "xong"**, chứ không chỉ chạy
một lần ở cuối buổi. Hai bước này bổ sung cho nhau: bước (a) bắt lỗi sớm khi còn nhớ
mình vừa làm gì, bước (b) là lưới an toàn cho những gì bước (a) bỏ sót.

Điểm chung của cả ba: hôm nay tôi qua được cổng không phải vì quy trình của tôi kín
kẽ, mà vì `clip_01` chỉ có 190 frame và 8 track nên sai sót còn nằm trong ngưỡng chịu
được. Với 10 clip, cùng tỉ lệ lỗi đó sẽ tích lũy thành một tập dữ liệu không dùng
được để train. Thứ cần đổi là quy trình, không phải cố gắng cẩn thận hơn.


## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt` — 570 bbox, 8 track, ĐẠT định dạng
- [x] `annotations/clip_02/gt.txt` — 234 bbox, 7 track, ĐẠT định dạng
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền — luật ID/bbox + 3 ca mơ hồ có frame/ID thật
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md` — 3 finding có frame/ID/rule/closure, reviewer checklist 8/8 đã điền
- [x] `reports/REPORT.md` (file này)
