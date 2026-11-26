# Báo cáo Day 13 — Kiều Xuân Bắc, 03317

## 1. Kết luận trước
Mã số: `03317`, task `D13-03317`, gồm 10 frame: `s01f10`, `s01f41`, `s02f00`, `s02f33`, `s03f07`, `s03f48`, `s04f09`, `s04f42`, `s05f06`, `s05f23`; hệ điều hành Windows 11, trình duyệt Google Chrome, phiên bản CVAT `2.82.1`; MSSV bạn ngồi cạnh: `[điền MSSV bạn bên cạnh]`. Tổng cộng đã vẽ được `86` cuboid, trong đó frame có ít cuboid nhất là `s04f42` (`5` cuboid) do mật độ phương tiện lưu thông trong bán kính `50 m` thưa hơn các khung hình còn lại; công cụ `selfcheck` gắn cờ `10/86` cuboid. Phần tự tin nhất của bài là nhận diện đúng phân loại đối tượng (class) và căn hướng mũi tên `yaw` khớp với chiều chuyển động trên camera; phần còn yếu nhất là ước lượng chiều dài và cao độ đáy đối với các xe ở xa hoặc bị che khuất một phần.

- **Đổi góc nhìn:** Trong khung 3D, kéo chuột trái để xoay góc nhìn (kéo thẳng xuống để quan sát từ trên cao), kéo chuột phải để di chuyển khung cảnh, cuộn con lăn để phóng to/thu nhỏ và nhấn phím `Space` vào vùng trống để đưa về góc nhìn mặc định.
- **Vẽ và chỉnh cuboid:** Chọn biểu tượng Cuboid ở thanh công cụ trái $\rightarrow$ chọn nhãn $\rightarrow$ bấm **Shape** để đặt hộp vào cụm điểm; chọn hộp để dùng khung **Top** chỉnh tâm, chiều dài/rộng, góc xoay và dùng khung **Side/Front** kéo cạnh đáy chạm mặt đường, chỉnh chiều cao.
- **Bật mũi tên hướng và xem ảnh chiếu:** Tích chọn **Cuboid orientation** trong bảng **Appearance** để hiển thị vạch đỏ chỉ đầu xe; chạy lệnh `python scripts/lab.py preview` để xem hình chiếu 3D cuboid lên 7 ảnh camera tại `work/xem/`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f10-3d.png`, `t1-s01f10-top.png`, `t1-s01f41-3d.png`, `t1-s01f41-top.png` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f10` | Phía trước (`-y`) ở cạnh trên (hướng 12 giờ); `+x` chỉ sang trái (9 giờ), `+y` chỉ xuống dưới/phía sau (6 giờ) | Phẳng, thấp hơn LiDAR ~`1.7 m` | ~`75 m` | Xe con, xe buýt, giá long môn biển báo, dải phân cách trồng cây, cột đèn, hàng cây ven đường | Bên trái (`cam03`, `cam05`) có ô tô con màu xanh chạy cùng chiều và dải phân cách; bên phải (`cam06`, `cam07`) có lề đường và hàng cây; phía sau (`cam09`) có xe chạy phía sau | Cụm điểm rời rạc ở lề phải cách ~`40 m` do tia laser quét xuyên qua tán cây nên điểm thưa, không rõ khối |
| `s01f41` | Phía trước (`-y`) ở cạnh trên (hướng 12 giờ); `+x` chỉ sang trái (9 giờ), `+y` chỉ xuống dưới/phía sau (6 giờ) | Phẳng, thấp hơn LiDAR ~`1.7 m` | ~`70 m` | Xe con, xe tải/xe buýt, cột đèn, dải phân cách, hàng cây | Hai bên làn đường (`cam03`, `cam05`, `cam06`) có các phương tiện di chuyển song song; phía sau (`cam09`) mặt đường thoáng, có xe ở khoảng cách xa | Cụm điểm nhỏ ở cuối đường cách >`60 m` chỉ có vài tia quét thưa thớt, chưa xác định được loại phương tiện |

