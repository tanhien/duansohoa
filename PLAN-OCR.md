# PLAN-OCR – Kế hoạch OCR trích xuất thông tin file scan

## Mục tiêu

Nhận file PDF scan + tham số định danh, trích xuất metadata để:
1. Tự động điền vào form chỉnh lý, giảm nhập liệu thủ công
2. Phục vụ đóng gói SIP/AIP

---

## Đầu vào

| Tham số | Kiểu | Ý nghĩa | Ví dụ |
| --- | --- | --- | --- |
| `file` | String | Đường dẫn file PDF scan | `uploads/g09/hs001/van_ban_001.pdf` |
| `record_id` | Number | Số thứ tự tài liệu trong hồ sơ | `3` |
| `type` | Number | `0` = Trang bìa hồ sơ · `1` = Văn bản hành chính | `1` |

---

## Phân loại tài liệu

| `type` | Loại | Dấu hiệu nhận dạng | Bộ trường |
| --- | --- | --- | --- |
| `0` | Trang bìa hồ sơ | "HỒ SƠ", "MỤC LỤC", tiêu đề hồ sơ, thời hạn lưu trữ | Bộ A |
| `1` | Văn bản hành chính | Số ký hiệu `123/QĐ-BNV`, quốc hiệu, chữ ký | Bộ B |

---

## Bộ A – Trang bìa hồ sơ (`type = 0`)

| Trường | Tên hiển thị | Kiểu | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| `fileCode` | Mã hồ sơ | String(50) | Bắt buộc | Mã định danh hồ sơ theo hệ thống, dạng `{mcq}.{năm}.{đơn vị}.{stt}` | `G09.2021.01.001` |
| `title` | Tiêu đề hồ sơ | String(500) | Bắt buộc | Tiêu đề đầy đủ của hồ sơ, thường in trên bìa | `Tập quyết định nhân sự năm 2021` |
| `maintenance` | Thời hạn lưu trữ | String(2) | Bắt buộc | Thời hạn bảo quản, tra bảng mã bên dưới | `01` |
| `startDate` | Ngày bắt đầu | Date `DD/MM/YYYY` | Bắt buộc | Ngày của văn bản đầu tiên trong hồ sơ | `01/01/2021` |
| `endDate` | Ngày kết thúc | Date `DD/MM/YYYY` | Bắt buộc | Ngày của văn bản cuối cùng trong hồ sơ | `31/12/2021` |
| `totalDoc` | Tổng số tài liệu | Number | Tùy chọn | Tổng số văn bản/tài liệu trong hồ sơ | `15` |
| `numberOfPaper` | Số tờ | Number | Tùy chọn | Tổng số tờ giấy vật lý | `120` |
| `numberOfPage` | Số trang | Number | Tùy chọn | Tổng số trang (= 2 × số tờ nếu in 2 mặt) | `240` |
| `language` | Ngôn ngữ | String(2) | Tùy chọn | Mã ngôn ngữ: `01` Tiếng Việt, `02` Tiếng Anh, `03` Tiếng Pháp | `01` |
| `mode` | Chế độ sử dụng | String(2) | Tùy chọn | Mức tiếp cận: `01` Thông thường, `02` Hạn chế, `03` Mật, `04` Tối mật, `05` Tuyệt mật | `01` |
| `keyword` | Từ khóa | String(200) | Tùy chọn | Từ khóa tìm kiếm, phân cách bằng dấu chấm phẩy | `nhân sự; quyết định` |
| `inforSign` | Ký hiệu thông tin | String(50) | Tùy chọn | Ký hiệu phân loại nội bộ của cơ quan, thường không có trên bìa scan | `BNV.2021.HS001` |
| `paperFileCode` | Mã hồ sơ giấy gốc | String(50) | Tùy chọn | Mã hồ sơ ghi trên bìa giấy gốc trước khi số hóa | `HOP-001` |
| `description` | Ghi chú | String(2000) | Tùy chọn | Thông tin bổ sung không thuộc các trường trên | |

**Mã `maintenance`:**

