# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Tên: `Nguyễn Trọng Minh Đức`
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

Bổ sung của nhóm: **xe đang đỗ, đứng yên vẫn phải gán.** Trạng thái chuyển động
không nằm trong định nghĩa lớp — tiêu chí duy nhất là "có phải xe bốn bánh nhìn
thấy được hay không". Cả hai clip đều có xe đỗ (`clip_01` ID 3; `clip_02` ID 1 và
2) và chúng đều được gán đầy đủ.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Trong 2 giây, một chiếc xe trong cảnh giao thông này không thể bị thay thế bởi một xe khác trông giống hệt ở cùng vị trí. Tạo ID mới ở đây là tự tay tạo ra fragmentation, loại lỗi mà IDF1 phạt rất nặng. |
| Xe bị che lâu hơn ngưỡng trên | tạo **track mới** | Sau 25 frame thì không còn đủ căn cứ để khẳng định là cùng một xe; đoán bừa mà sai sẽ thành ID switch, tệ hơn là để hai track riêng. *(Chưa gặp ca này trong `clip_01`/`clip_02`.)* |
| Xe rời khung hình rồi quay lại | **track mới** | Ngoài khung hình thì không quan sát được gì cả, không có cơ sở nào để nối lại danh tính. Áp dụng thật ở `clip_02` ID 3: xe ra khỏi rìa trái ở frame 3, ID 3 không bao giờ được dùng lại. |
| Hai xe cắt nhau / chồng lên nhau | **giữ nguyên ID của cả hai**, bám theo hướng đi của từng xe, và tua **từng frame một** qua đúng đoạn giao nhau thay vì tua nhanh | Đây là chỗ sinh ID switch nhiều nhất. Khi hai bbox chồng lên nhau, không được dựa vào "box nào gần box cũ hơn" mà phải nhìn xe nào đi tiếp về hướng nào. Kết quả: `clip_01` đạt **IDSW = 0** khi chấm với gold. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **phải thấy đủ hình khối để nói được đây là xe bốn bánh chứ không chỉ là một mảng màu lạ cạnh vật che.** Chỉ vài pixel lộ ra sau đuôi xe buýt thì **chưa** mở track — đợi thêm vài frame. Bài học có số liệu: mở track sớm 5 frame ở `clip_01` ID 5 (frame 75 thay vì 80) đã sinh ra 4 bbox thừa, FP 12 → 9 sau khi sửa lại |
| Xe đang đỗ, không di chuyển | **vẫn gán, giữ một ID xuyên suốt, bbox gần như không đổi qua các frame.** Đây **không** phải lý do để bật Outside — Outside chỉ dùng khi xe *vắng mặt*, còn xe đỗ vẫn nhìn thấy được. Validator sẽ cảnh báo "bbox gần như đứng im... hay bạn quên bấm outside?" — đó là cảnh báo cần kiểm, không phải lỗi |
| Keyframe đặt dày ở đâu | dày nhất ở **khúc cua và đoạn xe đổi hướng/đổi scale nhanh**, vì CVAT nội suy **tuyến tính** giữa hai keyframe nên không mô tả nổi quỹ đạo cong. Đoạn xe chạy thẳng đều thì hai keyframe đầu–cuối là đủ. Bằng chứng: hai chỗ drift còn sót lại (`clip_01` ID 5 frame 80–83 và ID 6 frame 107–109, IoU 0.546–0.574) **đều rơi vào lúc xe vào cua** |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1 — xe đỗ đứng yên, tưởng là quên Outside
- Clip / frame / ID: `clip_01` / MOT frame 1–15 (CVAT 0–14) / ID 3
- Tình huống: bbox gần như bất động suốt 15 frame. `check_mot_labels.py` cảnh báo
  *"bbox gần như đứng im — vật thể thật đứng yên, hay bạn quên bấm outside?"*
- Quyết định: **giữ nguyên bbox, không bật Outside.**
- Lý do: kiểm bằng mắt trên `outputs/vis_clip_01/` thấy đây là xe đỗ thật, vẫn nằm
  trong cảnh và nhìn thấy rõ. Outside mang nghĩa "xe không có bbox ở frame này",
  hoàn toàn khác với "xe không di chuyển". Bật Outside ở đây sẽ tạo ra FN.

