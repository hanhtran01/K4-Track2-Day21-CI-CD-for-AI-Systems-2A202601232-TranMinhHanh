# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Trần Minh Hạnh |
| MSSV | 2A202601232 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/hanhtran01/K4-Track2-Day21-CI-CD-for-AI-Systems-2A202601232-TranMinhHanh |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.874 |
| 4 | 200 | 0.05 | 4 | 0.7222 | 0.880 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.05`, `max_depth=4`.

**Lý do:** Bộ này cho f1_score cao nhất trong bốn lần chạy, vượt ngưỡng triển khai 0.65 một khoảng an toàn. So sánh giữa các lần chạy cho thấy hai chỉ số biến động rất khác nhau: f1_score trải trên 11,7 điểm phần trăm (0.605 đến 0.722) trong khi accuracy chỉ trải 3,4 điểm (0.846 đến 0.880). Đáng chú ý hơn là thứ tự xếp hạng không trùng nhau. Lần chạy 3 có f1 cao hơn lần 2 (0.7149 so với 0.7109) nhưng accuracy lại thấp hơn (0.874 so với 0.878). Nếu chọn mô hình theo accuracy thì đã chọn nhầm lần 2. Về đánh đổi giữa hai tham số, khi hạ learning_rate từ 0.1 xuống 0.05 phải nâng n_estimators lên 200 để bù lại; bộ 50 cây với learning_rate 0.05 học chưa đủ nên f1 tụt xuống 0.6051, dưới ngưỡng.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập Adult mất cân bằng lớp: chỉ 24,8% số mẫu thuộc nhóm thu nhập trên 50K. Hệ quả là một mô hình vô dụng, luôn trả lời "thu nhập thấp" cho mọi đầu vào, vẫn đạt accuracy 0,752 mà không bắt được một trường hợp thu nhập cao nào. Con số 0,752 gây hiểu nhầm vì nó chỉ phản ánh tỷ lệ lớp đa số, không phản ánh năng lực phân loại. Nếu đặt ngưỡng chất lượng trên accuracy thì mô hình rỗng đó vẫn được triển khai.

F1 của lớp dương là trung bình điều hòa của precision và recall trên riêng nhóm thu nhập cao, nên nó bằng 0 với mô hình nói trên. Đây chính là thứ accuracy không đo được. Cũng vì vậy không được truyền `average="weighted"` hay `average="macro"` khi gọi `f1_score`: cả hai đều gộp thêm điểm số của lớp đa số, kéo giá trị lên cao và làm ngưỡng 0.65 mất ý nghĩa.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lệch phiên bản scikit-learn giữa huấn luyện và phục vụ | Model pickle bằng 1.4.2, VM cài 1.7.2 nên báo `AttributeError: __pyx_unpickle_CyHalfBinomialLoss` | Pin `scikit-learn==1.4.2`, `joblib==1.4.2`, `numpy==1.26.4` trên VM cho khớp môi trường train |
| `AccessDenied` khi tạo S3 bucket | Nhóm IAM được cấp EC2, VPC, IAM nhưng thiếu hoàn toàn quyền S3 | Dùng `IAMFullAccess` tự tạo policy `IncomeLabS3` giới hạn ở prefix `income-lab-*`, theo nguyên tắc quyền tối thiểu |
| DVC remote trỏ về `gs:///dvc` | Chạy lệnh mẫu dạng bash `gs://$BUCKET/dvc` trong PowerShell nên biến nở thành chuỗi rỗng | Sửa bằng `dvc remote modify labstore url s3://income-lab-hanh-2a202601232/dvc` |
| `IndentationError` ở job Train | Heredoc Python trong khối `run: \|` bị thụt lề sâu hơn các dòng còn lại, YAML chỉ cắt phần thụt chung | Căn đều các dòng trong heredoc, đồng thời thêm khối `env:` vì secret không tự thành biến môi trường của Python |

Lab triển khai trên AWS thay vì GCP, nên `STORAGE_CREDENTIALS` được tách thành ba secret `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`. VM đọc model qua IAM Role gắn vào instance thay vì chép file khóa lên đĩa.

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7222 | 0.880 |
| Bước 3 (thêm `train_batch2`) | ___ | ___ |

**Nhận xét:** ___
