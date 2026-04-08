# Hệ thống Chỉnh lý và Đóng gói SIP/AIP

## Căn cứ pháp lý

| File | Nội dung |
|---|---|
| `FRD/05-bnv.pdf` (364 trang) | Thông tư /2025/TT-BNV – tài liệu kỹ thuật chính: cấu trúc gói tin, quy trình, metadata schemas |
| `FRD/20-2020-QD-TTg.pdf` | Quyết định 20/2020/QĐ-TTg (file scan) |
| `FRD/09-2025-QD-TTg.pdf` | Quyết định 09/2025/QĐ-TTg (file scan) |
| `FRD/7197-BKHCN-CDSQG.pdf` | Văn bản 7197/BKHCN về chuẩn dữ liệu quốc gia (file scan) |

---

## Tổng quan hệ thống

**Mô hình:** Web app (client-server), multi-user, triển khai nội bộ.

**Luồng chính:**
```
[Import danh sách + PDF] → [Chỉnh lý] → [Đóng gói SIP] → [Validate] → [Thu nộp & Phê duyệt] → [Chuyển AIP]
```

**4 loại gói tin (theo Phụ lục I–IV Thông tư):**
- `SIP_hoso` (Phụ lục III, trang 182) – Hồ sơ nộp, khác hệ thống
- `SIP_tailieu` (Phụ lục IV, trang 239) – Tài liệu lẻ nộp, khác hệ thống
- `AIP_hoso` (Phụ lục I, trang 33) – Hồ sơ lưu trữ bảo quản
- `AIP_tailieu` (Phụ lục II, trang 110) – Tài liệu lưu trữ bảo quản
- `DIP` (Phụ lục V, trang 292) – Gói cung cấp cho người dùng (giai đoạn sau)

---

## Cấu trúc gói tin (E-ARK CSIP + METS)

```
{uuid-UUID}/                            ← tên thư mục = OBJID (UUID viết hoa)
│
├── METS.xml                            ← METS cấp gói (root)
│
├── metadata/
│   ├── descriptive/
│   │   └── EAD.xml                     ← Metadata mô tả hồ sơ (simpledc schema)
│   └── preservation/
│       └── PREMIS.xml                  ← Metadata bảo quản (PREMIS v3)
│
├── representations/
│   └── rep1/
│       ├── METS.xml                    ← METS cấp bản đại diện
│       ├── data/
│       │   ├── {mã_hồ_sơ}.001.pdf     ← PDF/A 2 lớp, ≥200dpi
│       │   ├── {mã_hồ_sơ}.002.pdf
│       │   └── ...
│       └── metadata/
│           ├── descriptive/
│           │   └── EAD_{docCode}.xml   ← Metadata từng tài liệu
│           └── preservation/
│               └── PREMIS_{uuid}.xml
│
├── schemas/
│   ├── EAD.xsd
│   └── DILCISExtensionMETS.xsd
│
└── documentation/                      ← Tùy chọn
    └── UserManual.pdf
```

> Gói được nén thành file ZIP: tên = `{OBJID}.zip`

---

## Quy tắc đặt tên (Naming Conventions)

### 1. Tên thư mục gói & tên file ZIP

| Loại gói | Quy tắc OBJID | Tên thư mục | Tên file ZIP |
|---|---|---|---|
| **SIP_hoso** | `uuid-{UUID}` (viết hoa) | Giống OBJID | `uuid-{UUID}.zip` |
| **SIP_tailieu** | `uuid-{UUID}` (viết hoa) | Giống OBJID | `uuid-{UUID}.zip` |
| **AIP_hoso** | `urn:{MãPhông}:uuid:{UUID}` (viết hoa) | Thay `:` → `_` | `urn_{MãPhông}_uuid_{UUID}.zip` |
| **AIP_tailieu** | `urn:{MãPhông}:uuid:{UUID}` (viết hoa) | Thay `:` → `_` | `urn_{MãPhông}_uuid_{UUID}.zip` |

