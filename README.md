# Cheatsheet Volatility 2.6 – Phân tích Memory Dump Windows Server

Tài liệu dùng nhanh cho thí sinh khi phân tích file memory dump:

```text
windows-server-memory-final3.raw
```

Khuyến nghị dùng **Volatility 2.6 standalone trên Windows**. Bản standalone không cần cài Python.

---

## 1. Chuẩn bị thư mục làm bài

Thí sinh tự tạo một thư mục bất kỳ, ví dụ:

```text
D:\CTF\memory-final\
```

Hoặc:

```text
C:\CTF\memory-final\
```

Trong thư mục đó đặt file memory dump:

```text
memory-final\
│
├── windows-server-memory-final3.raw
└── tools\
    └── volatility_2.6_win64_standalone\
        └── volatility_2.6_win64_standalone.exe
```

> Không bắt buộc phải giống đường dẫn ví dụ. Chỉ cần các lệnh được chạy trong **thư mục chứa file `.raw`**.

---

## 2. Tải Volatility 2.6 standalone

Mở **CMD** tại thư mục chứa file `.raw`.

Ví dụ nếu file nằm ở `D:\CTF\memory-final\`:

```cmd
cd /d D:\CTF\memory-final
```

Tải Volatility 2.6:

```cmd
mkdir tools
cd tools
curl.exe -L -o volatility_2.6_win64_standalone.zip https://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_win64_standalone.zip
tar -xf volatility_2.6_win64_standalone.zip
cd ..
```

Kiểm tra:

```cmd
tools\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe --info | findstr /i "Win2016 Win10"
```

---

## 3. Profile dùng cho bài này

Profile đã kiểm tra chạy được:

```text
Win2016x64_14393
```

Thiết lập biến lệnh cho dễ dùng:

```cmd
set VOL=tools\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe
set MEM=windows-server-memory-final3.raw
set PROFILE=Win2016x64_14393
```

Sau đó chạy theo mẫu:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% <plugin>
```

Ví dụ:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% pslist
```

---

## 4. Xác định hệ điều hành

```cmd
%VOL% -f %MEM% --profile=%PROFILE% imageinfo
```

Nếu `imageinfo` chạy lâu, có thể bỏ qua vì profile dùng cho bài này là:

```text
Win2016x64_14393
```

Kết luận nền tảng của image:

```text
Windows Server
```

---

## 5. Liệt kê process

### Process đang active

```cmd
%VOL% -f %MEM% --profile=%PROFILE% pslist
```

Lưu ra file:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% pslist > vol2_pslist.txt
```

### Scan process, kể cả process đã thoát

```cmd
%VOL% -f %MEM% --profile=%PROFILE% psscan
```

Lưu ra file:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% psscan > vol2_psscan.txt
```

Process đáng chú ý trong bài:

```text
powershell.exe
updater.exe
wcache.exe
telemetry.exe
Robocopy.exe
winpmem_mini_x
```

Lọc nhanh:

```cmd
findstr /i "powershell updater wcache telemetry robocopy winpmem" vol2_pslist.txt vol2_psscan.txt
```

---

## 6. Xem command line

Đây là plugin rất quan trọng trong bài.

```cmd
%VOL% -f %MEM% --profile=%PROFILE% cmdline
```

Lưu ra file:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% cmdline > vol2_cmdline.txt
```

Lọc nhanh dòng đáng chú ý:

```cmd
findstr /i "powershell updater wcache telemetry portal ARTICLE OPERATOR SESSION URL winpmem" vol2_cmdline.txt
```

Các chuỗi có thể đáng chú ý:

```text
powershell.exe
portal_memory.ps1
PORTAL_SESSION
ARTICLE_ID
OPERATOR_HANDLE
PORTAL_URL
winpmem_mini_x64_rc2.exe
```

---

## 7. Registry hive

```cmd
%VOL% -f %MEM% --profile=%PROFILE% hivelist
```

Lưu ra file:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% hivelist > vol2_hivelist.txt
```

Hive thường cần chú ý:

```text
\REGISTRY\MACHINE\SYSTEM
\REGISTRY\MACHINE\SOFTWARE
\SystemRoot\System32\Config\SAM
\SystemRoot\System32\Config\SECURITY
\??\C:\Users\ADM\ntuser.dat
```

Đọc key registry nếu cần:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% printkey -K "Microsoft\Windows\CurrentVersion\Run"
```

---

## 8. Network / kết nối mạng

Thử lần lượt:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% netscan
```

```cmd
%VOL% -f %MEM% --profile=%PROFILE% connections
```

```cmd
%VOL% -f %MEM% --profile=%PROFILE% connscan
```

Lưu kết quả:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% netscan > vol2_netscan.txt
```

---

## 9. DLL / module / injection

Liệt kê DLL của process theo PID:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% dlllist -p <PID>
```

Ví dụ:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% dlllist -p 4492
```

Kiểm tra injection/malware memory:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% malfind
```

Theo PID:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% malfind -p <PID>
```

---

## 10. File scan / dump file

Với memory image này, `filescan` có thể không ra hoặc chỉ ra header. Vẫn có thể thử:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% filescan
```

Lưu ra file:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% filescan > vol2_filescan.txt
```

Lọc tài liệu:

```cmd
findstr /i ".docx .xlsx .pdf .txt .locked .enc" vol2_filescan.txt
```

Dump file theo offset nếu `filescan` có kết quả:

```cmd
mkdir output
%VOL% -f %MEM% --profile=%PROFILE% dumpfiles -Q <OFFSET> -D output
```

