# Báo cáo Day 13 — Phạm Hữu Hưng, 2A202603087

## 1. Kết luận trước

- **Thông tin:** MSSV `2A202603087`, task `D13-2A202603087`, gồm 10 frame `s01f13`, `s01f54`, `s02f13`, `s02f34`, `s03f04`, `s03f46`, `s04f07`, `s04f51`, `s05f05`, `s05f44`. Máy dùng macOS, trình duyệt Chrome, CVAT Server `v2.74.1` cùng giao diện `cvat-vf-overlay` bản `v2.83.1`. Bài làm cá nhân.
- **Tóm tắt:** Tổng đã vẽ 70 cuboid, gồm 56 `Car`, 5 `Truck`, 5 `Cyclist`, 3 `Bus` và 1 `Pedestrian`. Frame ít nhất là `s04f51` với 4 cuboid vì khung cảnh thưa xe rõ điểm trong tầm 30 m. Phiếu `selfcheck` gắn cờ 3 trên 70 cuboid. Phần tôi tin nhất là hướng `yaw_deg` và độ cao đáy `z_bottom`; phần còn yếu nhất là phân loại xe cỡ trung và ước lượng kích thước vật xa 30–50 m.
- **Cách dùng CVAT:**
  1. Đổi góc nhìn bằng chuột trái để xoay, chuột phải để dời, con lăn để thu phóng; xoay thẳng từ trên xuống để quét 360° và bấm biểu tượng bánh răng trên khung ảnh để đổi đủ 7 camera.
  2. Vẽ cuboid bằng nút **N** rồi nháy đúp đặt hộp; chỉnh tâm, dài, rộng và xoay yaw ở khung **Top**, còn khung **Side** và **Front** chỉ kéo lên xuống để đáy chạm đường và đặt chiều cao.
  3. Bật **Appearance → Cuboid orientation** để căn vạch đỏ `+X` chỉ về đầu xe; bấm **Save** rồi chạy `python scripts/lab.py sync` để xem cuboid chiếu lên 7 ảnh camera qua `preview`.

## 2. Đọc frame — Bước 2

Hai frame đọc kỹ gồm `s01f13` với ảnh `t1-s01f13-3d.png`, `t1-s01f13-top.png` và `s01f54` với ảnh `t1-s01f54-3d.png`, `t1-s01f54-top.png`. Trên cả hai ảnh nhìn từ trên xuống đã đánh dấu phía trước `−y`, trục `+x` và trục `+y`.

| Frame | Phía trước xe và hướng +x, +y | Mặt đường | Vật xa nhất đọc được | Loại vật thấy | Vật phía sau và hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f13` | Theo `cam01`, phía trước là `−y`, `+x` sang trái, `+y` ra sau | Tương đối phẳng, thấp hơn LiDAR khoảng 1,7 m | Khoảng 29,8 m | `Car`, `Bus`, `Pedestrian` | 1 `Pedestrian` sau-phải ở 9,45 m trên `cam08`; 2 `Car` phía sau ở 21,32 m và 29,78 m trên `cam09` | Cụm điểm thưa trên 35 m dọc lề đường, khó rõ là vật tĩnh hay người |
| `s01f54` | Theo `cam01`, phía trước là `−y`, `+x` sang trái, `+y` ra sau | Tương đối phẳng, thấp hơn LiDAR khoảng 1,7–1,8 m | Khoảng 16,2 m | `Car`, `Cyclist` | 2 `Car` phía sau ở 6,86 m và 11,37 m trên `cam09`; 1 `Cyclist` bên phải ở 10,93 m trên `cam07` | Cụm điểm rời rạc sát lề trên 30 m bị cây che một phần |

- **3 điều chỉ camera cho biết:** màu sắc cùng chi tiết đèn pha và đèn hậu để phân biệt đầu đuôi xe; phân biệt xe hai bánh có người lái hay dựng trống và người đi bộ với cột tĩnh; biển báo, đèn tín hiệu và vạch kẻ làn đường.
- **3 điều chỉ LiDAR cho biết:** khoảng cách chính xác theo mét tới từng vật; kích thước 3D thật và độ cao đáy so với mặt đường không bị méo phối cảnh; phát hiện đủ 360° kể cả ở góc mù giữa các camera.

Tám frame lướt:

| Frame | Phía trước trên ảnh nhìn từ trên xuống | Điều lạ |
|---|---|---|
| `s02f13` | Hướng `−y`, ngược chiều trục `+y` | Nhiều xe đi ngược chiều ở làn bên trái, `yaw_deg` quanh 88–90° |
| `s02f34` | Hướng `−y`, ngược chiều trục `+y` | Có xe phía sau bị che một phần ở 31,48 m |
| `s03f04` | Hướng `−y`, ngược chiều trục `+y` | Giao thông đông 10 cuboid, có 1 `Cyclist` rất xa ở 47,89 m |
| `s03f46` | Hướng `−y`, ngược chiều trục `+y` | Có 2 `Cyclist` bên phải ở 7,12 m và 12,07 m |
| `s04f07` | Hướng `−y`, ngược chiều trục `+y` | Làn ngược chiều bên trái có `Truck` và `Bus` cỡ lớn |
| `s04f51` | Hướng `−y`, ngược chiều trục `+y` | Đường thoáng, chỉ có 4 cuboid rõ điểm |
| `s05f05` | Hướng `−y`, ngược chiều trục `+y` | Có `Car` đi rất sát bên phải ở 3,53 m |
| `s05f44` | Hướng `−y`, ngược chiều trục `+y` | Giao lộ có xe cắt ngang bên phải, `yaw_deg` khoảng 8–18° và mặt đường nhánh hơi dốc lên |

## 3. Ba cuboid có hướng dẫn — Bước 3

Ảnh mẫu gồm `t2-a-car.png`, `t2-b-bus-truck.png` và `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front và ảnh? | Kích thước so với bảng 4.3 |
|---|---|---|---|---|---|---|
| a. Xe con gần | `s01f13`, idx 1 | `Car` | Có, `269,0°` chỉ về đầu xe cùng chiều | Có, `z_bottom = −1,766 m` | Có | Trong khoảng Sedan: `4,286 × 1,814 × 1,491 m` |
| b. Xe buýt hoặc xe tải | `s01f13`, idx 3 | `Bus` | Có, `284,3°` chỉ về đầu xe buýt | Có, `z_bottom = −1,751 m` | Có | Gần khớp Bus thường: `12,552 × 2,773 × 3,204 m`, dài và rộng nhỉnh nhẹ 0,05–0,17 m ở tầm 29,83 m |
| c. Người đi bộ hoặc xe hai bánh | `s01f13`, idx 2 | `Pedestrian` | Có, `0,0°` theo hướng người | Có, `z_bottom = −1,443 m` trên vỉa hè | Có | Trong khoảng Người lớn: `0,531 × 0,676 × 1,755 m` |

