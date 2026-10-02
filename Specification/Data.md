# 📊 Specification: Dữ liệu & Định dạng nộp bài (Data & Submission)

> **Tài liệu tham chiếu:** `Data.png` thuộc bài thi **Zalo AI Challenge - RoadBuddy: Understanding the Road through Dashcam AI**.

---

<p align="center">
  <img src="../Data.png" alt="Data Specification" width="850"/>
</p>

---

## 1. Thống kê tập dữ liệu (Dataset Statistics)

Bộ dữ liệu được ban tổ chức phân bổ thành 3 tập chính:

| Tập dữ liệu (Dataset Split) | Số lượng Video | Số lượng Mẫu (Samples) | Trường dữ liệu cung cấp |
| :--- | :--- | :--- | :--- |
| **Training Data** | $\sim 600$ videos | $\sim 1000$ samples | `id`, `question`, `choices`, `answer`, `support_frames`, `video_path` |
| **Public Test** | $\sim 300$ videos | $\sim 500$ samples | `id`, `question`, `choices`, `video_path` *(không có answer & support_frames)* |
| **Private Test** | $\sim 300$ videos | $\sim 500$ samples | `id`, `question`, `choices`, `video_path` *(dùng để chấm điểm chung cuộc)* |

---

## 2. Cấu trúc nhãn dữ liệu (Data Annotation Schema)

Tập dữ liệu huấn luyện bao gồm file nhãn `train.json` và thư mục chứa các video hành trình tương ứng. Mỗi phần tử trong `train.json` có cấu trúc:

- **`id`** (`string`): Mã định danh duy nhất cho từng câu hỏi/mẫu kiểm thử.
- **`video_path`** (`string`): Đường dẫn tương đối trỏ tới file video hành trình MP4 phục vụ câu hỏi.
- **`question`** (`string`): Nội dung câu hỏi truy vấn về tình huống trong video bằng tiếng Việt.
- **`choices`** (`array<string>`): Danh sách $4$ phương án lựa chọn trắc nghiệm (A, B, C, D).
- **`answer`** (`string`): Đáp án chính xác của câu hỏi (chỉ có trong tập `train.json`).
- **`support_frames`** (`array<float>`): Mốc thời gian (timestamp tính bằng giây) trong video chứa khung hình bằng chứng để trả lời câu hỏi.

### 📝 Ví dụ mẫu dữ liệu JSON chuẩn (Sample Annotation)

```json
{
  "id": "train_0001",
  "question": "Trong video này, vạch kẻ đường dạng chữ viết trên mặt đường có ý nghĩa gì?",
  "choices": [
    "A. Làn đường dành cho xe buýt",
    "B. Làn thu phí không dừng",
    "C. Làn đường dành cho xe tải",
    "D. Khu vực được phép quay đầu xe"
  ],
  "answer": "B. Làn thu phí không dừng",
  "support_frames": [
    5.05
  ],
  "video_path": "traffic_buddy_train/videos/train_0001.mp4"
}
```

> **Lưu ý:**
> Tập **Public Test** và **Private Test** có cấu trúc JSON tương đương, ngoại trừ hai trường `answer` và `support_frames` đã được ban tổ chức ẩn đi nhằm mục đích đánh giá độc lập.

---

## 3. Định dạng file nộp bài (Submission Format)

Mỗi đội thi xây dựng mô hình dự đoán đáp án trắc nghiệm cho từng câu hỏi trong tập kiểm thử (Public Test / Private Test) và xuất ra file nộp dạng **CSV** theo đúng quy định sau:

- Định dạng file: `submission.csv`
- Cột dữ liệu: Gồm 2 cột `id` và `answer`
- Ký tự đáp án: Chỉ ghi ký tự chữ cái đại diện cho đáp án (`A`, `B`, `C`, hoặc `D`).

### 📄 Cấu trúc file CSV mẫu:

```csv
id,answer
testa_0001,A
testa_0002,B
testa_0003,D
testa_0004,C
```

---

## 4. Hướng dẫn dữ liệu & Docker nghiệm thu (Guidelines)

- **Training data & Public test:** Được cung cấp chính thức qua cổng thi đấu để các đội huấn luyện và đánh giá mô hình cục bộ.
- **Private test:** Dùng để đánh giá thứ hạng cuối cùng thông qua quy trình Docker tự động không có kết nối Internet.
- **Docker Guideline for final Solution:** Cần tuân thủ cấu trúc nạp mã nguồn, mô hình đã huấn luyện và script chạy kiểm thử offline theo đúng chuẩn môi trường quy định.
