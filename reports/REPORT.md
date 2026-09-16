# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Lê Nguyễn Hà My   Nhóm: K4-L2   Ngày: 2026-09-16

## 1. Nhãn của tôi

| Chỉ số | Giá trị |
| --- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 29 |
| v=2 / v=1 / v=0 | 349 / 126 / 18 |
| Thời gian trung bình mỗi ảnh | 5 phút |

Ba khớp có `%v=1` cao nhất (chép từ `reports/visibility_report.md`):

1. `left_ear` — 69%
2. `right_ear` — 41%
3. `left_wrist` — 34%

Đúng. Tai trái có %v=1 cao nhất vì phần lớn ảnh trong bộ dữ liệu là người quay lưng hoặc nghiêng đầu, tai bị tóc hoặc mũ bảo hiểm che khuất, không nhìn thấy vành tai nhưng biết chắc còn trong khung — ví dụ `train_02` người đạp xe đội mũ, tai trái bị mũ che hoàn toàn. Tai phải tương tự nhưng ít hơn vì nhiều ảnh người hơi nghiêng sang trái. Cổ tay trái hay bị che bởi thân mình hoặc vật đang cầm (pizza ở `train_01`, điện thoại ở `train_13`), nên phải ước lượng vị trí và để v=1. Đây đúng là những khớp khó xác định cờ nhất: tai khó vì phải phân biệt "tóc che" (v=1) với "nhìn thấy vành tai" (v=2), còn cổ tay khó vì phải ước lượng vị trí giải phẫu khi bị vật khác che.

## 2. Chấm với gold

| Chỉ số | Trước rework | Sau rework |
| --- | ---: | ---: |
| OKS trung bình | 0.928 | 0.946 |
| OKS@0.50 | 0.966 | 1.000 |
| OKS@0.75 | 0.966 | 1.000 |
| Lỗi `dao_trai_phai` | 1 | 0 |
| Lỗi `nham_nguoi` | 1 | 0 |
| Lỗi `xoa_khop_bi_che` | 0 | 0 |

**Tôi đã sửa gì giữa hai lần chạy** (ghi cụ thể: ảnh nào, người thứ mấy, khớp nào):

- `train_02`, người #1: cả skeleton bị đảo trái/phải — người đạp xe quay lưng, tôi nhầm tay trái/phải theo mép ảnh thay vì theo cơ thể. Đã đổi lại toàn bộ cặp left/right cho vai, khuỷu tay, cổ tay, hông, gối, mắt cá. OKS tăng từ 0.443 lên 0.904.
- `train_01`, người #2 (ông già áo sọc), khớp `right_wrist`: cổ tay phải bị chấm lệch sang phía bàn tay cô phục vụ bên cạnh (hai người cùng cầm khay pizza). Đã kéo chấm về đúng mép khay phía ông, không còn gần `right_wrist` của người khác. OKS tăng từ 0.801 lên 0.867.
- `train_01`, người #1 (cô phục vụ), khớp `left_wrist`: chấm lệch ~35px so với gold, kéo lại sát cổ tay thực tế trên ảnh.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?** Ảnh đó dễ hay khó? Nếu là ảnh dễ, bạn nghĩ vì sao mình vẫn sai?

Lỗi đảo trái/phải ở `train_02`, người đạp xe đứng cạnh xe trên đường. Ảnh này thuộc loại trung bình — chỉ có 1 người, đứng rõ ràng, không bị che nhiều. Tôi sai vì người quay lưng về phía camera: tay trái của họ nằm bên phải khung hình, tôi vô thức gắn `left_shoulder` theo hướng nhìn của mình (bên trái ảnh) thay vì theo cơ thể người. Đây là lỗi slide 43 cảnh báo: ảnh dễ, làm nhanh, quên quy ước COCO "trái/phải tính theo cơ thể người, không theo camera". Khi mở `outputs/vis_train/train_02.jpg`, đường nối vai trái và phải cắt chéo nhau — dấu hiệu rõ ràng của lỗi đảo.

## 3. Kiểm chéo

Bạn cùng nhóm: ______

Khớp lệch `%v=1` nhiều nhất giữa hai bảng đếm:

| Khớp | Bạn | Họ | Lệch | Nguyên nhân (guideline hay gán sai?) |
| --- | ---: | ---: | ---: | --- |
| | | | | |
| | | | | |