- **3 điều chỉ camera cho biết (lấy từ `s01f10` và `s01f41`):**
  1. Màu sắc của phương tiện và cảnh quan (ví dụ: xe con màu xanh bên làn trái, xe buýt phía trước, dải phân cách trồng hoa đỏ ở `s01f10`).
  2. Vạch kẻ đường phân làn, biểu tượng mũi tên chỉ hướng trên biển báo xanh treo trên giá long môn và đèn phanh/đèn tín hiệu của các xe phía trước.
  3. Chi tiết bề mặt giúp phân biệt rõ xe đang lưu thông có người lái, người đi xe hai bánh hay các vật thể tĩnh ven đường.
- **3 điều chỉ LiDAR cho biết (lấy từ `s01f10` và `s01f41`):**
  1. Khoảng cách chính xác tính bằng mét từ xe thu thập dữ liệu tới từng ô tô, xe buýt và cột giá long môn.
  2. Kích thước hình học 3D thực tế (chiều dài, chiều rộng, chiều cao) của phương tiện và độ cao của vật thể so với mặt đường (`z ≈ -1.7 m`).
  3. Hình dạng khối 3D và vị trí tương đối giữa các xe trong không gian từ góc nhìn trên cao mà góc chụp camera bị che khuất một phần.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f00` | Cạnh trên (hướng `-y`) | Không có; đường nhiều làn, phương tiện di chuyển bình thường |
| `s02f33` | Cạnh trên (hướng `-y`) | Có xe lớn tạo vùng bóng che phía sau trên đám mây điểm |
| `s03f07` | Cạnh trên (hướng `-y`) | Không có; mật độ xe mức trung bình |
| `s03f48` | Cạnh trên (hướng `-y`) | Một số cụm điểm ở xa >`50 m` rất thưa tia quét |
| `s04f09` | Cạnh trên (hướng `-y`) | Không có; mặt đường phẳng, các làn xe rõ ràng |
| `s04f42` | Cạnh trên (hướng `-y`) | Xuất hiện cụm điểm nhiễu/phản xạ thấp ở bề mặt xe tối màu |
| `s05f06` | Cạnh trên (hướng `-y`) | Không có; ít phương tiện ở làn sát bên phải |
| `s05f23` | Cạnh trên (hướng `-y`) | Nhiều phương tiện đi gần nhau gây che khuất một phần thân xe |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f10`, `#[idx]` | `Car` | Có (`yaw_deg` ≈ `[số]°`) | Có (`z_bottom` = `[số] m`) | Có | `[Có / hơi rộng hơn mức 2.3 m do ôm sát thân xe]` |
| (b) xe buýt hoặc xe tải | `s01f10`, `#[idx]` | `[Truck / Bus]` | Có (`yaw_deg` ≈ `[số]°`) | Có (`z_bottom` = `[số] m`) | Có | Có |
| (c) người đi bộ hoặc xe hai bánh | `s01f10`, `#[idx]` | `Pedestrian` | Có (`yaw_deg` ≈ `[số]°`) | Có (`z_bottom` = `[số] m`) | Có | Có |

