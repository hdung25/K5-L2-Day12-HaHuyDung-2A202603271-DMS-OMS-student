# TEAM — Ngày 12

Mã bộ ảnh: **G01** · Hình thức: nhóm 3 người

Vòng review chéo: **Quốc Anh → Hiếu → Dũng → Quốc Anh** (Quốc Anh chấm Hiếu, Hiếu chấm Dũng, Dũng chấm Quốc Anh).

| Họ tên | MSSV | Đã gán đủ 22 ảnh? | Review chéo với (peer diff) | Số dòng disagreement đã xử lý |
|---|---|---|---|---|
| Hà Huy Dũng | 2A202603271 | có | Dũng chấm Dương Quốc Anh (`disagreement_log.csv`); được Triệu Hoàng Hiếu chấm (`disagreement_log_hieu.csv`) | 123 (bài Quốc Anh) + 46 (Hiếu chấm Dũng) |
| Triệu Hoàng Hiếu | 2A202603294 | có | chấm Dũng | |
| Dương Quốc Anh | 2A202603286 | có (bản nháp v2) | chấm Hiếu | |

Ca vẫn không thống nhất và đã hỏi coach (ghi tên ảnh + đối tượng):

- G01 D01 `person.seatbelt` (unknown / not_worn): vùng ngực tối, không phân biệt dây với áo.
- G01 D05 `face.glasses` (sun / clear / unknown): tròng hiện tối trên NIR.
- G01 D04 `hand` trái `on_wheel` (unknown / yes): tay sát vành vô lăng.
- G01 S01, S06, S07, S08 `infant_seat.facing`.
- Đã giải quyết bằng đề mới (labels v1 có `occupied = yes`): S03, S05 ghế trẻ em có trẻ ngồi → `occupied = yes`.

## Cá nhân

### HaHuyDung-2A202603271
- Phần mình làm: gán đủ 22 ảnh G01; chạy `cabin_check diff` với bài của Quốc Anh và điền 123 dòng; xử lý 46 dòng Hiếu chấm bài mình, sửa 2 lỗi (box face S04, cờ điểm miệng D01) và cập nhật theo đề mới (`occupied = yes` cho S03, S05), export lại `export_final_v2.zip` (0 lỗi).
- Điều mình học được hôm nay: hai người gán giống nhau chưa chắc đã đúng. Phần lớn bất đồng nằm ở các ca "không thấy" (mắt sau phản xạ kính, dây an toàn trong bóng tối). Guideline 4.4-1 ("không thấy thì unknown") là luật mình dùng nhiều nhất khi xử lý bất đồng.
