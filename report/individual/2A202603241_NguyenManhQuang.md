# Báo cáo Day 13 — NguyenManhQuang, 2A202603241

## 1. Kết luận trước
MSSV: `2A202603241`. Task cá nhân: `D13-2A202603241` tại `http://localhost:8080/tasks/18`; task chung vòng hiệu chuẩn: `D13-HC485087` tại `http://localhost:8080/tasks/19`. Mười frame: `s01f25`, `s01f49`, `s02f29`, `s02f54`, `s03f01`, `s03f57`, `s04f04`, `s04f58`, `s05f02`, `s05f42`. Hệ điều hành: Windows; trình duyệt: Chrome hoặc Edge; tài khoản CVAT: `quangkebg123`; phiên bản server CVAT: `2.81.0`. MSSV người ngồi cạnh: chưa ghi do chưa đổi kiểm trong phiên hiện tại.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-<frame>-3d`, `t1-<frame>-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

Frame `s01f25`: ảnh `t1-s01f25-3d.jpg`, `t1-s01f25-top.jpg`. Frame `s01f49`: ảnh `t1-s01f49-3d.jpg`, `t1-s01f49-top.jpg`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f25 | Trong ảnh top, phía trước là cạnh trên theo −y; +x chỉ sang trái; +y chỉ xuống cạnh sau. | Trung vị điểm mặt đường khoảng −1,699 m; đường tương đối phẳng, P10/P90 trong vùng phân tích là −1,917/-1,384 m. | Khoảng 196 m theo điểm đo xa nhất trong LiDAR; vật nhận dạng rõ nên dùng dưới 50 m. | Xe con, xe tải/buýt, người đi bộ và xe hai bánh; phân loại theo cụm điểm và ảnh camera. | Kiểm hai bên bằng top, phía sau bằng cam09, hai phía xéo bằng cam05/cam08 và cam06/cam07. | Cụm điểm xa hoặc bị che có ít điểm; chưa đủ cơ sở để xác định class thì bỏ qua. |
| s01f49 | Phía trước ở cạnh trên theo −y; +x sang trái, +y xuống cạnh sau. | Trung vị điểm mặt đường khoảng −1,751 m; P10/P90 là −1,846/-1,287 m. | Khoảng 199 m theo điểm LiDAR xa nhất; đối chiếu vật mục tiêu ưu tiên trong 50 m. | Xe con và các loại xe lớn gần/trung tâm, người đi bộ, xe hai bánh; cần cam01/cam03/cam06–09 đối chiếu. | Vật sau và hai bên không suy chỉ từ cam01; dùng top rồi chọn cam05–09 theo hướng. | Cụm điểm ở xa hoặc góc mù có thể bị phản xạ ít điểm; giữ `LOW_VISIBILITY` nếu không đủ thể tích. |

Ba điều chỉ camera cho biết trên hai frame: màu và kiểu xe/car/Truck/Bus, biển hiệu hoặc dấu hiệu đầu–đuôi để xác định yaw, hướng nhìn hoặc hướng đi của người đi bộ/xe hai bánh. Ba điều chỉ LiDAR cho biết: hình khối 3D, khoảng cách/cận xa và hệ tọa độ mét, đáy vật chạm mặt đường, cũng như vật bị che hiện thành cụm điểm ở phần còn lại.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s01f49 | Cạnh trên là −y; đã đọc kỹ ở bảng trên | Cụm điểm xa và phân tán |
| s02f29 | Cạnh trên là −y trong hệ VF | Xem top trước, đối chiếu camera theo hướng |
| s02f54 | Cạnh trên là −y trong hệ VF | Kiểm hai bên và phía sau, không nhìn chỉ bằng cam01 |
| s03f01 | Cạnh trên là −y trong hệ VF | Có thể có vật ở góc mù camera |
| s03f57 | Cạnh trên là −y trong hệ VF | Ưu tiên cụm điểm rõ trong 30–50 m |
| s04f04 | Cạnh trên là −y trong hệ VF | Khó phân loại thì ghi UNCERTAIN_CLASS |
| s04f58 | Cạnh trên là −y trong hệ VF | Bóng che hoặc cụm điểm bị che |
| s05f02 | Cạnh trên là −y trong hệ VF | Điểm lạ không gán thành vật |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f25, idx 5 | Car | Có, yaw 268.6° gần 270° | Có, z_bottom -1.612 m lệch 0.088 m so mặt đường | Đã kiểm trên Top/Side/Front và ảnh camera | Dài 4.550 m, rộng 1.700 m, cao 1.432 m; thuộc sedan/hatchback |
| (b) xe buýt hoặc xe tải | s05f42, idx 2 | Bus | Có, yaw 276.6° gần hướng phía trước | Có, z_bottom -1.756 m lệch 0.056 m | Đã kiểm Top/Side/Front và 7 camera | Dài 5.041 m, rộng 1.745 m, cao 1.958 m; nhỏ hơn xe buýt thường nhưng có thể do chỉ một phần nhìn thấy |
| (c) người đi bộ hoặc xe hai bánh | s03f57, idx 8 | Cyclist | Hướng đã đối chiếu, yaw 283.3° | z_bottom -1.335 m lệch 0.365 m; giữ để xem lại do vật thưa | Đã kiểm Top/Side/Front và ảnh camera | Dài 0.595 m, rộng 0.590 m, cao 1.000 m; nhỏ hơn bảng tham chiếu, có cảnh báo |