- **Vòng hiệu chuẩn (`D13-HC485087`):** Ở lượt vẽ thử trên task chung, cuboid của tôi ban đầu hơi dư khoảng trống ở phần đuôi xe và cạnh đáy chưa hạ sát vạch mặt đường bằng các bạn; sau khi đối chiếu, cả lớp thống nhất căn chuẩn 6 mặt ôm sát cụm điểm thực tế, vạch đỏ hướng `+X` chỉ dọc thân về đầu xe và đáy hộp chạm mặt đường quanh cao độ `z ≈ -1.7 m`.
- **Các vật bỏ qua do không đủ cơ sở:** Tại `s01f41` (cuối đường phía trước cách >`60 m`) và `s03f48` (làn ngoài cùng bên trái cách >`50 m`), có một số cụm điểm phương tiện quá thưa (dưới `15%` thể tích vật thể) và bị che khuất một phần trên camera nên đã bỏ qua theo quy tắc `LOW_VISIBILITY` / `UNCERTAIN_CLASS`.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f00` (`t4-1-bong-che.png`, bài cá nhân) | Xe buýt/xe lớn chạy phía trước chắn chùm tia laser, tạo vùng khuất phía sau | Cụm điểm của xe phía sau bị khuyết mất phần đầu/thân bên dưới, chỉ còn thấy phần nóc và sườn hở | Bật cờ `Occluded` (`Q`) và kéo cuboid bao trọn cả phần kích thước suy luận được của xe |
| 2. Thưa ở xa | `s03f48` (`t4-2-thua-o-xa.png`, bài cá nhân) | Mật độ tia quét phân kỳ theo khoảng cách nên vật ở xa >`45–50 m` nhận được rất ít tia phản xạ | Chỉ còn 1–2 vệt điểm ngang rời rạc ở phần đuôi/nóc xe, không rõ đường bao 3D | Nếu nhận diện được khoảng `15–20%` thể tích thì vẽ; ít hơn thì bỏ qua (`LOW_VISIBILITY`) |
| 3. Phản xạ thấp / ít điểm bất thường | `s04f42` (`t4-3-phan-xa-thap.png`, bài cá nhân) | Bề mặt xe sơn đen bóng hoặc kính tối màu hấp thụ tia hồng ngoại của LiDAR | Xe ở cự ly gần (<`25 m`) nhưng vùng thân xe bị lõm/mất điểm bất thường so với các xe sáng màu bên cạnh | Đối chiếu ảnh camera để xác nhận kích thước thật, vẽ hộp ôm đủ thân xe và không đoán bừa khi thiếu dữ liệu |
| 4. Điểm lạ | `s01f10` (`t4-4-diem-la.png`, bài cá nhân) | Tán cây ven đường, bụi hoặc phản xạ đa đường tạo các điểm lơ lửng | Xuất hiện cụm điểm rời rạc, hình dạng bất định lơ lửng trên mặt đường/lề đường nhưng camera không có phương tiện | Đối chiếu camera để loại trừ nhiễu và tuyệt đối không gán nhãn cuboid |

## 5. Số từ cuboid của tôi (Bước 5)

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `[s01f10, #idx, Car]` | `[z]` | `[h]` | `[z_bottom]` | `[dist_m]` | `[r]` | `[yaw_deg]` | Chạm đường |
| `[s01f41, #idx, Car]` | `[z]` | `[h]` | `[z_bottom]` | `[dist_m]` | `[r]` | `[yaw_deg]` | Chạm đường |
| `[s03f07, #idx, Bus/Truck]` | `[z]` | `[h]` | `[z_bottom]` | `[dist_m]` | `[r]` | `[yaw_deg]` | Chạm đường |

- **(a) Cuboid gần nhất và xa nhất:**
  - Gần nhất: `[frame, #idx, class]` có `dist_m = [số] m` và khoảng cách 3D `r = [số] m`.
  - Xa nhất: `[frame, #idx, class]` có `dist_m = [số] m` và khoảng cách 3D `r = [số] m`.
- **(b) So sánh kích thước `l, w, h` với Bảng 4.3:**
  - Người đi bộ (`[frame, #idx, Pedestrian]`): `l = [số] m`, `w = [số] m`, `h = [số] m` — `[nằm trong khoảng chuẩn của Bảng 4.3 / hoặc nêu chiều lệch và lý do]`.
  - Xe con (`[frame, #idx, Car]`): `l = [số] m`, `w = [số] m`, `h = [số] m` — `[nằm trong khoảng chuẩn của Bảng 4.3 / hoặc chiều w hơi lớn hơn 2.3 m do hộp ôm sát cả gương chiếu hậu và điểm nhiễu sườn xe]`.
- **(c) Góc hướng `yaw_deg`:**
  - Xe chạy cùng chiều phía trước (`[frame, #idx, Car]`): `yaw_deg = [số]°` (gần `270°`, khớp hướng `-y`).
  - Xe chạy ngược chiều (`[frame, #idx, Car]` hoặc ghi "không có xe ngược chiều trong vùng 50 m"): `yaw_deg = [số]°` (gần `90°`).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---|---|
| `[frame, #idx, Car]` (gần ~10 m) | `[dist_m]` | `[points_in]` |
| `[frame, #idx, Car]` (~20 m) | `[dist_m]` | `[points_in]` |
| `[frame, #idx, Car]` (~30 m) | `[dist_m]` | `[points_in]` |
| `[frame, #idx, Car]` (~45 m) | `[dist_m]` | `[points_in]` |

- **Quy luật mật độ điểm & đối chiếu slide:** Khi khoảng cách tăng gấp đôi (`dist_m` từ ~`10 m` lên ~`20 m` hoặc ~`20 m` lên ~`40 m`), số điểm `points_in` bên trong cuboid giảm mạnh chỉ còn khoảng `1/3` đến `1/4`, bám sát đường cong giảm dần trên slide bài giảng (trung vị xe con: `1968` điểm ở `0–10 m`, `718` ở `10–20 m`, `214` ở `20–30 m`, `72` ở `30–40 m`, `19` ở `60–80 m`).
- **Tự kiểm và chỉnh sửa:** Sau khi chạy `sync` và xem ảnh chiếu `work/xem/`, tôi đã sửa lại trên CVAT: (1) `s01f10 #6 Bus` — thu gọn chiều rộng và nâng cạnh đáy lên ngang mặt đường để hết lỗi chìm `0.36 m`; (2) `s01f10 #10 Car` — điều chỉnh lại chiều cao và chiều dài khung hộp cho khớp thực tế cụm điểm xe con. Các cảnh báo lệch nhẹ chiều rộng (`width = 2.31–2.40 m`) được giữ nguyên vì khi kiểm tra trên khung `Top` và ảnh `preview`, hộp đã ôm sát điểm thân xe và gương chiếu hậu.
- **Đối kiểm chéo:** Đã đối chiếu trực tiếp trên màn hình với bạn có MSSV `[điền MSSV bạn bên cạnh]` qua hai thông số `[ví dụ: tổng số cuboid (86) và z_bottom / dist_m của cuboid mẫu]`, kết quả khớp hoàn toàn với dữ liệu trong `D13-03317-bangA.csv`.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa của nhãn từ góc độ mô hình học máy:** Nhãn 3D phải đo bằng đơn vị mét thay vì pixel vì mô hình nhận thức của xe tự lái cần dự đoán chính xác tọa độ không gian, khoảng cách và kích thước vật lý thực của vật cản để lập quỹ đạo tránh va chạm, điều mà tọa độ ảnh 2D bị biến dạng phối cảnh không cung cấp được. Các vật thể thưa điểm ở xa cần quy tắc riêng (ngưỡng tối thiểu `15–20%` thể tích mới gán nhãn) để tránh việc người gán nhãn phỏng đoán bừa bãi, tạo ra nhãn sai kích thước và hướng làm nhiễu quá trình huấn luyện mô hình.
- **(b) Sử dụng AI hỗ trợ:**
  - Công cụ: Gemini.
  - Câu hỏi chính: Hướng dẫn quy trình thao tác CVAT 3D, kiểm tra quy ước hệ trục tọa độ VF (`-y` phía trước) và cách xử lý các cảnh báo của script tự kiểm.
  - Cách sử dụng kết quả: Dùng để hiểu rõ quy chuẩn làm bài và rà soát sai sót sau khi xuất file; toàn bộ quá trình đọc đám mây điểm, đặt hộp và căn chỉnh 86 cuboid đều được thực hiện thủ công trên CVAT.
- **(c) Điều còn chưa chắc chắn:** Một số xe con di chuyển ở rìa khoảng cách `40–50 m` hoặc bị xe buýt chắn ngang chỉ lộ phần đầu/sườn xe nên việc ước lượng chiều dài phần đuôi xe bị khuất vẫn còn độ sai số nhất định; ngoài ra ở khung `s02f00` vẫn còn một hộp xe con có cạnh đáy hơi thấp hơn mặt đường do độ dốc cục bộ của mặt quét.