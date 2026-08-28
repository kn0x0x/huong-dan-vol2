# Cheatsheet Volatility 2.6 – Memory Forensics Windows Server

Tài liệu dùng nhanh cho bài memory dump `windows-server-memory-final3.raw`.

> Khuyến nghị dùng **Volatility 2.6 standalone** trên Windows. Không cần cài Python.

---

## 1. Cấu trúc thư mục khuyến nghị

Đặt file như sau:

```text
C:\Users\Administrator\Downloads\windows-server-memory-final3\
│
├── windows-server-memory-final3.raw
└── tools\
    └── volatility_2.6_win64_standalone\
        └── volatility_2.6_win64_standalone.exe
```

---

## 2. Tải Volatility 2.6 standalone

Mở **CMD**:

```cmd
cd /d C:\Users\Administrator\Downloads\windows-server-memory-final3
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

Biến lệnh cho dễ dùng:

```cmd
set VOL=tools\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe
set MEM=windows-server-memory-final3.raw
set PROFILE=Win2016x64_14393
```

Sau đó chạy dạng:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% <plugin>
```

---

## 4. Xác định hệ điều hành

```cmd
%VOL% -f %MEM% --profile=%PROFILE% imageinfo
```

Nếu `imageinfo` chạy lâu, có thể bỏ qua vì profile đã xác định là:

```text
Win2016x64_14393
```

Kiểm tra nhanh bằng Volatility 3 nếu cần:

```cmd
python volatility3\vol.py -f windows-server-memory-final3.raw windows.info
```

Dấu hiệu quan trọng:

```text
NtProductType   NtProductServer
NtMajorVersion  10
NtMinorVersion  0
```

Kết luận nền tảng:

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

### Scan process kể cả process đã thoát

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

---

## 6. Xem command line

Đây là plugin quan trọng nhất trong bài.

```cmd
%VOL% -f %MEM% --profile=%PROFILE% cmdline
```

Lưu ra file:

```cmd
%VOL% -f %MEM% --profile=%PROFILE% cmdline > vol2_cmdline.txt
```

Lọc nhanh dòng đáng chú ý:

```cmd
findstr /i "powershell updater wcache telemetry portal ARTICLE OPERATOR winpmem" vol2_cmdline.txt
```

Các dòng quan trọng đã thấy trong bài:

```text
powershell.exe -NoExit -ExecutionPolicy Bypass -File C:\Lab\portal_memory.ps1
```

```text
PORTAL_SESSION=7F91A6C2
ARTICLE_ID=8841
OPERATOR_HANDLE=KazmiKami1
PORTAL_URL=http://89.106.78.147:8080/dashboad/admin-dientapgialai
```

```text
winpmem_mini_x64_rc2.exe C:\windows-server-memory-final3.raw
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

Hive quan trọng thường gặp:

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

Với dump này, `filescan` có thể không ra hoặc chỉ ra header. Vẫn có thể thử:

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
%VOL% -f %MEM% --profile=%PROFILE% dumpfiles -Q <OFFSET> -D output
```

Ví dụ:

```cmd
mkdir output
%VOL% -f %MEM% --profile=%PROFILE% dumpfiles -Q 0xffffce8e12345678 -D output
```

---

## 11. Tìm chuỗi trong memory

Volatility 2 có thể phối hợp với `strings` bên ngoài.

Nếu có Sysinternals `strings.exe`:

```cmd
strings.exe -n 6 windows-server-memory-final3.raw > strings.txt
```

Lọc nhanh:

```cmd
findstr /i "ATTT flag key AES decrypt locked docx portal ARTICLE OPERATOR" strings.txt
```

Nếu không có `strings.exe`, có thể dùng PowerShell đơn giản nhưng sẽ chậm với file 5GB. Khuyến nghị dùng Sysinternals Strings.

---

## 12. Workflow gợi ý cho bài này

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
```

Lọc:

```cmd
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

Tạo file `run_vol2.bat` trong thư mục chứa `.raw`:

```bat
@echo off
cd /d C:\Users\Administrator\Downloads\windows-server-memory-final3

set VOL=tools\volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe
set MEM=windows-server-memory-final3.raw
set PROFILE=Win2016x64_14393

echo [1] pslist
%VOL% -f %MEM% --profile=%PROFILE% pslist > vol2_pslist.txt

echo [2] psscan
%VOL% -f %MEM% --profile=%PROFILE% psscan > vol2_psscan.txt

echo [3] cmdline
%VOL% -f %MEM% --profile=%PROFILE% cmdline > vol2_cmdline.txt

echo [4] hivelist
%VOL% -f %MEM% --profile=%PROFILE% hivelist > vol2_hivelist.txt

echo [5] netscan
%VOL% -f %MEM% --profile=%PROFILE% netscan > vol2_netscan.txt

echo [6] interesting command lines
findstr /i "powershell updater wcache telemetry portal ARTICLE OPERATOR SESSION URL winpmem" vol2_cmdline.txt > vol2_interesting_cmdline.txt

echo Done.
echo Output files:
echo - vol2_pslist.txt
echo - vol2_psscan.txt
echo - vol2_cmdline.txt
echo - vol2_hivelist.txt
echo - vol2_netscan.txt
echo - vol2_interesting_cmdline.txt
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

Kiểm tra đang đứng đúng thư mục chưa:

```cmd
cd /d C:\Users\Administrator\Downloads\windows-server-memory-final3
```

Kiểm tra file:

```cmd
dir windows-server-memory-final3.raw
```

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

```cmd
cd /d C:\Users\Administrator\Downloads\windows-server-memory-final3
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

- Không dùng Volatility 3 cho các plugin `pslist`, `cmdline`, `filescan`, `hivelist` trong bài này nếu thấy rỗng.
- Dùng Volatility 2.6 standalone với profile `Win2016x64_14393`.
- Tập trung vào `cmdline`: đây là nơi có nhiều thông tin điều tra nhất.
- Khi thấy process đáng ngờ, luôn đối chiếu cả `pslist`, `psscan`, `cmdline`.
