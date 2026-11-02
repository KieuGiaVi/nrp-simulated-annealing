# Simulated Annealing cho bài toán xếp lịch trực điều dưỡng trên bộ dữ liệu chuẩn

Mã nguồn, kết quả thực nghiệm và lời giải kèm bài báo của **Kiều Gia Vĩ** và **Trần Bá Minh Sơn** (Viện Công nghệ Số, Trường Đại học Thủ Dầu Một), gửi Tạp chí Khoa học Trường Đại học Thủ Dầu Một.

*English summary: code, raw results and rosters for a study of a simple Simulated Annealing (SA) for the Nurse Rostering Problem on Instances 1–12 of the Curtois & Qu (2014) benchmark, compared with Hill Climbing (HC), random-restart HC and the HiGHS MILP solver under 60 s and 600 s budgets. Everything runs from one Jupyter/Colab notebook on a CPU.*

## Nội dung kho

```
NRP_Simulated_Annealing.ipynb     notebook duy nhất: đọc dữ liệu, bộ kiểm tra lời giải độc lập,
                                  Greedy, HC, SA, mô hình MILP (HiGHS), thực nghiệm, thống kê, hình vẽ
requirements.txt                  phiên bản thư viện đã dùng
nrp_sa_experiment/                kết quả (đúng cấu trúc thư mục OUT_DIR của notebook)
├── results_raw.csv               thực nghiệm gốc: từng lần chạy Greedy, MILP 600 s, HC và SA 60 s × 10 lần
├── table_summary.csv             bảng tổng hợp thực nghiệm gốc
├── table_instances.csv           đặc trưng 12 tập dữ liệu
├── solutions/                    lịch tốt nhất của mỗi (tập, phương pháp), dạng .txt và .json
├── figures/                      hình vẽ
└── revision/                     thực nghiệm bổ sung theo ý kiến phản biện (R0–R10)
    ├── environment.json          cấu hình máy và phiên bản thư viện của lần chạy
    ├── milp_model_size.csv       số biến, số ràng buộc, số phần tử khác 0 của mô hình MILP
    ├── highs_budget.csv          HiGHS với ngân sách 60 s
    ├── min_shortage.csv          số người thiếu tối thiểu có thể đạt (MILP chỉ tối thiểu hóa thiếu người)
    ├── sa_budget.csv             SA 60 s và 600 s, tập 5–12, seed 0–4 (cùng máy)
    ├── rrhc.csv                  HC khởi động lại ngẫu nhiên, 60 s × 10 lần
    ├── sensitivity.csv           độ nhạy T0, Tf, λ (tập 3, 5, 9, 11; 7 cấu hình × 5 lần)
    ├── coverage_best_solutions.csv  thiếu/thừa người và chi phí nguyện vọng của các lịch tốt nhất
    ├── traces_*.json             chi phí tốt nhất theo thời gian của từng lần chạy
    ├── revision_summary.xlsx     các bảng tổng hợp và kiểm định
    └── solutions_runs.zip        lịch trực của TỪNG lần chạy bổ sung (350 file .json)
```

Mỗi dòng trong các file `.csv` là một lần chạy độc lập. Các cột chính: `cost` (điểm phạt mềm, chỉ có nghĩa khi `feasible = True`), `gap` (% so với BKS), `time_to_best`, `first_feasible` (thời điểm có lịch khả thi đầu tiên), `search_time`, `setup_time` (tiền xử lý và lời giải khởi tạo, nằm ngoài đồng hồ tìm kiếm), `overrun` (mức vượt ngân sách), `under_staff` / `over_staff` (tổng số lượt thiếu / thừa người), `under_cost`, `over_cost`, `pref_cost` (chi phí nguyện vọng cá nhân), `viol_*` (số trường hợp vi phạm từng ràng buộc cứng theo bộ kiểm tra độc lập).

## Dữ liệu

Nurse Rostering Benchmark Instances (Curtois, T., & Qu, R., 2014), lưu trữ tại <https://schedulingbenchmarks.org/nrp/>. Dữ liệu **không** được đưa vào kho này; notebook tự tải `Instance1.txt` … `Instance12.txt` từ trang gốc. Giá trị BKS lấy từ cùng trang (truy cập 07/10/2026).

## Môi trường đã dùng

Google Colab (miễn phí), Intel Xeon CPU @ 2.20 GHz, 2 CPU logic, 13,6 GB RAM; Python 3.13.18, NumPy 2.1.3, SciPy 1.16.3, pandas 2.2.3. HiGHS được gọi qua `scipy.optimize.milp` với thiết lập luồng mặc định. SA và HC viết bằng Python thuần, chạy đơn luồng. Chi tiết trong `nrp_sa_experiment/revision/environment.json`.

## Chạy lại

**Xem lại bảng và hình từ kết quả có sẵn (vài phút):** mở notebook bằng Jupyter hoặc Google Colab (cùng thư mục với `nrp_sa_experiment/`), chọn *Run all*. Notebook bỏ qua mọi lần chạy đã có trong các file `.csv` và chỉ tính lại bảng, kiểm định, hình.

**Chạy lại toàn bộ:** xóa (hoặc đổi tên) thư mục `nrp_sa_experiment/`, đặt `RUN_ORIGINAL = True` ở mục 10, rồi *Run all*. Thời gian: thực nghiệm gốc khoảng 4–6 giờ, thực nghiệm bổ sung khoảng 12 giờ. Trên Colab nên đặt `USE_DRIVE = True` để kết quả lưu lên Google Drive và chạy tiếp được nếu bị ngắt.

**Lưu ý về tái lập:** SA làm nguội theo thời gian thực, nên cùng seed trên máy nhanh hơn hoặc chậm hơn sẽ thực hiện số bước lặp khác nhau và cho kết quả hơi khác. Không chạy song song nhiều thực nghiệm trên cùng máy. Mọi lịch trả về đều được kiểm tra lại bằng hàm `evaluate()` viết độc lập với hàm đánh giá dùng trong tìm kiếm.

`traces.json` (đường hội tụ) của thực nghiệm gốc không được lưu giữ; đường hội tụ và tỷ lệ lần chạy khả thi theo thời gian được vẽ từ dữ liệu R3–R4 (`traces_sa_budget.json`, hình `figures/fig_sa_60_vs_600.png`).

## Giấy phép

Mã nguồn phát hành theo giấy phép MIT (xem `LICENSE`). Bộ dữ liệu thuộc về tác giả gốc (Curtois & Qu).
