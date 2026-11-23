# Kết quả nghiên cứu | Kiểm định credit scorecard

**Kết luận: Đạt có điều kiện trong phạm vi nghiên cứu.** Scorecard phân hạng rủi ro tốt trên 37.500 hồ sơ validation. Benchmark có AUC cao hơn. Mức rủi ro dự báo của một số nhóm còn lệch so với tỷ lệ quá hạn thực tế, nên cần xác nhận calibration trước khi dùng xác suất để quyết định tín dụng.

## Từ hồ sơ khách hàng đến kết quả dự báo

Mục tiêu là ước tính xác suất một khách hàng quá hạn từ 90 ngày trở lên trong vòng hai năm. Từ 150.000 hồ sơ có nhãn, mô hình dùng 112.500 hồ sơ để phát triển và giữ riêng 37.500 hồ sơ để kiểm định, với stratified split và seed 42. Tập cs-test không có nhãn, dùng để chấm điểm và so sánh phân phối.

1. **Xử lý dữ liệu.** Tách mã quá hạn 96/98 thành nhóm riêng, xử lý tuổi ngoài phạm vi giả định, cap hai biến tỷ lệ ở phân vị 99,5% của train. Có 202 hồ sơ train mang mã 96/98; ý nghĩa nghiệp vụ của mã vẫn cần xác minh.
2. **Chấm từng hồ sơ.** Mỗi giá trị đầu vào được gán vào bin đã xác định trên train. Mô hình dùng WOE của từng bin, chuẩn hóa WOE rồi kết hợp bằng Logistic Regression để trả về xác suất quá hạn. Scorecard cuối giữ 8 biến. Xác suất được quy đổi thành điểm và 7 risk grade để đối chiếu với bad rate thực tế.
3. **Kiểm định.** So sánh với HistGradientBoosting trên cùng validation, dùng 5-fold CV trên train và paired bootstrap trên validation. Các threshold được chọn từ dự báo out-of-fold của train.

## Khả năng phân biệt

| Metric trên validation | Scorecard gốc | Scorecard v2 | HistGradientBoosting |
|---|---:|---:|---:|
| AUC | 0,85869 | 0,85970 | 0,86814 |
| Gini | 0,71738 | 0,71940 | 0,73628 |
| KS | 0,56096 | 0,56120 | 0,57986 |

Khoảng tin cậy 95% cho AUC của scorecard là **[0,85171; 0,86697]**. Với cùng hồ sơ validation, chênh lệch AUC giữa benchmark và scorecard có khoảng tin cậy **[0,00604; 0,01080]**. Kết quả 5-fold CV nhất quán với thứ tự này: AUC trung bình của scorecard là 0,85623 (độ lệch chuẩn 0,00406), của benchmark là 0,86456 (0,00390).

Sau khi thay binning, giữ đủ biến cho AUC 0,859762. Loại MonthlyIncome và NumberOfOpenCreditLinesAndLoans theo kiểm tra hệ số trên train làm AUC giảm nhẹ xuống 0,859699. Lợi ích của bước này là mô hình gọn hơn và dễ giải thích dấu hệ số.

## Threshold ảnh hưởng thế nào đến quyết định?

| Quy tắc chọn trên train | Threshold | Hồ sơ bị gắn cờ | Recall | Precision |
|---|---:|---:|---:|---:|
| Youden | 0,06542 | 24,54% | 76,66% | 20,87% |
| Chi phí bỏ sót:gắn cờ nhầm = 3:1 | 0,20710 | 6,75% | 44,01% | 43,58% |
| Chi phí bỏ sót:gắn cờ nhầm = 5:1 | 0,14345 | 9,69% | 53,71% | 37,05% |
| Chi phí bỏ sót:gắn cờ nhầm = 10:1 | 0,08455 | 18,91% | 70,11% | 24,78% |

Ví dụ, tại threshold Youden, trên validation mô hình bỏ sót **585** khách hàng quá hạn và gắn cờ nhầm **7.283** khách hàng. Nếu khâu thẩm định chỉ có thể xem xét khoảng 10% hồ sơ, kịch bản 5:1 minh họa mức recall có thể đạt trong giới hạn đó. Các tỷ lệ chi phí là giả định nghiên cứu.

## Calibration, độ ổn định và điều kiện sử dụng

- **Calibration.** Mean PD là 6,652%, gần bad rate thực tế 6,683%. Tuy vậy, ở grade an toàn nhất, mean PD khoảng 0,94% trong khi bad rate là 0,50%. Cần kiểm tra calibration theo nhóm trước khi dùng PD để ước tính tổn thất.
- **Risk grade.** Bad rate giảm đơn điệu từ 29,80% ở grade rủi ro nhất xuống 0,50% ở grade an toàn nhất. Đây là bằng chứng về thứ tự rủi ro trên validation.
- **Monitoring.** Score PSI giữa train và test là 0,00025. Hai mẫu cùng nguồn và thiếu thời điểm quan sát. Đánh giá độ ổn định qua kỳ cần có hồ sơ kỳ sau cùng kết quả trả nợ đã đủ thời gian quan sát.
- **Phạm vi áp dụng.** Trước khi dùng tại một công ty tài chính Việt Nam, cần xác minh định nghĩa biến và mã đặc biệt, kiểm định trên dữ liệu địa phương, rà soát sai lệch do chỉ quan sát người được duyệt và chốt chi phí sai phân loại với nghiệp vụ.

[Nguồn số liệu, bảng chi tiết và code tái lập](outputs/GiveMeSomeCredit_Model_Validation_v2.ipynb)