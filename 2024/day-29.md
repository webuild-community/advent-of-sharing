# Tìm hiểu về Barcode EAN

Bạn đã bao giờ đứng xếp hàng tại siêu thị, nghe tiếng *"bíp"* khi nhân viên quét sản phẩm qua máy và tự hỏi: **"Cái barcode này là gì mà thần kỳ đến vậy?"**

Barcode (mã vạch) là một phát minh mang tính cách mạng trong ngành bán lẻ, logistics, và quản lý chuỗi cung ứng. Những vạch đen trắng tưởng chừng đơn giản này lại chứa đựng hàng loạt thông tin như:

- Mã quốc gia.
- Thông tin nhà sản xuất.
- Định danh sản phẩm.

## Tổng quan về Barcode

Barcode được chia làm hai loại chính:

1. **1D (mã vạch một chiều):** Bao gồm các loại mã như UPC - Universal Product Code, EAN, Code 39, Code 128. Đây là loại mã sử dụng các vạch ngang và khoảng trắng để mã hóa thông tin.
2. **2D (mã vạch hai chiều):** Như QR code, Data Matrix, Aztec Code. Loại này mã hóa dữ liệu theo cả chiều ngang lẫn chiều dọc, chứa được nhiều thông tin hơn, bao gồm cả văn bản và liên kết.

Trong bài viết này, chúng ta sẽ tập trung vào một loại mã vạch 1D phổ biến nhất trong ngành bán lẻ: **Barcode EAN**. Cụ thể, chúng ta sẽ khám phá chi tiết về **EAN-13**, tiêu chuẩn thường thấy nhất.

## 1. Barcode EAN Là Gì?
EAN (European Article Number) là một hệ thống mã vạch được phát triển để nhận diện sản phẩm trên toàn cầu. Trong đó:

- **EAN-13** là chuẩn phổ biến nhất, bao gồm 13 chữ số.
- **EAN-8** được sử dụng cho các sản phẩm nhỏ, có dung lượng thông tin ít hơn.

### Mục Đích Của Barcode EAN

- Đơn giản hóa quy trình nhận diện sản phẩm trong chuỗi cung ứng.
- Đảm bảo tính duy nhất của sản phẩm trên toàn cầu.
- Phù hợp với các hệ thống bán lẻ, siêu thị, và logistics quốc tế.

## 2. Cấu Trúc Barcode EAN-13
Barcode EAN-13 gồm 13 chữ số, được chia thành 4 thành phần chính:

| Thành Phần | Độ Dài | Ý Nghĩa |
| --- | --- | --- |
| **Mã quốc gia** | 2 hay 3 chữ số | Xác định quốc gia hoặc tổ chức quản lý mã vạch. Ví dụ: `893` là Việt Nam. |
| **Mã doanh nghiệp** | 4 (nếu mã QG 3 chữ số) hay 5 (nếu mã QG 2 chữ số) chữ số | Nhận diện công ty, doanh nghiệp phát hành sản phẩm. |
| **Mã sản phẩm** | 5 chữ số | Xác định từng loại hàng hóa riêng biệt, thường đánh mã sản phẩm từ **00000** đến **99999**. Như vậy có thể có tới 100.000 chủng loại sản phẩm khác nhau đối với một nhà sản xuất. |
| **Số kiểm tra** | 1 chữ số | Đảm bảo mã vạch được quét chính xác, tránh lỗi. Phụ thuộc vào 12 chữ số trước đó |

Ví Dụ
![Image1](./media/29/sample_barcode.jpeg)
Barcode EAN-13: `8938505922027`

- **Mã quốc gia:** `893` (Việt Nam).
- **Mã doanh nghiệp:** `8505` (Một công ty Việt Nam).
- **Mã sản phẩm (Product code):** `92202` (Mã riêng của sản phẩm cụ thể).
- **Số kiểm tra (Checksum):** `7` (Tự động tính toán để kiểm tra độ chính xác khi quét).

Một số mã quốc gia