| Từ khóa OCR | Mã |
| --- | --- |
| Vĩnh viễn | `01` |
| 70 năm | `02` |
| 50 năm | `03` |
| 30 năm | `04` |
| 20 năm | `05` |
| 10 năm | `06` |
| Khác / không xác định | `07` |

---

## Bộ B – Văn bản hành chính (`type = 1`)

| Trường | Tên hiển thị | Kiểu | Bắt buộc | Mô tả | Ví dụ |
| --- | --- | --- | --- | --- | --- |
| `organName` | Tên cơ quan ban hành | String(200) | Bắt buộc | Tên cơ quan hoặc chức danh ban hành, in ở góc trên trái văn bản | `BỘ NỘI VỤ` |
| `codeNumber` | Số văn bản | String(11) | Bắt buộc | Phần số trong số ký hiệu, trích từ `123/QĐ-BNV` → `123` | `123` |
| `codeNotation` | Ký hiệu văn bản | String(30) | Bắt buộc | Phần ký hiệu sau dấu `/`, trích từ `123/QĐ-BNV` → `QĐ-BNV` | `QĐ-BNV` |
| `issuedDate` | Ngày ban hành | Date `DD/MM/YYYY` | Bắt buộc | Ngày ký ban hành văn bản | `15/03/2021` |
| `typeName` | Loại văn bản | String(100) | Bắt buộc | Tên loại văn bản in trên đầu trang, tra bảng mapping bên dưới | `QUYẾT ĐỊNH` |
| `typeDoc` | Mã loại văn bản | String(2) | Bắt buộc | Mã số tương ứng với `typeName`, sinh tự động từ bảng mapping | `02` |
| `subject` | Trích yếu nội dung | String(500) | Bắt buộc | Nội dung tóm tắt, thường sau cụm "V/v" hoặc "Về việc" | `Về việc bổ nhiệm Vụ trưởng` |
| `autograph` | Người ký | String(200) | Tùy chọn | Họ tên người ký ghi dưới chữ ký ở cuối văn bản | `Nguyễn Văn A` |
| `process` | Chức vụ người ký | String(200) | Tùy chọn | Chức vụ/chức danh của người ký, in trên họ tên | `BỘ TRƯỞNG` |
| `pageAmount` | Số trang | Number(4) | Tùy chọn | Số trang của văn bản này (không phải toàn bộ hồ sơ) | `3` |
| `language` | Ngôn ngữ | String(2) | Tùy chọn | Mã ngôn ngữ: `01` Tiếng Việt, `02` Tiếng Anh, `03` Tiếng Pháp | `01` |
| `mode` | Chế độ sử dụng | String(2) | Tùy chọn | Mức tiếp cận, nhận dạng từ dấu đóng dấu mật trên văn bản, mặc định `01` | `01` |
| `keyword` | Từ khóa | String(100) | Tùy chọn | Từ khóa tìm kiếm, phân cách bằng dấu chấm phẩy | |
| `description` | Ghi chú / nơi nhận | String(500) | Tùy chọn | Danh sách nơi nhận hoặc ghi chú cuối văn bản | |
| `source` | Chiều văn bản | Boolean | Tùy chọn | `0` = văn bản đi (cơ quan ban hành), `1` = văn bản đến (cơ quan nhận) | `0` |

**Mapping `typeName` → `typeDoc`:**

| Từ khóa OCR | `typeName` | `typeDoc` |
| --- | --- | --- |
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
| Không nhận dạng | Khác | `32` |

---

## Đặt tên file đóng gói (`packedFileName`)

Theo **7197/BKHCN-CDSQG Mục 2.4b**:

```
packedFileName = {DocCode}_{YYYYMMDD}.pdf
DocCode        = {mcq}_{LV}_{sttSeq}_V1
```

| Thành phần | Ý nghĩa | Ví dụ |
| --- | --- | --- |
| `mcq` | Mã cơ quan (QĐ 20/2020 + QĐ 09/2025) | `G09.01` |
| `LV` | Viết tắt loại văn bản (từ `typeName`) | `QD` |
| `sttSeq` | Số thứ tự số hoá, 6 chữ số bù 0 | `000021` |
| `V1` | Phiên bản bản quét gốc | `V1` |
| `YYYYMMDD` | Ngày số hoá | `20250414` |