### Ca 2 — track chỉ dài 2 frame, trông như vẽ nhầm
- Clip / frame / ID: `clip_02` / MOT frame 1–2 (CVAT 0–1) / ID 3
- Tình huống: một track chỉ tồn tại đúng 2 frame rồi biến mất. Validator cảnh báo
  *"track chỉ có 2 frame — vẽ nhầm hay vật thể thật?"*. Nhìn thoáng qua rất giống
  một box rác do lỡ tay click.
- Quyết định: **giữ track ngắn này, và bật Outside tại frame 3.**
- Lý do: đó là phần đuôi một chiếc xe đang ra khỏi khung bên trái — `bb_left = 0`
  ở cả hai frame và width co từ 18.6 xuống 11.5 px, đúng dấu hiệu xe đang rời
  khung. Rule "mọi xe bốn bánh nhìn thấy được đều phải có bbox" không có ngoại lệ
  cho xe chỉ xuất hiện ngắn. Xóa track này mới là lỗi bỏ sót.

### Ca 3 — lúc nào mới được mở track cho xe vừa ló ra sau vật che
- Clip / frame / ID: `clip_01` / MOT frame 75–80 (CVAT 74–79) / ID 5
- Tình huống: một xe con bị xe buýt (ID 4) che gần hết, chỉ lộ ra vài pixel ở mép
  trái xe buýt. Không rõ nên mở track ngay từ lúc thấy mảng pixel đầu tiên, hay đợi
  tới lúc nhìn ra hình dáng xe.
- Quyết định: ban đầu mở track từ frame 75; sau khi chấm với gold đã **lùi điểm bắt
  đầu về frame 80**.
- Lý do: gold không tính xe này là đối tượng cho tới khi nó lộ ra đủ. Bốn bbox ở
  frame 75–78 trở thành FP. Sau khi sửa, FP giảm 12 → 9 và HOTA/DetA/IDF1/MOTA đều
  tăng. Đây chính là khoảng trống mà mục 3 của file này ban đầu chưa nói rõ.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Thiếu hẳn tiêu chí "đủ rõ để mở một track".** Bản đầu chỉ ghi "bắt đầu track từ
  frame đầu tiên xác định được là xe bốn bánh" — nghe thì hợp lý nhưng không dùng
  được khi đứng trước màn hình, vì mỗi người hiểu "xác định được" một kiểu. Đã viết
  lại ở mục 3 kèm ví dụ số thật (ca 3). Người gán tiếp theo cần đọc dòng này **trước
  khi** chạm vào các đoạn có xe che nhau.
- **Chưa nói keyframe phải đặt dày ở đâu.** Bản đầu để trống, hậu quả là cả hai chỗ
  drift còn sót lại đều nằm ở khúc cua — đúng chỗ nội suy tuyến tính sai nhiều nhất.
  Đã bổ sung ở mục 3.
- **Chưa có luật cho xe đỗ**, dẫn tới việc phải dừng lại tra cứu mỗi lần validator
  cảnh báo "bbox đứng im". Đã bổ sung ở mục 1 và mục 3.
- **Quy trình còn thiếu một bước: kiểm lại sau khi sửa, trước khi export.** Lúc sửa
  `clip_01` ID 5 đã vô tình làm mất bbox của ID 4 ở frame 75 (xe buýt đang chạy liên
  tục 74 → 76 bỗng đứt một frame). Lỗi này do `check_mot_labels.py` bắt được chứ
  không phải do mắt. Luật mới: **mỗi lần sửa xong một track, phải soát lại các track
  lân cận ở đúng khoảng frame đó rồi mới export.**
- **Mắt thường không bắt được drift ở mức IoU ~0.55.** Cả ba lượt QC đều báo PASS
  nhưng gold vẫn chỉ ra 5 điểm drift. Nếu gán thêm clip mới mà không có gold, phải
  giả định loại lỗi này luôn còn sót và bù lại bằng cách đặt keyframe dày hơn ở
  những đoạn đã biết là rủi ro (cua, đổi scale nhanh, ra/vào vùng bị che).
