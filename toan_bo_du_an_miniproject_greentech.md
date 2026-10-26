# BÁO CÁO TOÀN DIỆN VÀ MÃ NGUỒN HOÀN CHỈNH
## MINI PROJECT: QUẢN LÝ HẠ TẦNG GREENTECH IOT
**Môn học:** Nhập môn CNTT / Hạ tầng CNTT  
**Vai trò đảm nhận:** Kỹ sư Hạ tầng (DevOps Specialist)  
**Cấu trúc Repository GitHub:** `[TênLớp]_[MônHọc]_MiniProject_GreenTech`

---

## PHẦN 1: MÔ HÌNH HỆ THỐNG (IPO & DIKW)

### 1. Mô hình xử lý thông tin IPO (Input - Process - Output)
- **Input (Đầu vào):** Tệp nhật ký máy chủ `log-he-thong.log` (500MB+), bảng danh mục mã linh kiện `danh-muc-linh-kien` và danh sách 100 trạm cảm biến `trang-thai-tram`.
- **Process (Xử lý):** Lọc theo luồng dữ liệu (Stream extraction) với đường ống Pipe (`|`) & `findstr`; Làm sạch chuỗi với `TRIM` và tra cứu chính xác bằng `VLOOKUP` với địa chỉ tuyệt đối (`$`).
- **Output (Đầu ra):** Tệp báo cáo sự cố `backup/bao-cao-su-co.txt`, tài liệu kĩ thuật `README.md`, file `srs-he-thong.docx` có Auto TOC, và bảng đối soát Excel không bị lỗi `#N/A`.

### 2. Chuỗi chuyển hóa DIKW
1. **Data (Dữ liệu):** Các ký tự log thô và mã trạm IoT chưa chuẩn hóa.
2. **Information (Thông tin):** Các dòng lỗi `CRITICAL`, `ERROR 404`, `TIMEOUT` được lọc ra cùng tên thiết bị tương ứng với từng trạm.
3. **Knowledge (Tri thức):** Thống kê được các trạm IoT và linh kiện nào có tần suất gặp sự cố cao nhất ngoài trang trại.
4. **Wisdom (Quyết định):** Đưa ra kế hoạch bảo trì phòng ngừa, nâng cấp thiết bị và tối ưu hóa băng thông truyền tải.

---

## PHẦN 2: THIẾT LẬP HẠ TẦNG & TỆP CẤU HÌNH (MODULE 1)

### 1. Câu lệnh khởi tạo cây thư mục 3 cấp (01 dòng lệnh duy nhất)
Thực thi tại Terminal/Command Prompt để tạo cấu trúc `kebab-case` không dấu, không khoảng trắng:

```bash
mkdir D23CNTT02_NhapMonCNTT_MiniProject_GreenTech\docs D23CNTT02_NhapMonCNTT_MiniProject_GreenTech\scripts D23CNTT02_NhapMonCNTT_MiniProject_GreenTech\data D23CNTT02_NhapMonCNTT_MiniProject_GreenTech\backup && cd D23CNTT02_NhapMonCNTT_MiniProject_GreenTech && echo DB_HOST=localhost > .env && echo DB_PORT=5432 >> .env && echo DB_USER=greentech_admin >> .env && echo DB_PASS=GT2026_SecureKey!@# >> .env && echo LOG_LEVEL=CRITICAL >> .env
```

### 2. Nội dung tệp cấu hình ẩn `.env` (Đường dẫn tương đối: `.env`)

```ini
# GreenTech System Configuration Environment File
# Sensitive Credentials & System Thresholds

DB_HOST=localhost
DB_PORT=5432
DB_USER=greentech_admin
DB_PASS=GT2026_SecureKey!@#
LOG_THRESHOLD_LEVEL=CRITICAL
MAX_LOG_SIZE_MB=500
NETWORK_SPEED_LIMIT_MBPS=100
```

---

## PHẦN 3: ROBOT LỌC LOG TỰ ĐỘNG (MODULE 2)

