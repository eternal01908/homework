# **Báo cáo Lab 1: Text Tokenization**
## **1. Các bước triển khai**
\- Interface (Tokenizer): Tạo abstract class Tokenizer trong src/core/interfaces.py.\
\- SimpleTokenizer: lowercase, tách theo whitespace và dấu câu, cho phép '\_' nằm trong từ.\
\- RegexTokenizer: dùng regex \w+|[^\w\s], có chỉnh sửa để đồng bộ với SimpleTokenizer.\
\- Dataset Loader: src/core/dataset\_loaders.py để load dữ liệu từ file .txt.\
\- main.py: test cả SimpleTokenizer và RegexTokenizer trên câu ví dụ và dataset UD English EWT.
## **2. Cách chạy code và log kết quả**
Cách chạy: \
`    `python main.py\
\
Ví dụ log:\
Input: "Hello, world!"\
`    `SimpleTokenizer: ['hello', ',', 'world', '!']\
`    `RegexTokenizer: ['hello', ',', 'world', '!']\
\
Input: "abc.zyx real-time com\_dot!"\
`    `SimpleTokenizer: ['abc', '.', 'zyx', 'real', '-', 'time', 'com\_dot', '!']\
`    `RegexTokenizer: ['abc', '.', 'zyx', 'real', '-', 'time', 'com\_dot', '!']
## **3. Giải thích kết quả**
\- Với SimpleTokenizer: chỉ giữ '\_' trong từ, còn '.' và '-' bị tách.\
\- Với RegexTokenizer: cho kết quả gần giống, linh hoạt hơn trong một số trường hợp.\
\- Trên dataset UD English EWT: tokenizer tách câu thành từ và dấu câu chính xác.
## **4. Khó khăn và cách giải quyết**
\- Khó khăn: Dataset gốc là .conllu, không phải .txt.\
`  `Giải pháp: viết lại load\_raw\_text\_data để bỏ qua metadata và chỉ lấy cột word form.\
\- Khó khăn: Xác định quy tắc cho SimpleTokenizer.\
`  `Giải pháp: Cho phép '\_' nhưng tách '.' và '-'.\
\- Khó khăn: Import module trong Python.\
`  `Giải pháp: Thêm sys.path.append(...) trong main.py.
## **5. Tham khảo**
\- Python re (regular expressions).\
\- Universal Dependencies dataset: https://universaldependencies.org/\
\- Tài liệu và slide lab giảng viên cung cấp.
## **6. Nhận xét**
\- Cách chia token có phần cứng nhắc và sẽ còn phải chỉnh sửa nhiều theo token\
Ví dụ: trong các trường hợp . nằm dưới dạng trang web như google.com, thì nên coi cả cụm là 1 token thay vì 3 token, nhưng việc này yêu cầu phải nhận diện về mặt ngữ nghĩa
