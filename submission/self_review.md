# Tự đánh giá — Hà Huy Dũng · G01

## 1. Tự rà soát 4 cảnh

Loại lỗi: `seat_flip`, `wrong_class`, `guess`, `box`, `missing`, `extra`, `lr_flip` (trái/phải mặt), `flag` (thiếu cờ điểm), `ok`.

| Ảnh | Đối tượng | Loại lỗi | Ghi chú | Đã sửa? |
|---|---|---|---|---|
| S01_rgb | child_seat rear_left | guess | Lúc đầu định chép `occupied = no` từ S01_ir; trên RGB ghế chìm trong bóng tối, không thấy bên trong → đổi thành `unknown` (4.4-1: không suy từ ảnh kia) | Có |
| S06_rgb | object rear_middle | ok | RGB tối, không rõ hình → `type = other`; IR-sim thấy ba lô → `bag`. Hai ảnh khác nhau là đúng | — |
| D05 | face.glasses | guess | Đầu tiên để `unknown`; xem lại thấy tròng tối, không nhìn xuyên được tới mắt (khác kính cận D02/D03) → `sun`, mắt `unknown`; đưa vào câu hỏi leo thang | Có |
| D06 | hand trái | missing | Bàn tay trái bị đèn NIR làm cháy sáng nên lúc đầu bỏ qua; xem lại thấy rõ đường viền bàn tay nối với cổ tay áo → thêm `hand` (left, on_wheel = no) | Có |

## 2. Lỗi mình lặp lại nhiều nhất hôm nay

Lỗi: `missing` / `guess` ở vùng ảnh khó nhìn (quá tối hoặc cháy sáng) — có xu hướng bỏ qua đối tượng hoặc chép giá trị từ ảnh cặp thay vì nhìn chính ảnh đó.

Vì sao mình mắc: Ảnh RGB tối và ảnh NIR cháy sáng làm mắt "bỏ qua" vùng đó; còn khi đã gán ảnh `_ir` rõ hơn thì dễ copy luôn thuộc tính sang ảnh `_rgb`.

Lần sau tránh bằng cách: Luôn chỉnh brightness/contrast hiển thị trước khi gán mỗi ảnh tối; sau khi copy shape RGB ↔ IR thì đọc lại **từng thuộc tính** trên chính ảnh đó (đặc biệt `occupied`, `type`, mắt), trường nào không thấy thì đổi `unknown`.

## 3. Ghi chú về disagreement_log

Một dòng bạn của mình đúng và mình sai (Hiếu chấm bài mình): **G01 S04, box `face`**. Mình vẽ box theo cả khuôn mặt ước lượng, kể cả phần bị tóc che, nên box rộng tới x = 550. Hiếu vẽ sát phần da mặt thấy được, và cách đó đúng theo 4.4-5 ("hộp sát mép, ôm đủ phần thấy được"). Mình đã sửa ở `export_final_v2.zip`.

Mình học được: khi mặt hoặc vật bị che một phần, box ôm **phần thấy được** chứ không ôm phần mình đoán. Đây cùng một kiểu lỗi với `guess` ở mục 2.

Tương tự ở D01: mình đánh `occluded` cho 4 điểm miệng vì có râu, nhưng môi và khóe miệng vẫn thấy được. Bỏ cờ là đúng theo 4.6.
