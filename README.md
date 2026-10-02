# 🚗 RoadBuddy: Traffic Scene Multi-Modal Reasoning & Legal Grounding System

<p align="center">
  <img src="Overview.png" alt="RoadBuddy Overview" width="850"/>
</p>

<p align="center">
  <a href="#-tổng-quan"><img src="https://img.shields.io/badge/Competition-Zalo_AI_Challenge-blue.svg?style=flat-square" alt="Zalo AI Challenge"/></a>
  <a href="#giai-đoạn-4-tri-modal-reasoning-engine"><img src="https://img.shields.io/badge/VLM-Qwen2.5--VL--7B-orange.svg?style=flat-square" alt="Qwen2.5-VL"/></a>
  <a href="#giai-đoạn-4-tri-modal-reasoning-engine"><img src="https://img.shields.io/badge/LLM-Llama--3.1--8B-purple.svg?style=flat-square" alt="Llama 3.1"/></a>
  <a href="#giai-đoạn-1-temporal-sampling--dual-stream-perception"><img src="https://img.shields.io/badge/Perception-YOLOv11_%7C_UFLDv2-red.svg?style=flat-square" alt="Perception"/></a>
  <a href="#giai-đoạn-3-knowledge-base-rag--vector-database"><img src="https://img.shields.io/badge/Legal_RAG-QCVN_41:2019/BGTVT-green.svg?style=flat-square" alt="Legal RAG"/></a>
  <a href="#giai-đoạn-5-backend-serving--interactive-dashboard"><img src="https://img.shields.io/badge/Serving-FastAPI_%7C_Celery_%7C_Redis-cyan.svg?style=flat-square" alt="Serving"/></a>
</p>

---

## 📖 Mục lục

