# Báo cáo Day 13 — Ngô Thị Minh Quang, 2A202603146

## 1. Kết luận trước

Tôi thực hiện task `D13-2A202603146` trên Windows, Google Chrome, CVAT 2.74.1. Mười frame: `s01f22`, `s01f52`, `s02f05`, `s02f21`, `s03f18`, `s03f50`, `s04f17`, `s04f45`, `s05f08`, `s05f28`. Bảng A sau `sync` có **84 cuboid**: 73 Car, 4 Bus, 1 Truck, 5 Cyclist, 1 Pedestrian. Hai frame ít hộp nhất là `s03f50`, `s04f45`, mỗi frame 6 hộp. Phiếu tự kiểm có **0/84 cuboid bị cờ**. Tôi tự tin hơn với vị trí và hướng của vật gần; chiều cao xe hai bánh, vật xa và các cụm khó nhận dạng là những phần còn hạn chế.

Tôi kéo chuột trái để xoay khung 3D, chuột phải để dời, con lăn để phóng to/thu nhỏ. Tôi vẽ bằng Shape, nháy đúp đặt hộp, chỉnh tâm/kích thước/yaw ở Top và kiểm đáy ở Side, Front. Tôi bật Cuboid orientation, đối chiếu camera và ảnh `preview` sau `sync` để kiểm hướng hộp.

## 2. Đọc frame (Bước 2)

Ảnh: `t1-s01f22-3d.png`, `t1-s01f22-top.png`, `t1-s01f52-3d.png`, `t1-s01f52-top.png`. Trên hai ảnh Top: **phía trước (−y) sang phải, +y sang trái, +x lên trên**. Gốc ở LiDAR; x trái, y sau, z lên, đơn vị mét.

| Frame | Mặt đường | Vật xa nhất trong các hộp đã gán | Loại vật, phía sau/hai bên | Cụm chưa đọc chắc |
|---|---|---|---|---|
| s01f22 | Mốc tham khảo z ≈ −1,7 m; cần xét đường cục bộ | Car #6, 37,90 m, phía sau | Car, Bus; bus bên trái và xe phía sau cần camera bên/sau | Vùng ít điểm bên kia vật lớn; chưa xác định vật bị che |
| s01f52 | Mốc z ≈ −1,7 m; chưa kết luận độ dốc chỉ từ một góc nhìn | Car #7, 22,10 m, phía sau | Car, Cyclist; xe bên trái, xe hai bánh bên phải | Vệt trước dãy ô tô và cụm vùng xe đạp chưa rõ bản chất |

Camera cho biết màu thân xe, bề mặt kính và người đang đi xe hay đứng/dắt xe. LiDAR cho biết tọa độ theo mét, khoảng cách đến LiDAR và hình học 3D/độ cao đáy. Cam01 không đủ đọc vật phía sau.

Tám frame lướt dưới đây được xem qua ảnh preview. **Trên ô BEV của script**, phía trước đều hướng lên; không mặc định hướng này cho mọi góc nhìn CVAT.

