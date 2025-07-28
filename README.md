# So sánh các Bộ Lọc Không Gian trong Xử Lý Ảnh Số
**Sinh viên thực hiện:** Nhóm 06  
**Môn học:** Nhập môn Xử lý ảnh số  
**Giảng viên:** TS. Đỗ Hữu Quân

---

## Giới thiệu

Hệ thống này so sánh **5 bộ lọc không gian phổ biến** cho ảnh số: Mean, Gaussian, Median, Bilateral và Non-local Means. Mục tiêu là đánh giá khả năng loại nhiễu, giữ chi tiết và hiệu quả của từng bộ lọc bằng nhiều chỉ số định lượng, minh họa trên dashboard và sinh báo cáo tự động.

---

## Công nghệ sử dụng

- **Python**
- **OpenCV**: Tiền xử lý, áp dụng các bộ lọc ảnh
- **scikit-image**: Cung cấp metrics (PSNR, SSIM), bộ lọc NLM
- **NumPy**: Tính toán trên ma trận ảnh
- **Matplotlib, Seaborn**: Vẽ biểu đồ, trực quan hóa dữ liệu
- **ipywidgets**: Xây dựng dashboard đa tab

---

## Tóm tắt bộ lọc, công thức và ví dụ

### 1. Mean Filter (Lọc trung bình)

- **Mục đích:** Làm mượt, giảm nhiễu đều.
- **Công thức:**
  ```
  g(x, y) = (1 / (m * n)) * Tổng [f(x+i, y+j)], i = 0..m-1, j = 0..n-1
  ```
- **Ví dụ:** Ảnh có nhiễu nhẹ sau khi lọc mean sẽ trở nên mờ, các đường biên giảm độ nét.
- **Code:**
  ```
  # Áp dụng bộ lọc trung bình với kernel 5x5
  filtered, t = cv2.blur(image, (5, 5))
  ```

---

### 2. Gaussian Filter (Lọc Gaussian)

- **Mục đích:** Làm mượt một cách tự nhiên, giữ biên khá tốt.
- **Công thức:**
  ```
  G(x, y) = (1 / (2 * π * σ^2)) * exp( - (x^2 + y^2) / (2 * σ^2) )
  ```
- **Ví dụ:** Ảnh nhiễu sẽ mịn hơn, các đường biên phần lớn vẫn được giữ lại.
- **Code:**
  ```
  # Áp dụng bộ lọc Gaussian với kernel 5x5 và sigma = 1.0
  filtered, t = cv2.GaussianBlur(image, (5, 5), 1.0)
  ```

---

### 3. Median Filter (Lọc trung vị)

- **Mục đích:** Loại bỏ nhiễu "muối tiêu" (salt-and-pepper), giữ nét tốt.
- **Công thức:**
  ```
  g(x, y) = Median{ f(x+i, y+j) | (i, j) trong vùng kernel }
  ```
- **Ví dụ:** Ảnh có các đốm đen trắng ngẫu nhiên sau khi qua bộ lọc median sẽ trở nên sạch rõ.
- **Code:**
  ```
  # Áp dụng bộ lọc trung vị với kernel 5x5
  filtered, t = cv2.medianBlur(image, 5)
  ```

---

### 4. Bilateral Filter (Lọc song phương)

- **Mục đích:** Làm mịn nhưng vẫn giữ lại các cạnh sắc nét.
- **Công thức:**
  ```
  g(x, y) = (1 / W) * Tổng [ f(x+i, y+j) * exp( - (||p-q||^2) / (2σ_s^2) - (f(p) - f(q))^2 / (2σ_r^2) ) ]
  ```
- **Ví dụ:** Dùng để làm mịn da trong ảnh chân dung mà vẫn giữ lại độ nét của mắt, mũi, miệng.
- **Code:**
  ```
  # Áp dụng bộ lọc song phương
  filtered, t = cv2.bilateralFilter(image, d=9, sigmaColor=75, sigmaSpace=75)
  ```

