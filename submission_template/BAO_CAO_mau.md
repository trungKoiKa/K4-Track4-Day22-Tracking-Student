# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** làm cá nhân **Thành viên:** Hoàng Trung Anh — MSSV 2A202602521

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

Chạy trên CPU, `video_1.txt` … `video_5.txt` trong `runs/nop_bai/` đều chạy đủ frame (600 / 1050 / 837 / 900 / 750).

**Cách quan sát `video_2`–`video_5`:** không có nhãn nên tôi so sánh các tracker bằng hai thứ: (1) vẽ hộp và ID của hai tracker cạnh nhau trên cùng một frame mẫu (frame 120), (2) thống kê từ file kết quả: số hộp trung bình mỗi frame, tổng số ID, tỉ lệ track ngắn (dưới 10 frame, thường là hộp nháy hoặc ID bị vỡ). Tôi không xem lại toàn bộ video từng frame, nên phần "quan sát" dưới đây chỉ nói về những gì thấy trên frame mẫu và con số thống kê.

## 1. Cấu hình đã chọn

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.3 | 0.7 | Chỉ bắt được khoảng 22% số người trong nhãn (recall thấp), nhưng hộp bắt được khá chuẩn (precision 87%). 33 lần đổi ID, 55 ID so với 62 ID thật. | `bytetrack` 0.3/0.5 (HOTA 26.9); `botsort` conf 0.5 (HOTA 27.2, bỏ sót nhiều người); `strongsort` conf 0.15 (IDF1 cao nhất 32.6 nhưng 110 lần đổi ID) |
| video_2 (phố đêm, tĩnh, rất đông) | botsort | 0.15 | 0.5 | Trên frame mẫu, nhóm người đứng sát nhau ở khu vực xa phía trên bị bỏ sót ở cả hai tracker; `botsort` bắt thêm vài người đi ở giữa đường. ID khá ổn định: track trung bình dài 223 frame, chỉ 6% track ngắn. | `botsort` conf 0.3 (20% track ngắn, nhiều ID nháy hơn); `botsort` conf 0.5 (bỏ sót nhiều, 7.5 hộp/frame so với 11.6 ở 0.15) |
| video_3 (camera di động, ảnh nhỏ) | bytetrack | 0.15 | 0.5 | Cảnh quay sát người đi đường, hộp rất lớn, người che nhau liên tục. ID bị vỡ nhiều: 130 ID cho khoảng 5 người mỗi frame, 31% track ngắn. | `botsort` conf 0.15 (161 ID, 34% track ngắn); `botsort` conf 0.3 (167 ID); `ocsort` conf 0.15 (221 ID, tệ nhất) |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.15 | 0.5 | Cảnh thưa, người to và rõ. Cả hai tracker cho kết quả gần như nhau ở frame mẫu. `botsort` conf 0.15 cho nhiều hộp hơn mà tỉ lệ track ngắn chỉ 16% trên cả video. | `bytetrack` conf 0.15 (ít hộp hơn, 5.8 so với 7.0 hộp/frame ở bản thử 200 frame); `botsort` conf 0.5 |
| video_5 (trên xe bus, giao lộ đông) | botsort | 0.15 | 0.5 | Người nhỏ ở xa (hộp cao khoảng 130 px). `botsort` bắt được nhiều người nhỏ ở lề trái hơn `bytetrack` trên frame mẫu. Camera rung nên ID vẫn bị vỡ: 79 ID, 23% track ngắn. | `bytetrack` conf 0.15 (14% track ngắn nhưng chỉ 4 hộp/frame, bỏ sót nhiều người); `botsort` conf 0.5 (3.2 hộp/frame, bỏ sót nhiều) |

Ghi chú về `iou`: ở `video_1`, `iou` 0.4 / 0.5 / 0.7 chỉ chênh HOTA từ 29.3 đến 30.0, nên tôi chọn 0.7 vì HOTA cao nhất. Chênh lệch này nhỏ, có thể chỉ là nhiễu. Ở `video_2`–`video_5` tôi giữ `iou` 0.5 và chỉ thử đổi `conf`.

## 2. Số liệu video_1

Kết quả của `scripts/evaluate_practice.py` cho `runs/nop_bai/video_1.txt` (`botsort`, conf 0.3, iou 0.7):

```
HOTA: nhom01_video1-pedestrian     HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            29.969    18.408    49.061    19.176    74.385    52.38     80.959    83.019    30.624    37.181    76.409    28.409
COMBINED                           29.969    18.408    49.061    19.176    74.385    52.38     80.959    83.019    30.624    37.181    76.409    28.409

CLEAR: nhom01_video1-pedestrian    MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag
video_1                            19.025    80.817    19.202    22.491    87.244    14.516    17.742    67.742    14.71     4179      14402     611       33        9         11        42        105

Identity: nhom01_video1-pedestrian IDF1      IDR       IDP       IDTP      IDFN      IDFP
video_1                            29.703    18.68     72.463    3471      15110     1319

Count: nhom01_video1-pedestrian    Dets      GT_Dets   IDs       GT_IDs
video_1                            4790      18581     55        62
```

Tóm tắt: HOTA 29.97, MOTA 19.03, IDF1 29.70.