Luật mới đã bổ sung vào `GUIDELINE_MINI.md` sau khi thống nhất:

<!-- Viết một rule kiểm chứng được: điều kiện nhìn thấy/căn cứ vị trí → chọn v=1 hoặc v=0.
Không chỉ ghi “cẩn thận hơn khi gán”. -->


## 4. Model

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | +0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6908 | +0.0055 |
| pose_precision | 0.9734 | 0.9792 | +0.0058 |
| pose_recall | 0.8462 | 0.8462 | +0.0000 |
| box_mAP50-95 | 0.8119 | 0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

1. `pose_mAP50-95` tăng 0.0055 (từ 0.6853 lên 0.6908). Mức tăng rất nhỏ, chưa đủ để kết luận model giỏi hơn hẳn — tập test chỉ có 10 ảnh nên sai số dao động lớn. Tuy vậy, kết quả cho thấy 20 ảnh train của tôi không phá hỏng kiến thức gốc COCO mà model đã học. Đồng thời, `pose_precision` cũng tăng nhẹ +0.0058, nghĩa là model sau fine-tune ít đoán sai khớp hơn một chút, có thể vì nhãn của tôi nhất quán về cờ occluded cho hông và cổ tay.

2. Sau fine-tune, `box_mAP50-95` = 0.8041, `pose_mAP50-95` = 0.6908, chênh 0.1133. Model tìm người (ô chữ nhật) dễ hơn tìm khớp. Điều này hợp lý: để đúng box chỉ cần bao quanh đúng thân người, còn 17 khớp phải đúng từng điểm cụ thể trên cơ thể — khớp bị che, khớp chồng lên nhau giữa hai người, và dễ bị đảo trái/phải khi người quay lưng. Chênh lệch ~0.11 cho thấy bài toán pose khó hơn detection rõ rệt.

3. Ảnh `test_03`: model **nhầm người** — ảnh có 2 người đứng gần nhau, model phát hiện đúng 2 người nhưng xương khuỷu tay và cổ tay của người đứng trước bị gắn lệch sang phía thân người đứng sau. Đây không phải lệch nhẹ vì khoảng cách giữa chấm đoán và khớp thật quá xa, rơi hẳn vào bounding box của người kia.

4. Ảnh có OKS thấp nhất giữa nhãn của tôi và model là `train_02` với OKS = 0.353. Tôi đúng: đây là ảnh người đạp xe quay lưng, sau khi rework tôi đã gắn trái/phải đúng theo cơ thể (đối chiếu gold đạt OKS 0.904). Model bị đảo hai vai và hai hông — đường nối vai trên ảnh model cắt chéo nhau. Nhìn ảnh gốc thấy balo nằm sát vai phải của người (phía xa camera), khớp với nhãn của tôi chứ không khớp với model.

5. Không trùng. Ảnh tôi gán tệ nhất so với gold là `train_02` (OKS trước rework = 0.443, lỗi đảo trái/phải). Ảnh model tệ nhất so với nhãn tôi cũng là `train_02` (OKS = 0.353). Trong trường hợp này lại trùng: `train_02` khó cho cả người lẫn máy vì người đạp xe quay lưng hoàn toàn, tất cả khớp đều phải gắn "ngược" so với trực giác nhìn ảnh. Cả tôi (trước rework) và model đều mắc cùng một lỗi — đảo trái/phải — cho thấy đây là ca khó thuộc tính của dữ liệu (người quay lưng), không chỉ lỗi thao tác riêng lẻ.

## 5. Một rule evidence bạn đã dùng

Ảnh `train_13`, người #1 (đàn ông mặc vest ở tiền cảnh), khớp `left_knee`. Người này đứng chính diện, nhìn thấy rõ đầu, vai, khuỷu tay, cổ tay, và hông — nhưng phần chân từ đầu gối trở xuống bị che hoàn toàn bởi vạt áo vest dài và cặp táp cầm tay. Tôi vẫn thấy hai vai và phần hông, nên biết chắc hai đầu gối còn nằm trong khung ảnh, chỉ bị che bởi quần áo chứ không ra ngoài mép. Theo luật lớp: bị che mà còn trong ảnh thì đặt chấm ước lượng trên đường nối hông–mắt cá (ước lượng vị trí giải phẫu) và để v=1, không dùng Outside. Nếu để v=0, khớp đầu gối bị loại khỏi OKS và model sẽ học rằng "người mặc vest dài = không có đầu gối".
