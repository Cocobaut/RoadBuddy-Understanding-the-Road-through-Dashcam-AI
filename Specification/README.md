# 📑 Thư mục Đặc tả Kỹ thuật Cuộc thi (Specification Directory)

Thư mục `Specification/` tập hợp các tài liệu đặc tả chi tiết được biên soạn trực tiếp từ thông số chính thức của cuộc thi **Zalo AI Challenge - RoadBuddy: Understanding the Road through Dashcam AI**.

---

## 📂 Danh mục tài liệu đặc tả

| Tài liệu | File ảnh nguồn | Tóm tắt nội dung chính |
| :--- | :--- | :--- |
| [**1. Tổng quan & Bài toán (Overview.md)**](./Overview.md) | `Overview.png` | Mục tiêu trợ lý lái xe ảo, bài toán VLM/LLM đa phương thức, bối cảnh giao thông Việt Nam, đặc tả Input (video $5-15\text{s}$ + câu hỏi) và Output (đáp án pháp lý). |
| [**2. Dữ liệu & Định dạng nộp bài (Data.md)**](./Data.md) | `Data.png` | Quy mô tập dữ liệu (Train $\sim 600$ videos/1000 samples, Public Test $\sim 300$ videos/500 samples, Private Test $\sim 300$ videos/500 samples), schema JSON chi tiết và định dạng file CSV nộp bài. |
| [**3. Phương pháp đánh giá & Xếp hạng (Evaluation.md)**](./Evaluation.md) | `Evaluation.png` | Cách tính điểm độ chính xác (Accuracy), giới hạn thời gian thực thi $\le 30\text{s}$/test case, chế tài trừ điểm quá giờ và quy tắc phân hạng ưu tiên theo thời gian suy luận (Tie-breaker). |
| [**4. Quy chế & Ràng buộc kỹ thuật (Rule.md)**](./Rule.md) | `Rule.png` | Ràng buộc kích thước mô hình ($\le 9\text{B}$ tham số), cấu hình phần cứng Docker (1 GPU RTX 3090/A30, 16 CPU cores, 64GB RAM), quy định chạy Offline hoàn toàn và giao thức kiểm thử 3 bước. |

---

## 🖼️ Bộ ảnh quy chuẩn gốc

<p align="center">
  <img src="../Overview.png" width="48%" alt="Overview"/>
  <img src="../Data.png" width="48%" alt="Data"/>
</p>
<p align="center">
  <img src="../Evaluation.png" width="48%" alt="Evaluation"/>
  <img src="../Rule.png" width="48%" alt="Rule"/>
</p>

---

> Trở về trang tài liệu chính của dự án: [README.md](../README.md)
