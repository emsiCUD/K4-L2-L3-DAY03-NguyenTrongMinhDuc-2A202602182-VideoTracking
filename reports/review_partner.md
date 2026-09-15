# Peer review — Day 3

| Trường | Giá trị |
| --- | --- |
| Author | `Nguyễn Trọng Minh Đức` |
| Reviewer | `Nguyễn Đăng Vĩ Anh` |
| Pair ID | `N/A` |
| CVAT version | `2.74.1` |
| Thời điểm review | `trước 11:28 15/09/2026` (trước khi khóa pre-gold) |

Phạm vi review: `clip_01` (190 frame, 8 track, 575 bbox tại thời điểm review) và
`clip_02` (60 frame, 7 track, 234 bbox). Review thực hiện **trước** khi khóa
pre-gold, chưa mở teaching reference.

Ghi chú nguồn finding: các mục dưới đây được `tools/check_mot_labels.py` nêu lên
dưới dạng CẢNH BÁO, sau đó author và reviewer cùng kiểm bằng mắt trên
`outputs/vis_clip_0X/` rồi thống nhất closure. Reviewer không nêu thêm defect nào
ngoài danh sách này.

## Danh sách finding

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 0–14 | 1–15 | 3 (clip_01) | endpoint/scope — nghi ngờ | bbox gần như đứng im suốt 15 frame; validator hỏi "vật thể thật đứng yên, hay quên bấm Outside?" | kiểm bằng mắt `vis_clip_01` frame 1–15, nếu xe đã rời khung thì bật Outside | **not-a-defect** — xe đỗ thật, vẫn nằm trong cảnh và nhìn thấy được, nên phải có bbox theo rule "mọi xe bốn bánh nhìn thấy được đều phải có bbox" |
| 2 | 0–1 | 1–2 | 3 (clip_02) | endpoint/scope — nghi ngờ | track chỉ tồn tại 2 frame; validator hỏi "vẽ nhầm hay vật thể thật?" | kiểm frame 1–2, nếu là box rác thì xóa track | **not-a-defect** — là phần đuôi xe đang ra khỏi khung bên trái (`bb_left = 0`, width co từ 18.6 → 11.5 px). Áp rule "bbox chạm rìa ảnh, không đoán phần ngoài ảnh" + "Outside tại frame đầu xe vắng mặt"; export không còn dòng nào từ frame 3 → xác nhận Outside đã đặt đúng |
| 3 | 11–25 / 0–14 | 12–26 / 1–15 | 1 và 2 (clip_02) | endpoint/scope — nghi ngờ | hai track đứng im dài, cùng dạng cảnh báo như finding #1 | kiểm bằng mắt, đối chiếu rule xe đỗ | **not-a-defect** — cả hai là xe đỗ đứng yên trong cảnh, cùng lý do với finding #1 |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | clip_01 có 8 track (ID 1–8) ≥ 6; không có pedestrian/xe máy trong danh sách |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | soát `vis_clip_01` frame 1–190, ID không bị gán lại cho xe khác sau khi track kết thúc |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | kiểm các frame có 4–6 bbox cùng lúc (đông xe nhất), số ID không nhảy qua lại |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | clip_02 ID 3 frame 2→3 là ca mẫu: box dừng đúng frame xe khuất, không còn dòng treo |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | clip_02 ID 3 bbox dừng ở `bb_left = 0`, không vẽ nối phần đã ra ngoài ảnh |
| Frame giữa hai keyframe không bị interpolation drift | PASS | soát midpoint các đoạn dài bằng mắt, không thấy box trượt khỏi xe ở mức nhận ra được |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `check_mot_labels.py` cả hai clip: "ĐẠT phần định dạng: 0 lỗi"; frame range 1..190 và 1..60 |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | 3/3 finding ở bảng trên đều có closure `not-a-defect` kèm rule viện dẫn |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | tua toàn bộ 190 ảnh `vis_clip_01` chỉ nhìn số ID; không phát hiện ID switch, fragmentation hay reuse ID |
| 2 — endpoint/scope | PASS | đi frame đầu/cuối của cả 8 track; 3 cảnh báo validator đều được adjudicate thành not-a-defect (bảng trên) |
| 3 — geometry/interpolation | PASS tại thời điểm review | soát midpoint bằng mắt không thấy lệch rõ. **Ghi nhận sau khi mở gold:** vẫn còn drift ở MOT frame 80–83 (ID 5) và 107–109 (ID 6) với IoU 0.53–0.58 — mức lệch này mắt thường không bắt được, chỉ lộ ra khi chấm định lượng |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: **Finding #2 (clip_02, MOT frame 1–2, ID 3).** Một track chỉ dài 2 frame trông y hệt lỗi vẽ nhầm, nhưng rule "bbox chạm rìa ảnh nếu xe bị cắt khung" + "Outside tại frame đầu object vắng mặt" cho thấy đây là cách xử lý đúng của một xe đang rời khung. Nếu xóa track này thì mới là lỗi (bỏ sót xe nhìn thấy được).
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do: **Finding #1 (clip_01, MOT frame 1–15, ID 3)** — xe đỗ đứng yên. Validator cảnh báo "quên Outside?" vì bbox gần như bất động, nhưng Outside chỉ dùng khi xe **vắng mặt**; xe đỗ vẫn nhìn thấy được nên bắt buộc phải có bbox.
3. Một rule cần Lab Coach làm rõ: **ngưỡng "đủ rõ để bắt đầu một track".** Khi một xe mới chỉ lộ ra vài pixel phía sau xe buýt, chưa có tiêu chí định lượng cho frame đầu tiên được phép mở track. Thực tế chấm với gold cho thấy chênh lệch 4–5 frame ở ca này (clip_01, ID 5) đủ để sinh ra FP.
