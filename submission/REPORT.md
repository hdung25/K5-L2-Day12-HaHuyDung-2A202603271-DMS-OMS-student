# Báo cáo Lab Ngày 12 — DMS & OMS: gán nhãn khung hình cabin RGB + IR

**Họ tên:** Hà Huy Dũng · **MSSV:** 2A202603271 · **Bộ ảnh:** G01 · **Hình thức:** cá nhân

## 1. Môi trường

- CVAT local v2.82.0 chạy bằng Docker trên máy cá nhân (`localhost:8080`).
- Tôi tạo project `Day12` bằng cách dán `labels_cabin_v1.json` vào tab Raw (đủ 9 nhãn), và tạo task `Day12-G01` với 22 ảnh cốt lõi: `S01–S08` (`_rgb` và `_ir`) và `D01–D06`. Tôi không đưa `S09`, `S10` và `manifest.csv` vào task.
- Script kiểm: `scripts/cabin_check.py` của repo, chạy bằng Python 3.11.

## 2. Task 1 — OMS hàng ghế sau (S01–S08, 16 ảnh)

Camera SVIRO đặt phía trước và nhìn về phía sau xe, nên tôi gọi ghế theo xe: **bên trái ảnh là `rear_right`**. Tôi gán ảnh `_rgb` trước, sau đó copy shape sang ảnh `_ir` và đọc lại từng thuộc tính trên chính ảnh IR.

| Cảnh | rear_right (trái ảnh) | rear_middle | rear_left (phải ảnh) |
|---|---|---|---|
| S01 | infant_seat (trống) | trống | child_seat — RGB `unknown`, IR `no` |
| S02 | infant_seat (có em bé) | trống | person + face, `not_worn` |
| S03 | object `cup` | object `other` | child_seat (có trẻ) `yes` |
| S04 | object `box` | person + face, `not_worn` | trống |
| S05 | child_seat (có trẻ) `yes` | trống | child_seat (có trẻ) `yes` |
| S06 | child_seat `no` | object — RGB `other`, IR `bag` | infant_seat (trống) |
| S07 | trống | infant_seat (có em bé) | trống |
| S08 | infant_seat (có em bé) | child_seat `no` | child_seat `no` |

Các điểm chính:
- **Dây an toàn:** SVIRO không có ai thắt dây, nên tôi chỉ dùng `not_worn` hoặc `unknown`.
- **frame_meta:** ảnh dựng nên `light = unknown`, `camera_blocked = no`.
- **Ghế trống:** không vẽ box, chỉ đếm vào ghi chú (S01: 1, S02: 1, S04: 1, S05: 1, S07: 2).
- **Trẻ em trong ghế:** guideline yêu cầu không gán trẻ em. Tôi chỉ vẽ box ghế, không vẽ `person` cho trẻ. Theo `labels_cabin_v1.json` mới (`child_seat.occupied` có thêm `yes`), ghế trẻ em có trẻ ngồi ghi `occupied = yes`. Nôi (`infant_seat`) không có thuộc tính này nên tôi đưa vào câu hỏi leo thang.

## 3. Task 2 — DMS tài xế (D01–D06, NIR thật)

Ảnh Drive&Act bị xoay khoảng 90° và tôi **không xoay lại**. Trái/phải gọi theo người lái: mắt trái có **y lớn hơn**.

| Khung | Hoạt động | seatbelt | Mắt (trái / phải) | glasses | gaze_zone | Tay | phone |
|---|---|---|---|---|---|---|---|
| D01 | cúi người, tay phải với xuống | unknown | unknown / unknown | clear | other | phải (on_wheel unknown), trái (no) | — |
| D02 | uống nước | worn_correct | unknown / unknown (phản xạ kính) | clear | other | phải cầm chai, trái cầm thẻ | — |
| D03 | đang thắt dây | unknown (thắt dở) | unknown / unknown | clear | lap | hai tay ở khóa dây | — |
| D04 | quay đầu sang phải | worn_correct | unknown / unknown | clear | other | phải (no), trái (unknown) | — |
| D05 | gọi điện | worn_correct | unknown / unknown | sun | other | phải áp tai, trái trên đùi | held |
| D06 | mắt nhắm, cầm đồ | worn_correct | closed / closed | none | other | phải (no), trái (no, cháy sáng) | — |

- **face13:** tôi vẽ đủ 13 điểm trên cả 6 khung, vì mọi mặt đều rộng hơn 60 px. Điểm bị phản xạ kính, chai nước hoặc râu che thì đánh `occluded`. Mắt phải của D04 bị sống mũi che hẳn nên đánh `outside`.
- **on_wheel:** xe đang tự lái nên tay không đặt trên vô lăng (`on_wheel = no`) là bình thường.
- **Không vẽ** chai nước (D02), thẻ hay giấy (D02, D06), hay kính và mũ trên bảng điều khiển (D05).

## 4. Tự kiểm bằng `cabin_check`

| Lần kiểm | Phạm vi | Kết quả |
|---|---|---|
| Checkpoint A (`check_cpA.txt`) | S01, S02 — 4 ảnh | 0 lỗi, 0 cảnh báo |
| Checkpoint B (`check_cpB.txt`) | D01, D02 | 0 lỗi, 0 cảnh báo |
| Lần cuối (`check_final.txt`) | 22 ảnh | **0 lỗi**, 1 cảnh báo |
| Sau review chéo (`check_final_v2.txt`) | 22 ảnh, `export_final_v2.zip` | **0 lỗi**, 1 cảnh báo |