Ví dụ:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% dumpfiles -Q 0xffffce8e12345678 -D output
```

---

## 11. Tìm chuỗi trong memory

Có thể dùng Sysinternals `strings.exe` để hỗ trợ tìm chuỗi trong file `.raw`.

Nếu có `strings.exe`:

```cmd
strings.exe -n 6 windows-server-memory-final3.raw > strings.txt
```

Lọc nhanh:

```cmd
findstr /i "ATTT flag key AES decrypt locked docx portal ARTICLE OPERATOR" strings.txt
```

Nếu chưa có `strings.exe`, có thể bỏ qua bước này và tập trung vào `cmdline`, `pslist`, `psscan`, `hivelist`.

---

## 12. Workflow gợi ý

### Bước 1 – Xác định OS

```cmd
%VOL% -f %MEM% --profile=%PROFILE% imageinfo
```

Kết luận:

```text
Windows Server
```

### Bước 2 – Tìm process

```cmd
%VOL% -f %MEM% --profile=%PROFILE% pslist > vol2_pslist.txt
%VOL% -f %MEM% --profile=%PROFILE% psscan > vol2_psscan.txt
findstr /i "powershell updater wcache telemetry robocopy winpmem" vol2_pslist.txt vol2_psscan.txt
```

### Bước 3 – Tìm command line

```cmd
%VOL% -f %MEM% --profile=%PROFILE% cmdline > vol2_cmdline.txt
findstr /i "portal ARTICLE OPERATOR SESSION URL powershell updater wcache telemetry" vol2_cmdline.txt
```

### Bước 4 – Kiểm tra registry/network nếu cần

```cmd
%VOL% -f %MEM% --profile=%PROFILE% hivelist > vol2_hivelist.txt
%VOL% -f %MEM% --profile=%PROFILE% netscan > vol2_netscan.txt
```

---

## 13. File chạy nhanh `run_vol2.bat`

Tạo file `run_vol2.bat` trong **cùng thư mục với file `.raw`**.

Nội dung:

```bat
@echo off
cd /d "%~dp0"

set VOL=tools\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe
set MEM=windows-server-memory-final3.raw
set PROFILE=Win2016x64_14393

if not exist "%MEM%" (
    echo [ERROR] Khong tim thay file %MEM%
    echo Hay dat run_vol2.bat trong cung thu muc voi file .raw
    pause
    exit /b
)

if not exist "%VOL%" (
    echo [ERROR] Khong tim thay Volatility 2.6 standalone.
    echo Hay tai va giai nen vao thu muc tools\
    pause
    exit /b
)

echo [1] pslist
"%VOL%" -f "%MEM%" --profile=%PROFILE% pslist > vol2_pslist.txt

echo [2] psscan
"%VOL%" -f "%MEM%" --profile=%PROFILE% psscan > vol2_psscan.txt

echo [3] cmdline
"%VOL%" -f "%MEM%" --profile=%PROFILE% cmdline > vol2_cmdline.txt

echo [4] hivelist
"%VOL%" -f "%MEM%" --profile=%PROFILE% hivelist > vol2_hivelist.txt

echo [5] netscan
"%VOL%" -f "%MEM%" --profile=%PROFILE% netscan > vol2_netscan.txt

echo [6] interesting command lines
findstr /i "powershell updater wcache telemetry robocopy portal ARTICLE OPERATOR SESSION URL winpmem" vol2_cmdline.txt > vol2_interesting_cmdline.txt

echo.
echo Done.
echo Nen mo file vol2_interesting_cmdline.txt truoc.
echo.
pause
```

Chạy:

```cmd
run_vol2.bat
```

---

## 14. Lỗi thường gặp

### Lỗi: Invalid profile

Dùng sai profile. Với bài này dùng:

```text
Win2016x64_14393
```

### Lỗi: File not found

Kiểm tra đang đứng đúng thư mục chứa file `.raw` chưa:

```cmd
dir windows-server-memory-final3.raw
```

Nếu không thấy file, hãy `cd` vào đúng thư mục hoặc chuyển file `.raw` vào thư mục đang làm bài.

### Lệnh chạy quá lâu

Một số plugin scan toàn bộ RAM 5GB nên có thể lâu. Ưu tiên chạy trước:

```cmd
pslist
cmdline
hivelist
psscan
```

### `filescan` không ra

Với memory image này, `filescan` có thể không hữu ích. Tập trung vào:

```text
pslist
psscan
cmdline
hivelist
strings
```

---

## 15. Bộ lệnh tối thiểu cần nhớ

Chạy trong thư mục chứa file `.raw`:

```cmd
set VOL=tools\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe
set MEM=windows-server-memory-final3.raw
set PROFILE=Win2016x64_14393

%VOL% -f %MEM% --profile=%PROFILE% pslist
%VOL% -f %MEM% --profile=%PROFILE% psscan
%VOL% -f %MEM% --profile=%PROFILE% cmdline
%VOL% -f %MEM% --profile=%PROFILE% hivelist
```

---

## 16. Ghi chú cho thí sinh

- Dùng **Volatility 2.6 standalone** với profile `Win2016x64_14393`.
- Chạy lệnh trong thư mục chứa file `windows-server-memory-final3.raw`.
- Không cần cài Python nếu dùng bản standalone.
- Tập trung vào `cmdline`: đây là nơi có nhiều thông tin điều tra nhất.
- Khi thấy process đáng ngờ, luôn đối chiếu cả `pslist`, `psscan`, `cmdline`.