Các cấu hình đã thử trên `video_1` (đủ 600 frame):

| Cấu hình | HOTA | MOTA | IDF1 | Đổi ID |
|---|---|---|---|---|
| bytetrack 0.3 / 0.5 | 26.91 | 17.29 | 25.71 | 12 |
| ocsort 0.3 / 0.5 | 27.45 | 19.81 | 28.73 | 42 |
| deepocsort 0.3 / 0.5 | 27.38 | 19.76 | 27.80 | 51 |
| strongsort 0.3 / 0.5 | 28.66 | 19.70 | 29.85 | 41 |
| strongsort 0.15 / 0.5 | 29.19 | 19.91 | 32.57 | 110 |
| botsort 0.15 / 0.5 | 29.34 | 20.73 | 29.56 | 27 |
| botsort 0.3 / 0.4 | 29.32 | 19.47 | 29.82 | 19 |
| botsort 0.3 / 0.5 | 29.46 | 19.81 | 29.35 | 25 |
| **botsort 0.3 / 0.7 (nộp)** | **29.97** | 19.02 | 29.70 | 33 |
| botsort 0.5 / 0.5 | 27.17 | 15.25 | 24.56 | 10 |

`video_2` đến `video_5` không có nhãn trong gói lab, nên tôi không điền số cho các video đó.

## 3. Phân tích

**video_1 (quảng trường, camera tĩnh, ban ngày).** Điểm yếu lớn nhất ở đây không phải là đổi ID mà là bỏ sót người: recall chỉ 22%, nhưng precision 87%. Nghĩa là detector `yolo26n` cố định ở 640 px bỏ qua nhiều người nhỏ hoặc bị che, và tracker không cứu được phần này. Vì vậy MOTA thấp (19) dù ít hộp giả. Về tracker, `bytetrack` có ít lần đổi ID nhất (12) nhưng HOTA thấp nhất (26.9), vì nó bỏ sót nhiều hơn. `botsort` có thêm Re-ID nên giữ danh tính tốt hơn và HOTA cao hơn khoảng 3 điểm. `strongsort` ở conf 0.15 cho IDF1 cao nhất (32.6) nhưng sinh ra 110 lần đổi ID, cho thấy conf thấp làm tăng hộp giả nhấp nháy, gây vỡ ID.

**video_2 (phố đêm, camera tĩnh, rất đông).** Cảnh tối và đông nên `conf` cao (0.5) làm mất nhiều người; `conf` 0.15 bắt được nhiều người hơn (11.6 hộp/frame so với 7.5 ở conf 0.5). Điều đáng chú ý là ở conf 0.15, tỉ lệ track ngắn của `botsort` lại *giảm* xuống 4% (so với 20% ở conf 0.3 trong bản thử 200 frame). Tôi đoán nguyên nhân là người đi bộ tối bị detector lúc thấy lúc không quanh ngưỡng 0.3, nên hạ ngưỡng giúp track liền mạch hơn. Camera đứng yên nên chuyển động dễ dự đoán, nhưng người đứng sát nhau khiến Re-ID có ích khi hai người cắt nhau. Đây là điều tôi suy ra từ thống kê, chưa kiểm chứng bằng cách xem từng cặp người cắt nhau.

**video_3 (camera di động, ảnh nhỏ, ít khung hình/giây).** Đây là video khó nhất cho cả tracker: trên toàn video, tất cả cấu hình đều tạo ra từ 130 đến 221 ID cho khoảng 5 người mỗi frame. Khi camera di chuyển và khung hình thưa, người nhảy xa giữa hai frame và che nhau liên tục. Tôi chọn `bytetrack` vì nó cho ít ID nhất (130, so với 161 của `botsort` và 221 của `ocsort`) và ít track ngắn nhất (31.5%). Tôi đoán Re-ID của `botsort` kém ổn định khi bị che mạnh và ảnh bị mờ do chuyển động. Đổi lại `bytetrack` ra ít hộp hơn (5.2 so với 6.0 hộp/frame). Đây là sự đánh đổi, không phải thắng rõ ràng.

**video_4 và video_5.** Ở `video_4` (trong nhà, thưa, người to) cả hai tracker gần như bằng nhau, nên chọn `botsort` conf 0.15 chủ yếu vì nhiều hộp hơn. Ở `video_5` (xe bus rung, người nhỏ ở xa), `conf` thấp cần thiết để bắt người nhỏ, và `botsort` bắt được nhiều hơn `bytetrack` (5.7 so với 4.0 hộp/frame ở bản thử 200 frame).

## 4. Nếu có thêm thời gian

- Với `video_3` và `video_5` (camera di động / rung), thử bù chuyển động camera: `botsort` có tùy chọn này trong cấu hình riêng của nó, nhưng lab không cho đổi nên tôi không thử.
- Quét `conf` mịn hơn (0.1, 0.2, 0.25) và xem trực tiếp các frame có hai người cắt nhau, thay vì chỉ dựa vào thống kê số ID.
- Vì recall ở `video_1` chỉ 22%, thử detector mạnh hơn hoặc ảnh đầu vào lớn hơn 640 px để xem phần lớn lỗi có thật sự đến từ detector không (phần này nằm ngoài luật chơi của lab).
