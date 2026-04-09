# Kế hoạch OCR – Trích xuất thông tin từ file scan

## Mục tiêu

Nhận vào danh sách hồ sơ + file PDF đã scan. OCR sẽ đọc từng file, trích xuất các trường metadata để:
1. Tự động điền vào form chỉnh lý, giảm nhập liệu thủ công
2. Phục vụ đóng gói SIP/AIP

---

## Luồng xử lý OCR

**Tham số đầu vào khi gọi OCR**:

| Tham số | Ý nghĩa | Ví dụ |
| --- | --- | --- |
| `file` | Đường dẫn file PDF scan trên MinIO/local | `uploads/phong_g09/hs_2025_001/van_ban_001.pdf` |
| `record_id` | Số thứ tự tài liệu trong hồ sơ (dùng để sắp xếp) | `3` |
| `type` | Loại tài liệu để phân loại(0: Trang bìa hồ sơ - 1: Văn bản hành chính) | `0` |

```text
[file +record_id]
         ↓
PDF scan → Phát hiện loại tài liệu → OCR từng trang → Trích xuất trường                                             
         ↓ output
[extractedFields + docCode + packedFileName + confidenceLevel]
```

---

## Phân loại tài liệu (bước đầu tiên)

OCR cần xác định loại tài liệu trước khi áp dụng bộ trường tương ứng.

| Loại | Dấu hiệu nhận dạng | Bộ trường áp dụng |
|---|---|---|
| **Trang bìa hồ sơ** | Chứa "HỒ SƠ", "MỤC LỤC", tiêu đề hồ sơ, thời hạn lưu trữ | Bộ A – Hồ sơ |
| **Văn bản hành chính** | Có số ký hiệu dạng `123/QĐ-BNV`, có quốc hiệu, có chữ ký | Bộ B – Văn bản |

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
| 17 | `docOrdinal` | Số thứ tự trong hồ sơ | **INPUT** – truyền từ Module 1, không OCR (xem Luồng xử lý OCR) | Number | Bắt buộc | `1` |

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

## Quy tắc đặt tên file đóng gói (`packedFileName`)

Trường `packedFileName` là **địa chỉ tài liệu gốc** – tên file sẽ được dùng khi copy vào thư mục `representations/rep1/data/` trong gói SIP/AIP. Tuân theo quy tắc đặt tên tệp số hoá của **7197/BKHCN-CDSQG Mục 2.4b**.

### Công thức

```text
packedFileName = {DocCode}_{YYYYMMDD}.pdf
DocCode        = {mcq}_{LV}_{sttSeq}_V1
```

| Thành phần | Ý nghĩa | Lấy từ | Ví dụ |
| --- | --- | --- | --- |
| `mcq` | Mã cơ quan phân cấp (theo QĐ 20/2020 + QĐ 09/2025) | Tham số đầu vào từ Module 1 | `G09.01` |
| `LV` | Viết tắt loại văn bản | OCR `typeName` → map sang LV (xem bảng Bộ B) | `QD` |
| `sttSeq` | Số thứ tự số hoá tuần tự, **6 chữ số, bù 0** | Tham số đầu vào (hệ thống tự cấp) | `000021` |
| `V1` | Phiên bản (mặc định bản quét gốc) | Cố định `V1` | `V1` |
| `YYYYMMDD` | Ngày số hoá | Hệ thống tự lấy (hoặc `digitizationDate` truyền vào) | `20250414` |

> `docOrdinal` và `fileCode` vẫn được lưu trong JSON output để liên kết hồ sơ và sắp xếp thứ tự — nhưng **không dùng để đặt tên file**.

### Ví dụ

| `mcq` | `LV` | `sttSeq` | `YYYYMMDD` | `packedFileName` |
| --- | --- | --- | --- | --- |
| `G09.01` | `QD` | `000001` | `20250102` | `G09.01_QD_000001_V1_20250102.pdf` |
| `G09.01` | `CV` | `000021` | `20250102` | `G09.01_CV_000021_V1_20250102.pdf` |
| `G22` | `QD` | `000009` | `20250414` | `G22_QD_000009_V1_20250414.pdf` |

### Vị trí trong gói SIP

```text
{UUID}/
  representations/
    rep1/
      data/
        G09.01_QD_000001_V1_20250102.pdf   ← tài liệu #1 trong hồ sơ
        G09.01_CV_000002_V1_20250102.pdf   ← tài liệu #2 trong hồ sơ
        G09.01_BB_000003_V1_20250103.pdf   ← tài liệu #3 trong hồ sơ
      metadata/
        descriptive/
          EAD_doc_G09.01_QD_000001_V1.xml
        preservation/
          PREMIS_{uuid}.xml
```

> **Thứ tự trong hồ sơ** được xác định bởi `docOrdinal` (lưu trong JSON và EAD), không phải tên file. Tên file theo DocCode đảm bảo duy nhất toàn hệ thống.

---

## Đầu ra của bước OCR

Mỗi file scan sau OCR sinh ra một object JSON nháp:

```json
{
  "fileId": "uuid-...",
  "filePath": "uploads/HS001/van_ban_001.pdf",
  "fileCode": "G09.2025.01.001",
  "docOrdinal": 1,
  "docCode": "G09.01_QD_000001_V1",
  "packedFileName": "G09.01_QD_000001_V1_20250102.pdf",
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

## Ví dụ thực tế – OCR file `09/2025/QĐ-TTg`

> **File gốc:** `FRD/09-2025-QD-TTg.pdf` – Quyết định 09/2025/QĐ-TTg của Thủ tướng Chính phủ  
> **Loại tài liệu phát hiện:** Văn bản hành chính → áp dụng **Bộ B**  
> **Dấu hiệu nhận dạng:** Có quốc hiệu, có "QUYẾT ĐỊNH" in hoa, có số ký hiệu dạng `09/2025/QĐ-TTg`

### Nội dung văn bản (nhận biết qua OCR)

```text
THỦ TƯỚNG CHÍNH PHỦ          CỘNG HOÀ XÃ HỘI CHỦ NGHĨA VIỆT NAM
                                   Độc lập - Tự do - Hạnh phúc
Số: 09/2025/QĐ-TTg            Hà Nội, ngày 14 tháng 4 năm 2025

                            QUYẾT ĐỊNH
Sửa đổi, bổ sung Quyết định số 20/2020/QĐ-TTg ngày 22 tháng 7 năm 2020
của Thủ tướng Chính phủ về mã định danh điện tử của các cơ quan, tổ chức
phục vụ kết nối, chia sẻ dữ liệu với các bộ, ngành, địa phương
                                           ...
                                    KT. THỦ TƯỚNG
                                    PHÓ THỦ TƯỚNG
                                    Nguyễn Chí Dũng
```

### Kết quả trích xuất (Bộ B)

| Trường | Giá trị OCR trích xuất | Nguồn nhận dạng | Confidence |
| --- | --- | --- | --- |
| `organName` | `THỦ TƯỚNG CHÍNH PHỦ` | Góc trên trái | 98% |
| `codeNumber` | `09` | Regex `RE_CODE` → phần số trước `/2025` | 99% |
| `codeNotation` | `QĐ-TTg` | Regex `RE_CODE` → phần sau `/2025/` | 99% |
| `issuedDate` | `14/04/2025` | Regex `RE_PLACE_DATE` → "ngày 14 tháng 4 năm 2025" | 97% |
| `typeName` | `QUYẾT ĐỊNH` | Tiêu đề in hoa ở trung tâm trang | 99% |
| `typeDoc` | `02` | Mapping: QUYẾT ĐỊNH → `02` | — |
| `subject` | `Sửa đổi, bổ sung Quyết định số 20/2020/QĐ-TTg ... chia sẻ dữ liệu với các bộ, ngành, địa phương` | Dòng in đậm sau "QUYẾT ĐỊNH" | 91% |
| `autograph` | `Nguyễn Chí Dũng` | Tên in dưới chữ ký | 88% |
| `process` | `KT. THỦ TƯỚNG / PHÓ THỦ TƯỚNG` | Dòng trên tên người ký | 85% |
| `pageAmount` | `3` | Đếm tổng số trang PDF | 100% |
| `language` | `01` | Suy từ văn bản (tiếng Việt) | — |
| `mode` | `01` | Không có dấu MẬT/TỐI MẬT | — |
| `source` | `0` | Văn bản đi (do cơ quan ban hành) | — |
| `confidenceLevel` | `93` | Trung bình có trọng số các trường | — |
| **`docCode`** | **`G22_QD_000009_V1`** | **`mcq`(input) + `LV`(OCR) + `sttSeq`(input) + `V1`(mặc định)** | **—** |
| **`packedFileName`** | **`G22_QD_000009_V1_20250414.pdf`** | **`docCode` + `_` + `digitizationDate` (7197 Mục 2.4b)** | **—** |

### JSON output

```json
{
  "fileId": "uuid-A1B2C3D4-E5F6-7890-ABCD-EF1234567890",
  "filePath": "uploads/phong_g09/hs_2025_001/09-2025-QD-TTg.pdf",
  "fileCode": "G09.2025.01.001",
  "docOrdinal": 1,
  "docCode": "G22_QD_000009_V1",
  "packedFileName": "G22_QD_000009_V1_20250414.pdf",
  "docType": "vanban",
  "confidenceLevel": 93,
  "status": "draft",
  "needsReview": false,
  "extractedFields": {
    "organName": "THỦ TƯỚNG CHÍNH PHỦ",
    "codeNumber": "09",
    "codeNotation": "QĐ-TTg",
    "issuedDate": "14/04/2025",
    "typeName": "QUYẾT ĐỊNH",
    "typeDoc": "02",
    "subject": "Sửa đổi, bổ sung Quyết định số 20/2020/QĐ-TTg ngày 22 tháng 7 năm 2020 của Thủ tướng Chính phủ về mã định danh điện tử của các cơ quan, tổ chức phục vụ kết nối, chia sẻ dữ liệu với các bộ, ngành, địa phương",
    "autograph": "Nguyễn Chí Dũng",
    "process": "KT. THỦ TƯỚNG / PHÓ THỦ TƯỚNG",
    "pageAmount": 3,
    "language": "01",
    "mode": "01",
    "source": "0"
  },
  "fieldConfidence": {
    "organName": 98,
    "codeNumber": 99,
    "codeNotation": 99,
    "issuedDate": 97,
    "typeName": 99,
    "subject": 91,
    "autograph": 88,
    "process": 85
  },
  "flaggedFields": []
}
```