- [🌟 Tổng quan](#-tổng-quan)
- [🏗️ Kiến trúc tổng thể](#️-kiến-trúc-tổng-thể)
- [🧩 Chi tiết 5 giai đoạn Pipeline](#-chi-tiết-5-giai-đoạn-pipeline)
  - [Giai đoạn 1: Temporal Sampling & Dual-Stream Perception](#giai-đoạn-1-temporal-sampling--dual-stream-perception)
  - [Giai đoạn 2: Keyframe Selection & Temporal Grounding](#giai-đoạn-2-keyframe-selection--temporal-grounding)
  - [Giai đoạn 3: Knowledge Base RAG & Vector Database](#giai-đoạn-3-knowledge-base-rag--vector-database)
  - [Giai đoạn 4: Tri-Modal Reasoning Engine](#giai-đoạn-4-tri-modal-reasoning-engine)
  - [Giai đoạn 5: Backend Serving & Interactive Dashboard](#giai-đoạn-5-backend-serving--interactive-dashboard)
- [⚙️ Ràng buộc kỹ thuật & Benchmark](#️-ràng-buộc-kỹ-thuật--benchmark)
- [📂 Cấu trúc mã nguồn](#-cấu-trúc-mã-nguồn)
- [🚀 Hướng dẫn cài đặt & Triển khai](#-hướng-dẫn-cài-đặt--triển-khai)
- [📜 Quy chuẩn & Tài liệu tham chiếu](#-quy-chuẩn--tài-liệu-tham-chiếu)

---

## 🌟 Tổng quan

**RoadBuddy** là hệ thống thị giác máy tính kết hợp lập luận đa phương thức (*VLM*, *LLM*, *Hybrid RAG*) phân tích tình huống giao thông đường bộ từ video dashcam theo thời gian thực. Hệ thống tự động phát hiện đối tượng, phân tích ngữ cảnh làn đường và đối chiếu trực tiếp với **Quy chuẩn Kỹ thuật Quốc gia QCVN 41:2019/BGTVT**, **Luật Trật tự, An toàn Giao thông Đường bộ**, cùng các **Nghị định xử phạt vi phạm hành chính**.

- **Đầu vào (Input):** Video dashcam hành trình (5 – 15 giây, đa dạng điều kiện thời tiết, ngày/đêm, đô thị/cao tốc) và câu hỏi truy vấn của người dùng.
- **Đầu ra (Output):** Đáp án chính xác kèm trích xuất bằng chứng hình ảnh (*support frames*), bounding box đối tượng vi phạm, căn cứ điều luật cụ thể và chuỗi suy luận (*Chain-of-Thought*).

---

## 🏗️ Kiến trúc tổng thể

```
[Raw Dashcam Video]
         │
         ▼
[Stage 1: Dual-Stream Perception] ──► (YOLOv11, UFLDv2, ByteTrack, DBNet + VietOCR) ──► [Perception Cache]
         │
         ▼
[Stage 2: Temporal Grounding]     ──► (SigLIP Cross-Modal Matching + Gaussian Prior)
         │
         ▼
[Stage 3: Legal RAG Engine]       ──► (Hybrid Search: BM25 Sparse + FAISS/Qdrant Dense)
         │
         ▼
[Stage 4: Tri-Modal Reasoning]   ──► (Qwen2.5-VL / Llama-3.1 Symbolic / Cloud API)
         │
         ▼
[Stage 5: Dashboard Delivery]     ──► (FastAPI, Redis, Video Sync, SVG Overlay, Matrix View)
```

---

## 🧩 Chi tiết 5 giai đoạn Pipeline

### Giai đoạn 1: Temporal Sampling & Dual-Stream Perception

#### 1. Trích xuất khung hình và Scene Chunking
- **Sampling Rate:** $10 - 15 \text{ FPS}$ tùy thuộc vào vận tốc trung bình của xe, đảm bảo không bỏ sót biển báo ở cự ly gần và tiết kiệm tài nguyên tính toán.
- **Scene Windowing:** Gom cụm cửa sổ $W = 4$ khung hình liên tiếp thành một Scene. Mỗi Scene gán nhãn metadata:
```json
{
  "scene_id": "int",
  "timestamp_range": [3.0, 4.0],
  "frame_indices": [30, 31, 32, 33]
}
```

#### 2. Multi-Task Perception Branches
- **Nhánh 1 - Biển báo & Tín hiệu đèn (Object Detection):**
  - *Kiến trúc:* YOLOv11x / RT-DETR fine-tuned trên tập gộp Zalo AI Traffic, VNTSD (*Vietnam Traffic Sign Dataset*) và Road Damage Dataset.
  - *Phân lớp:* Biển cấm ($P$), Biển nguy hiểm ($W$), Biển hiệu lệnh ($R$), Biển chỉ dẫn ($I$), Đèn tín hiệu giao thông (Đỏ, Vàng, Xanh, Mũi tên chuyển hướng).
- **Nhánh 2 - Làn đường & Vị trí xe chủ (Lane & Ego-Offset):**
  - *Kiến trúc:* UFLDv2 / CLRNet / YOLOPv2 trích xuất tọa độ đa thức của vạch kẻ đường trái ($L$) và phải ($R$).
  - *Mục tiêu:* Đo lường độ lệch tâm (`offset_pixels`) để xác định trạng thái di chuyển đúng làn bên phải.
- **Nhánh 3 - Mũi tên & Vạch mặt đường (Road Surface Marking):**
  - *Kiến trúc:* YOLOv11m phát hiện: Đi thẳng, Rẽ trái, Rẽ phải, Quay đầu, Vạch hỗn hợp, Vạch mắt võng, Vạch dừng (*Stop line*).

#### 3. Optical Character Recognition (OCR) Biển báo
Pipeline 2 giai đoạn chuyên biệt hóa cho biển phụ, biển giới hạn tốc độ và biển tải trọng:
1. **Text Detection:** DBNet / CRAFT cắt vùng bounding box chứa ký tự bên trong biển báo.
2. **Text Recognition:** VietOCR (kiến trúc Transformer / Seq2Seq) nhận diện tiếng Việt có dấu và số hiệu.
3. **Regex Post-processing:**
   - *Giới hạn tốc độ:*
     `r"(?i)\b(tối đa|tối thiểu)?\s*(\d{2,3})\s*(km/h)?\b"` $\rightarrow$ `{"type": "speed_limit", "val": 60, "unit": "km/h"}`
   - *Tải trọng / Chiều cao:*
     `r"(\d+(\.\d+)?)\s*(t|m|tấn|mét)"` $\rightarrow$ `{"type": "dimension_limit", "val": 10, "unit": "T"}`
   - *Khung giờ cấm:*
     `r"(\d{1,2})h?(\d{2})?\s*-\s*(\d{1,2})h?(\d{2})?"` $\rightarrow$ Dải thời gian hiệu lực.

#### 4. Multi-Object Tracking & Optimal Crop Selection
- **Tracking:** ByteTrack duy trì `track_id` cho từng biển báo và cụm đèn qua các khung hình, loại bỏ phân mảnh quỹ đạo do xe xóc hoặc bị che khuất ngắn hạn.
- **Optimal Frame Scoring:** Chọn $1 - 2$ khung hình có độ phân giải và chất lượng tốt nhất trước khi biển báo ra khỏi trường nhìn (FOV):

$$S(f) = \alpha \cdot \frac{\text{Area}(B_f)}{\max_t \text{Area}(B_t)} + (1 - \alpha) \cdot \frac{\operatorname{LaplacianVar}(I_{B_f})}{\max_t \operatorname{LaplacianVar}(I_{B_t})}$$

Trong đó:
- $\text{Area}(B_f) = w \times h$ của bounding box tại frame $f$.
- $\operatorname{LaplacianVar}(I_{B_f}) = \operatorname{Var}(\nabla^2 I_{B_f})$ đo độ sắc nét (chống nhòe chuyển động).
- Trọng số thực nghiệm: $\alpha = 0.6$.

```json
{
  "scene_id": 12,
  "timestamp_range": [3.0, 4.0],
  "ego_position": {
    "offset_pixels": 14.5,
    "lane_status": "center_aligned"
  },
  "tracked_objects": [
    {
      "track_id": 104,
      "class": "prohibitory_sign",
      "best_frame_idx": 8,
      "bbox_normalized": [0.72, 0.25, 0.81, 0.38],
      "ocr_text": "CẤM RẼ TRÁI",
      "attributes": {
        "code": "P.123a",
        "sharpness_score": 340.2
      }
    }
  ],
  "road_markings": [
    {
      "type": "arrow_straight_only",
      "bbox": [0.45, 0.70, 0.55, 0.85]
    }
  ]
}
```

---

### Giai đoạn 2: Keyframe Selection & Temporal Grounding

Định vị khoảng thời gian và các khung hình phục vụ suy luận dựa trên câu hỏi người dùng:

1. **Cross-Modal Embedding:**
   - Chiếu câu hỏi $Q$ và các frame $F_t$ vào cùng không gian tiềm ẩn bằng **SigLIP** (hoặc `CLIP-ViT-B/16-multilingual`):
     $$e_Q = \operatorname{Embed}_{\text{text}}(Q), \quad e_{F_t} = \operatorname{Embed}_{\text{img}}(F_t)$$
   - Điểm tương đồng ngữ nghĩa: $S_t = \cos(e_Q, e_{F_t})$.

2. **Temporal Prior Regularization:**
   - Nhân thêm hàm suy giảm Gauss quanh vùng timestamp biến cố $t_{\text{event}}$ để loại bỏ các đoạn video trống:
     $$S'_t = S_t \cdot \exp\left(-\frac{(t - t_{\text{event}})^2}{2\sigma^2}\right)$$

3. **Trích xuất:** Lấy Top $2 - 4$ frames có $S'_t$ cao nhất kèm bounding box crop độ phân giải cao tương ứng theo `track_id`.

---

### Giai đoạn 3: Knowledge Base RAG & Vector Database

- **Tài liệu nguồn:** QCVN 41:2019/BGTVT (Quy chuẩn báo hiệu đường bộ), Luật Trật tự, An toàn Giao thông Đường bộ và các Nghị định xử phạt vi phạm hành chính (Nghị định 100/2019/NĐ-CP, Nghị định 123/2021/NĐ-CP).
- **Structured Legal Chunking:** Không chia chunk theo độ dài token cố định mà chia theo từng thực thể điều luật độc lập:

```json
{
  "entity_id": "P.124a",
  "category": "Biển báo cấm",
  "name": "Cấm quay đầu xe",
  "description": "Báo cấm các loại xe quay đầu (theo kiểu chữ U). Chiều mũi tên phù hợp với chiều cấm quay đầu.",
  "legal_scope": "Biển có hiệu lực cấm các loại xe cơ giới và thô sơ quay đầu xe, trừ các xe được quyền ưu tiên. Biển này KHÔNG cấm rẽ trái.",
  "keywords": ["cấm quay đầu", "quay đầu", "P124a", "rẽ trái"],
  "related_articles": [
    "QCVN 41:2019 - Điều B.24",
    "Nghị định 100/2019/NĐ-CP - Điểm k Khoản 3 Điều 5"
  ]
}
```

- **Hybrid Search (Dense + Sparse Retrieval):**
  - **Sparse Search (BM25):** Khớp từ khóa định danh chính xác từ OCR (`P.124a`, `Cấm quay đầu`, `60`, `Vạch 1.1`).
  - **Dense Retrieval (FAISS / Qdrant):** Truy vấn ngữ nghĩa tổng quát bằng embedding models (`bge-m3` hoặc `text-embedding-3-large`).
  - **Hợp nhất Reciprocal Rank Fusion (RRF):**
    $$\text{RRF\_Score}(d) = \frac{1}{60 + \text{Rank}_{\text{BM25}}(d)} + \frac{1}{60 + \text{Rank}_{\text{Dense}}(d)}$$

---

### Giai đoạn 4: Tri-Modal Reasoning Engine

| Tiêu chí | Mô hình 1: Fine-tuned VLM *(Core Research)* | Mô hình 2: Symbolic LLM *(Core Engineering)* | Mô hình 3: Cloud API *(Baseline)* |
| :--- | :--- | :--- | :--- |
| **Base Model** | **Qwen2.5-VL-7B-Instruct** | **Llama-3.1-8B-Instruct** (4-bit AWQ/GPTQ) | **Gemini 1.5 Flash / GPT-4o-mini** |
| **Đầu vào** | Keyframes gốc + Inset visual crop biển báo + Prompt + RAG text | Text-only: Chuỗi bảng trạng thái Perception (Event Log) + RAG text | Video clip/Keyframes + CoT Prompt + RAG context qua API |
| **Kỹ thuật** | QLoRA ($r = 64, \alpha = 128$, target full projection layers) | Structured prompt engineering không nhận pixel | Zero-shot / Few-shot Chain-of-Thought |
| **Mục đích** | Suy luận trực quan kết hợp chi tiết điểm ảnh | Cô lập lỗi thị giác (Ablation Benchmark) | Mốc tham chiếu trần (Upper-bound performance) |

#### Structured Output Schema
Cả 3 engines đều đồng nhất định dạng phản hồi chuẩn JSON:

```json
{
  "selected_option": "B",
  "confidence": 0.94,
  "evidence": {
    "support_frames": [8, 12],
    "target_track_id": 104,
    "legal_reference": "QCVN 41:2019/BGTVT - Biển P.124a"
  },
  "reasoning_steps": [
    "Khung hình thứ 8 cho thấy biển báo P.124a (Cấm quay đầu) ở làn ngoài cùng bên phải.",
    "Theo quy chuẩn, biển P.124a cấm xe quay đầu nhưng không cấm rẽ trái.",
    "Xe chủ đang bật đèn tín hiệu rẽ trái và vạch kẻ đường là vạch nét đứt, do đó hành vi rẽ trái là hợp lệ."
  ]
}
```

---

### Giai đoạn 5: Backend Serving & Interactive Dashboard

#### 1. Kiến trúc Asynchronous Inference

```
[Client (Next.js / React)]
       │
       ▼  POST /api/v1/analyze-video (Multipart Form)
[FastAPI Gateway]
       ├── 1. Generate task_id & Save raw video to MinIO/Local Disk
       ├── 2. Push task message to Redis Queue
       └── 3. Return {"task_id": "uuid-xxx", "status": "PENDING"} immediately
       │
       ▼
[Celery Distributed Workers]
  ├── Step 1: Check Redis Cache for Hash(video)
  │     ├── Cache HIT  ──► Load Perception State JSON directly
  │     └── Cache MISS ──► Run (Sampling -> YOLO -> ByteTrack -> OCR)
  │                        └── Save result to Redis (TTL = 7 days)
  ├── Step 2: Temporal Grounding (SigLIP) -> Extract Keyframes
  ├── Step 3: Hybrid Search (RAG) -> Retrieve Law Chunks
  └── Step 4: Dispatch to Selected Engine (Qwen2.5-VL / Llama-3.1 / API)
       │
       ▼
[Redis Result Backend] ◄── Polling or WebSocket pushes status to Client
```

- **Perception Cache Engine:**
  - Key lưu trữ: `perception:cache:<sha256_of_video>`.
  - Khi chuyển đổi qua lại giữa các engine suy luận trên cùng một video, hệ thống tái sử dụng toàn bộ Perception JSON đã trích xuất, giảm thời gian xử lý từ $\sim 15\text{s}$ xuống còn **dưới $1.5\text{s}$**.

#### 2. UI/UX Dashboard
- **Video-Evidence Synchronization:** Tích hợp trực tiếp qua HTML5 Video API (`videoRef.current.currentTime`). Nhấp vào `support_frames` hoặc từng bước `reasoning_steps` sẽ tự động tua video đến đúng timestamp phát hiện lỗi.
- **SVG Bounding Box Overlay:** Sử dụng layer `<svg>` đè tuyệt đối (`position: absolute`) trên khung video. Render các bounding box chuẩn hóa theo thời gian thực; đối tượng liên quan trực tiếp đến suy luận được viền sáng cùng nhãn OCR và mã hiệu luật tương ứng.
- **Benchmark Matrix Table:** Chế độ xem đa luồng (split-view) hiển thị song song kết quả, độ trễ và chuỗi lập luận giữa 3 mô hình trên cùng một video đầu vào.

---

## ⚙️ Ràng buộc kỹ thuật & Benchmark

<p align="center">
  <img src="Rule.png" alt="Competition Rules" width="48%"/>
  <img src="Evaluation.png" alt="Evaluation Metric" width="48%"/>
</p>

| Tiêu chuẩn | Giới hạn quy định | Tối ưu hóa RoadBuddy |
| :--- | :--- | :--- |
| **Kích thước mô hình** | $\le 9\text{B}$ tham số / mô hình | Qwen2.5-VL-7B (Fine-tuned), Llama-3.1-8B (AWQ 4-bit) |
| **Thời gian suy luận** | $\le 30\text{s}$ / testcase | $\sim 2.5\text{s} - 8\text{s}$ (với Perception Cache $< 1.5\text{s}$) |
| **Phần cứng đánh giá** | 1 GPU RTX 3090 / NVIDIA A30, 16 Core CPU, 64GB RAM | Tối ưu hóa VRAM qua QLoRA, vLLM / FlashAttention-2 |
| **Kết nối mạng** | Không có Internet trong lúc chấm giải | Toàn bộ Vector DB (FAISS) & Weights offline độc lập |
| **Độ chính xác đánh giá** | $\text{Accuracy} = \frac{\text{Số câu đúng}}{\text{Tổng testcase}}$ | Phối hợp Cross-Modal Reasoning + Legal Verification |

---

## 📂 Cấu trúc mã nguồn

```bash
RoadBuddy/
├── Dataset/
│   ├── traffic_buddy_train+public_test/
│   │   ├── train/                 # Tập video huấn luyện (~600 videos)
│   │   ├── public_test/           # Tập video public test (~300 videos)
│   │   └── train.json             # Nhãn Q&A, choices, support_frames
│   └── traffic_buddy_private_test/ # Tập video private test (~300 videos)
├── FinalCode/
│   ├── Task_1/                    # Stage 1: Dual-Stream Perception (YOLO, UFLDv2, OCR)
│   ├── Task_2/                    # Stage 2: Temporal Grounding (SigLIP / Cross-Modal)
│   ├── Task_3/                    # Stage 3: Legal RAG Engine (Vector DB, BM25, QCVN 41)
│   ├── Task_4/                    # Stage 4: Tri-Modal Reasoning (Qwen2.5-VL, Llama-3.1)
│   └── Task_5/                    # Stage 5: Backend Serving & Interactive Dashboard
├── Overview.png                   # Hình ảnh tổng quan bài toán
├── Data.png                       # Mô tả định dạng dữ liệu
├── Rule.png                       # Ràng buộc cuộc thi
├── Evaluation.png                 # Tiêu chí đánh giá
├── Pipeline.pdf                   # Bản đặc tả kỹ thuật chi tiết
└── README.md                      # Tài liệu dự án
```

---

## 🚀 Hướng dẫn cài đặt & Triển khai

### 1. Chuẩn bị môi trường

```bash
# Clone repository
git clone https://github.com/Cocobaut/RoadBuddy-Understanding-the-Road-through-Dashcam-AI.git
cd RoadBuddy-Understanding-the-Road-through-Dashcam-AI

# Tạo môi trường ảo conda / venv
conda create -n roadbuddy python=3.10 -y
conda activate roadbuddy

# Cài đặt PyTorch và các phụ thuộc
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
pip install ultralytics transformers accelerate peft vllm faiss-gpu vietocr celery redis fastapi uvicorn
```

### 2. Chạy dịch vụ Backend & Worker

```bash
# Khởi chạy Redis Broker & Cache
redis-server --port 6379

# Khởi chạy Celery Distributed Worker
celery -A FinalCode.Task_5.worker worker --loglevel=info -c 2

# Khởi chạy FastAPI Gateway
uvicorn FinalCode.Task_5.main:app --host 0.0.0.0 --port 8000 --reload
```

### 3. Đánh giá kiểm thử (Offline Batch Submission)

```bash
python FinalCode/Task_4/inference_submission.py \
    --test_json "Dataset/traffic_buddy_train+public_test/public_test.json" \
    --video_dir "Dataset/traffic_buddy_train+public_test/public_test/" \
    --output_csv "submission.csv"
```

---

## 📜 Quy chuẩn & Tài liệu tham chiếu

1. **QCVN 41:2019/BGTVT**: Quy chuẩn kỹ thuật quốc gia về báo hiệu đường bộ của Bộ Giao thông Vận tải.
2. **Luật Trật tự, An toàn Giao thông Đường bộ Việt Nam**.
3. **Nghị định 100/2019/NĐ-CP & Nghị định 123/2021/NĐ-CP**: Quy định xử phạt vi phạm hành chính trong lĩnh vực giao thông đường bộ.
4. **Zalo AI Challenge 2024**: Track RoadBuddy - *Understanding the Road through Dashcam AI*.

---

<p align="center">
  Được phát triển bởi <b>Cocobaut Team</b> cho <b>Zalo AI Challenge</b> 🚦
</p>
