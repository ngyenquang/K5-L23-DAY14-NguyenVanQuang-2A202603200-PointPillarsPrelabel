# Báo cáo Day 14 — Nguyễn Văn Quang — 2A202603200

- Mã học viên dùng với `--student`: `2A202603200`
- `seed_tag` trong `frames_A.txt`: `82ac3100fd`

## 1. Dự đoán trước sweep (bước 5b)

Ghi **trước** khi chạy `scripts/pp_sweep.py`, rồi commit và push ngay (lịch sử commit là bằng chứng thời điểm viết). Không sửa phần này sau khi có kết quả.

- Tham số tôi nghĩ giảm công sửa nhiều nhất: **`score_thr`**, tăng từ 0.1 lên khoảng **0.3–0.4**. `min_points` gần như không giúp được (dự đoán `min_points` = 0 hoặc 5 cho kết quả tương đương).
- Vì sao (nhìn ảnh BEV tập A: box thừa nhiều hay box sót nhiều?):
  - Tập A chủ yếu là **box thừa**: với cấu hình gốc, trong 50 m tôi phải xóa 48 box, chỉ thêm 5 và sửa 6 (chi phí 22.4). Phần xóa chiếm 14.4/22.4 chi phí.
  - Box thừa phần lớn nằm trên bụi cây, dải phân cách và hàng rào ven đường (nhiều "Cyclist" xếp thẳng hàng ở s03f44), cộng với box nằm ngay trên xe thu dữ liệu (r ≈ 1 m).
  - Box tôi xóa có score thấp hơn rõ: trung vị 0.34, so với 0.84 ở box tôi giữ. Box giữ thấp nhất là 0.30. Vì vậy ngưỡng 0.3 bỏ được khoảng 18 box thừa mà không mất box đúng; ngưỡng 0.4 bỏ được khoảng 30 box thừa nhưng mất khoảng 4 box đúng (mỗi box mất thành 1 lần "thêm").
  - `min_points` không tách được hai nhóm, vì box thừa trên bụi cây vẫn có nhiều điểm (trung vị 80 điểm): `min_points=10` chỉ bỏ được khoảng 3 box thừa và đã mất 1 box đúng ở xa.
  - Rủi ro: chỉ có 3 frame, nên ngưỡng tốt trên tập A có thể không giữ được trên tập B.

## 2. Tập A: sửa tay và tune

| Cấu hình | Xóa | Thêm | Sửa | Chi phí |
|---|---:|---:|---:|---:|
| Gốc | | | | |
| `best_config.json` | | | | |

- Tham số đã đổi trong `best_config.json`: ...
- Kết quả leave-one-out (dán các dòng `scripts/pp_sweep.py` in ra): ...
- Cấu hình có ổn định trên từng frame không? Vì sao tôi vẫn chọn (hoặc không chọn) nó: ...

## 3. Trước/sau (bước 8)

![so sánh](../../d14_<ID>/compare/<frame>.jpg)

- `scripts/pp_compare.py` báo giảm (hoặc tăng) ...% công sửa trên tập A.
- Dự đoán ở mục 1 đúng hay sai? Giải thích bằng số box xóa/thêm/sửa: ...

## 4. Tập B

Số đo từ `scripts/pp_prelabel.py --frames d14_<ID>/frames_B.txt --config d14_<ID>/best_config.json --gt d14_<ID>/ref_B.json`:

| Xóa | Thêm | Sửa | Chi phí |
|---:|---:|---:|---:|
| | | | |

- Loại box tốn công sửa nhất ở tập B: ...

## 5. Giới hạn

- **Chỉ tune trên 3 frame (tập A).** Cấu hình "tốt nhất" có thể chỉ khớp với 3 frame này; kết quả leave-one-out ở mục 2 cho thấy mức độ ổn định, nhưng 3 frame vẫn quá ít để kết luận chắc chắn.
- **Model học từ KITTI, khác miền dữ liệu VF.** KITTI chỉ có Car/Cyclist/Pedestrian, không có Truck/Bus; Truck/Bus chỉ được suy ra bằng luật kích thước nên dễ sai lớp. LiDAR và cảnh đường phố VF cũng khác KITTI, nên model sinh nhiều box thừa trên bụi cây, dải phân cách và trên chính xe thu dữ liệu.
- **Chi phí sửa là công thức (0.3·xóa + 1.0·thêm + 0.5·sửa), chưa phải thời gian sửa thật.** Trọng số do lab đặt; thời gian thực tế phụ thuộc người gán nhãn và giao diện CVAT.
- **Chỉ tính box trong bán kính 50 m.** Box tôi thêm ngoài 50 m (ví dụ Car 57.8 m và Bus 68.1 m ở s03f44) không được tính vào chi phí.
- **Nhãn tham chiếu do chính tôi sửa**, không phải nhãn chuẩn của coach, nên có thể còn sót hoặc sai; tune theo nhãn này sẽ mang theo sai số đó.
