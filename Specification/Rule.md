# ⚖️ Specification: Quy chế & Ràng buộc kỹ thuật (Competition Rules)

> **Tài liệu tham chiếu:** `Rule.png` thuộc bài thi **Zalo AI Challenge - RoadBuddy: Understanding the Road through Dashcam AI**.

---

<p align="center">
  <img src="../Rule.png" alt="Rule Specification" width="850"/>
</p>

---

## 1. Ràng buộc kỹ thuật cốt lõi (Core Technical Constraints)

Các đội tham dự cần đáp ứng đầy đủ các tiêu chuẩn bắt buộc sau trong giải pháp cuối cùng:

| Tiêu chuẩn | Quy định chi tiết | Lưu ý thực thi trong hệ thống |
| :--- | :--- | :--- |
| **Kích thước mô hình (Model Size)** | $\le 9\text{B}$ tham số (*parameters*) cho **từng mô hình đơn lẻ** tại thời điểm suy luận. | Được phép kết hợp nhiều mô hình nhỏ (ensemble/pipeline), miễn sao mỗi mô hình $\le 9\text{B}$. |
| **Thời gian suy luận (Inference Time)** | $\le 30\text{s}$ cho mỗi mẫu kiểm thử (*sample/testcase*). | Vượt quá $30\text{s}$ bị tính 0 điểm ngay lập tức. |
| **Môi trường phần cứng (Hardware)** | **1 GPU:** NVIDIA RTX 3090 (24GB VRAM) hoặc NVIDIA A30 (24GB VRAM)<br>**CPU:** 16 Cores Intel(R) Xeon(R) Gold 6442Y<br>**RAM:** 64GB | Mọi mô hình phải nạp vừa vặn trong 24GB VRAM mà không bị OOM (*Out Of Memory*). |
| **Kết nối mạng (Internet Access)** | **Hoàn toàn KHÔNG có Internet** trong suốt quá trình chạy đánh giá nghiệm thu. | Mọi trọng số mô hình (*weights*), cơ sở dữ liệu vector (*FAISS index*), thư viện phụ thuộc phải được đóng gói sẵn offline. |
| **Dữ liệu & Mô hình nguồn mở** | **Được phép** sử dụng các nguồn dữ liệu và mô hình mã nguồn mở hợp lệ (*Open-source data/models*). | Ví dụ: YOLOv11, ByteTrack, VietOCR, SigLIP, Qwen2.5-VL, Llama-3.1. |
| **Dữ liệu tổng hợp (Synthetic Data)** | **Được phép** sinh dữ liệu nhân tạo thông qua dịch vụ hoặc mô hình khác (LLM, VLM...). | Hữu ích cho việc làm giàu mẫu câu hỏi và sinh bổ sung tình huống luật. |
| **Bảo mật dữ liệu (Data Privacy)** | Sau khi cuộc thi kết thúc, các thí sinh cam kết **không lưu trữ hoặc phát tán** dữ liệu huấn luyện cho mục đích cá nhân. | Tuân thủ cam kết bản quyền của ban tổ chức. |

---

## 2. Phụ lục giải thích chi tiết (Appendix: Clarifications)

### 📌 1. Về kích thước mô hình (Model Size $\le 9\text{B}$)
- Ban tổ chức điều chỉnh giới hạn từ $8\text{B}$ lên **$9\text{B}$** để phù hợp với kích thước thực tế của các mô hình open-source phổ biến hiện nay (như Llama-3.1-8B, Qwen2.5-8B, v.v.).
- Mỗi mô hình đơn lẻ trong hệ thống không được vượt quá $9\text{B}$ tham số. Tổng tham số của toàn bộ hệ thống (gộp các mô hình phát hiện, nhận dạng, embedding, LLM) có thể vượt quá $9\text{B}$, nhưng thí sinh cần lưu ý:
  > *Việc sử dụng quá nhiều mô hình hoặc mô hình cồng kềnh sẽ làm tăng thời gian tính toán và có nguy cơ tràn bộ nhớ GPU (24GB VRAM RTX 3090 / A30).*

