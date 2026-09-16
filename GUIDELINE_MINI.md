# Mini guideline - nhóm: K4-L2  |  người gán: Lê Nguyễn Hà My  |  ngày: 2026-09-16

> Điền file này **trong lúc** gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng lại
> hơn 10 giây để phân vân, đó là một dòng phải ghi vào đây.

## 1. Luật bắt buộc (đã thống nhất cả lớp - không sửa)

- Bộ 17 điểm COCO, đúng tên, đúng thứ tự. Lấy từ file `.SVG` chung.
- Mọi người trong ảnh đều có **đủ 17 điểm**. Điểm không dùng được thì gắn cờ, không xoá.
- Trái/phải tính theo **cơ thể người**, không theo bức ảnh.
- Bị che, còn trong khung -> `v = 1`, **vẫn đặt chấm** ở vị trí ước lượng.
- Ra ngoài mép ảnh -> `v = 0`, **không** đặt chấm.
- Không dùng `Hidden` (`h`) - nó không được lưu vào file.

## 2. Luật của nhóm bạn (phải điền)

| Tình huống | Luật nhóm bạn chọn | Vì sao |
| --- | --- | --- |
| Hông của người mặc quần áo dài | Luôn `v=1`, đặt chấm ước lượng trên đường nối vai–gối, chia tỉ lệ khoảng 1/2 từ vai xuống gối. Ảnh mẫu: `train_13` người #1 — hông bị vạt vest che, chấm đặt giữa vai và gối. | Hông không bao giờ nhìn thấy qua quần áo. Nếu để `v=2` thì cả nhóm không nhất quán, và model nhầm lẫn giữa "nhìn thấy" và "ước lượng". |
| Tai bị tóc hoặc mũ bảo hiểm che một phần | Nếu **không thấy vành tai**: `v=1`, đặt chấm theo vị trí giải phẫu (ngang mắt, sau gò má). Nếu **thấy ít nhất một phần vành tai**: `v=2`. Ảnh mẫu: `train_02` người #1 — mũ bảo hiểm che hoàn toàn tai trái → `v=1`. | Ranh giới rõ ràng: "vành tai" là bề mặt nhìn thấy được. Tóc/mũ che hết vành tai = bị che. Thấy một mảnh vành tai = nhìn thấy. |
| Người bị cắt ở mép ảnh (chỉ thấy từ hông trở lên) | Khớp **còn pixel trong khung**: `v=1` hoặc `v=2` tuỳ có nhìn thấy không. Khớp **ra ngoài mép hoàn toàn** (gối, mắt cá): `v=0`. Ảnh mẫu: `train_04` người #2 — cắt ở mép dưới, gối và mắt cá ra ngoài → `v=0`. | Luật lớp: `v=0` chỉ dành cho khớp ra ngoài mép. Khớp còn trong khung dù bị che vẫn là `v=1`. |
| Cổ tay nằm sau tay lái / sau thân mình | `v=1`, đặt chấm ở vị trí ước lượng dựa trên hướng cánh tay và vật che. Ảnh mẫu: `train_01` người #1 — cổ tay phải nằm sau khay pizza, chấm đặt ở mép khay phía bàn tay. | Cổ tay vẫn trong khung, chỉ bị vật cầm/tay lái che. Nếu để `v=0` thì model mất dữ liệu về vị trí cổ tay khi cầm đồ vật. |
| Hai người chồng lên nhau | Gán **riêng từng người**, xong hẳn người trước mới sang người sau. Khớp của người bị che bởi người kia: `v=1`, đặt chấm ước lượng. Ảnh mẫu: `train_01` có 2 người đứng gần nhau — gán cô phục vụ trước, rồi mới gán ông già. | Tránh lỗi "nhầm người": xương của A kéo sang B. Làm lần lượt giúp giữ skeleton đúng cơ thể. |
| Người nhỏ đến mức nào thì không gán nữa | Bộ ảnh đã chọn sao cho mọi người đều đủ lớn. Nếu người < 40px chiều cao: ghi vào đây và hỏi nhóm. Trong 20 ảnh core, chưa gặp ca nào quá nhỏ. | Người quá nhỏ (<40px) thì 17 chấm chồng lên nhau, không có ý nghĩa cho model. |

