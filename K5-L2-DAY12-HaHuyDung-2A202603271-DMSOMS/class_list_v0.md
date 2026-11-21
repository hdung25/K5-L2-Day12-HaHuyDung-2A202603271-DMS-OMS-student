# Danh sách lớp v0 — Hà Huy Dũng · G01

Viết tiếp từ slide 54 bài giảng. Mỗi lớp trả lời: **model cần nhận ra cái này để làm gì?**

## DMS (8 lớp / thuộc tính)

| # | Lớp / thuộc tính | Giá trị | Để làm gì (chức năng DMS) | Nhìn thấy được trên IR? |
|---|---|---|---|---|
| 1 | `face` + `eye_left` / `eye_right` | open / closed / unknown (từng mắt) | Phát hiện buồn ngủ (PERCLOS, mắt nhắm lâu); `unknown` để model không học nhầm "phản xạ = nhắm" | Có, trừ khi đốm phản xạ LED trên kính che mắt (D02, D04) |
| 2 | `gaze_zone` | road, mirror_l/r/c, cluster, center, lap, other | Phát hiện mất tập trung: thời gian liên tục không nhìn `road` (ví dụ > 2 s) | Có, nhờ hướng đầu + mắt; kém khi mắt unknown |
| 3 | `face13` (13 điểm mặt, cờ occluded/outside) | tọa độ + cờ | Ước lượng tư thế đầu (yaw/pitch), độ mở mắt (EAR), ngáp (độ mở miệng) | Có; điểm mắt hay bị occluded vì phản xạ kính |
| 4 | `glasses` | none / clear / sun / unknown | Chọn chiến lược đo mắt (kính gây phản xạ); cảnh báo khi kính râm che mắt hoàn toàn | Một phần: kính cận trong suốt; kính râm trên NIR có thể trong suốt hoặc tối (D05) |
| 5 | `phone` (`state`: held / mounted / lap) | box + trạng thái | Phát hiện dùng điện thoại khi lái (gọi điện, nhắn tin) | Có theo hình dạng; khó khi tay che hoặc cháy sáng (D06) |
| 6 | `hand` (`whose`, `side`, `on_wheel`) | box + thuộc tính | Hands-on-wheel cho mức tự động L2/L3; nhận biết tay đang làm việc khác (uống, gọi) | Có; tay gần camera dễ bị cháy sáng |
| 7 | `person.seatbelt` (driver) | worn_correct / worn_incorrect / not_worn / unknown | Nhắc thắt dây, phát hiện thắt sai (dưới nách, sau lưng) | Có, dây đen nổi rõ trên áo sáng; khó trên áo tối (D01) |
| 8 | `frame_meta.camera_blocked` | yes / no | Phát hiện camera bị che → báo lỗi hệ thống DMS thay vì đoán sai | Có |

## OMS (7 lớp / thuộc tính)

| # | Lớp / thuộc tính | Giá trị | Để làm gì (chức năng OMS) | Nhìn thấy được trên IR? |
|---|---|---|---|---|
| 1 | `person` + `seat` | box, ghế theo xe | Đếm người theo ghế: nhắc thắt dây hàng sau, cấu hình túi khí, điều hòa theo vùng | Có (hình dạng) |
| 2 | `person.seatbelt` (hàng sau) | not_worn / unknown / worn_* | Seatbelt reminder hàng ghế sau (luật Euro NCAP) | Có nếu người không bị che |
| 3 | `infant_seat` (`facing`) | box + rear / front, theo ghế | Tắt túi khí ghế trước khi có nôi quay sau; nhắc quên trẻ trong xe | Có (hình dạng vỏ nôi, quai) |
| 4 | `child_seat` (`occupied`) | box + no / unknown, theo ghế | Phân biệt ghế trẻ em trống với có trẻ → cảnh báo quên trẻ, tránh báo động giả | Có hình dạng; bên trong ghế khó thấy khi tối (S01_rgb) |
| 5 | `object` (`type`: bag, box, cup, other) | box + loại, theo ghế | Nhắc quên đồ khi rời xe; phân biệt vật với người để không báo "có người" sai | Có theo hình dạng; mất màu (S03, S06) |
| 6 | `face` (OMS, chỉ box) | box | Xác nhận ghế có người thật (không phải vật), định vị đầu cho túi khí thông minh | Có |
| 7 | `frame_meta.light` + `modality` | day / night / unknown; rgb / ir | Chọn model/ngưỡng theo điều kiện sáng; đánh giá lỗi theo modality | — (siêu dữ liệu) |

## 2 câu hỏi leo thang

1. **Câu hỏi:** Labels v1 mới đã thêm `child_seat.occupied = yes`, nhưng `infant_seat` vẫn **không có** thuộc tính `occupied`. Vậy nôi có em bé nằm trong thì ghi nhận "có trẻ" ở đâu? Đây chính là ca "quên trẻ trong xe" mà OMS cần học.
   **Khung ví dụ:** G01 S02 (em bé trong nôi, rear_right), S07 (rear_middle), S08 (rear_right); so với S01, S06 (nôi trống).
   **Tạm gán thế nào và vì sao:** Vẽ box `infant_seat` (đúng lớp ghế để khớp đáp án SVIRO), **không** vẽ `person` cho em bé (GUIDE 3), ghi chú "có em bé". Với `child_seat` có trẻ (S03, S05) mình đã dùng `occupied = yes` theo labels mới. Đề xuất thêm `occupied` (no / unknown / yes) cho `infant_seat` ở v1.x.

2. **Câu hỏi:** `infant_seat.facing` tính theo hướng nào khi camera OMS nhìn về phía sau? Trong SVIRO nôi được dựng với miệng nôi (mặt em bé) hướng về camera, tức là em bé nhìn về **phía trước xe**, trái với thực tế (nôi luôn lắp quay về sau).
   **Khung ví dụ:** G01 S07 (em bé nhìn thẳng vào camera, rear_middle), S08 (rear_right), S01 / S06 (nôi trống, miệng nôi hướng camera).
   **Tạm gán thế nào và vì sao:** `facing = front` cho cả 5 nôi, theo hướng em bé nhìn so với xe (gọi theo xe, 4.4-3). Cần người viết guideline xác nhận `facing` mô tả hướng em bé (như mình gán) hay hướng lưng nôi.

(Câu phụ) Trên NIR, tròng kính hiện tối và không nhìn xuyên được (G01 D05): có nên có giá trị riêng kiểu "kính, không rõ loại" thay vì `unknown`? Tạm ghi `glasses = unknown`, mắt `unknown`.