**Ví dụ:**
```
SIP:  uuid-7D0D1987-0F1C-47A7-8FD6-CC5C7DE4064F/     → uuid-7D0D1987-0F1C-47A7-8FD6-CC5C7DE4064F.zip
AIP:  urn_G09_uuid_9C13E70E-08B2-4C54-8BAF-979B35D01B4D/  → urn_G09_uuid_9C13E70E-08B2-4C54-8BAF-979B35D01B4D.zip
```

> **Lưu ý:** UUID tự sinh, tất cả ký tự hex phải **VIẾT HOA**.

---

### 2. Mã định danh nghiệp vụ

#### arcFileCode – Mã lưu trữ hồ sơ (trong EAD.xml cấp hồ sơ)

Cấu trúc: `{MãCQLT}.{MãCQHT}.{Năm}.{SốLầnNộp}.{STTHồSơ}`

| Thành phần | Mô tả | Ví dụ |
|---|---|---|
| MãCQLT | Mã cơ quan quản lý lưu trữ (cơ quan nhận) | `PARTCODE` |
| MãCQHT | Mã cơ quan hình thành phông (cơ quan nộp) | `G09` |
| Năm | Năm hình thành tài liệu | `2021` |
| SốLầnNộp | Số lần nộp lưu (hệ thống tự cập nhật) | `01` |
| STTHồSơ | Số thứ tự hồ sơ trong lần nộp | `001` |

**Ví dụ:** `PARTCODE.G09.2021.01.001`

> Trường `arcFileCode` dùng cho **AIP** (hồ sơ lưu trữ). Trường `fileCode` dùng cho **SIP** (hồ sơ nộp).

#### arcDocCode – Mã lưu trữ tài liệu (trong EAD_doc.xml cấp tài liệu)

Cấu trúc: `{MãCQLT}{MãCQHT}{Năm}{SốLầnNộp}{STTTàiLiệu}`

| Thành phần | Mô tả | Ví dụ |
|---|---|---|
| MãCQLT | Mã cơ quan quản lý lưu trữ | `PARTCODE` |
| MãCQHT | Mã định danh cơ quan, tổ chức nộp | `G09` |
| Năm | Năm hình thành tài liệu | `2021` |
| SốLầnNộp | Số lần nộp lưu | `01` |
| STTTàiLiệu | **7 ký tự**, bắt đầu từ `0000001` | `0000001` |

**Ví dụ:** `PARTCODE.G09.2021.01.0000001`

#### Mã gói tin (trong metadata SIP_tailieu)

Cấu trúc: `{MãCQHT}{Năm}{SốLầnNộp}{STTGóiTin}`

**Ví dụ:** `G09.2021.01.001`

#### Mã yêu cầu đăng ký nộp

Cấu trúc: `{STTĐăngKý}.{KýHiệuNộp}.{TênyViếtTắtCQ}.{SốLầnNộp}.{NămĐăngKý}`

**Ví dụ:** `NL.BNV.2023.01`

---

### 3. Tên file tài liệu trong `data/`

| Trường hợp | Quy tắc đặt tên | Ví dụ |
|---|---|---|
| Tách từng tài liệu | `{mã_hồ_sơ}.{STT}.{ext}` | `G09.2021.01.001.001.pdf` |
| Không tách (cả hồ sơ 1 file) | `{mã_hồ_sơ}.{ext}` | `G09.2021.01.001.pdf` |
| Tài liệu liên kết (holey file) | `{tên_file}.fetch.txt` | `abc.pdf.fetch.txt` |

**Định dạng file cho phép:**

| Loại tài liệu | Định dạng | Yêu cầu kỹ thuật |
|---|---|---|
| Văn bản hành chính | `.pdf/a` (PDF/A 2 lớp) | ≥200 dpi, màu 24-bit |
| Bản đồ, bản vẽ | `.pdf/a` (PDF/A 2 lớp) | ≥300 dpi |
| Ảnh (dương bản) | `.jpeg`, `.pdf`, `.tiff`, `.png` | ≥200 dpi |
| Phim âm bản | `.jpeg`, `.tiff` | Quét chuyên dụng |
| Ghi âm | `.mp3`, `.wav` (không nén), `.wma` | ≥1500 kbps |
| Ghi hình / Video | `.mp4` (MPEG-4), `.avi`, `.wmv` | ≥1500 kbps |

