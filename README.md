# Give Me Some Credit | Credit Scorecard Validation

Project kiểm định một **WOE Logistic Scorecard** dự báo khách hàng quá hạn từ 90 ngày trở lên trong vòng hai năm. Bài phân tích tái lập mô hình gốc, kiểm tra các giả định về dữ liệu và binning, rồi đối chiếu với HistGradientBoosting trên cùng tập validation.

## Kết quả kiểm định

| Metric trên 37.500 hồ sơ validation | Scorecard gốc | Scorecard v2 |
|---|---:|---:|
| AUC | 0,858692 | 0,859699 |
| Gini | 0,717383 | 0,719397 |
| KS | 0,560965 | 0,561199 |

**Đạt có điều kiện trong phạm vi nghiên cứu**, theo tiêu chí giả định của tác giả. Bad rate giảm từ 29,80% xuống 0,50% qua 7 risk grade. HistGradientBoosting đạt AUC 0,86814. Khoảng tin cậy 95% cho chênh lệch AUC giữa benchmark và scorecard là [0,00604; 0,01080] theo paired bootstrap.

| Câu hỏi kiểm định | Phương pháp và phát hiện |
|---|---|
| Các giá trị bất thường ảnh hưởng thế nào? | Có 202 hồ sơ train mang mã 96/98 ở các biến quá hạn. Scorecard tách bin riêng. Ý nghĩa nghiệp vụ của hai mã vẫn cần xác minh. |
| Có nên giữ đủ biến? | Rà soát WOE, dấu hệ số, VIF và khoảng tin cậy. Với cùng binning mới, loại hai biến làm AUC giảm nhẹ từ 0,859762 xuống 0,859699. |
| Threshold có được đánh giá độc lập? | Chọn threshold bằng dự báo out-of-fold trên train, sau đó đánh giá trên validation. So sánh Youden với các kịch bản chi phí bỏ sót và từ chối nhầm. |
| PD có sát thực tế? | Mean PD đạt 6,652%, gần bad rate 6,683%. Phân tích theo decile vẫn cho thấy sai lệch cục bộ cần kiểm tra trên mẫu khác. |
| Mô hình có ổn định theo thời gian? | Score PSI giữa train và test là 0,00025. Dữ liệu thiếu thời điểm quan sát để thực hiện out-of-time validation. |

Mẫu Kaggle có giới hạn khi suy rộng sang khách hàng Việt Nam. Dữ liệu cũng thiếu kết quả trả nợ của người bị từ chối vay để đánh giá tác động của chính sách phê duyệt.

## Chạy lại trên máy

1. Tải và giải nén dữ liệu từ [Give Me Some Credit trên Kaggle](https://www.kaggle.com/competitions/GiveMeSomeCredit/data). Đặt bốn file dưới đây cạnh README. Nếu dùng thư mục khác, đặt biến môi trường `GMC_DATA_DIR` trỏ tới thư mục chứa các file.

   ```text
   cs-training.csv
   cs-test.csv
   sampleEntry.csv
   Data Dictionary.xls
   ```

2. Từ thư mục gốc của repo, tạo môi trường Python 3.11 và cài các phiên bản thư viện đã dùng. Ví dụ trên Windows PowerShell:

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\python.exe -m pip install -r requirements.txt
   ```

3. Mở notebook bằng VS Code với extension Jupyter, chọn kernel Python trong `.venv` và bấm **Run All**. Báo cáo ở đầu notebook cập nhật khi ô tổng hợp chạy xong. Notebook dùng seed 42 và xuất `submission_scorecard_v2.csv` vào thư mục làm việc của kernel.

Notebook đã lưu output nên có thể đọc kết quả ngay trên GitHub. Dữ liệu nguồn và các file submission được loại khỏi danh sách file đưa lên Git.