---

### 5. Non-local Means (NLM)

- **Mục đích:** Loại bỏ nhiễu mạnh, duy trì chi tiết dựa trên các vùng (patch) giống nhau trên toàn ảnh.
- **Ví dụ:** Ảnh có nhiễu nặng nhưng cấu trúc tổng thể vẫn được phục hồi gần như gốc.
- **Code:**
  ```
  from skimage import restoration
  # Áp dụng bộ lọc NLM
  filtered = restoration.denoise_nl_means(img_float, h=0.1)
  ```

---

### Một số chỉ số đánh giá

- **PSNR (Peak Signal-to-Noise Ratio)**
  - *Mô tả:* Đo lường chất lượng tái tạo của ảnh sau khi lọc so với ảnh gốc. Giá trị càng cao thì chất lượng càng tốt.
  - *Công thức:*
    ```
    PSNR = 10 * log10(MAX^2 / MSE)
    ```

- **SSIM (Structural Similarity Index)**
  - *Mô tả:* So sánh sự tương đồng về cấu trúc, độ sáng và độ tương phản giữa hai ảnh. Giá trị càng gần 1 thì ảnh càng giống nhau.
  - *Công thức:*
    ```
    SSIM(x, y) = [(2 * μx * μy + C1) * (2 * σxy + C2)] /
                 [(μx^2 + μy^2 + C1) * (σx^2 + σy^2 + C2)]
    ```

- **Edge Preservation (Bảo toàn cạnh)**
  - *Mô tả:* Đánh giá khả năng bộ lọc giữ lại các chi tiết cạnh. Thường so sánh SSIM của ảnh biên (tạo bởi Canny, Sobel, Laplacian) trước và sau khi lọc.

- **MSE (Mean Squared Error)**
  - *Mô tả:* Đo lỗi trung bình bình phương giữa hai ảnh (càng thấp càng tốt).
  - *Công thức:*
    ```
    MSE = (1 / (m * n)) * Σ [ I1(x, y) - I2(x, y) ]^2
    ```
    Trong đó I1 và I2 là hai ảnh cần so sánh, m và n là kích thước ảnh.

- **SNR (Signal-to-Noise Ratio)**
  - *Mô tả:* Tỷ số tín hiệu trên nhiễu, đánh giá chất lượng tín hiệu so với mức độ nhiễu (càng cao càng tốt).
  - *Công thức:*
    ```
    SNR = 10 * log10(Mean(I^2) / MSE)
    ```

- **Entropy**
  - *Mô tả:* Đo mức độ phức tạp/thông tin của ảnh; entropy càng lớn thì ảnh càng nhiều chi tiết.
  - *Công thức:*
    ```
    Entropy = - Σ [p_i * log2(p_i)]
    ```
    Trong đó p_i là xác suất xuất hiện của mức xám i trong ảnh.

- **Độ tương phản (Contrast)**
  - *Mô tả:* Đo mức độ dao động cường độ sáng tối trên toàn ảnh.
  - *Công thức:*
    ```
    Contrast = sqrt( (1/(m*n)) * Σ [I(x, y) - μ]^2 )
    ```
    Với μ là giá trị trung bình của ảnh.

- **Độ sắc nét (Sharpness)**
  - *Mô tả:* Đo tổng biến thiên hoặc phương sai của đạo hàm bậc 2 (Laplacian) ảnh; sắc nét càng cao giá trị càng lớn.
  - *Công thức:*
    ```
    Sharpness = Var( Laplacian( I(x, y) ) )
    ```
    Laplacian là toán tử đạo hàm cấp 2 áp dụng lên ảnh.

---

## Minh họa đơn giản