---

### 4. Tên file metadata trong `rep1/metadata/`

#### Metadata mô tả (descriptive)

Quy tắc: `{TiêuChuẩn}_{LoạiTàiLiệu}_{TênFile}.xml`

| Loại tài liệu | Schema file | Ví dụ tên file EAD |
|---|---|---|
| Văn bản | `EAD_doc.xsd` | `EAD_doc_File1.xml` |
| Phim âm bản / Ảnh | `EAD_pic.xsd` | `EAD_pic_File2.xml` |
| Ghi âm / Ghi hình | `EAD_media.xsd` | `EAD_media_File3.xml` |

> File EAD cấp gói hồ sơ (không phải tài liệu) đặt tên là `EAD.xml`, dùng schema `EAD.xsd`.

#### Metadata bảo quản (preservation)

| Phạm vi | Tên file |
|---|---|
| Bản đại diện (`rep1`) | `PREMIS_rep1.xml` |
| Từng tài liệu | `PREMIS_{uuid}.xml` |
| Gói hồ sơ (root) | `PREMIS.xml` |

---

### 5. Tên file schemas trong `schemas/`

| File schema | Mục đích |
|---|---|
| `METS.xsd` | Cấu trúc file METS |
| `EAD.xsd` | Metadata hồ sơ |
| `EAD_doc.xsd` | Metadata tài liệu văn bản |
| `EAD_pic.xsd` | Metadata tài liệu phim/ảnh |
| `EAD_media.xsd` | Metadata tài liệu âm thanh/video |
| `DILCISExtensionMETS.xsd` | Extension CSIP |
| `DILCISExtensionSIPMETS.xsd` | Extension SIP (chỉ cho SIP) |

---

### 6. Ví dụ đầy đủ cấu trúc gói SIP_hoso

```
uuid-7D0D1987-0F1C-47A7-8FD6-CC5C7DE4064F/
│
│  [METS.xml root - csip:OAISPACKAGETYPE="SIP"]
│  OBJID="uuid-7D0D1987-0F1C-47A7-8FD6-CC5C7DE4064F"
│  LABEL="Hồ sơ G09.2021.01.TCCB về tập quyết định nhân sự năm 2021"
│
├── METS.xml
├── metadata/
│   ├── descriptive/
│   │   └── EAD.xml            ← fileCode="G09.2021.01.001", title=...
│   └── preservation/
│       └── PREMIS.xml
├── representations/
│   └── rep1/
│       ├── METS.xml           ← csip:OAISPACKAGETYPE="SIP"
│       ├── data/
│       │   ├── G09.2021.01.001.001.pdf   ← tài liệu 1
│       │   ├── G09.2021.01.001.002.pdf   ← tài liệu 2
│       │   └── G09.2021.01.001.003.pdf   ← tài liệu 3
│       └── metadata/
│           ├── descriptive/
│           │   ├── EAD_doc_G09.2021.01.001.001.xml
│           │   ├── EAD_doc_G09.2021.01.001.002.xml
│           │   └── EAD_doc_G09.2021.01.001.003.xml
│           └── preservation/
│               ├── PREMIS_rep1.xml
│               ├── PREMIS_uuid-AAA...xml
│               ├── PREMIS_uuid-BBB...xml
│               └── PREMIS_uuid-CCC...xml
├── schemas/
│   ├── METS.xsd
│   ├── EAD.xsd
│   ├── EAD_doc.xsd
│   ├── DILCISExtensionMETS.xsd
│   └── DILCISExtensionSIPMETS.xsd
└── documentation/             ← tùy chọn
    └── UserManual.pdf
```

---

## Cấu trúc METS.xml

