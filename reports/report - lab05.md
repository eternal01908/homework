# BÁO CÁO LAB 5: SEQUENCE MODELS FOR NLP (RNNs, LSTM, Bi-LSTM)
---

## I. MỤC TIÊU
Bài thực hành Lab 5 tập trung vào việc áp dụng các kiến trúc mạng nơ-ron hồi quy (RNN, LSTM, Bi-LSTM) để giải quyết các bài toán cơ bản trong NLP:
1.  **Part 2 - Text Classification:** Phân loại ý định (Intent Detection) trên tập dữ liệu HWU64.
2.  **Part 3 - POS Tagging:** Gán nhãn từ loại (Part-of-Speech) trên tập dữ liệu UD English EWT.
3.  **Part 4 - NER:** Nhận dạng thực thể tên (Named Entity Recognition) trên tập dữ liệu CoNLL 2003.

---

## II. PART 2: PHÂN LOẠI VĂN BẢN (TEXT CLASSIFICATION)

### 1. Phương pháp thực hiện (Implementation)
Chúng tôi đã xây dựng 4 pipeline khác nhau để so sánh hiệu năng giữa Machine Learning truyền thống và Deep Learning:

* **Tiền xử lý:**
    * Xử lý file CSV lỗi định dạng (thiếu header).
    * Tokenization và Padding dữ liệu về độ dài cố định (`max_len=50`).
* **Mô hình 1: TF-IDF + Logistic Regression:** Sử dụng vector tần suất từ (Baseline 1).
* **Mô hình 2: Word2Vec (Average) + Dense Layer:** Lấy trung bình cộng vector của các từ trong câu (Baseline 2).
* **Mô hình 3: LSTM (Pre-trained):** Sử dụng trọng số Word2Vec từ Mô hình 2 và đóng băng (`trainable=False`).
* **Mô hình 4: LSTM (End-to-End):** Lớp Embedding học từ đầu (`trainable=True`) kết hợp với LSTM.

### 2. Kết quả thực nghiệm

| Mô hình | Accuracy | Macro F1-Score | Nhận xét |
| :--- | :---: | :---: | :--- |
| **TF-IDF + LogReg** | ~85% | 0.82 | Hiệu năng tốt, thời gian huấn luyện nhanh. |
| **Word2Vec (Avg)** | ~18% | 0.13 | **Thất bại.** Việc lấy trung bình làm mất thông tin thứ tự và ngữ nghĩa của câu. |
| **LSTM (Frozen)** | ~2% | 0.02 | **Thất bại.** Vector Word2Vec chất lượng thấp và việc đóng băng trọng số khiến LSTM không thể tối ưu. |
| **LSTM (End-to-End)** | **~75%** | **0.72** | **Thành công.** Sau khi xử lý lỗi padding, mô hình hoạt động tốt nhất trong nhóm Deep Learning. |

### 3. Phân tích & Đánh giá
* **Hạn chế của Word2Vec Average:** Các câu lệnh như *"alarm set"* và *"alarm remove"* có vector trung bình rất giống nhau do chia sẻ nhiều từ chung. Mô hình Dense không thể phân biệt ranh giới.
* **Sức mạnh của LSTM:** Mô hình xử lý dữ liệu theo tuần tự (Sequence), do đó nắm bắt được ngữ pháp và ngữ cảnh (ví dụ: từ "set" đứng sau "alarm" mang ý nghĩa khác với đứng trước).
* **Vai trò của Masking:** Ban đầu, mô hình LSTM End-to-End chỉ đạt 2% độ chính xác. Nguyên nhân là do các câu lệnh ngắn (5-7 từ) bị chèn quá nhiều số 0 (Padding). Việc thêm tham số `mask_zero=True` vào lớp Embedding đã giải quyết triệt để vấn đề này.

---

## III. PART 3: GÁN NHÃN TỪ LOẠI (POS TAGGING)

### 1. Phương pháp thực hiện
Chuyển đổi sang framework **PyTorch** để giải quyết bài toán Sequence Labeling (Many-to-Many).

* **Xử lý dữ liệu:**
    * Viết hàm `load_jsonl` tùy chỉnh để đọc định dạng JSONL.
    * Implement `Dataset` và `collate_fn` để thực hiện **Dynamic Padding** (đệm theo batch).
* **Kiến trúc Mô hình:**
    * Sử dụng **LSTM** thay vì SimpleRNN để tránh Vanishing Gradient.
    * Thêm lớp **Dropout (rate=0.5)** để chống Overfitting.
    * Output: `Embedding` -> `LSTM` -> `Dropout` -> `Linear`.
* **Huấn luyện:** Sử dụng `CrossEntropyLoss` với `ignore_index=PAD_IDX` (bỏ qua loss tại vị trí padding). Lưu checkpoint model tốt nhất dựa trên Validation Loss.

### 2. Kết quả thực nghiệm
Mô hình được huấn luyện trong 15 Epochs.

