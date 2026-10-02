# 📈 Specification: Phương pháp đánh giá & Xếp hạng (Evaluation)

> **Tài liệu tham chiếu:** `Evaluation.png` thuộc bài thi **Zalo AI Challenge - RoadBuddy: Understanding the Road through Dashcam AI**.

---

<p align="center">
  <img src="../Evaluation.png" alt="Evaluation Specification" width="850"/>
</p>

---

## 1. Quy trình đánh giá (Evaluation Process)

1. **Thực thi kiểm thử độc lập (Test Execution):**
   - Từng mẫu kiểm thử (*test case*) trong tập dữ liệu kiểm thử sẽ được đưa qua hệ thống suy luận của đội thi để dự đoán câu trả lời.
   - Quá trình chạy diễn ra tuần tự hoặc theo batch trong môi trường máy chủ Docker khép kín do ban tổ chức thiết lập.

2. **Giới hạn thời gian tối đa (Maximum Runtime Limit):**
   - **Thời gian chạy tối đa cho mỗi mẫu kiểm thử:** $\le 30 \text{ giây / test case}$.
   - Thời gian được tính từ thời điểm script nhận đầu vào (câu hỏi và đường dẫn video) cho tới khi kết xuất ra đáp án cuối cùng.

---

## 2. Cách tính điểm (Scoring Metric)

- **Điểm cho từng mẫu kiểm thử:**
  - **1 điểm:** Nếu đáp án dự đoán trùng khớp hoàn toàn với đáp án chuẩn (*ground truth*).
  - **0 điểm:** Nếu đáp án dự đoán sai.
  - **0 điểm (Phạt quá giờ):** Bất kỳ câu trả lời nào vượt quá giới hạn thời gian $30\text{s}$ đều tự động nhận **0 điểm** cho test case đó, bất kể đáp án có đúng hay không.

- **Điểm tổng kết hệ thống (System Score):**
  - Điểm của hệ thống được tính bằng tỷ lệ số lượng câu trả lời đúng trên tổng số câu hỏi trong tập kiểm thử.

### 📐 Công thức tính độ chính xác (Formula):

$$\text{Accuracy} = \frac{\text{Number of correct answers}}{\text{Total number of test cases}}$$

---

## 3. Tiêu chí phân định thứ hạng khi bằng điểm (Tie-breaker Rule)

- Trong trường hợp hai hoặc nhiều đội thi đạt **cùng điểm số Accuracy**, thứ hạng chung cuộc sẽ được phân định dựa trên **thời gian suy luận trung bình (Average Inference Time)** trên toàn bộ tập kiểm thử.
- **Quy tắc xếp hạng:** Đội nào có thời gian suy luận trung bình **thấp hơn (nhanh hơn)** sẽ xếp hạng cao hơn.

$$\text{Ranking Priority: } \text{Accuracy} \uparrow \longrightarrow \text{Average Inference Time} \downarrow$$

---

## 4. Chiến lược tối ưu cho hệ thống RoadBuddy

Để tối đa hóa điểm số và ưu thế tie-breaker theo bảng tiêu chí trên, kiến trúc RoadBuddy đã được thiết kế chuyên biệt:
1. **Kiểm soát thời gian nghiêm ngặt:** Luồng pipeline (Stage 1 đến Stage 4) được tối ưu hóa chỉ dao động trong khoảng $2.5\text{s} - 8\text{s}$ cho mỗi test case (thấp hơn nhiều so với mốc trần $30\text{s}$).
2. **Perception Cache Engine:** Sử dụng hàm băm video SHA-256 để lưu trữ và tái sử dụng kết quả trích xuất nhận diện (YOLO/ByteTrack/OCR), giảm thời gian xử lý xuống dưới $1.5\text{s}$ khi cần thử nghiệm lặp lại.
3. **Hybrid RAG + Tri-Modal Reasoning:** Tăng cường tính chính xác của đáp án thông qua việc đối chiếu trực tiếp điều luật QCVN 41:2019 và kiểm chứng chéo giữa mô hình ngôn ngữ và thị giác.