Mỗi file METS.xml gồm các phần tử:

| Phần tử | Thuộc tính quan trọng | Ghi chú |
|---|---|---|
| `<mets>` | `OBJID="uuid-{UUID}"`, `TYPE="Mixed"`, `PROFILE` | UUID tự sinh, viết hoa |
| `<metsHdr>` | `CREATEDATE`, `RECORDSTATUS="NEW"`, **`csip:OAISPACKAGETYPE="SIP"`** hoặc `"AIP"` | Bắt buộc |
| `<agent ROLE="CREATOR">` | `TYPE="ORGANIZATION"` | Tên + mã phông cơ quan nộp |
| `<agent ROLE="ARCHIVIST">` | `TYPE="ORGANIZATION"` | Tên + mã cơ quan lưu trữ |
| `<dmdSec>` | `ID="uuid-..."`, `CREATED` | Tham chiếu EAD.xml (URL + SHA-256 + size) |
| `<amdSec>/<digiprovMD>` | | Tham chiếu PREMIS.xml |
| `<fileSec>/<fileGrp>` | `USE="Schemas"`, `USE="Representations/rep1"` | Danh sách file với checksum |
| `<structMap LABEL="CSIP">` | `TYPE="PHYSICAL"` | Cấu trúc: Metadata, Schemas, Representations/rep1 |

---

## Cấu trúc Metadata XML

### EAD.xml – Metadata hồ sơ (simpledc)
```xml
<simpledc>
  <fileCode>G09.2021.01.TCCB</fileCode>
  <title>Tập quyết định nhân sự năm 2021</title>
  <maintenance>01</maintenance>       <!-- 01=Vĩnh viễn, 02=70 năm... -->
  <mode>01</mode>                     <!-- 01=Công khai, 02=Có điều kiện, 03=Mật -->
  <language>01</language>             <!-- 01=Tiếng Việt -->
  <startDate>2021-01-01</startDate>
  <endDate>2021-12-31</endDate>
  <keyword>nhân sự</keyword>
  <totalDoc>10</totalDoc>
  <numberOfPaper>10</numberOfPaper>
  <numberOfPage>50</numberOfPage>
  <riskRecovery>1</riskRecovery>
  <riskRecoveryStatus>01</riskRecoveryStatus>
</simpledc>
```

### EAD_{docCode}.xml – Metadata tài liệu văn bản
```xml
<simpledc>
  <arcDocCode>PARTCODE.G09.2021.01.0000001</arcDocCode>
  <maintenance>01</maintenance>
  <typeDoc>01</typeDoc>               <!-- 01=Nghị quyết, 02=Quyết định... 32=Khác -->
  <codeNumber>123/QĐ-BNV</codeNumber>
  <signer>Bộ trưởng Nguyễn Văn A</signer>
  <issuedDate>2021-03-15</issuedDate>
  <pageCount>3</pageCount>
  <mode>01</mode>
  <riskRecovery>1</riskRecovery>
</simpledc>
```

### PREMIS.xml – Metadata bảo quản (PREMIS v3)
```xml
<premis xmlns:premis="http://www.loc.gov/premis/v3" version="3.0">
  <object xmlID="...">
    <objectIdentifier>
      <objectIdentifierType>UUID</objectIdentifierType>
      <objectIdentifierValue>uuid-...</objectIdentifierValue>
    </objectIdentifier>
    <objectCategory>file</objectCategory>
    <objectCharacteristics>
      <fixity>
        <messageDigestAlgorithm>SHA-256</messageDigestAlgorithm>
        <messageDigest>ABC123...</messageDigest>
      </fixity>
      <size>12345</size>
      <format>
        <formatDesignation>
          <formatName>PDF/A</formatName>
        </formatDesignation>
      </format>
    </objectCharacteristics>
  </object>
  <event>
    <eventIdentifier>...</eventIdentifier>
    <eventType>ingest</eventType>
    <eventDateTime>2025-01-13T15:46:25+07:00</eventDateTime>
    <linkingAgentIdentifier>...</linkingAgentIdentifier>
  </event>
</premis>
```