> `record_id` và `fileCode` lưu trong JSON để sắp xếp, liên kết hồ sơ – không dùng trong tên file.

---

## JSON Output – Mẫu chung

Một object JSON thống nhất cho cả Bộ A và Bộ B. Các trường không áp dụng để `null`.

```json
{
  "file": "uploads/.../van_ban_001.pdf",
  "record_id": 1,
  "type": 0,
  "docCode": "G09.01_QD_000001_V1",
  "confidenceLevel": 87,

  "extractedFields": {

    "fileCode":       null,
    "title":          null,
    "maintenance":    null,
    "startDate":      null,
    "endDate":        null,
    "totalDoc":       null,
    "numberOfPaper":  null,
    "numberOfPage":   null,
    "paperFileCode":  null,

    "organName":      null,
    "codeNumber":     null,
    "codeNotation":   null,
    "issuedDate":     null,
    "typeName":       null,
    "typeDoc":        null,
    "subject":        null,
    "autograph":      null,
    "process":        null,
    "pageAmount":     null,
    "source":         null,

    "language":       null,
    "mode":           null,
    "keyword":        null,
    "inforSign":      null,
    "description":    null
  }
}
```

---

## Ví dụ 1 – Bộ A: Trang bìa hồ sơ

```json
{
  "file": "uploads/g09/hs_2021_001/bia_ho_so.pdf",
  "record_id": 0,
  "type": 0,
  "docCode": null,
  "confidenceLevel": 91,

  "extractedFields": {
    "fileCode":       "G09.2021.01.001",
    "title":          "Tập quyết định nhân sự năm 2021",
    "maintenance":    "01",
    "startDate":      "01/01/2021",
    "endDate":        "31/12/2021",
    "totalDoc":       15,
    "numberOfPaper":  120,
    "numberOfPage":   240,
    "paperFileCode":  "HOP-001",

    "organName":      null,
    "codeNumber":     null,
    "codeNotation":   null,
    "issuedDate":     null,
    "typeName":       null,
    "typeDoc":        null,
    "subject":        null,
    "autograph":      null,
    "process":        null,
    "pageAmount":     null,
    "source":         null,

    "language":       "01",
    "mode":           "01",
    "keyword":        "nhân sự; quyết định",
    "inforSign":      "BNV.2021.HS001",
    "description":    null
  }
}
```

---

## Ví dụ 2 – Bộ B: Văn bản hành chính (QĐ 09/2025/QĐ-TTg)

```json
{
  "file": "uploads/g09/hs_2025_001/09-2025-QD-TTg.pdf",
  "record_id": 1,
  "type": 1,
  "docCode": "G22_QD_000009_V1",
  "confidenceLevel": 93,

  "extractedFields": {
    "fileCode":       null,
    "title":          null,
    "maintenance":    null,
    "startDate":      null,
    "endDate":        null,
    "totalDoc":       null,
    "numberOfPaper":  null,
    "numberOfPage":   null,
    "paperFileCode":  null,

    "organName":      "THỦ TƯỚNG CHÍNH PHỦ",
    "codeNumber":     "09",
    "codeNotation":   "QĐ-TTg",
    "issuedDate":     "14/04/2025",
    "typeName":       "QUYẾT ĐỊNH",
    "typeDoc":        "02",
    "subject":        "Sửa đổi, bổ sung Quyết định số 20/2020/QĐ-TTg ngày 22 tháng 7 năm 2020 của Thủ tướng Chính phủ về mã định danh điện tử của các cơ quan, tổ chức phục vụ kết nối, chia sẻ dữ liệu với các bộ, ngành, địa phương",
    "autograph":      "Nguyễn Chí Dũng",
    "process":        "KT. THỦ TƯỚNG / PHÓ THỦ TƯỚNG",
    "pageAmount":     3,
    "source":         "0",

    "language":       "01",
    "mode":           "01",
    "keyword":        null,
    "inforSign":      null,
    "description":    null
  }
}
```