Vòng hiệu chuẩn chưa ghi chi tiết phần lệch giữa các cuboid trong báo cáo vì không còn dữ liệu ghi nhận riêng. Các vật bỏ qua: những cụm rất thưa hoặc không đủ hình dạng để phân class đã bỏ, ghi dạng `UNCERTAIN_CLASS` / `LOW_VISIBILITY` khi cần; không liệt kê thành vật cụ thể.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f00` — `t4-1-bong-che.png` — task chung `D13-HC485087` | Xe buýt đỏ sát camera có thể che vùng phía sau vật theo hướng quét; vùng sau xe buýt là vùng chưa quan sát được chứ không nhất thiết trống do vật bị khử điểm. | Xe buýt gần tạo cụm điểm dày, phía sau xe gần như không có điểm trong khi hai bên góc che vẫn có điểm. | Cần bật `occluded` và suy luận cuboid bao phần vật còn che được, không thu nhỏ hộp chỉ theo phần điểm nhìn thấy. |
| 2. Thưa ở xa | `s04f58`, cuboid idx 4 — `t4-2-thua-o-xa.png` — task cá nhân `D13-2A202603241` | Khoảng cách xa làm ít tia LiDAR chạm xe, nên vật có thể chỉ còn một phần nhỏ hoặc vài cụm điểm. | Cuboid `Car` cách khoảng `45,19 m` (`x=-3.844`, `y=-45.025 m`), thuộc vùng 30–50 m và có ít điểm hơn các xe gần. | Chỉ vẽ nếu đánh giá còn khoảng 15–20% thể tích hoặc suy luận hình khối đủ cơ sở; không đủ thì bỏ qua và ghi `LOW_VISIBILITY`. |
| 3. Phản xạ thấp / ít điểm bất thường | `s04f02` hoặc `s02f00` — `t4-3-phan-xa-thap.png` — task chung `D13-HC485087` | Cùng khoảng cách nhưng vật tối màu, kính hoặc mặt phản xạ kém có thể trả ít điểm; đây chỉ là giả thuyết vì intensity không phải căn cứ khẳng định nguyên nhân. | Vật có cụm điểm thưa hoặc không đều so với xe cùng tầm; cần đối chiếu ảnh camera để xem màu, kiểu vật và phần bị che. | Không dùng số điểm hay quầng sáng để tự động đổi kích thước; giữ cuboid sát điểm và hình học đã suy ra, ghi giả thuyết thay vì khẳng định. |
| 4. Điểm lạ | `s02f35` — `t4-4-diem-la.png` — task chung `D13-HC485087` | Cụm điểm có thể do sai hình học/quét hoặc nhiễu, không thể xác định nguyên nhân chỉ từ một khung nhìn. | Cụm khoảng 46 điểm ở phía sau bên trái, cách khoảng 29 m và ở `z ≈ -13,5 m`, thấp hơn mặt đường khoảng 11 m; cụm nằm dưới mặt đường. | Không gán cụm này thành bất kỳ vật nào vì điểm dưới mặt đường không khớp vật có hình học hợp lệ. |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-<MÃ SỐ>-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f25, idx 2, Car | -1.000 | 1.400 | -1.700 | 12.78 | 14.04 | 347.0 | Chạm đường sau sửa |
| s02f29, idx 5, Car | -0.845 | 1.710 | -1.700 | 18.08 | 18.11 | 277.5 | Chạm đường sau sửa |
| s05f42, idx 7, Car | -0.932 | 1.536 | -1.700 | 7.29 | 7.35 | 270.4 | Chạm đường |

(a) Cuboid **gần nhất** là `s05f42, idx 6, Car`: `dist_m = 3,69 m`, `r = √(3,109²+1,986²+(-0,761)²) = 3,77 m`. Cuboid **xa nhất** là `s02f29, idx 8, Car`: `dist_m = 45,22 m`, `r = √(0,717²+(-45,211)²+(-1,078)²) = 45,23 m`.

(b) Cyclist `s03f57, idx 8` có `l = 0,595 m`, `w = 0,590 m`, `h = 1,000 m`. So bảng 4.3, chiều dài nhỏ hơn khoảng xe đạp hoặc xe máy có người (1,6–2,3 m), chiều cao thấp hơn khoảng tham chiếu (1,4–1,9 m); có thể do hộp vẫn bao không đủ xe hoặc vật bị che. Một xe con hợp lý hơn là `s01f25, idx 2`: `l = 4,870 m`, `w = 1,686 m`, `h = 1,400 m`, thuộc kích thước sedan/hatchback tham chiếu (dài 3,8–5,0 m, rộng 1,70–1,90 m, cao 1,40–1,60 m).

(c) Xe cùng chiều phía trước dùng `s01f25, idx 9, Car` với `yaw_deg = 267,4°`, gần 270°. Xe ngược chiều dùng `s01f25, idx 4, Car` với `yaw_deg = 91,1°`, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f25, idx 5, Car | 9.15 | 1421 |
| s01f25, idx 9, Car | 18.87 | 460 |
| s01f25, idx 1, Car | 24.29 | 319 |
| s02f29, idx 8, Car | 45.22 | 40 |

Số điểm giảm rõ khi khoảng cách tăng: ở 9,15 m có 1421 điểm, 18,87 m còn 460 điểm, 24,29 m còn 319 điểm và 45,22 m còn 40 điểm. Quy luật này phù hợp xu hướng đường cong trong slide: xe con trung vị giảm mạnh theo khoảng cách, nhưng tại nhóm dữ liệu này số điểm không trùng tuyệt đối do góc nhìn và che khuất.

Sau `preview/selfcheck`, đã sửa hai cuboid: `s01f25, idx 2, Car` chỉnh chiều cao từ 1,034 m lên 1,400 m và tâm z để `z_bottom = -1,700 m`; `s02f29, idx 5, Car` đưa tâm z xuống `-0,845 m` để `z_bottom = -1,700 m`. Sau khi `sync`, số cảnh báo giảm từ 18 còn 16.

Các cảnh báo giữ nguyên gồm ít nhất: `s01f25, idx 7, Car` bị báo hộp rỗng; đã xem ảnh preview nhưng không đủ cơ sở xác định vật để đoán thêm. `s03f57, idx 8, Cyclist` dài 0,595 m và có 20 điểm sát mép; giữ nguyên vì cảnh báo có thể do vật thưa hoặc cách nhìn, chưa có bằng chứng chắc chắn nới/thu hộp. Các cảnh báo kích thước Bus/xe to ở frame xa chỉ là gợi ý; đã giữ khi preview cho thấy vật nằm trong hộp và chưa xác định sai class.

Đổi kiểm: chưa đổi kiểm với bạn bên cạnh trong phiên làm bài này; hai con số dự kiến đối chiếu là `dist_m` gần nhất/xa nhất và `z_bottom` ba cuboid khi thực hiện đổi kiểm.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
Nhãn 3D đo bằng mét vì model cần học kích thước vật, khoảng cách và vị trí trong không gian thực; tọa độ pixel chỉ phụ thuộc góc camera và không thể hiện trực tiếp độ sâu hay kích thước 3D. Vật thưa điểm cần quy tắc riêng vì số điểm nhận được giảm theo khoảng cách và vật liệu/che khuất; chỉ vẽ khi còn khoảng 15–20% thể tích hoặc có đủ cơ sở suy luận, tránh tạo hộp tự ý từ quầng điểm trống.

Dùng AI: công cụ là GitHub Copilot trong VS Code; câu hỏi chính là cách điền Bước 4–5 theo mẫu, tính số liệu từ Bảng A và xử lý cảnh báo. Kết quả AI được đối chiếu với file CSV, guideline và ảnh preview trước khi ghi vào báo cáo; không sao chép số chưa xác minh.

Điều còn chưa chắc: một số cuboid ở xa hoặc bị che có thể vừa thưa điểm vừa sai kích thước nhẹ, và việc phân loại một số xe trong frame chưa chắc chắn. Lỗi phát hiện sau hạn sẽ được ghi lại và sửa cuboid trong CVAT trước deadline nếu cần.