### Tệp Kịch Bản `scripts/auto_report.bat`

```batch
@echo off
:: =================================================================
:: GreenTech Infrastructure - Automatic Log Filtering Script
:: Target: Process 500MB+ log streams using pipes to avoid RAM overload
:: Output: Append results to backup/bao-cao-su-co.txt
:: =================================================================

set LOG_FILE=..\data\log-he-thong.log
set REPORT_FILE=..\backup\bao-cao-su-co.txt

:: -----------------------------------------------------------------
:: STEP 1: AUDIT VERIFICATION (Tham dinh an toan he thong)
:: Tranh viec thuc thi cac lenh xoa nguy hiem nhu: del /f /s /q
:: -----------------------------------------------------------------
if not exist "%LOG_FILE%" (
    echo [AUDIT ERROR] Tep nhat ky nguon "%LOG_FILE%" khong ton tai!
    echo Quyen truy cap hoac duong dan khong hop le. Khong the xu ly.
    pause
    exit /b 1
)

:: -----------------------------------------------------------------
:: STEP 2: Khoi tao Tieu de Bao cao chuyen nghiep voi toan tu >
:: -----------------------------------------------------------------
echo ======================================================= > %REPORT_FILE%
echo             BAO CAO SU CO HA TANG GREENTECH             >> %REPORT_FILE%
echo             Thoi gian xu ly: %date% %time%               >> %REPORT_FILE%
echo ======================================================= >> %REPORT_FILE%
echo. >> %REPORT_FILE%

echo [STATUS] Dang tien hanh loc Log va trich xuat du lieu...

:: -----------------------------------------------------------------
:: STEP 3: Xu ly luong voi Duong ong Pipe (|) va findstr
:: Loc cac dong chua "CRITICAL" va "ERROR" (bao gom ERROR 404, TIMEOUT)
:: -----------------------------------------------------------------
type "%LOG_FILE%" | findstr /C:"CRITICAL" /C:"ERROR" >> %REPORT_FILE%

echo. >> %REPORT_FILE%
echo ======================================================= >> %REPORT_FILE%
echo [STATUS] DA HOAN THANH QUA TRINH LOC NHAT KY CHUYEN NGHIEP. >> %REPORT_FILE%
echo ======================================================= >> %REPORT_FILE%

echo [SUCCESS] Bao cao da duoc xu ly va luu tai: %REPORT_FILE%
pause
```

### Tệp Nhật Ký Mẫu `data/log-he-thong.log`

```text
2026-10-26 08:00:01 [INFO] Station_001 connected to server successfully.
2026-10-26 08:00:05 [INFO] Soil moisture sensor sending data: 45%.
2026-10-26 08:01:12 [WARNING] High temperature detected on Gateway_002: 42C.
2026-10-26 08:02:30 [ERROR] Connection lost to Sensor Station_014 on port 8080.
2026-10-26 08:03:00 [INFO] System heartbeat check OK.
2026-10-26 08:04:15 [CRITICAL] Memory exhaustion imminent on Edge Node_005. RAM usage 98%.
2026-10-26 08:05:22 [ERROR] ERROR 404: Sensor Hub_088 failed to acknowledge data packet.
2026-10-26 08:06:10 [ERROR] TIMEOUT: Connection reset by peer at Station TRAM-012.
2026-10-26 08:07:01 [INFO] Backup process completed successfully.
```

---

## PHẦN 4: TÀI LIỆU KỸ THUẬT (MODULE 3)

### 1. Tệp Hướng Dẫn Vận Hành (`docs/README.md`)