Tôi không làm vòng hiệu chuẩn trên task chung mà rà soát trực tiếp theo guideline trên task `D13-2A202603087`. Các vật bỏ qua vì không đủ cơ sở gồm vật xa trên 40–50 m quá thưa điểm thuộc nhóm `LOW_VISIBILITY` và `UNCERTAIN_SCOPE`, vật bị che gần hết sau xe lớn, cùng xe hai bánh dựng trống không có người lái.

## 4. Bốn hiện tượng chất lượng — Bước 4

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân — giả thuyết | Thấy gì trên point cloud | Hệ quả khi gán theo guideline |
|---|---|---|---|---|
| 1. Bóng che | `t4-1-bong-che.png`, task cá nhân `D13-2A202603087` | Giả thuyết tia LiDAR bị thân xe gần hoặc vật lớn phía trước chắn hoàn toàn | Tạo vùng quạt trống điểm sau vật che, xe phía sau chỉ hiện một phần thân | Bật `occluded` và vẽ cuboid bao cả phần suy luận được, không coi vùng trống là đường trống |
| 2. Thưa ở xa | `t4-2-thua-o-xa.png`, task cá nhân `D13-2A202603087` | Giả thuyết tia LiDAR 40 kênh phân kỳ theo góc nên khoảng cách tia giãn rộng ở 30–50 m | Vật ở xa chỉ còn vài chục đến vài điểm rời rạc, khó rõ đủ 4 góc | Chỉ vẽ khi thấy khoảng 15–20% thể tích hoặc đủ cơ sở suy luận; không đủ thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp hoặc ít điểm bất thường | `t4-3-phan-xa-thap.png`, task cá nhân `D13-2A202603087` | Giả thuyết sơn tối màu hấp thụ tia laser hoặc mặt kính phản xạ gương làm tia bật chệch | Vật tối màu hoặc vùng kính thưa và rỗ điểm hơn hẳn xe sáng màu ở cùng tầm | Ghi nguyên nhân là giả thuyết, không khẳng định; kết hợp camera và bảng 4.3 để không co hẹp hộp |
| 4. Điểm lạ | `t4-4-diem-la.png`, task cá nhân `D13-2A202603087` | Giả thuyết phản xạ đa đường qua mặt kính, mặt đường hoặc nhiễu cảm biến | Xuất hiện điểm lẻ hoặc cụm điểm lơ lửng trên không hay dưới mặt đường, không có vật thật trên ảnh camera | Không gán thành vật |

## 5. Số từ cuboid của tôi — Bước 5

Số liệu lấy từ `submission/D13-2A202603087-bangA.csv` và `submission/D13-2A202603087-tukiem.csv`.

