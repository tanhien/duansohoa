# Kế hoạch OCR – Trích xuất thông tin từ file scan

## Mục tiêu

Module 1 nhận vào danh sách hồ sơ + file PDF đã scan. OCR sẽ đọc từng file, trích xuất các trường metadata để:
1. Tự động điền vào form chỉnh lý (Module 2), giảm nhập liệu thủ công
2. Sinh EAD.xml (Module 3) với dữ liệu chính xác
3. Phục vụ đóng gói SIP/AIP (Module 4)

---

## Luồng xử lý OCR

```text
PDF scan → Phát hiện loại tài liệu → OCR từng trang → Trích xuất trường → Validate → Lưu nháp
                                          ↓ thất bại
                                     Gắn cờ "cần nhập thủ công"
```

---

## Phân loại tài liệu (bước đầu tiên)

OCR cần xác định loại tài liệu trước khi áp dụng bộ trường tương ứng.

| Loại | Dấu hiệu nhận dạng | Bộ trường áp dụng |
|---|---|---|
| **Trang bìa hồ sơ** | Chứa "HỒ SƠ", "MỤC LỤC", tiêu đề hồ sơ, thời hạn lưu trữ | Bộ A – Hồ sơ |
| **Văn bản hành chính** | Có số ký hiệu dạng `123/QĐ-BNV`, có quốc hiệu, có chữ ký | Bộ B – Văn bản |
| **Ảnh / phim âm bản** | File JPEG/TIFF, hoặc PDF 1 trang toàn ảnh | Bộ C – Ảnh |
| **Ghi âm / ghi hình** | File MP3/MP4/AVI hoặc metadata nhúng | Bộ D – Media |

---

## Bộ A – Trích xuất thông tin Hồ sơ (trang bìa)

Áp dụng cho: trang bìa hoặc trang đầu của mỗi hồ sơ trong lô scan.

| STT | Trường | Tên hiển thị | Vị trí trên văn bản | Kiểu | Bắt buộc | Ví dụ |
|---|---|---|---|---|---|---|
| 1 | `fileCode` | Mã hồ sơ | Header / góc trên | String(50) | Bắt buộc | `G09.2021.01.001` |
| 2 | `title` | Tiêu đề hồ sơ | Tiêu đề in đậm trung tâm | String(500) | Bắt buộc | `Tập quyết định nhân sự năm 2021` |
| 3 | `maintenance` | Thời hạn lưu trữ | Góc phải hoặc dòng "Thời hạn lưu trữ:" | String(2) | Bắt buộc | `01` (Vĩnh viễn) |
| 4 | `startDate` | Ngày bắt đầu | Dòng "Thời gian:" hoặc "Từ ngày" | Date(DD/MM/YYYY) | Bắt buộc | `01/01/2021` |
| 5 | `endDate` | Ngày kết thúc | Dòng "Đến ngày" hoặc kế bên startDate | Date(DD/MM/YYYY) | Bắt buộc | `31/12/2021` |
| 6 | `totalDoc` | Tổng số tài liệu | Dòng "Số tài liệu:" hoặc "Gồm ... văn bản" | Number | Tùy chọn | `15` |
| 7 | `numberOfPaper` | Số lượng tờ | Dòng "Số tờ:" | Number | Tùy chọn | `120` |
| 8 | `numberOfPage` | Số lượng trang | Dòng "Số trang:" | Number | Tùy chọn | `240` |
| 9 | `language` | Ngôn ngữ | Dòng "Ngôn ngữ:" hoặc suy từ nội dung | String(2) | Tùy chọn | `01` (Tiếng Việt) |
| 10 | `mode` | Chế độ sử dụng | Dòng "Chế độ:" hoặc đóng dấu MẬT | String(2) | Tùy chọn | `01` (Công khai) |
| 11 | `keyword` | Từ khóa | Dòng "Từ khóa:" hoặc trích từ tiêu đề | String(200) | Tùy chọn | `nhân sự; quyết định` |
| 12 | `inforSign` | Ký hiệu thông tin | Góc / mã lưu kho | String(50) | Tùy chọn | `BNV.2021.HS001` |
| 13 | `paperFileCode` | Mã hồ sơ giấy gốc | Dòng "Hộp số:" / "Cặp số:" | String(50) | Tùy chọn | `HOP-001` |
| 14 | `description` | Ghi chú | Dòng "Ghi chú:" | String(2000) | Tùy chọn | |