---

## Các Module hệ thống

### Module 1: Import & Quản lý danh sách
- Nhập danh sách hồ sơ/tài liệu từ file **Excel/CSV**
- Mapping cột: mã hồ sơ, tiêu đề, thời hạn, ngôn ngữ, ngày bắt đầu/kết thúc, mức tiếp cận
- Upload file PDF đã scan, gán vào hồ sơ/tài liệu tương ứng
- Validate: định dạng PDF/A, độ phân giải ≥200dpi (hành chính) / ≥300dpi (bản vẽ)

### Module 2: Chỉnh lý (Tổ chức, Phân loại)
- Quản lý **Phông lưu trữ**: mã phông, tên phông, lịch sử đơn vị hình thành phông
- Quản lý **Hồ sơ**: mã hồ sơ, tiêu đề, thời hạn, ngôn ngữ, từ khóa, tổng số tài liệu
- Quản lý **Tài liệu**: mã lưu trữ, loại VB (01–32), số ký hiệu, ngày, tác giả
- Hỗ trợ tài liệu phim/ảnh (EAD_pic) và âm thanh/video (EAD_media)
- Phân quyền: CREATOR, ARCHIVIST, viên chức nghiệp vụ
- Kiểm tra trùng lặp tài liệu

### Module 3: Sinh Metadata XML
- Sinh **EAD.xml** theo loại tài liệu (văn bản / ảnh / phim âm bản / âm thanh-video)
- Sinh **PREMIS.xml** với object/event/agent theo PREMIS v3
- Sinh **METS.xml** hai cấp (gói + bản đại diện) với UUID viết hoa
- Tính **SHA-256** cho tất cả file trước khi sinh METS

### Module 4: Đóng gói & Nén ZIP
- Tạo cấu trúc thư mục đúng quy định
- Copy PDF vào `representations/rep1/data/`, đặt tên `{mã_hồ_sơ}.{stt}.pdf`
- Copy schemas XSD vào `schemas/`
- Ký số file PDF (chữ ký số cơ quan có thẩm quyền)
- Nén thành ZIP: tên = `{OBJID}.zip`

### Module 5: Validate gói tin
- Validate METS.xml theo `mets1_12.xsd` + `DILCISExtensionMETS.xsd`
- Validate EAD.xml theo XSD tương ứng (văn bản / ảnh / media)
- Validate PREMIS.xml theo `premis-3-0.xsd`
- Kiểm tra SHA-256 toàn bộ file referenced trong METS
- Kiểm tra đủ số tài liệu theo mục lục
- Kiểm tra virus

### Module 6: Thu nộp & Phê duyệt
- **Đăng ký nộp**: thông tin đơn vị, khối tài liệu, phương án phân loại
- **Trạng thái SIP**: `Chờ tiếp nhận` → `Đang xử lý` → `Đạt/Không đạt` → `Hoàn thành`
- **Kiểm tra nghiệp vụ**: virus, xác thực gói SIP, đối chiếu mục lục, nội dung/thời hạn
- **Biên bản**: giao nhận, trả lại, phê duyệt
- Thời hạn: ≤60 ngày xử lý, ≤7 ngày làm việc trả lời đăng ký

### Module 7: Chuyển đổi SIP → AIP
- Sau phê duyệt → tự động tạo AIP
- Cập nhật `csip:OAISPACKAGETYPE="SIP"` → `"AIP"`
- Thêm PREMIS `ingest event`
- Lưu vào kho bảo quản
- Backup: hằng ngày (incremental), hằng tháng (full), 3 năm (full archive)

---

## Tech Stack