| Cuboid | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét |
|---|---|---|---|---|---|---|---|
| `s01f13`, idx 1, `Car` | −1,021 m | 1,491 m | −1,766 m | 5,12 m | 5,22 m | 269,0° | Chạm đường, `ground_gap_m = −0,01 m` |
| `s01f13`, idx 3, `Bus` | −0,149 m | 3,204 m | −1,751 m | 29,83 m | 29,83 m | 284,3° | Chạm đường, `ground_gap_m = 0,00 m` |
| `s05f44`, idx 9, `Car` | −0,573 m | 1,525 m | −1,336 m | 32,68 m | 32,68 m | 10,0° | Cao hơn mốc −1,7 m khoảng 0,364 m do đường nhánh bên phải dốc lên; so mặt đường tại chỗ `ground_gap_m = +0,12 m` nên vẫn chạm đường |

- **Gần nhất và xa nhất:** Gần nhất là `s05f05`, idx 1, `Car` với `dist_m = 3,53 m` và `r = 3,63 m`. Xa nhất là `s03f04`, idx 10, `Cyclist` với `dist_m = 47,89 m` và `r = 47,90 m`.
- **Kích thước so với bảng 4.3:** Người đi bộ `s01f13`, idx 2 có `l = 0,531 m`, `w = 0,676 m`, `h = 1,755 m`, cả ba chiều đều nằm trong khoảng Người lớn. Xe con `s01f13`, idx 5 có `l = 5,652 m`, `w = 1,961 m`, `h = 1,528 m`; rộng và cao nằm trong khoảng Car, riêng chiều dài vượt trần 5,3 m khoảng 0,35 m do nới hơi rộng khi bao cụm điểm ở tầm 21,32 m.
- **Góc `yaw_deg`:** Xe cùng chiều phía trước `s01f13`, idx 1 có `yaw_deg = 269,0°`, rất sát 270°. Xe ngược chiều `s02f13`, idx 1 có `yaw_deg = 89,3°`, rất sát 90°.

Mật độ theo khoảng cách trên 4 cuboid `Car`:

| Cuboid | dist_m | Số điểm `points_in` |
|---|---|---|
| `s01f13`, idx 1, `Car` | 5,12 m | 2675 |
| `s02f13`, idx 8, `Car` | 11,60 m | 750 |
| `s01f13`, idx 5, `Car` | 21,32 m | 294 |
| `s05f44`, idx 9, `Car` | 32,68 m | 114 |

- **Quy luật mật độ:** Khi khoảng cách tăng gấp khoảng 2 lần thì số điểm trong cuboid giảm còn khoảng 1/3 đến 2/5, khớp xu hướng đường cong slide bài giảng với trung vị xe con khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m và 19 ở 60–80 m.
- **Tự kiểm sau `preview` và `selfcheck`:** Tôi đã sửa lại độ cao đáy và mép hộp cho 2 cuboid `Car` là `s01f54`, idx 2 và `s03f04`, idx 1, sau đó bấm Save và chạy lại `sync`. Ba cảnh báo được giữ nguyên gồm: `s01f13`, idx 2 `Pedestrian` vì điểm sát mép thuộc vỉa hè liền kề; `s05f44`, idx 1 `Bus` vì đây là xe chở khách cỡ nhỏ ngắn hơn xe buýt chuẩn; và `s05f44`, idx 7 `Car` vì các điểm sát mép ở 29,32 m thuộc mặt đường sát gầm xe.
- **Đổi kiểm:** Làm bài độc lập nên tôi tự đối chiếu toàn bộ số liệu ở mục 5 với `D13-2A202603087-bangA.csv` và `D13-2A202603087-tukiem.csv`, các số liệu đều khớp; chưa đổi kiểm trực tiếp với bạn ngồi cạnh.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **Từ phía model:** Nhãn 3D đo bằng mét thay vì pixel vì kích thước pixel thay đổi theo khoảng cách và góc chiếu camera, trong khi model 3D cần kích thước và vị trí vật lý thật theo mét để ước lượng khoảng cách an toàn và quy hoạch quỹ đạo. Vật thưa điểm cần quy tắc riêng vì đoán bừa khi chỉ còn vài điểm sẽ tạo nhiễu nhãn về tâm, hướng và kích thước, khiến model học sai hình học và tăng báo động giả.
- **Cách dùng AI:**
  - Công cụ: Antigravity trong IDE.
  - Câu hỏi chính: hướng dẫn lệnh Terminal, Docker, cài `cvat-vf-overlay`, giải thích hệ tọa độ VF và góc `yaw`.
- **Điều còn chưa chắc:** Cuboid `s05f44`, idx 1 phân vân `UNCERTAIN_CLASS` giữa minibus `Bus` và van chở người `Car`; cuboid `s03f04`, idx 10 ở 47,89 m chỉ có 23 điểm thuộc nhóm `LOW_VISIBILITY` nên góc yaw chưa thật chắc chắn; hai frame `s01f13` và `s04f51` có 6 và 4 cuboid, thấp hơn mức gợi ý 8 và 5 do ít vật rõ ở gần.