### Quy tắc nhận dạng thời hạn lưu trữ

| Từ khóa OCR nhận dạng | Mã `maintenance` |
|---|---|
| Vĩnh viễn, permanent | `01` |
| 70 năm | `02` |
| 50 năm | `03` |
| 30 năm | `04` |
| 20 năm | `05` |
| 10 năm | `06` |
| Khác hoặc không xác định | `07` |

---

## Bộ B – Trích xuất thông tin Văn bản hành chính

Áp dụng cho: từng file PDF tương ứng 1 văn bản trong hồ sơ.

| STT | Trường | Tên hiển thị | Vị trí trên văn bản | Kiểu | Bắt buộc | Ví dụ |
|---|---|---|---|---|---|---|
| 1 | `organName` | Tên cơ quan ban hành | Góc trên trái (dưới logo) | String(200) | Bắt buộc | `BỘ NỘI VỤ` |
| 2 | `codeNumber` | Số của văn bản | Dòng "Số:" – phần số | String(11) | Bắt buộc | `123` |
| 3 | `codeNotation` | Ký hiệu văn bản | Dòng "Số:" – phần ký hiệu sau `/` | String(30) | Bắt buộc | `QĐ-BNV` |
| 4 | `issuedDate` | Ngày tháng năm | Dòng địa danh + ngày, thường góc phải | Date(DD/MM/YYYY) | Bắt buộc | `15/03/2021` |
| 5 | `typeName` | Loại văn bản | Tiêu đề in hoa (QUYẾT ĐỊNH, CÔNG VĂN...) | String(100) | Bắt buộc | `QUYẾT ĐỊNH` |
| 6 | `typeDoc` | Mã loại văn bản | Suy từ `typeName` (xem bảng bên dưới) | String(2) | Bắt buộc | `02` |
| 7 | `subject` | Trích yếu nội dung | Dòng "Về việc..." hoặc tiêu đề in đậm sau loại VB | String(500) | Bắt buộc | `Về việc bổ nhiệm Vụ trưởng` |
| 8 | `autograph` | Người ký | Cuối văn bản – tên in dưới chữ ký | String(200) | Tùy chọn | `Nguyễn Văn A` |
| 9 | `process` | Chức vụ người ký | Cuối văn bản – dòng trên tên người ký | String(200) | Tùy chọn | `BỘ TRƯỞNG` |
| 10 | `pageAmount` | Số trang văn bản | Đếm tổng số trang PDF | Number(4) | Tùy chọn | `3` |
| 11 | `language` | Ngôn ngữ | Suy từ nội dung văn bản | String(2) | Tùy chọn | `01` |
| 12 | `mode` | Chế độ sử dụng | Đóng dấu MẬT / TỐI MẬT hoặc không có | String(2) | Tùy chọn | `01` |
| 13 | `confidenceLevel` | Mức độ tin cậy OCR | Hệ thống tự tính (0–100%) | Number | Tự động | `87` |
| 14 | `keyword` | Từ khóa | Trích từ trích yếu | String(100) | Tùy chọn | |
| 15 | `description` | Ghi chú / nơi nhận | Phần "Nơi nhận:" cuối văn bản | String(500) | Tùy chọn | |
| 16 | `source` | Chiều văn bản | Xác định VB đi (0) hay VB đến (1) | Boolean | Tùy chọn | `0` |
| 17 | `docOrdinal` | Số thứ tự trong hồ sơ | Lấy từ danh sách import, không OCR | Number | Bắt buộc | `1` |

### Nhận dạng loại văn bản (typeName → typeDoc)