Ảnh mẫu minh hoạ có thể xem trong `outputs/vis_train/` — file visualize đã vẽ xương với
màu xanh (bên trái cơ thể) và cam (bên phải cơ thể).

## 3. Ba ca mơ hồ đã gặp (bắt buộc, ghi ít nhất 3)

### Ca 1 - ảnh `train_02`, người thứ `1`, khớp `left_shoulder` / `right_shoulder`

- Mơ hồ ở chỗ nào: Người đạp xe **quay lưng** về phía camera. Tay trái của họ nằm bên phải khung hình. Không biết nên gán `left` theo mép ảnh hay theo cơ thể.
- Bạn quyết thế nào: Gán theo **cơ thể người** — `left_shoulder` ở bên phải khung hình (vì người quay lưng).
- Vì sao: Luật lớp quy định rõ: trái/phải tính theo cơ thể. Tưởng tượng mình đứng vào vị trí người đó rồi giơ tay trái — tay đó nằm phía nào trên ảnh thì đó là `left`.
- Nếu người khác quyết ngược lại thì model học sai cái gì: Model học rằng khi người quay lưng thì `left_shoulder` nằm ở bên trái ảnh — sai hoàn toàn. Kết hợp với `flip_idx` augmentation, lỗi này bị nhân đôi.

### Ca 2 - ảnh `train_13`, người thứ `1`, khớp `left_knee`

- Mơ hồ ở chỗ nào: Đàn ông mặc vest dài, phần chân từ đầu gối trở xuống bị vạt áo và cặp táp che hoàn toàn. Không thấy bất kỳ phần da hay nếp gối nào. Phân vân giữa `v=1` (bị che) và `v=0` (coi như không thấy).
- Bạn quyết thế nào: `v=1`, đặt chấm ước lượng trên đường nối hông–mắt cá (ước lượng vị trí giải phẫu giữa đùi).
- Vì sao: Nhìn thấy cả hai vai và hông → biết chắc đầu gối còn trong khung ảnh, chỉ bị quần áo che chứ không ra ngoài mép. Theo luật: bị che + còn trong ảnh = `v=1`.
- Nếu người khác quyết ngược lại thì model học sai cái gì: Nếu để `v=0`, khớp đầu gối bị loại khỏi OKS và model sẽ học rằng "người mặc vest dài = không có đầu gối", dẫn đến model bỏ qua gối khi gặp người mặc đồ công sở.

### Ca 3 - ảnh `train_09`, người thứ `1`, khớp `left_ear`

- Mơ hồ ở chỗ nào: Người quay nghiêng ~45°, tóc dài phủ xuống. Tai trái bị tóc che gần hết, chỉ thấy lờ mờ một mảng da nhỏ phía sau tóc. Phân vân giữa `v=1` (che hoàn toàn) và `v=2` (vẫn thấy một phần).
- Bạn quyết thế nào: `v=1` — vì không thấy rõ **vành tai**, chỉ thấy da vùng sau tai qua kẽ tóc.
- Vì sao: Theo luật nhóm ở mục 2: phải thấy ít nhất một phần vành tai mới được `v=2`. Mảng da lờ mờ qua tóc không phải vành tai.
- Nếu người khác quyết ngược lại thì model học sai cái gì: Nếu để `v=2`, model nhận được tín hiệu "tai luôn nhìn thấy" ở góc nghiêng — khi inference gặp người tương tự, model sẽ tự tin đặt chấm tai ở chỗ tóc che, dẫn đến sai vị trí.

## 4. Sau khi so visibility report với bạn cùng nhóm

- Khớp lệch `%v=1` nhiều nhất: `left_hip` (bạn `28%` / họ `10%`)
- Nguyên nhân là **guideline chưa rõ**: hai bên chưa thống nhất khi nào hông "nhìn thấy được" khi người mặc quần dài. Tôi để `v=1` vì hông luôn bị che bởi quần, bạn kia để `v=2` vì "đoán được vị trí = nhìn thấy". Lệch xảy ra nhất quán trên nhiều ảnh, không phải sót thao tác ở một ảnh cụ thể.
- Luật mới bổ sung vào mục 2 sau khi thống nhất: **Hông người mặc quần: luôn `v=1`**, đặt chấm ước lượng trên đường nối vai–gối. Không bao giờ để `v=2` cho hông khi mặc quần dài, vì không nhìn thấy bề mặt xương/da của khớp hông.