```markdown
# GreenTech Infrastructure Management System

Hệ thống quản lý hạ tầng máy chủ và danh mục phần cứng 100 trạm cảm biến IoT ngoài trang trại cho Startup công nghệ **GreenTech**. Dự án được xây dựng nhằm tự động hóa quy trình xử lý sự cố nhật ký máy chủ và quản lý tài nguyên linh kiện IoT theo tư duy Database Mindset.

---

## 1. Cấu trúc Cây thư mục (Project Directory)

```text
greentech-system/
├── docs/
│   ├── README.md              # Tài liệu hướng dẫn hệ thống
│   └── srs-he-thong.docx      # Đặc tả yêu cầu phần mềm (SRS Document)
├── scripts/
│   └── auto_report.bat        # Script tự động lọc Log sự cố
├── data/
│   ├── log-he-thong.log       # Tệp nhật ký hệ thống (mô phỏng 500MB+)
│   └── quan-ly-thiet-bi.xlsx  # Bảng đối soát tài nguyên linh kiện IoT
├── backup/
│   └── bao-cao-su-co.txt      # Báo cáo kết xuất sự cố định kỳ
└── .env                       # File cấu hình ẩn chứa thông số bảo mật
```

---

## 2. Hướng dẫn Vận hành Script

Thực hiện theo các bước sau tại cửa sổ dòng lệnh (Command Prompt / PowerShell):

```bash
# Bước 1: Di chuyển vào thư mục chứa script
cd scripts

# Bước 2: Thực thi file tự động hóa lọc log
auto_report.bat
```

> **Lưu ý Thẩm định (Audit Check):** Kịch bản `auto_report.bat` đã tích hợp lệnh kiểm tra sự tồn tại của tệp nguồn. Tuyệt đối **không** áp dụng các lệnh xóa nguy hiểm như `del /f /s /q` do AI đề xuất mà chưa qua thẩm định.

---

## 3. Checklist Kiểm tra Trạm Cảm biến IoT

- [x] Khởi tạo cấu trúc cây thư mục chuẩn bằng `mkdir` 01 dòng lệnh.
- [x] Tạo tệp cấu hình ẩn `.env` bằng đường dẫn tương đối (Relative Path).
- [x] Xây dựng Robot lọc file Log bằng đường ống Pipe (`|`) và `findstr`.
- [x] Chuẩn hóa dữ liệu mã linh kiện bằng hàm `TRIM` trên Excel.
- [x] Thực hiện đối soát `VLOOKUP` không còn bị lỗi `#N/A`.
- [x] Kiểm tra kết nối mạng và tính toán tốc độ truyền tải log (8 Mbps = 1 MB/s).

---

## 4. Bảng Thông số Phần cứng & Thiết bị Mẫu

| Mã Linh Kiện | Tên Thiết Bị / Cảm Biến | Loaị Storage | Dung Lượng RAM | Đơn Giá (VND) |
| :--- | :--- | :---: | :---: | ---: |
| **LK-001** | Cảm biến Độ ẩm Đất v2 | N/A | N/A | 250,000 |
| **LK-002** | Gateway Hub IoT | SSD | 4GB | 2,500,000 |
| **LK-003** | Trạm Thu Phát LoRaWAN | N/A | 512MB | 1,200,000 |
| **LK-004** | Edge Computing Node | SSD | 16GB | 15,000,000 |

---

## 5. Phân tích Bài toán Băng thông & Thời gian Tải Log

Giả định tệp nhật ký hệ thống phát sinh dung lượng **500 MB**, đường truyền mạng đạt tốc độ **100 Mbps**:

1. **Quy đổi tốc độ mạng sang Megabytes/giây (MB/s):**
   $$\text{Tốc độ (MB/s)} = \frac{100 \text{ Mbps}}{8} = 12.5 \text{ MB/s}$$

2. **Thời gian tải tệp Log lý thuyết:**
   $$\text{Thời gian} = \frac{500 \text{ MB}}{12.5 \text{ MB/s}} = 40 \text{ giây}$$