| Frame | Phía trước trên BEV preview | Điểm chú ý |
|---|---|---|
| s02f05 | Trên | Bus #8 ngoài vùng chấm 50 m |
| s02f21 | Trên | Có Cyclist và Pedestrian |
| s03f18 | Trên | Đáy hộp thay đổi theo vị trí, cần xét đường cục bộ |
| s03f50 | Trên | Có xe hai bánh bên phải |
| s04f17 | Trên | Xe ở cả hai chiều và phía sau |
| s04f45 | Trên | Xe xa #6 thưa điểm |
| s05f08 | Trên | Có Truck, cần phân biệt với Car |
| s05f28 | Trên | Hai Car bên trái đã bật occluded |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`. Idx dưới đây theo Bảng A. Tôi đối chiếu ba hình chiếu Top/Side/Front với camera.

| Vật mẫu | Frame, idx, class | Yaw | Đáy | Kích thước l × w × h và bảng 4.3 |
|---|---|---|---|---|
| Xe con gần | s01f52 #2, Car | 268,9°, gần hướng cùng chiều 270° | −1,749 m; ground_gap +0,06 m | 4,748 × 1,934 × 1,539 m; rộng hơn mức sedan 1,90 m khoảng 0,034 m, chưa xác định phần chênh có thuộc gương xe |
| Xe buýt | s01f22 #1, Bus | 270,0° | −1,723 m; gap +0,04 m | 13,029 × 2,827 × 3,151 m; dài/rộng vượt khoảng bus thường 10–12,5 m/2,5–2,6 m |
| Xe hai bánh có người | s02f21 #6, Cyclist | 275,7° | −1,541 m; gap +0,22 m | 1,841 × 0,675 × 1,227 m; chiều cao thấp hơn khoảng xe đạp có người 1,60–1,90 m, khả năng thiếu phần người lái còn chưa chắc |

Tôi chưa có biên bản vòng hiệu chuẩn để khẳng định chênh lệch với cả lớp. Với cụm chưa nhận dạng chắc, tôi ghi LOW_VISIBILITY/UNCERTAIN_CLASS để xem lại, không gọi là vật mới hoặc điểm lạ chỉ vì không nhận ra.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Nguồn các ảnh hiện tại là task cá nhân. Nguyên nhân được ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh | Nguyên nhân (giả thuyết), quan sát | Hệ quả theo guideline |
|---|---|---|---|
| 1. Bóng che | s05f28; `t4-1-bong-che.png` | Vật/cây trên đường ngắm có thể che một phần xe; Car #1, #2 đã bật occluded | Bao phần suy luận được, không chỉ bao điểm còn thấy; kiểm camera |
| 2. Thưa ở xa | s04f45; `t4-2-thua-o-xa.png` | Car #6 ở 47,28 m, có 44 điểm trong hộp; khoảng cách lớn có thể làm giảm số tia trúng vật | Vẽ nếu thấy khoảng 15–20% thể tích hoặc có cơ sở suy luận; không đủ thì bỏ qua, ghi LOW_VISIBILITY |
| 3. Phản xạ thấp/ít điểm bất thường | s01f52, Car #1 (thẻ CVAT 8); `t4-3-phan-xa-thap.png` | Cam06 xác nhận xe sedan đen; Side/Front cho thấy vùng phía trên thưa điểm hơn phần dưới. Giả thuyết: kính có đáp ứng LiDAR khác phần thân; góc tới cũng có thể góp phần, chưa đủ để tách riêng nguyên nhân | Dựa vào điểm thân xe và ảnh camera để suy luận kích thước đầy đủ; không co hộp theo khoảng thiếu điểm ở vùng kính |
| 4. Nghi vấn điểm lạ | s01f52; `t4-4-diem-la.png` | Cụm điểm gần khu vực xe đạp chưa phân biệt được là xe, người hay cây. Giả thuyết: điểm từ các vật gần nhau chồng trong góc nhìn; chưa xác nhận là outlier | Tôi không gán thêm cuboid cho cụm chưa xác định được vật thật |

## 5. Số từ cuboid của tôi (Bước 5)

Nguồn: `D13-2A202603146-bangA.csv`, `D13-2A202603146-tukiem.csv` sau `sync`. Khoảng cách 3D r = √(x² + y² + z²); số có sai số làm tròn theo CSV.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r (m) | yaw (°) | Nhận xét |
|---|---|---|---|---|---|---|---|
| s01f22 #2 Car | −0,966 | 1,432 | −1,682 | 7,04 | 7,109 | 271,7 | Gần đường; gap +0,09 m |
| s01f52 #6 Car | −1,044 | 1,522 | −1,805 | 16,10 | 16,129 | 268,9 | Gần đường cục bộ; gap +0,03 m |
| s05f28 #2 Car | −1,010 | 1,304 | −1,662 | 8,36 | 8,422 | 84,4 | Gap −0,19 m khi vật bị che; chưa đến ngưỡng cờ chìm |

Gần nhất: Car `s01f22#3`, dist_m **3,19 m**, r **3,345 m**. Xa nhất toàn bảng: Bus `s02f05#8`, dist_m **59,31 m**, r **59,314 m**, ngoài vùng chấm 50 m; tôi giữ nguyên dữ liệu gốc. Trong 50 m, xa nhất là Car `s04f45#6`, dist_m **47,28 m**, r **47,289 m**.

So kích thước Car và Cyclist ở mục 3 cho thấy không có cờ vẫn có thể ngoài bảng 4.3. Xe cùng chiều phía trước `s01f52#6` có yaw **268,9°**, gần 270°; Car ngược chiều `s02f21#5` có yaw **92,6°**, gần 90°.

| Car (frame, idx) | dist_m (m) | Số điểm points_in |
|---|---|---|
| s01f22 #3 | 3,19 | 3178 |
| s01f52 #6 | 16,10 | 414 |
| s01f52 #7 | 22,10 | 280 |
| s01f22 #6 | 37,90 | 90 |

Trong mẫu này, xa khoảng **11,9 lần** thì số điểm còn khoảng **1/35,3**. Xu hướng giống slide: 1968, 718, 214, 72, 19 điểm ở các khoảng 0–10, 10–20, 20–30, 30–40, 60–80 m; các vật/frame khác nhau nên không suy thành quy luật chung.

Tôi đã bật occluded cho `s04f45#6`, `s05f28#1`, `s05f28#2` (trước 0, sau 1), rồi Save và chạy lại `sync`. Phiếu mới có 0 cờ kích thước, lơ lửng, chìm hoặc điểm sát mép. Bốn hộp `s02f05#7`, `s02f05#8`, `s03f18#6`, `s04f45#6` không ước được ground_gap_m nên chưa đánh giá đáy bằng chỉ số này.

Tôi đã kiểm chéo với MSSV **2A202603176**. Hai số dùng đối chiếu là chiều dài Bus `s01f22#1` **13,029 m** và chiều cao Cyclist `s02f21#6` **1,227 m**; số trong báo cáo khớp hai ô tương ứng ở Bảng A.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Model cần nhãn theo mét để học vị trí/kích thước trong không gian; pixel chỉ là phép chiếu phụ thuộc camera. Vật thưa điểm cần quy tắc riêng để model không học hộp co theo phần nhìn thấy hoặc nhãn đoán khi thiếu bằng chứng.

- Công cụ: ChatGPT/Codex.
- Câu hỏi chính: thao tác CVAT, lệnh chạy script và cấu trúc bài nộp.
- Cách dùng: hỗ trợ kiểm tra tệp, đối chiếu số liệu, ghi hướng trục trên ảnh Top và soạn thảo; tôi gắn nhãn trên CVAT và sinh CSV bằng script gốc sau Save/sync.

Tôi còn chưa chắc về chiều cao Cyclist, kích thước Bus, nguyên nhân vùng ít điểm ở kính xe và bản chất cụm điểm gần xe đạp. Với T4-d, tôi ghi nhận cụm chưa xác định thay vì kết luận đó là outlier.