| Quốc gia | Mã |
| --- | --- |
| Việt Nam | 893 |
| Mỹ | 001 – 019 & 030 - 039 & 060 - 139 |
| Trung Quốc | 680 - 681 & 690 - 699 |
| Úc | 930 - 939 |
| Brasil | 789 - 790 |
| Mexico | 750 |
| Anh | 500 - 509 |
| Nhật | 450 - 459 & 490 - 499 |

## 3. Checksum
Số kiểm tra (checksum) là con số cuối cùng trong barcode, được tính toán dựa trên 12 chữ số trước đó. Công thức tính như sau:

1. Lấy tổng các số ở vị trí **lẻ** (1, 3, 5, ...).
2. Lấy tổng các số ở vị trí **chẵn** (2, 4, 6, ...), rồi nhân với 3.
3. Cộng hai kết quả trên lại với nhau.
4. Lấy số gần nhất chia hết cho 10 lớn hơn hoặc bằng tổng ở bước 3, rồi trừ đi tổng đó.

Ví dụ: Với mã 8938505922027, số kiểm tra 8 được tính như sau:
- Tổng vị trí lẻ: `8 + 3 + 5 + 5 + 2 + 0 = 23`.
- Tổng vị trí chẵn nhân 3: `(9 + 8 + 0 + 9 + 2 + 2) * 3 = 90`.
- Tổng cộng: `113`.
- Số gần nhất chia hết cho 10 là `120`.
- Số kiểm tra: `120 - 113 = 7`.

Code python tính checksum đơn giản
```
def calculate_ean13_checksum(_barcode):
    """
    Calculate the checksum for an EAN-13 barcode.

    Args:
        _barcode (str): A string of the first 12 digits of the barcode.

    Returns:
        int: The checksum digit (13th digit of the barcode).
    """
    if len(_barcode) != 12 or not _barcode.isdigit():
        raise ValueError("The barcode must be a string of 12 numeric digits.")

    # Convert the string into a list of integers
    digits = [int(d) for d in _barcode]

    # Calculate the sum of digits at odd positions (1-based index)
    odd_sum = sum(d for d in digits[::2])

    # Calculate the sum of digits at even positions (1-based index), multiplied by 3
    even_sum = sum(d for d in digits[1::2]) * 3

    # Total sum
    total_sum = odd_sum + even_sum

    # Calculate the checksum
    _checksum = (10 - (total_sum % 10)) % 10

    return _checksum


# Example
barcode = "8938505922027"
checksum = calculate_ean13_checksum(
    barcode[:-1]
)  # input is first 12 digits of the barcode
print(f"The checksum for the barcode {barcode} is: {checksum}")

```

## 4. Các loại Barcode EAN
Ngoài EAN-13, còn có:
- **EAN-8:** Loại rút gọn, chỉ chứa 8 chữ số, phù hợp với các sản phẩm nhỏ như mỹ phẩm, kẹo.
- **EAN-11 (ít gặp):** Một biến thể của EAN, sử dụng trong các ứng dụng đặc biệt hoặc nội bộ.
## 5. Ưu điểm và ứng dụng của Barcode EAN
### Ưu Điểm
- **Dễ nhận diện:** Barcode đơn giản, dễ quét bằng các loại máy scanner phổ biến.
- **Quản lý hàng hóa hiệu quả:** Gắn mã vạch giúp đồng bộ thông tin sản phẩm trên toàn hệ thống.
- **Tích hợp quốc tế:** Dễ dàng sử dụng ở nhiều quốc gia, nhờ vào mã quốc gia.

### Ứng Dụng
- **Siêu thị:** Nhận diện và kiểm tra giá sản phẩm nhanh chóng.
- **Logistics:** Theo dõi và quản lý kho hàng.
- **E-commerce:** Hỗ trợ quản lý sản phẩm trên các nền tảng thương mại điện tử.

## Tham khảo
- https://www.barcodestalk.com/learn-about-barcodes/european-article-number
- https://en.wikipedia.org/wiki/International_Article_Number
- https://nbc.gov.vn/tintuc/danh-sach-ma-so-ma-vach-cac-nuoc/
- https://en.wikipedia.org/wiki/List_of_GS1_country_codes
- https://en.wikipedia.org/wiki/Barcode