```

### 2. Cấu trúc Tài liệu Đặc tả SRS (`docs/srs-he-thong.docx`)
*Thiết lập trong MS Word:*
- Trang 1: **MỤC LỤC TỰ ĐỘNG (Auto TOC)** (Insert -> Table of Contents).
- Dùng **Heading 1**: `1. GIỚI THIỆU HỆ THỐNG`, `2. YÊU CẦU CHỨC NĂNG`, `3. YÊU CẦU PHI CHỨC NĂNG`.
- Dùng **Heading 2**: `1.1 Mục tiêu dự án`, `1.2 Phạm vi hệ thống`, `2.1 Module Căn cứ số`, `2.2 Module Lọc log`, `2.3 Module Excel đối soát`, `3.1 Hiệu năng`, `3.2 Bảo mật`.

---

## PHẦN 5: BẢNG TÍNH EXCEL QUẢN LÝ THIẾT BỊ (MODULE 4)

### Bảng tính Asset Database (`data/quan-ly-thiet-bi.xlsx`)

#### Sheet 1: `danh-muc-linh-kien` (Khóa chính)
| Cell | A (ma_linh_kien) | B (ten_thiet_bi) | C (don_gia) | D (loai_storage) |
| :--- | :--- | :--- | :--- | :--- |
| **1** | **ma_linh_kien** | **ten_thiet_bi** | **don_gia** | **loai_storage** |
| **2** | LK001 | Cảm biến độ ẩm đất | 250000 | N/A |
| **3** | LK002 | IoT Gateway Core Node | 1800000 | SSD |
| **4** | LK003 | Module LoRaWAN | 450000 | N/A |
| **5** | LK004 | Edge Computing Server | 12000000 | SSD |

#### Sheet 2: `trang-thai-tram` (Khóa ngoại & Tra cứu)
| Cell | A (ma_tram) | B (ma_linh_kien) | C (ten_thiet_bi) | D (dung_luong_ram) | E (ngay_lap_dat) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **ma_tram** | **ma_linh_kien** | **ten_thiet_bi** | **dung_luong_ram** | **ngay_lap_dat** |
| **2** | TRAM-001 | LK001 | `=VLOOKUP(TRIM(B2), 'danh-muc-linh-kien'!$A$2:$D$10, 2, FALSE)` | 512MB | 2026-01-15 |
| **3** | TRAM-002 | LK002 *(có khoảng trắng)* | `=VLOOKUP(TRIM(B3), 'danh-muc-linh-kien'!$A$2:$D$10, 2, FALSE)` | 4GB | 2026-01-18 |
| **4** | TRAM-003 | LK004 | `=VLOOKUP(TRIM(B4), 'danh-muc-linh-kien'!$A$2:$D$10, 2, FALSE)` | 16GB | 2026-02-01 |

#### Giải thích kỹ thuật giải quyết bẫy Excel:
1. **Hàm `TRIM(B2)`**: Tự động loại bỏ khoảng trắng thừa đầu/cuối của mã linh kiện nhập sai (ví dụ `"LK002 "`), triệt tiêu 100% lỗi `#N/A`.
2. **Ký tự cố định `$A$2:$D$10`**: Khóa vùng dữ liệu tìm kiếm tuyệt đối, đảm bảo khi sao chép công thức xuống 100 dòng không bị lệch ô.
3. **Tham số `FALSE`**: Đảm bảo chế độ tìm kiếm chính xác tuyệt đối (Exact Match).
4. **Không Merge Cell / Dòng trống**: Đảm bảo tính năng `AutoFilter` và `Sort` hoạt động chính xác.

---

## PHẦN 6: QUY TRÌNH NỘP BÀI LÊN GITHUB & PORTAL

1. Khởi tạo Git repository và commit dữ liệu local:
   ```bash
   git init
   git add .
   git commit -m "Complete GreenTech Infrastructure MiniProject"
   git branch -M main
   ```
2. Liên kết remote GitHub repository:
   ```bash
   git remote add origin https://github.com/[TàiKhoảnGitHub]/[TênLớp]_[MônHọc]_MiniProject_GreenTech.git
   git push -u origin main
   ```
3. Copy đường link Repository public và nộp lên Portal theo yêu cầu.