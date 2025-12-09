# BÁO CÁO TỔNG HỢP THỰC HÀNH NLP
**Lab 6: Introduction to Transformers**

---

# LAB 6 - TRANSFORMERS

## I. MỤC TIÊU
Làm quen với kiến trúc Transformer và thư viện Hugging Face thông qua các tác vụ NLP cơ bản.

## II. KẾT QUẢ THỰC HIỆN

### Task 1: Masked Language Modeling (BERT)
* **Input:** *"Hanoi is the `<mask>` of Vietnam."*
* **Output:** `capital` (Độ tin cậy: 40.33%).
* **Phân tích:** Mô hình Encoder-only (DistilRoBERTa) sử dụng cơ chế **Self-Attention hai chiều**, cho phép nhìn thấy cả "Hanoi" và "Vietnam" để điền đúng từ "capital".

### Task 2: Text Generation (GPT)
* **Input:** *"The best thing about learning NLP is"*
* **Output:** *"The best thing about learning NLP is that there are a few ways to pick up resources: Learn something from others..."*
* **Phân tích:** Mô hình Decoder-only (GPT-2) sử dụng **Causal Masking (một chiều)**, chỉ nhìn về quá khứ để dự đoán từ tiếp theo, phù hợp cho tác vụ sinh văn bản.

### Task 3: Sentence Representation (BERT Embedding)
* **Mô hình:** `bert-base-uncased`.
* **Input:** *"This is a sample sentence."*
* **Output:** Vector kích thước `[1, 768]`.
* **Kỹ thuật:** Sử dụng **Mean Pooling** kết hợp với **Attention Mask** để tính trung bình cộng các vector token mà không bị nhiễu bởi các token đệm (padding).