### ⏱️ 2. Cách tính thời gian suy luận cho mỗi Test Case
- Thời gian suy luận được đo lường chính xác từ **thời điểm hệ thống nhận đầu vào** (`question` và `video_path`) cho đến **thời điểm trả về đáp án** cho test case đó.
- Mọi khâu tiền xử lý (đọc video, trích frame, chạy detection, OCR, trích xuất embedding, RAG retrieval) đều được tính vào thời gian suy luận.
- **Tối ưu hóa tài nguyên:** Tất cả mô hình và tài nguyên tĩnh (weights, tokenizer, index) **phải được tải trước lên bộ nhớ (pre-loaded)** trước khi bước vào vòng lặp kiểm thử.

### 🧪 3. Quy chuẩn cấu trúc mã nguồn kiểm thử (3-Step Evaluation Protocol)
File script dự đoán nghiệm thu của đội thi (Python script hoặc Jupyter Notebook chạy trong Docker) bắt buộc phải tuân theo cấu trúc 3 bước sau:

```python
# ==============================================================================
# BƯỚC 1: Tải trước toàn bộ mô hình và tài nguyên lên bộ nhớ (Preload Resources)
# (Thời gian ở bước này KHÔNG bị tính vào thời gian suy luận của từng mẫu)
# ==============================================================================
perception_models = load_perception_models()  # YOLO, ByteTrack, VietOCR
rag_retriever = load_vector_db_and_bm25()     # FAISS + Legal Database
reasoning_engine = load_vlm_or_llm_model()    # Qwen2.5-VL / Llama-3.1

# ==============================================================================
# BƯỚC 2: Đọc file dữ liệu kiểm thử (Load Test Cases)
# ==============================================================================
with open(test_json_path, "r", encoding="utf-8") as f:
    test_data = json.load(f)

# ==============================================================================
# BƯỚC 3: Lặp qua từng test case, sinh dự đoán và đo thời gian
# ==============================================================================
results = []
for sample in test_data:
    start_time = time.time()
    
    # Thực hiện suy luận trọn gói cho sample
    predicted_answer = predict_sample(
        sample["question"], 
        sample["video_path"], 
        sample["choices"]
    )
    
    elapsed_time = time.time() - start_time
    assert elapsed_time <= 30.0, f"Timeout on sample {sample['id']}"
    
    results.append({"id": sample["id"], "answer": predicted_answer})
```

---

## 3. Kiến trúc RoadBuddy thích ứng với Quy chế

Để tuân thủ hoàn hảo các điều kiện nghiêm ngặt trên, RoadBuddy áp dụng các kỹ thuật kỹ thuật cốt lõi:
1. **Lượng tử hóa & Tinh chỉnh hiệu quả (Quantization & QLoRA):**
   - Mô hình suy luận chính **Qwen2.5-VL-7B** sử dụng QLoRA ($r=64, \alpha=128$) hoặc **Llama-3.1-8B** lượng tử hóa 4-bit (AWQ/GPTQ) đảm bảo VRAM chỉ tiêu tốn $\approx 8 - 14\text{GB}$, vận hành ổn định trên 1 GPU RTX 3090 / A30 (24GB).
2. **Offline Local Vector DB:**
   - Cơ sở dữ liệu pháp luật QCVN 41:2019 và Nghị định được lập chỉ mục cục bộ bằng FAISS và BM25, không gửi bất kỳ request ra bên ngoài nào.
3. **Cơ chế Pipeline phân tách:**
   - Các mô hình phụ trợ (YOLOv11x, UFLDv2, DBNet/VietOCR) được tối ưu hóa luồng suy luận tuần tự, tránh tải đồng thời làm tràn bộ nhớ VRAM.