| Bộ lọc       | Kết quả lọc & nhận xét ngắn |
|--------------|----------------------------|
| Mean         | Ảnh mịn, biên mờ mạnh, cực nhanh |
| Gaussian     | Mịn vừa phải, giữ biên tốt hơn Mean   |
| Bilateral    | Rất mịn nhưng cạnh vẫn rõ ràng          |
| Median       | Sạch nhiễu muối tiêu, giữ nét cạnh|
| Non-local Means | Giữ chi tiết tốt, ít mất nét, nhưng chậm|

---

## Pipeline vận hành

```
[Ảnh gốc] → [Thêm nhiễu] → [5 bộ lọc song song] → [Tính toán metrics] → [So sánh & trực quan hóa] → [Xuất báo cáo]
```
- Toàn bộ quá trình được quản lý qua **dashboard nhiều tab**.

---

## Cấu trúc file

```
├── main.ipynb      # Notebook pipeline chính
├── images/         # Thư mục chứa ảnh gốc, ảnh test
├── README.md       # File này
└── data_output/    # Thư mục tự động tạo để chứa báo cáo
```

---

## Hướng dẫn nhanh

1.  **Clone repo và cài đặt các thư viện cần thiết:**
    ```
    pip install opencv-python scikit-image numpy matplotlib pandas seaborn ipywidgets
    ```
2.  **Chạy notebook** trong Jupyter, Google Colab, hoặc VSCode.
3.  **Theo dõi kết quả** trên các tab của dashboard và kiểm tra thư mục `data_output` để lấy báo cáo cuối cùng.

---

## Tham khảo

- Tài liệu của OpenCV và scikit-image
- Slide bài giảng môn Xử lý ảnh số
- **Sách kinh điển & giáo khoa**
    - Gonzalez, R.C. & Woods, R.E. *Digital Image Processing*, 4th Edition, Pearson.  
      *(Sách giáo khoa chuẩn quốc tế, bao phủ lý thuyết & thực hành bộ lọc không gian. Có chương chi tiết về các loại spatial filters, công thức, ví dụ minh họa.)* 
    - Jain, A.K., *Fundamentals of Digital Image Processing*, Prentice Hall.  
      *(Toàn diện về toán học nền tảng, các phép biến đổi ảnh cơ bản và nâng cao.)*
    - Solomon, C., Breckon, T., *Fundamentals of Digital Image Processing: A Practical Approach with Examples in Matlab*, Wiley.  
      *(Rất phù hợp cho sinh viên muốn code trực tiếp; có giới thiệu phong phú về các lọc không gian, minh họa MATLAB.)* 
    - Maria Petrou & Costas Petrou, *Image Processing: The Fundamentals*, Wiley.  
      *(Tập trung lý thuyết filter, biến đổi ảnh, có ví dụ, hình minh họa cụ thể.)* 

- **Bài báo và nguồn học thuật**
    - Desai, B. et al, *Study on Image Filtering -- Techniques, Algorithm and Applications*, arXiv:2207.06481  
      *(Tổng hợp các kỹ thuật lọc kinh điển và ứng dụng lọc ảnh không gian trong khoa học máy tính.)* 
    - ScienceDirect – Từ khóa: "Image Filtering"  
      *(Nền tảng học thuật, truy cập nhiều chương review về bộ lọc không gian, lý thuyết và ứng dụng thực tiễn.)* 

- **Đặc biệt cho y sinh & nâng cao**
    - Geoff Dougherty, *Digital Image Processing for Medical Applications*, Cambridge University Press.  
      *(Trình bày rõ các trường hợp sử dụng thực tiễn bộ lọc không gian trong ảnh y sinh, nhiều minh họa với dữ liệu thực tế.)* 

- **Nguồn trực tuyến chất lượng cao**
    - MathWorks Blog: Steve on Image Processing  
      *(Chia sẻ chuyên sâu về lý thuyết, ví dụ thực hành về filter, image enhancement và xử lý ảnh trên MATLAB.)* 
    - OpenCV, scikit-image Documentation  
      *(Thư viện chuẩn quốc tế, đầy đủ hướng dẫn và code mẫu bộ lọc ảnh.)*

---
