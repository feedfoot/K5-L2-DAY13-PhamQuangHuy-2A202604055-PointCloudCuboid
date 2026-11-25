# Báo cáo Day 13 — Phạm Quang Huy, 2A202604055

## 1. Kết luận trước
Mã số `2A202604055`, task `D13-2A202604055`, 10 frame: `s01f19 s01f56 s02f25 s02f41 s03f08 s03f24 s04f40 s04f58 s05f23 s05f41`. Windows 11, trình duyệt Chrome, CVAT 2.81.0. MSSV ngồi cạnh: 2A202604043.

Tổng **54 cuboid** (50 Car, 2 Bus, 1 Truck, 1 Cyclist). Ít nhất là 5 cuboid ở mỗi frame từ `s02f25` đến `s05f41` (8 frame), vì tôi chỉ vẽ đủ mức tối thiểu để kịp thời gian; các frame vẫn còn vật chưa vẽ. Có 3 hộp xa hơn 50 m (`s01f19` #8, `s03f24` #4 và #5). `selfcheck` cờ **6/54** cuboid. Phần tin nhất là đáy: 49/50 hộp đo được khoảng hở với mặt đường trong khoảng −0,02 đến 0,23 m. Phần yếu nhất là bề rộng (16/50 xe con rộng hơn 2,10 m), class của `s01f19` #2 và các vật ở xa.

Cách dùng CVAT:
- Góc nhìn: kéo chuột trái để xoay, chuột phải để dời, con lăn để phóng; xoay nhìn thẳng xuống để xem toàn cảnh.
- Vẽ: nút cuboid → label → Shape → nháy đúp để đặt; Top chỉnh tâm, dài, rộng, yaw; Side và Front chỉnh đáy, chiều cao.
- Bật Appearance → Cuboid orientation (vạch đỏ chỉ đầu xe); xem cuboid trên ảnh camera bằng `python scripts/lab.py preview`.

## 2. Đọc frame (Bước 2)
Ảnh: `t1-s01f19-3d`, `t1-s01f19-top`, `t1-s01f56-3d`, `t1-s01f56-top`. Hệ VF: x sang trái, y ra sau, phía trước xe là −y.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f19` | Trước: cạnh trái; +y sang phải, +x xuống dưới | Dốc rất nhẹ, thấp dần về phía trước; đáy các hộp đã gán ở z −1,58 đến −2,51 m | 53,5 m (hộp xa nhất đã gán) | Xe con, xe buýt | Car sau-phải ~9,9 m (#3); Car sau-trái ~14,3 m (#4); Car phía sau ~30 m (#7) | Các cụm rải rác ngoài dải cây bên trái, xa hơn khoảng 25 m |
| `s01f56` | Trước: cạnh trái; +y sang phải, +x xuống dưới | Gần phẳng; đáy các hộp đã gán ở z −1,58 đến −1,83 m | 16,8 m (hộp xa nhất đã gán) | Xe con | Car sau-trái ~4,7 m (#3); Car phía sau ~6,7 m (#4) | Các cụm nhỏ phía trước sau hàng xe, khoảng 25–35 m |

Cạnh phía trước ghi theo góc xem mặc định của CVAT (câu trả lời của AI); tôi chưa đối chiếu lại góc xoay của ảnh top.

Chỉ camera cho biết: (1) màu sắc của xe; (2) đâu là đầu, đuôi xe và kiểu xe qua đèn, kính, hình dáng; (3) màu và nội dung biển báo, đèn tín hiệu. Ví dụ ở cả hai frame: biển chỉ làn màu xanh có mũi tên trên giá long môn.

Chỉ LiDAR cho biết trực tiếp: (1) khoảng cách tới vật theo mét; (2) kích thước dài, rộng, cao theo mét; (3) vị trí quanh 360°, kể cả góc mù của camera. Ví dụ: ở `s01f56`, xe #3 cách 4,66 m và dài 4,58 m.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f25` | Cạnh trái | Rào chia đôi đường, có xe chạy ngược chiều |
| `s02f41` | Cạnh trái | Hộp #3 gần vuông góc với hướng đường; vài điểm nằm sâu dưới mặt đường |
| `s03f08` | Cạnh trái | Kẹt xe, tường quây hai bên, đám điểm thành hành lang hẹp |
| `s03f24` | Cạnh trái | Xe buýt ở khoảng 74 m và xe con ở khoảng 53 m rất thưa điểm |
| `s04f40` | Cạnh trái | Rào vàng và tường quây xanh; không thấy gì lạ rõ |
| `s04f58` | Cạnh trái | Nhiều người đi xe hai bánh bên kia rào, mới gán 1 |
| `s05f23` | Cạnh trái | Ngã tư, người đi xe máy ở vạch qua đường chưa gán; vài điểm sâu dưới mặt đường |
| `s05f41` | Cạnh trái | Xe buýt sát bên trái che khuất; nhiều điểm dưới mặt đường ở phía trước bên trái |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f19` #1 | Car | Đúng, hướng về trước (268,9°) | Có: z đáy −1,86 m, khoảng hở −0,02 m | Có | 4,51 × 1,94 × 1,58 m: dài trong khoảng; rộng hơn mức sedan 0,04 m |
| (b) xe buýt hoặc xe tải | `s01f19` #2 | Truck (camera cho thấy xe buýt: nhãn sai, chưa sửa) | Đúng, hướng về trước (268,4°) | Có so với đường quanh hộp: hở 0,02 m (z đáy −2,06 m) | Có | 12,47 × 2,81 × 3,08 m: so với xe buýt thường, dài và cao trong khoảng, rộng vượt 0,21 m |
| (c) người đi bộ hoặc xe hai bánh | `s04f58` #4 | Cyclist | Hướng về sau (89,5°), khớp chiều đi ở làn bên kia; độ chắc vừa | Có: z đáy −1,67 m, khoảng hở 0,04 m | Có | 1,90 × 0,79 × 1,80 m: so với xe đạp có người, dài và cao trong khoảng, rộng vượt 0,09 m |

Ghi chú:tôi đã kiểm lại ba hộp trên Top/Side/Front và ảnh camera.

Vòng hiệu chuẩn: tôi chưa xuất bảng A của vòng hiệu chuẩn, nên không có kết quả so sánh.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân dưới đây đều là giả thuyết.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s05f41`; `t4-1-bong-che`; frame của tôi | Giả thuyết: xe buýt chắn tia LiDAR nên phía sau nó ít điểm trả về | Xe buýt sát bên trái; vùng phía ngoài nó rất ít điểm | Vùng trống là chưa quan sát được; vật bị che thì bật occluded, hộp bao cả phần suy luận được |
| 2. Thưa ở xa | `s03f24`; `t4-2-thua-o-xa`; frame của tôi | Giả thuyết: các tia toả rộng theo khoảng cách nên ít điểm rơi trên vật | Xe buýt #4 ở 74,04 m chỉ có 78 điểm, trong khi xe con `s04f58` #3 ở 8,61 m có 2951 điểm | Vẽ nếu thấy khoảng 15–20% thể tích hoặc đủ cơ sở; không thì bỏ qua, ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f08`; `t4-3-phan-xa-thap`; frame của tôi | Giả thuyết: sơn tối hoặc kính phản xạ yếu hay lệch tia | Sedan đen #2 có intensity thấp hơn nhẹ so với xe #1, số điểm không giảm rõ: bằng chứng yếu | Đối chiếu camera, không khẳng định nguyên nhân, không đoán khi thiếu cơ sở |
| 4. Điểm lạ | `s02f41`; `t4-4-diem-la`; frame của tôi | Giả thuyết: tia phản xạ nhiều lần tạo điểm ảo | Một số điểm nằm sâu dưới mặt đường, phía trước hơi lệch phải, khoảng 29 m | Không gán thành vật |

tôi đã đối chiếu lại với bốn ảnh t4.

## 5. Số từ cuboid của tôi (Bước 5)

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f19` #1 Car | −1,072 | 1,582 | −1,863 | 4,89 | 5,00 | 268,9° | Lệch mốc −1,7 m là 0,16 m; khoảng hở −0,02 m: chạm đường |
| `s01f19` #5 Car | −1,273 | 1,655 | −2,101 | 16,17 | 16,22 | 91,1° | Thấp hơn mốc −1,7 m 0,40 m, nhưng cao hơn đường quanh hộp 0,21 m; giả thuyết: đường chỗ này thấp hơn |
| `s03f24` #3 Car | −0,943 | 1,544 | −1,715 | 16,08 | 16,10 | 89,8° | Sát mốc −1,7 m, nhưng thấp hơn đường quanh hộp 0,52 m: còn bị cờ chìm |

(a) Gần nhất: `s03f08` #1, dist_m 4,19 m, r 4,31 m. Xa nhất: `s03f24` #4 (Bus), dist_m 74,04 m, r 74,05 m (ngoài 50 m).

(b) Xe hai bánh `s04f58` #4: l 1,898 m, w 0,792 m, h 1,803 m; so với xe đạp có người (1,6–1,9 / 0,50–0,70 / 1,60–1,90 m) chỉ chiều rộng vượt 0,092 m; giả thuyết: nhiều người đi sát nhau nên hộp bị rộng. Xe con `s01f19` #1: 4,511 × 1,935 × 1,582 m; so với sedan (3,8–5,0 / 1,70–1,90 / 1,40–1,60 m) chỉ chiều rộng vượt 0,035 m; giả thuyết: hộp ôm cả gương hoặc kéo cạnh hơi rộng.

(c) Xe cùng chiều phía trước `s05f23` #4: yaw 267,2° (gần 270°). Xe quay ngược chiều `s02f25` #2: yaw 86,2° (gần 90°).

Mật độ theo khoảng cách:

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `s01f19` #3 Car | 9,85 | 1500 |
| `s01f19` #4 Car | 14,29 | 720 |
| `s02f41` #1 Car | 23,67 | 214 |
| `s02f41` #4 Car | 37,54 | 79 |

Quy luật: xa gấp khoảng 3,8 lần (9,85 → 37,54 m) thì số điểm còn khoảng 1/19 (1500 → 79). Trung vị xe con của tôi: 1907 (0–10 m), khoảng 716 (10–20 m), khoảng 247 (20–30 m), 104 (30–40 m), gần với slide (1968, 718, 214, 72).

Hai cuboid đã sửa: `s03f24` #2, bề rộng 2,49 → 2,11 m (hết cờ); `s03f24` #3, chỉnh đáy, độ chìm theo selfcheck 0,71 → 0,52 m (vẫn còn cờ). Hai thay đổi này suy ra từ hai lần chạy selfcheck.


Đổi kiểm: người ngồi cạnh là 2A202604043
## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model học nhãn này phải ước lượng vị trí và kích thước thật của vật để xe tính khoảng cách, nên nhãn đo bằng mét, còn kích thước theo pixel thay đổi theo khoảng cách và từng camera. Vật thưa điểm cho rất ít bằng chứng, nên cần quy tắc chung để model không học nhãn đoán mò, mỗi người một kiểu.

(b) Có dùng AI:
- Công cụ: Claude Code (hai phiên).
- Câu hỏi chính: cài môi trường, lệnh PowerShell và Git, cách dùng CVAT, quy tắc guideline; soạn báo cáo từ bảng A và phiếu tự kiểm; một phiên khác xem dữ liệu trên CVAT local (point cloud, 7 camera) để quan sát, đo và trả lời các câu hỏi cho mục 2–5.

