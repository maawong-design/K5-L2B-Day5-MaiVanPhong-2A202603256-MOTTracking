# BÁO CÁO CÁ NHÂN — LAB DAY 5: MULTI-OBJECT TRACKING

## 0. Thông tin sinh viên

| Thông tin | Nội dung |
|---|---|
| Họ và tên | Mai Văn Phong |
| MSSV | 2A202603256 |
| Email | 26ai.phongmv@vinuni.edu.vn |
| Lớp | K5 — Level 2 |
| Bài thực hành | Day 5 — MOT Tracking |
| Dataset | MOT17-09 |
| Task ID trên CVAT | 9 |
| Công cụ gán nhãn | CVAT |
| Phương pháp | Multi-Object Tracking |
| Số frame | 150 |
| FPS | 30 |
| Số đối tượng mục tiêu | 5 người đi bộ |

---

## 1. Preview video — CP0

### 1.1. Thông tin dữ liệu

- Dataset: MOT17-09.
- Tổng số frame: 150.
- Frame được đánh số từ 0 đến 149 trong CVAT.
- Số lượng đối tượng theo yêu cầu: 5 người đi bộ.
- Nhãn sử dụng: `person`.
- Chế độ gán nhãn: Track.

### 1.2. Quan sát đối tượng

Dựa trên dữ liệu xuất MOT 1.1, năm track có thông tin như sau:

| Track ID (MOT) | Frame bắt đầu | Frame kết thúc | Số frame có mặt | Frame outside |
|---|---:|---:|---:|---|
| 1 | 0 | 149 | 144 | 40–45 |
| 2 | 0 | 149 | 150 | Không có |
| 3 | 0 | 146 | 117 | 110–139 |
| 4 | 40 | 149 | 110 | Không có |
| 5 | 60 | 149 | 90 | Không có |

**Nhận xét:**

- Có tổng cộng 5 track ID khác nhau.
- Các đối tượng xuất hiện tại những thời điểm khác nhau trong video.
- Một số track có khoảng thời gian không xuất hiện do thuộc tính outside.
- Track 2 xuất hiện liên tục trong toàn bộ 150 frame.
- Track 3 có khoảng outside từ frame 110 đến frame 139.
- Track 1 có khoảng outside từ frame 40 đến frame 45.

---

## 2. Keyframes và thời gian gán nhãn — CP1, CP4

### 2.1. Thống kê kết quả

Kết quả kiểm tra từ file CVAT for video 1.1:

| Track ID (CVAT) | Nhãn | Frame thấy | Keyframe | KF/100 frame | Frame occluded | Lần bật outside |
|---|---|---:|---:|---:|---:|---:|
| 0 | person | 144 | 49 | 34,0 | 24 | 1 |
| 1 | person | 90 | 20 | 22,2 | 10 | 0 |
| 2 | person | 150 | 31 | 20,7 | 0 | 0 |
| 3 | person | 110 | 23 | 20,9 | 0 | 0 |
| 4 | person | 117 | 17 | 14,5 | 3 | 2 |
| **Tổng** | | **611** | **140** | **22,9** | **37** | **3** |

Lưu ý: CVAT sử dụng Track ID bắt đầu từ 0, trong khi file MOT sử dụng Track ID bắt đầu từ 1.

### 2.2. Mật độ keyframe

- Tổng số keyframe: 140.
- Tổng số frame có đối tượng: 611.
- Mật độ keyframe trung bình: 22,9 keyframe / 100 frame có mặt.

**Nhận xét:**

- Track 0 có số keyframe cao nhất với 49 keyframe.
- Track 4 có số keyframe thấp nhất với 17 keyframe.
- Track 2 xuất hiện trong toàn bộ 150 frame và có 31 keyframe.
- Các keyframe được đặt tại những thời điểm khác nhau nhằm mô tả sự thay đổi vị trí, hình dạng và chuyển động của đối tượng.
- Việc sử dụng keyframe giúp giảm thao tác gán nhãn thủ công trên từng frame, đồng thời duy trì tính liên tục của track.

### 2.3. Chiến lược đặt keyframe

Các keyframe được sử dụng để đánh dấu những thời điểm cần cập nhật vị trí bounding box của đối tượng.

Nên bổ sung keyframe khi:
- Đối tượng thay đổi hướng di chuyển.
- Đối tượng thay đổi tốc độ hoặc dừng lại.
- Bounding box thay đổi đáng kể về kích thước hoặc vị trí.
- Đối tượng bị che khuất hoặc xuất hiện trở lại.
- Nội suy giữa các keyframe không còn phản ánh chính xác vị trí thực tế.

---

## 3. Occlusion và xử lý đối tượng bị che khuất — CP2

### 3.1. Thống kê occlusion

Dựa trên file CVAT for video 1.1:

| Track ID (CVAT) | Số frame occluded | Số lần bật outside |
|---|---:|---:|
| 0 | 24 | 1 |
| 1 | 10 | 0 |
| 2 | 0 | 0 |
| 3 | 0 | 0 |
| 4 | 3 | 2 |
| **Tổng** | **37** | **3** |

