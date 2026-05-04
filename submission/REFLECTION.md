# Reflection - Day 18 Lakehouse Lab

**Họ tên:** Nguyễn Mậu Lân
**MSSV:** 2A202600400

### Anti-pattern dễ vướng phải nhất: "The Data Swamp" (Đầm lầy dữ liệu)

Qua quá trình thực hiện Lab, em nhận thấy anti-pattern dễ vướng phải nhất trong thực tế là việc bỏ qua tầng Silver mà đi thẳng từ Bronze lên Gold, hoặc không thực hiện nghiêm ngặt việc kiểm soát chất lượng dữ liệu tại tầng Silver.

**Lý do:**
1. **Dữ liệu trùng lặp và rác:** Như kết quả tại NB4, dữ liệu Bronze chứa tới 200,000 bản ghi nhưng có gần 10,000 bản ghi bị trùng lặp (dropped 9,948 rows). Nếu không có tầng Silver để Deduplicate, các báo cáo Gold sẽ sai lệch hoàn toàn.
2. **Schema Enforcement:** Trong NB1, nếu cho phép mọi thay đổi schema tự do mà không có cơ chế `opt-in` (như `merge`), bảng dữ liệu sẽ nhanh chóng mất đi tính nhất quán, gây hỏng các hệ thống hạ nguồn.

**Bài học rút ra:**
Kiến trúc Medallion không chỉ là chia folder, mà là cam kết về chất lượng dữ liệu. Tận dụng **Transaction Log** (NB1) và **Time Travel** (NB3) là cách duy nhất để quản trị Lakehouse tin cậy thay vì chỉ lưu các file Parquet rời rạc.

---

### Tổng kết kết quả Lab:
* **NB1 — Delta Basics**: Hoàn thành Schema Enforcement và kiểm tra Log giao dịch (Transaction Log) với các file JSON trong `_delta_log`.
* **NB2 — Optimize**: Đạt tốc độ tối ưu vượt trội (**8.5x Speedup**) nhờ `Z-ORDER` trên cột `user_id`, giúp giảm số lượng file cần quét (Files-pruned ratio 55.0x).
* **NB3 — Time Travel**: Duy trì lịch sử 5 phiên bản (v0-v4) và thực hiện thành công lệnh `RESTORE` về phiên bản v2 sạch sau khi giả lập ghi đè dữ liệu lỗi.
* **NB4 — Medallion**: Xây dựng thành công pipeline: Bronze (200k rows) -> Silver (190,052 rows sau khi dedup) -> Gold (Báo cáo đầy đủ 8 ngày x 3 model LLM).