| Thành phần | Khuyến nghị | Ghi chú |
|---|---|---|
| Backend API | **Python FastAPI** | Hoặc Java Spring Boot |
| Frontend | **React + TypeScript** | Ant Design / MUI |
| Database | **PostgreSQL** | Metadata phông/hồ sơ/tài liệu |
| File Storage | **MinIO** | S3-compatible, lưu PDF gốc + ZIP |
| XML Processing | **lxml + xmlschema** | Sinh và validate METS/EAD/PREMIS |
| ZIP packaging | Python `zipfile` | Đóng gói SIP/AIP |
| PDF validation | **pikepdf** | Kiểm tra PDF/A |
| Checksum | `hashlib` SHA-256 | Tính cho tất cả file |
| UUID | Python `uuid` | UUID viết hoa theo chuẩn |
| Chữ ký số | OpenSSL / SignServer | Cần hạ tầng PKI |
| Background jobs | **Celery + Redis** | Đóng gói, validate async |

---

## Thứ tự triển khai (6 Sprints)

| Sprint | Nội dung |
|---|---|
| **Sprint 1** | DB schema (phông/hồ sơ/tài liệu) + project setup + Import module |
| **Sprint 2** | Module chỉnh lý: CRUD hồ sơ, tài liệu, phân loại theo loại |
| **Sprint 3** | Sinh XML: EAD + PREMIS + METS + tính SHA-256 |
| **Sprint 4** | Đóng gói ZIP + Validate XSD + kiểm tra integrity |
| **Sprint 5** | Quy trình thu nộp, phê duyệt, quản lý trạng thái SIP |
| **Sprint 6** | Chuyển đổi SIP→AIP + Bảo quản + Backup |

---

## Verification (kiểm thử end-to-end)

1. Tạo 1 hồ sơ test với 3 file PDF → đóng gói SIP_hoso → giải nén kiểm tra cấu trúc thư mục
2. Validate METS.xml bằng `xmllint` với schema E-ARK CSIP
3. Validate EAD.xml và PREMIS.xml bằng XSD tương ứng
4. Kiểm tra SHA-256 của tất cả file referenced trong METS khớp với checksum thực tế
5. Chuyển đổi SIP → AIP: kiểm tra `csip:OAISPACKAGETYPE="AIP"` + PREMIS ingest event

---

## Danh mục loại văn bản (typeDoc 01–32)

| Mã | Loại | Mã | Loại |
|---|---|---|---|
| 01 | Nghị quyết | 17 | Phiếu gửi |
| 02 | Quyết định | 18 | Phiếu chuyển |
| 03 | Chỉ thị | 19 | Phiếu báo |
| 04 | Quy định | 20 | Biên bản |
| 05 | Thông tư | 21 | Hợp đồng |
| 06 | Thông tư liên tịch | 22 | Công văn |
| 07 | Thông báo | 23 | Công điện |
| 08 | Hướng dẫn | 24 | Bản ghi nhớ |
| 09 | Chương trình | 25 | Bản thỏa thuận |
| 10 | Kế hoạch | 26 | Giấy mời |
| 11 | Phương án | 27 | Giấy giới thiệu |
| 12 | Đề án | 28 | Giấy nghỉ phép |
| 13 | Dự án | 29 | Thư công |
| 14 | Báo cáo | 30 | Bản đồ |
| 15 | Tờ trình | 31 | Bản vẽ kỹ thuật |
| 16 | Giấy ủy quyền | 32 | Khác |

---

## Danh mục thời hạn lưu trữ (maintenance)

| Mã | Thời hạn |
|---|---|
| 01 | Vĩnh viễn |
| 02 | 70 năm |
| 03 | 50 năm |
| 04 | 30 năm |
| 05 | 20 năm |
| 06 | 10 năm |
| 07 | Khác |

---

## Danh mục ngôn ngữ (language)

| Mã | Ngôn ngữ | Mã | Ngôn ngữ |
|---|---|---|---|
| 01 | Tiếng Việt | 07 | Việt – Nga |
| 02 | Tiếng Anh | 08 | Việt – Pháp |
| 03 | Tiếng Pháp | 09 | Hán Nôm |
| 04 | Tiếng Nga | 10 | Việt – Trung |
| 05 | Tiếng Trung | 11 | Khác |
| 06 | Việt – Anh | | |