Cảnh báo còn lại nằm ở D03: luật so trục x giả định mặt nhìn thẳng, nhưng ảnh bị xoay 90°. Tôi kiểm lại theo trục y: `left_eye_outer` (y = 528) lớn hơn `right_eye_outer` (y = 398), tức là đúng quy ước. Giải thích đã ghi ở cuối `check_final.txt`.

File `export_final.zip` và `export_final_v2.zip` được export theo định dạng CVAT for images 1.1 và bỏ chọn "Save images", nên chỉ chứa `annotations.xml`.

## 5. IR làm mất gì so với ảnh màu (tóm tắt `ir_log.csv`)

**IR mô phỏng (S##):**
- Màu áo và màu vật mất hoàn toàn (S02, S03), nhưng không ảnh hưởng nhãn vì không có trường nào dựa vào màu.
- Ở những cảnh RGB tối, IR-sim lại giữ được thông tin: trong S01, IR-sim cho thấy ghế trống (`occupied` đổi từ `unknown` sang `no`); trong S06, nó cho thấy hình ba lô (`type` đổi từ `other` sang `bag`).
- Tôi không ghi các hiện tượng của IR thật (chói kính, mắt sáng) từ ảnh S##.

**IR thật (D##):**
- Đốm phản xạ LED trên tròng kính che mắt (D02, D04) → mắt `unknown`, điểm mắt đánh `occluded`.
- Tròng kính hiện tối, không nhìn xuyên được (D05) → `glasses = sun`, mắt `unknown`.
- Bàn tay ở gần đèn LED bị cháy sáng (D06) → vẫn vẽ `hand` theo đường viền, không vẽ `phone`.
- Vùng ngực tối nên không phân biệt được dây an toàn với áo (D01) → `seatbelt = unknown`.

## 6. Danh sách lớp v0 và câu hỏi leo thang

Chi tiết nằm trong `class_list_v0.md`: 8 lớp/thuộc tính DMS và 7 lớp/thuộc tính OMS, mỗi lớp có mục "để làm gì" và "có thấy trên IR không". Hai câu hỏi leo thang:

1. Labels mới đã có `child_seat.occupied = yes`, nhưng `infant_seat` chưa có `occupied`: nôi có em bé thì ghi nhận ở đâu (S02, S07, S08)?
2. `infant_seat.facing` tính theo hướng nào, khi nôi trong SVIRO quay miệng về phía camera (S01, S06, S07, S08)?

## 7. Review chéo

Nhóm 3 người cùng mã G01 chấm theo vòng **Quốc Anh → Hiếu → Dũng → Quốc Anh**.

- **Tôi chấm bài Dương Quốc Anh** (`disagreement_log.csv`, 123 dòng): 115 dòng tôi đúng theo guideline, 8 dòng mơ hồ cần hỏi coach. Lỗi chính trong bài Quốc Anh:
  - Gán trẻ em thành `person` / `face` (S03, S05, S07), trong khi GUIDE 3 yêu cầu không gán trẻ em.
  - Nhầm nôi xách thành `child_seat` (S01, S02, S06).
  - Thiếu box `object` (S03, S04).
  - Ghi mắt `open` khi đốm phản xạ kính che mắt (D01, D02, D04), trái 4.4-1.
  - Thiếu cờ `occluded` ở các điểm mắt (4.6).
  - Ghi `seatbelt = unknown` khi dây thấy rõ (4.7).
- **Triệu Hoàng Hiếu chấm bài tôi** (`disagreement_log_hieu.csv`, 46 dòng): 24 dòng tôi giữ nguyên, 14 dòng mơ hồ, 8 dòng Hiếu đúng. Tôi đã sửa:
  - Box `face` S04 ôm sát phần mặt thấy được (4.4-5).
  - Bỏ cờ `occluded` ở 4 điểm miệng/môi D01 (4.6).
- **Đề mới:** cập nhật `occupied = yes` cho S03 và S05.
- Tôi export lại thành `export_final_v2.zip`; `check_final_v2.txt` báo 0 lỗi.

## 8. Điều tôi rút ra

- Mỗi ảnh gán theo những gì nhìn thấy **trên chính ảnh đó**. Cặp RGB/IR của cùng một cảnh có thuộc tính khác nhau là đúng, không phải lỗi.
- Trên NIR thật, nguyên nhân chính khiến mắt thành `unknown` là phản xạ đèn LED trên kính. Không thấy mắt thì ghi `unknown`, không ghi `closed`.
- Lỗi tôi hay mắc là bỏ sót hoặc chép thuộc tính ở vùng quá tối hay cháy sáng (xem `self_review.md`). Cách khắc phục: luôn chỉnh brightness hiển thị và đọc lại từng thuộc tính sau khi copy shape.

## 9. Công cụ AI đã dùng

- **Claude Code (Anthropic, model Claude Opus 5.5):** hỗ trợ gán nhãn và đưa nhãn vào CVAT, chạy và đọc kết quả `cabin_check`, viết nháp các tài liệu nộp. Khai báo chi tiết trong `prompts.md`.