| Từ khóa OCR nhận dạng | `typeName` | `typeDoc` |
|---|---|---|
| NGHỊ QUYẾT | Nghị quyết | `01` |
| QUYẾT ĐỊNH | Quyết định | `02` |
| CHỈ THỊ | Chỉ thị | `03` |
| QUY ĐỊNH | Quy định | `04` |
| THÔNG TƯ | Thông tư | `05` |
| THÔNG TƯ LIÊN TỊCH | Thông tư liên tịch | `06` |
| THÔNG BÁO | Thông báo | `07` |
| HƯỚNG DẪN | Hướng dẫn | `08` |
| CHƯƠNG TRÌNH | Chương trình | `09` |
| KẾ HOẠCH | Kế hoạch | `10` |
| PHƯƠNG ÁN | Phương án | `11` |
| ĐỀ ÁN | Đề án | `12` |
| DỰ ÁN | Dự án | `13` |
| BÁO CÁO | Báo cáo | `14` |
| TỜ TRÌNH | Tờ trình | `15` |
| GIẤY ỦY QUYỀN | Giấy ủy quyền | `16` |
| PHIẾU GỬI | Phiếu gửi | `17` |
| PHIẾU CHUYỂN | Phiếu chuyển | `18` |
| PHIẾU BÁO | Phiếu báo | `19` |
| BIÊN BẢN | Biên bản | `20` |
| HỢP ĐỒNG | Hợp đồng | `21` |
| CÔNG VĂN | Công văn | `22` |
| CÔNG ĐIỆN | Công điện | `23` |
| BẢN GHI NHỚ | Bản ghi nhớ | `24` |
| BẢN THỎA THUẬN | Bản thỏa thuận | `25` |
| GIẤY MỜI | Giấy mời | `26` |
| GIẤY GIỚI THIỆU | Giấy giới thiệu | `27` |
| GIẤY NGHỈ PHÉP | Giấy nghỉ phép | `28` |
| THƯ CÔNG | Thư công | `29` |
| BẢN ĐỒ | Bản đồ | `30` |
| BẢN VẼ | Bản vẽ kỹ thuật | `31` |
| Không nhận dạng được | Khác | `32` |

### Pattern regex hỗ trợ OCR

```python
# Số và ký hiệu văn bản: "123/QĐ-BNV" hoặc "Số: 123/QĐ-BNV"
RE_CODE = r'(?:Số[:\s]*)?(\d+)\s*/\s*([A-ZĐÀÁÂÃÈÉÊÌÍÒÓÔÕÙÚĂẮẶĐÊỐỜỢƯỚỮ\-]+)'

# Ngày tháng năm: "ngày 15 tháng 3 năm 2021" hoặc "15/03/2021"
RE_DATE_VN = r'ngày\s+(\d{1,2})\s+tháng\s+(\d{1,2})\s+năm\s+(\d{4})'
RE_DATE_NUM = r'(\d{1,2})[/\-](\d{1,2})[/\-](\d{4})'

# Địa danh + ngày (tiêu chuẩn hành chính):
RE_PLACE_DATE = r'([\w\s]+),\s*ngày\s+(\d{1,2})\s+tháng\s+(\d{1,2})\s+năm\s+(\d{4})'

# Đóng dấu mật
RE_SECRET = r'\b(MẬT|TỐI MẬT|TUYỆT MẬT)\b'
```

---

## Bộ C – Trích xuất thông tin Ảnh / Phim âm bản

Áp dụng cho: file JPEG, TIFF, PNG hoặc PDF chứa toàn ảnh (không có text layer).