### 3.2. Cách xử lý đối tượng bị che khuất

Khi đối tượng bị che khuất một phần nhưng vẫn có thể xác định được danh tính, tiếp tục duy trì Track ID cũ và đánh dấu thuộc tính occluded phù hợp.

Khi đối tượng không còn xuất hiện trong khung hình, sử dụng thuộc tính outside theo đúng tình huống thực tế.

Khi đối tượng xuất hiện trở lại:
- Kiểm tra vị trí và đặc điểm nhận dạng của đối tượng.
- Đối chiếu với track trước đó.
- Tiếp tục ID cũ nếu có đủ cơ sở xác định đó là cùng một đối tượng.
- Chỉ tạo ID mới nếu xác định đây là một đối tượng khác.

### 3.3. Minh chứng hình ảnh

Bổ sung ảnh chụp màn hình từ CVAT vào các vị trí sau:

- **Ảnh 1:** Đối tượng trước khi bị che khuất.
- **Ảnh 2:** Đối tượng trong trạng thái occluded.
- **Ảnh 3:** Đối tượng xuất hiện trở lại.

> Chưa đính kèm ảnh minh chứng trong bản báo cáo này. Cần chèn ảnh thực tế từ CVAT trước khi nộp.

---

## 4. Tự kiểm tra chất lượng ID — CP3

### 4.1. Quy trình self-review

Thực hiện kiểm tra chất lượng tracking theo hai lượt:

**Lượt 1 — Kiểm tra từng đối tượng:**
- Chọn từng Track ID.
- Kiểm tra ngược từ frame 149 về frame 0.
- Theo dõi tính liên tục của ID.
- Kiểm tra các vị trí đối tượng bị che khuất, biến mất hoặc xuất hiện trở lại.
- Phát hiện các trường hợp ID bị thay đổi không hợp lý.

**Lượt 2 — Kiểm tra toàn cảnh:**
- Quan sát toàn bộ video.
- Kiểm tra theo chu kỳ mỗi 10 frame.
- Đối chiếu các đối tượng với Track ID tương ứng.
- Kiểm tra khả năng trùng ID hoặc gán nhầm ID giữa các đối tượng.

### 4.2. Các lỗi cần kiểm tra

| Loại lỗi | Mô tả |
|---|---|
| id_switch | Một đối tượng bị thay đổi ID trong quá trình theo dõi |
| fragmentation | Một đối tượng bị chia thành nhiều đoạn track |
| lost | Đối tượng bị mất track khi vẫn còn xuất hiện |
| merge | Hai đối tượng bị gộp chung một ID |
| false_track | Tạo track cho đối tượng không tồn tại |
| drift | Bounding box lệch khỏi đối tượng |

### 4.3. Kết quả tự kiểm tra

Kết quả self-review thủ công: **Chưa ghi nhận trong dữ liệu được cung cấp.**

Cần thực hiện và ghi lại kết quả thực tế sau khi hoàn thành hai lượt kiểm tra. Nếu không phát hiện lỗi, có thể ghi:

"Đã kiểm tra hai lượt: lượt 1 theo từng đối tượng từ frame 149 về frame 0; lượt 2 theo toàn cảnh mỗi 10 frame. Không phát hiện lỗi tracking."

---

## 5. File xuất và kết quả kiểm tra — CP4

### 5.1. Các file đã xuất

| Tên file | Định dạng | Nội dung |
|---|---|---|
| `MaiVanPhong_mot.zip` | MOT 1.1 | Dữ liệu tracking theo định dạng MOT |
| `MaiVanPhong_cvat_video.zip` | CVAT for video 1.1 | Dữ liệu annotation từ CVAT |
| `Task9_150_images.zip` | CVAT for images 1.1 | 150 ảnh của Task 9 |

### 5.2. Kết quả kiểm tra MOT 1.1

```text
Số track ID: 5
Tổng số box: 611

[OK] Đúng 5 track ID.

Track | Frame có mặt | Số frame | Đoạn vắng (outside)
1     | 0–149        | 144      | 40–45
2     | 0–149        | 150      | -
3     | 0–146        | 117      | 110–139
4     | 40–149       | 110      | -
5     | 60–149       | 90       | -
```

### 5.3. Kết quả kiểm tra CVAT for video 1.1

```text
Track | Nhãn   | Frame thấy | Keyframe | KF/100 frame
0     | person | 144        | 49       | 34.0
1     | person | 90         | 20       | 22.2
2     | person | 150        | 31       | 20.7
3     | person | 110        | 23       | 20.9
4     | person | 117        | 17       | 14.5

Tổng: 140 keyframe / 611 frame có mặt
= 22.9 keyframe / 100 frame

[OK] Đọc được 5 track từ annotations.xml.
```

### 5.4. Tóm tắt kiểm tra tự động

```text
=== Tóm tắt ===
Lỗi: 0 | Cảnh báo: 0
Đạt.
```

### 5.5. Đánh giá

Kết quả kiểm tra cho thấy:

- File MOT 1.1 được đọc thành công.
- File CVAT for video 1.1 được đọc thành công.
- Có đúng 5 Track ID theo yêu cầu.
- Tổng cộng có 611 bounding box.
- Tổng cộng có 140 keyframe.
- Mật độ keyframe trung bình đạt 22,9 keyframe trên 100 frame có đối tượng.
- Không phát hiện lỗi hoặc cảnh báo từ công cụ kiểm tra tự động.

**Kết luận:** Hai file annotation đáp ứng các điều kiện kiểm tra tự động của bài Lab Day 5.

---

## 6. Câu hỏi tổng kết

### Câu 1. Trong trường hợp nào cần bổ sung keyframe?

Cần bổ sung keyframe khi đối tượng có sự thay đổi đáng kể về vị trí, kích thước, hướng di chuyển hoặc trạng thái xuất hiện trong video.

Đặc biệt, cần bổ sung keyframe khi đối tượng thay đổi hướng, tăng hoặc giảm tốc độ, bị che khuất hoặc xuất hiện trở lại. Điều này giúp bounding box bám sát đối tượng và hạn chế sai lệch do nội suy giữa các keyframe.

### Câu 2. Khi không chắc chắn đối tượng xuất hiện trở lại có phải cùng một người hay không, nên giữ ID cũ hay tạo ID mới?

Cần kiểm tra các đặc điểm nhận dạng, vị trí xuất hiện, hướng di chuyển và mối liên hệ giữa các frame trước và sau.

Nếu có đủ bằng chứng cho thấy đó là cùng một người, nên tiếp tục sử dụng ID cũ để duy trì tính liên tục của tracking.

Nếu không đủ bằng chứng hoặc có khả năng đó là một người khác, không nên tự ý gán ID cũ. Cần kiểm tra kỹ hơn để tránh lỗi ID switch.

### Câu 3. Lỗi ID ảnh hưởng thế nào đến xe tự hành? MOTA và IDF1 phản ánh lỗi này ra sao?

Trong hệ thống xe tự hành, tracking giúp xe xác định và theo dõi vị trí của người đi bộ, phương tiện và các đối tượng xung quanh.

Nếu xảy ra lỗi ID switch, hệ thống có thể nhầm một người đi bộ thành một đối tượng khác hoặc không duy trì được lịch sử chuyển động của cùng một đối tượng. Điều này có thể ảnh hưởng đến việc dự đoán hướng di chuyển, tính toán khoảng cách và đưa ra quyết định phanh hoặc tránh chướng ngại vật.

MOTA đánh giá chất lượng tracking dựa trên các lỗi false positive, false negative và ID switch. IDF1 tập trung vào độ chính xác của việc duy trì danh tính đối tượng xuyên suốt các frame.

Vì vậy, MOTA phản ánh chất lượng tracking tổng thể, còn IDF1 giúp đánh giá khả năng duy trì đúng danh tính của từng đối tượng.

---

## 7. Tài liệu tham khảo và công cụ AI

### 7.1. Tài liệu tham khảo

- MOTChallenge — Dataset và benchmark cho Multi-Object Tracking.
- CVAT Documentation — Hướng dẫn tạo task, annotation và export dataset.
- Tài liệu hướng dẫn Lab Day 5 — MOT Tracking.
- Python Documentation — Hỗ trợ xử lý và kiểm tra dữ liệu.

### 7.2. Công cụ sử dụng

| Công cụ | Mục đích |
|---|---|
| CVAT | Gán nhãn và theo dõi đối tượng trong video |
| Python | Kiểm tra và xử lý dữ liệu |
| PowerShell | Thực hiện lệnh xuất và kiểm tra file |
| Git / GitHub | Quản lý mã nguồn và báo cáo |
| ChatGPT | Hỗ trợ giải thích quy trình, xử lý lỗi và hoàn thiện báo cáo |

AI được sử dụng như công cụ hỗ trợ kỹ thuật và trình bày. Kết quả thống kê trong báo cáo được lấy từ file xuất và công cụ `src/check_submission.py`.

---

## 8. Kết luận

Qua bài thực hành Lab Day 5 — Multi-Object Tracking, em đã thực hiện quy trình gán nhãn và theo dõi 5 đối tượng người đi bộ trên video MOT17-09 bằng CVAT.

Kết quả xuất dữ liệu gồm định dạng MOT 1.1 và CVAT for video 1.1. Công cụ kiểm tra tự động xác nhận có đúng 5 Track ID, tổng cộng 611 bounding box và 140 keyframe. Mật độ keyframe trung bình đạt 22,9 keyframe trên 100 frame có đối tượng.

Hai file annotation đều được kiểm tra thành công, không phát hiện lỗi hoặc cảnh báo.

Qua bài thực hành, em hiểu rõ hơn về cách duy trì ID của đối tượng qua nhiều frame, sử dụng keyframe, xử lý occlusion và outside, cũng như tầm quan trọng của tracking chính xác trong các hệ thống thị giác máy tính và xe tự hành.