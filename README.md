# The Planet’s Prestige — Blue Team Labs Online Walkthrough

Trong nhiệm vụ này, chúng ta nhận được một tập tin email cần phân tích, chứa thông tin về những người CoCanDian bị bắt cóc và con gái của ngài tổng thống. Bài lab cung cấp một file email để kiểm tra bằng ứng dụng đọc mail hoặc text editor (ví dụ: Notepad++). Bên trong email còn đính kèm một file có vẻ là PDF.

![Email attachment file](images/image_01.png)

File "pdf" trước khi được giải mã:

```text
--BOUND_600FB98E0DCEE8.49207210
Content-Type: application/pdf; name="PuzzleToCoCanDa.pdf"
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename="PuzzleToCoCanDa.pdf"
```

Sau khi giải mã và giải nén thư mục:

![Thư mục sau khi giải mã](images/image_02.png)

---

## Các câu hỏi và lời giải chi tiết

### Q1: What is the email service used by the malicious actor?
> **Đáp án:** `emkei.cz`

**Phân tích:**  
Trong header hiển thị ban đầu, bức thư có vẻ được gửi từ tên miền `microapple.com`. Tuy nhiên, khi kiểm tra kỹ trường `Received:` trong header email bằng Notepad++ hoặc terminal Linux:

```text
Received: from localhost (emkei.cz. [93.99.104.210])
```

Dịch vụ được kẻ tấn công sử dụng là **emkei.cz** (Emkei's Fake Mailer).

---

### Q2: What is the Reply-To email address?
> **Đáp án:** `negeja3921@pashter.com`

**Phân tích:**  
Trường này hiển thị trực tiếp trong phần MIME header của tập tin `.eml`:

```text
Reply-To: negeja3921@pashter.com
```

---

### Q3: What is the filetype of the received attachment which helped to continue the investigation?
> **Đáp án:** `.zip`

**Phân tích:**  
Tập tin đính kèm có tên `PuzzleToCoCanDa.pdf`, nhưng khi mở ra thì nội dung bị hỏng hoặc mã hóa dưới dạng Base64. 

Trong nội dung thư có lời khuyên: *"Don’t Trust Your Eyes"* (Đừng tin vào mắt mình), gợi ý phần mở rộng `.pdf` chỉ là ngụy trang.

1. Đưa chuỗi Base64 vào **CyberChef** và sử dụng công thức: `From Base64` -> `To Hex`.
2. Kiểm tra các byte đầu tiên (File Signature / Magic Bytes):
   ```text
   50 4B 03 04
   ```
3. Signature `50 4B 03 04` (ký tự ASCII: `PK..`) tương ứng với định dạng nén **ZIP (PKZIP archive file)**.

Sau khi đổi định dạng và giải nén thư mục `PuzzleToCoCanDa`, ta thu được danh sách các file:
- `DaughtersCrown`
- `GoodJobMajor`
- `Money.xlsx`

Kiểm tra định dạng thực tế của các file không có phần mở rộng:
- `DaughtersCrown.jpeg` (JPEG image)
- `GoodJobMajor.pdf` (PDF document)
- `Money.xlsx` (Excel Spreadsheet)

---

### Q4: What is the name of the malicious actor?
> **Đáp án:** `Pestero Negeja`

**Phân tích:**  
Sử dụng công cụ `exiftool` trên PowerShell/Linux để trích xuất metadata của các file đính kèm:

```powershell
exiftool .\GoodJobMajor.pdf
```

Kết quả metadata trả về:
```text
File Name     : GoodJobMajor.pdf
File Type     : PDF
MIME Type     : application/pdf
Author        : Pestero Negeja
Producer      : Skia/PDF m90
```

---

### Q5: What is the location of the attacker in this Universe?
> **Đáp án:** `The Martian Colony, Beside Interplanetary Spaceport.`

**Phân tích:**  
1. Mở file `Money.xlsx` và chuyển sang `Sheet3`.
2. Toàn bộ sheet có vẻ trống do định dạng màu sắc ẩn. Chọn toàn bộ sheet và thực hiện **Xóa định dạng (Clear Formatting)** (phím tắt `Ctrl + \` trên Google Sheets/Excel).
3. Tại ô `F4`, xuất hiện chuỗi mã hóa:
   ```text
   VGhlIE1hcnRpYW4gQ29sb255LCBCZXNpZGUgSW50ZXJwbGFuZXRhcnkgU3BhY2Vwb3J0Lg==
   ```
4. Giải mã chuỗi Base64 bằng CyberChef thu được kết quả:
   ```text
   The Martian Colony, Beside Interplanetary Spaceport.
   ```

---

### Q6: What could be the probable C&C domain to control the attacker’s autonomous bots?
> **Đáp án:** `pashter.com`

**Phân tích:**  
Máy chủ C&C (Command & Control) đóng vai trò là trung tâm chỉ huy mà kẻ tấn công dùng để ra lệnh cho mã độc hoặc bots. Kết hợp các manh mối:
- Tên kẻ tấn công: **Pestero Negeja**
- Địa chỉ nhận phản hồi: `negeja3921@pashter.com`

Tên miền `pashter.com` khả năng cao chính là hạ tầng C&C server được dựng lên phục vụ chiến dịch tấn công này.

---

## Kết luận

Đây là một bài lab điều tra số (Digital Forensics / SOC Analysis) phong cách CTF rất trực quan, rèn luyện kỹ năng phân tích Email Header, xác định Magic Bytes của file đính kèm, trích xuất Metadata qua Exiftool và phát hiện dữ liệu ẩn trong bảng tính.
