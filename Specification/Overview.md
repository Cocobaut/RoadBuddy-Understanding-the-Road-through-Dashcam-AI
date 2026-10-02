# 📋 Specification: Bài toán & Tổng quan (Overview)

> **Tài liệu tham chiếu:** `Overview.png` thuộc bài thi **Zalo AI Challenge - RoadBuddy: Understanding the Road through Dashcam AI**.

---

<p align="center">
  <img src="../Overview.png" alt="Overview Specification" width="850"/>
</p>

---

## 1. Giới thiệu bài toán (Problem Description)

Giao thông là một trong những vấn đề quan trọng và thách thức hàng đầu trong xã hội hiện đại. Cuộc thi **"RoadBuddy – Understanding the Road through Dashcam AI"** hướng tới mục tiêu xây dựng một **trợ lý lái xe ảo thông minh (Driving Assistant)** có năng lực thấu hiểu nội dung video thu nhận từ camera hành trình (dashcam) của ô tô, từ đó trả lời nhanh chóng và chính xác các câu hỏi liên quan đến:
- Biển báo giao thông (*Traffic signs*)
- Tín hiệu đèn (*Traffic signals*)
- Vạch kẻ và chỉ dẫn di chuyển trên mặt đường (*Driving instructions & Road markings*)

Hệ thống giúp nâng cao tính an toàn khi tham gia giao thông, đảm bảo tuân thủ nghiêm ngặt quy định pháp luật và giảm thiểu sự mất tập trung của người điều khiển phương tiện.

---

## 2. Ý nghĩa ứng dụng thực tiễn (Practical Applications)

Giải pháp phát triển không chỉ giới hạn trong việc trợ lý trực tiếp cho tài xế mà còn mở rộng sang nhiều ứng dụng quan trọng:
1. **Phân tích sau tai nạn (Post-accident analysis):** Tự động tái dựng hiện trường và truy vết chuỗi hành vi từ video hành trình.
2. **Truy xuất bằng chứng (Evidence retrieval):** Định vị nhanh khung hình chứa vi phạm hoặc biển báo mâu thuẫn để phục vụ điều tra, khiếu nại.
3. **Tối ưu hóa logistics & vận tải (Logistics optimization):** Hỗ trợ đội xe vận tải kiểm soát tuyến đường, hạn chế vi phạm biển cấm giờ/cấm tải trọng.
4. **Làm giàu dữ liệu hạ tầng & bản đồ số (Enriching map & infrastructure data):** Tự động phát hiện, cập nhật hệ thống biển báo giao thông mới từ nguồn dữ liệu camera cộng đồng.

---

## 3. Ý nghĩa nghiên cứu khoa học (Research Perspective)

Từ góc độ nghiên cứu học thuật và trí tuệ nhân tạo:
- Thử thách xây dựng bộ tiêu chuẩn đánh giá (**Vietnamese Traffic Benchmark**) được thiết kế chuyên biệt và phù hợp với bối cảnh giao thông đặc thù tại Việt Nam (hỗn hợp xe máy - ô tô, mật độ biển báo dày đặc, điều kiện hạ tầng biến thiên).
- Đẩy mạnh sự kết hợp giữa **Computer Vision truyền thống** (Object Detection, Tracking, OCR) và **Mô hình đa phương thức thế hệ mới** (Vision-Language Models - VLM, Large Language Models - LLM, Retrieval-Augmented Generation - RAG).

---

## 4. Đặc tả Đầu vào & Đầu ra (Input & Output Specification)

### 📥 Đầu vào (Input)
1. **Video hành trình (Traffic Dashcam Video):**
   - Được ghi lại từ camera hành trình gắn trên xe ô tô.
   - **Độ dài:** Từ $5$ đến $15$ giây.
   - **Môi trường & Ngữ cảnh:** Đa dạng điều kiện thực tế bao gồm đô thị (*urban*), đường cao tốc (*highway*), ban ngày (*day*), ban đêm (*night*), trời mưa (*rain*), trời nắng (*sun*).
   - **Thành phần xuất hiện:** Biển báo giao thông, đèn tín hiệu, mũi tên phân làn, vạch kẻ mặt đường, các phương tiện xung quanh, người đi bộ, chướng ngại vật...
2. **Câu hỏi người dùng (User Question):**
   - Câu hỏi trắc nghiệm bằng tiếng Việt liên quan đến tình huống giao thông trong video.

### 📤 Đầu ra (Output)
- **Đáp án tương ứng (Corresponding Answer):** Lựa chọn phương án chính xác nhất cho câu hỏi.
- **Ràng buộc pháp lý:** Tri thức được sử dụng để lập luận và trả lời câu hỏi **bắt buộc phải tuân thủ Luật Giao thông Đường bộ và các Quy chuẩn Kỹ thuật Quốc gia hiện hành của Việt Nam** (đặc biệt là QCVN 41:2019/BGTVT và các Nghị định liên quan).