* **Best Validation Accuracy:** **87.33%**
* **Training Loss:** 0.381 (giảm đều).
* **Validation Loss:** 0.409 (không bị tăng vọt như khi dùng SimpleRNN).

**Báo cáo chi tiết theo nhãn:**
* **Nhóm hiệu năng cao (>95%):** PUNCT (Dấu câu), DET (Mạo từ), PRON (Đại từ). Đây là các từ loại đóng, ít biến đổi.
* **Nhóm hiệu năng khá (>80%):** NOUN, VERB, ADJ. Mô hình phân biệt tốt dựa trên ngữ cảnh dù vốn từ vựng lớn.
* **Nhóm hiệu năng thấp (<70%):** PROPN (Tên riêng). Do nhiều từ lạ (`UNK`) chưa xuất hiện trong tập train.

### 3. Ví dụ dự đoán
* **Câu:** "I love learning NLP"
* **Dự đoán:** `[('I', 'PRON'), ('love', 'VERB'), ('learning', 'VERB'), ('NLP', 'NOUN')]`
* **Kết luận:** Mô hình dự đoán chính xác cấu trúc ngữ pháp.

---

## IV. PART 4: NHẬN DẠNG THỰC THỂ TÊN (NER)

### 1. Phương pháp thực hiện
Sử dụng bộ dữ liệu chuẩn **CoNLL 2003** để nhận dạng các thực thể: PER (Người), LOC (Địa điểm), ORG (Tổ chức), MISC (Khác).

* **Tải dữ liệu:** Sử dụng phương pháp tải thủ công (`wget`) từ nguồn GitHub do thư viện HuggingFace gặp lỗi script bảo mật.
* **Mô hình nâng cao: Bi-LSTM (Bidirectional LSTM)**.
    * Sử dụng LSTM 2 chiều để tận dụng thông tin ngữ cảnh từ cả quá khứ (trước) và tương lai (sau) của một từ.
    * Điều này cực kỳ quan trọng để phân biệt thực thể. Ví dụ: "Washington" trong "Washington DC" (LOC) vs "George Washington" (PER).

### 2. Kết quả thực nghiệm
* **Mô hình:** Bi-LSTM (Hidden dim = 256).
* **Kết quả:** Mô hình đạt độ chính xác ~88-90% trên tập Validation.
* **Ví dụ thực tế:**
    * **Input:** *"VNU University is located in Hanoi"*
    * **Output:**
        * `VNU`: **I-ORG**
        * `University`: **I-ORG**
        * `Hanoi`: **I-LOC**
    * **Đánh giá:** Mô hình nhận diện chính xác ranh giới và loại thực thể.

---

## V. NHỮNG KHÓ KHĂN VÀ GIẢI PHÁP (CHALLENGES & SOLUTIONS)

Trong quá trình thực hiện Lab, nhóm đã gặp phải 3 thách thức kỹ thuật lớn:

| Vấn đề | Mô tả chi tiết | Giải pháp đã thực hiện |
| :--- | :--- | :--- |
| **1. Vanishing Gradient do Padding (Part 2)** | Mô hình LSTM kẹt ở Accuracy 2%, Loss không giảm. Do chuỗi input chứa quá nhiều số 0 (Padding) lấn át tín hiệu thật. | Sử dụng tham số `mask_zero=True` trong lớp Embedding (Keras) để thông báo cho LSTM bỏ qua các bước thời gian là padding. |
| **2. Định dạng dữ liệu không khớp (Part 3)** | Đề bài yêu cầu file `.conllu` nhưng dữ liệu cung cấp là `.jsonl`. Code mẫu không chạy được. | Phân tích cấu trúc file JSON, viết lại hàm `load_jsonl` tùy chỉnh để trích xuất đúng trường `words` và `tags`. |
| **3. Lỗi thư viện HuggingFace (Part 4)** | Lỗi `trust_remote_code is not supported` khi tải dataset `conll2003` do chính sách bảo mật mới của thư viện `datasets`. | Chuyển sang tải dữ liệu thô (`raw text`) từ GitHub bằng lệnh `wget` và viết hàm tiền xử lý thủ công (Manual Loading). |

---

## VI. HƯỚNG DẪN CHẠY CODE

Để tái lập kết quả, vui lòng thực hiện theo trình tự sau trên Google Colab:

1.  **Cài đặt thư viện:**
    ```bash
    !pip install gensim torch tensorflow scikit-learn pandas seqeval
    ```
2.  **Chuẩn bị dữ liệu:**
    * Upload file `hwu.tar.gz` và `en_ewt-ud.jsonl.tar.gz` lên Colab.
    * Chạy các cell giải nén ở đầu mỗi Part.
    * Đối với Part 4, code sẽ tự động tải dữ liệu bằng `wget`.
3.  **Thực thi:**
    * Chạy lần lượt các cell từ trên xuống dưới (Run All).
    * Kết quả Training, biểu đồ Loss và các ví dụ dự đoán sẽ được in ra màn hình console.