| STT | Trường | Tên hiển thị | Nguồn lấy | Kiểu | Bắt buộc |
|---|---|---|---|---|---|
| 1 | `typePic` | Phân loại | OCR chú thích / EXIF / nhập thủ công | String(2) | Bắt buộc |
| 2 | `archivesNumber` | Số lưu trữ đặc thù | Chú thích dưới ảnh hoặc nhập thủ công | String(50) | Tùy chọn |
| 3 | `eventName` | Tên sự kiện | Chú thích dưới ảnh / metadata nhúng | String(500) | Tùy chọn |
| 4 | `imageTitle` | Tiêu đề ảnh | Chú thích dưới ảnh | String(500) | Tùy chọn |
| 5 | `photographer` | Tác giả / Người chụp | Chú thích hoặc EXIF Artist | String(300) | Tùy chọn |
| 6 | `photoPlace` | Địa điểm chụp | Chú thích hoặc EXIF GPS | String(300) | Tùy chọn |
| 7 | `photoTime` | Thời gian chụp | Chú thích hoặc EXIF DateTimeOriginal | Date | Tùy chọn |
| 8 | `colour` | Màu sắc | Phân tích ảnh tự động (màu/trắng đen) | String | Tùy chọn |
| 9 | `filmSize` | Cỡ phim/ảnh | Metadata file (width x height, DPI) | String | Tùy chọn |
| 10 | `docAttached` | Tài liệu đi kèm | Kiểm tra có file đi kèm không | Boolean | Tùy chọn |
| 11 | `mode` | Chế độ sử dụng | Nhập thủ công | String(2) | Tùy chọn |
| 12 | `format` | Tình trạng vật lý | Nhập thủ công sau kiểm tra | String(2) | Tùy chọn |
| 13 | `description` | Ghi chú | Chú thích đầy đủ | String(500) | Tùy chọn |

### Giá trị `typePic`

| Mã | Ý nghĩa |
|---|---|
| `01` | Ảnh dương bản |
| `02` | Phim âm bản |

---

## Bộ D – Trích xuất thông tin Ghi âm / Ghi hình

Áp dụng cho: file MP3, MP4, AVI, WMA, WAV, WMV. Chủ yếu lấy từ metadata nhúng trong file (ID3, MP4 tags), không OCR.

| STT | Trường | Tên hiển thị | Nguồn lấy | Kiểu | Bắt buộc |
|---|---|---|---|---|---|
| 1 | `typeMedia` | Phân loại | Từ định dạng file | String(2) | Bắt buộc |
| 2 | `archivesNumber` | Số lưu trữ | Nhập thủ công | String(50) | Tùy chọn |
| 3 | `eventName` | Tên sự kiện | Tag `Album` hoặc nhập thủ công | String(500) | Tùy chọn |
| 4 | `movieTitle` | Tiêu đề | Tag `Title` | String(500) | Tùy chọn |
| 5 | `recorder` | Tác giả | Tag `Artist` / `Author` | String(300) | Tùy chọn |
| 6 | `recordPlace` | Địa điểm | Tag `Comment` hoặc nhập thủ công | String(300) | Tùy chọn |
| 7 | `recordDate` | Thời gian | Tag `Year` / `Date` | Date | Tùy chọn |
| 8 | `language` | Ngôn ngữ | Tag `Language` hoặc nhập thủ công | String(2) | Tùy chọn |
| 9 | `playTime` | Thời lượng | Duration từ file (HH:MM:SS) | String(8) | Tùy chọn |
| 10 | `quality` | Chất lượng | Bitrate, sample rate từ file | String(50) | Tùy chọn |
| 11 | `docAttached` | Tài liệu đi kèm | Kiểm tra file đi kèm | Boolean | Tùy chọn |
| 12 | `mode` | Chế độ sử dụng | Nhập thủ công | String(2) | Tùy chọn |
| 13 | `format` | Tình trạng vật lý | Nhập thủ công | String(2) | Tùy chọn |
| 14 | `description` | Ghi chú | Tag `Comment` | String(500) | Tùy chọn |

### Giá trị `typeMedia`

| Mã | Ý nghĩa |
|---|---|
| `01` | Ghi âm (audio) |
| `02` | Ghi hình (video) |

---

## Validate sau OCR

Sau khi OCR trích xuất, hệ thống kiểm tra trước khi lưu nháp:

| Trường | Quy tắc validate | Hành động nếu sai |
|---|---|---|
| `issuedDate` / `startDate` | Định dạng DD/MM/YYYY, năm 1945–nay | Gắn cờ, yêu cầu nhập thủ công |
| `codeNumber` | Chỉ chứa số | Strip ký tự thừa, nếu rỗng → cờ đỏ |
| `codeNotation` | Chứa dấu `/` và `-`, không có số | Cờ vàng nếu không đúng pattern |
| `typeDoc` | Phải nằm trong `01`–`32` | Default `32` (Khác) |
| `maintenance` | Phải nằm trong `01`–`07` | Default `07` (Khác) |
| `mode` | Phải là `01`, `02`, `03` | Default `01` (Công khai) |
| `organName` | Không rỗng | Cờ đỏ |
| `subject` | Không rỗng, ≤500 ký tự | Cắt bớt + cờ vàng nếu >500 |
| `confidenceLevel` | < 70% | Gắn cờ "độ tin cậy thấp", highlight màu vàng |
| `confidenceLevel` | < 40% | Gắn cờ "cần kiểm tra lại toàn bộ", highlight màu đỏ |

---

## Mức độ tin cậy OCR (`confidenceLevel`)

| Ngưỡng | Trạng thái | Hiển thị UI | Hành động |
|---|---|---|---|
| ≥ 90% | Tin cậy cao | Xanh lá | Tự động lưu nháp, người dùng xem lại nhẹ |
| 70–89% | Chấp nhận được | Xanh dương | Lưu nháp, highlight các trường có confidence thấp |
| 40–69% | Cần xem lại | Vàng | Bắt buộc người dùng xác nhận từng trường nghi ngờ |
| < 40% | Không tin cậy | Đỏ | OCR thất bại, chuyển sang nhập tay toàn bộ |

---

## Đầu ra của bước OCR

Mỗi file scan sau OCR sinh ra một object JSON nháp:

```json
{
  "fileId": "uuid-...",
  "filePath": "uploads/HS001/van_ban_001.pdf",
  "docType": "vanban",
  "confidenceLevel": 87,
  "status": "draft",
  "needsReview": false,
  "extractedFields": {
    "organName": "BỘ NỘI VỤ",
    "codeNumber": "123",
    "codeNotation": "QĐ-BNV",
    "issuedDate": "15/03/2021",
    "typeName": "QUYẾT ĐỊNH",
    "typeDoc": "02",
    "subject": "Về việc bổ nhiệm Vụ trưởng Vụ Tổ chức cán bộ",
    "autograph": "Nguyễn Văn A",
    "process": "BỘ TRƯỞNG",
    "pageAmount": 3,
    "mode": "01",
    "language": "01"
  },
  "fieldConfidence": {
    "organName": 95,
    "codeNumber": 99,
    "codeNotation": 99,
    "issuedDate": 88,
    "typeName": 97,
    "subject": 82,
    "autograph": 71
  },
  "flaggedFields": ["autograph"]
}
```

---

## Thư viện OCR đề xuất

| Thư viện | Dùng cho | Ghi chú |
|---|---|---|
| `PyMuPDF` (fitz) | Render PDF → ảnh, extract text layer sẵn có | Ưu tiên dùng trước khi OCR |
| `Tesseract` + `pytesseract` | OCR tiếng Việt (`vie`) | Cần cài language pack `vie` |
| `VietOCR` / `PaddleOCR` | Thay thế Tesseract, tốt hơn cho tiếng Việt | Khuyến nghị cho production |
| `Pillow` | Tiền xử lý ảnh (deskew, contrast) trước OCR | |
| `python-doctr` | Deep learning OCR, chính xác cao | Cần GPU nếu xử lý batch lớn |
| `mutagen` | Đọc metadata file audio/video (MP3, MP4) | Cho Bộ D |
| `exifread` | Đọc EXIF ảnh (ngày chụp, GPS, thiết bị) | Cho Bộ C |

### Thứ tự ưu tiên xử lý PDF

```text
1. Thử extract text layer (PyMuPDF) → nếu có text → dùng text, không OCR
2. Nếu text rỗng / < 50 ký tự/trang → đây là scan → chạy OCR
3. Render PDF → ảnh 300dpi → tiền xử lý → Tesseract/PaddleOCR
4. Áp dụng regex/NLP trích xuất từng trường
5. Tính confidenceLevel → lưu JSON nháp
```
