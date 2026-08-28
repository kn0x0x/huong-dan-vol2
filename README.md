# Cheatsheet Volatility 2.6 

> Tài liệu này chỉ hướng dẫn sử dụng công cụ **Volatility 2.6** ở mức cơ bản và so sánh cú pháp tương ứng với **Volatility 3**.  

---

## 1. Chuẩn bị Volatility 2.6 trên Windows

### Cách nhanh nhất: dùng bản standalone

Tải Volatility 2.6 standalone cho Windows:

```text
https://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_win64_standalone.zip
```

Giải nén ra một thư mục bất kỳ, ví dụ:

```text
volatility_2.6_win64_standalone\
```

Bên trong sẽ có file:

```text
volatility_2.6_win64_standalone.exe
```

Bản standalone không cần cài Python 2.

---

## 2. Cấu trúc thư mục khuyến nghị

Để dễ chạy lệnh, có thể đặt như sau:

```text
memory-lab\
├── memory.raw
└── volatility_2.6_win64_standalone\
    └── volatility_2.6_win64_standalone.exe
```

Trong đó:

```text
memory.raw
```

là file memory dump được cung cấp trong bài.

Tên file `.raw` có thể khác, tự thay lại đúng tên file đang có.

---

## 3. Cú pháp chung

### Volatility 2

```cmd
volatility_2.6_win64_standalone.exe -f <file_memory> --profile=<profile> <plugin>
```

Ví dụ:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=Win2016x64_14393 pslist
```

### Volatility 3

```cmd
python vol.py -f <file_memory> <plugin>
```

Ví dụ:

```cmd
python vol.py -f memory.raw windows.pslist
```

---

## 4. Xác định profile trong Volatility 2

Volatility 2 cần `profile`.

Chạy:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw imageinfo
```

Hoặc:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw kdbgscan
```

Sau đó chọn profile phù hợp từ kết quả gợi ý.

Một số profile Windows thường gặp:

```text
Win7SP1x64
Win10x64
Win10x64_14393
Win2012R2x64
Win2016x64_14393
```

---

## 5. Bảng lệnh tương ứng Volatility 2 và Volatility 3

| Mục đích | Volatility 2 | Volatility 3 |
|---|---|---|
| Gợi ý profile / thông tin image | `imageinfo` | `windows.info` |
| Quét KDBG | `kdbgscan` | thường không cần trực tiếp |
| Liệt kê process | `pslist` | `windows.pslist` |
| Quét process trong memory | `psscan` | `windows.psscan` |
| Cây tiến trình | `pstree` | `windows.pstree` |
| Command line process | `cmdline` | `windows.cmdline` |
| DLL đã load | `dlllist` | `windows.dlllist` |
| Handles | `handles` | `windows.handles` |
| Biến môi trường | `envars` | `windows.envars` |
| Kết nối mạng | `netscan` | tùy bản Vol3 có thể là plugin network khác hoặc không có |
| Scan file object | `filescan` | `windows.filescan.FileScan` hoặc `windows.filescan` tùy bản |
| Dump file từ memory | `dumpfiles` | `windows.dumpfiles.DumpFiles` hoặc `windows.dumpfiles` tùy bản |
| Registry hive list | `hivelist` | `windows.registry.hivelist` |
| In registry key | `printkey` | `windows.registry.printkey` |
| UserAssist | `userassist` | `windows.registry.userassist` |
| Shimcache | `shimcache` | `windows.shimcachemem` |
| Malfind | `malfind` | `windows.malfind` hoặc `windows.malware.malfind` |
| Memory map process | `memmap` | `windows.memmap` |
| Dump process memory | `memdump` | `windows.memmap --dump` hoặc plugin tương ứng tùy bản |
| YARA scan | `yarascan` | `yarascan` / `vadyarascan` tùy bản |
| Strings mapping | `strings` | `windows.strings` |

---

## 6. Các nhóm lệnh cơ bản trong Volatility 2

### 6.1. Thông tin image

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw imageinfo
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw kdbgscan
```

---

### 6.2. Process

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> pslist
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> psscan
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> pstree
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> cmdline
```

Lọc theo PID:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> cmdline -p <PID>
```

---

### 6.3. DLL, handle, environment

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> dlllist
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> handles
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> envars
```

Lọc theo PID:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> dlllist -p <PID>
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> handles -p <PID>
```

---

### 6.4. Network

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> netscan
```

---

### 6.5. File trong memory

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> filescan
```

Dump file theo offset:

```cmd
mkdir dump
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> dumpfiles -Q <OFFSET> -D dump
```

---

### 6.6. Registry

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> hivelist
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> printkey
```

Chỉ định hive offset:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> printkey -o <HIVE_OFFSET>
```

Chỉ định key:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> printkey -K "<REGISTRY_KEY>"
```

---

### 6.7. Malfind và vùng nhớ tiến trình

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> malfind
```

Lọc theo PID:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> malfind -p <PID>
```

Memory map:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> memmap -p <PID>
```

Dump memory process:

```cmd
mkdir dump
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> memdump -p <PID> -D dump
```

---

## 7. Ghi output ra file

Nên ghi kết quả ra file để dễ đọc và tìm kiếm:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> pslist > pslist.txt
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> cmdline > cmdline.txt
```

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> filescan > filescan.txt
```

Tìm kiếm trong output trên Windows:

```cmd
findstr /i "keyword" cmdline.txt
```

Ví dụ cú pháp chung:

```cmd
findstr /i "powershell" cmdline.txt
```

---

## 8. Mẫu chạy nhanh

Sau khi biết profile, có thể chạy nhóm lệnh cơ bản:

```cmd
set VOL=volatility_2.6_win64_standalone\volatility_2.6_win64_standalone.exe
set MEM=memory.raw
set PROFILE=Win2016x64_14393

%VOL% -f %MEM% --profile=%PROFILE% pslist > pslist.txt
%VOL% -f %MEM% --profile=%PROFILE% psscan > psscan.txt
%VOL% -f %MEM% --profile=%PROFILE% pstree > pstree.txt
%VOL% -f %MEM% --profile=%PROFILE% cmdline > cmdline.txt
%VOL% -f %MEM% --profile=%PROFILE% hivelist > hivelist.txt
%VOL% -f %MEM% --profile=%PROFILE% filescan > filescan.txt
%VOL% -f %MEM% --profile=%PROFILE% netscan > netscan.txt
```

Thay:

```text
memory.raw
```

bằng đúng tên file memory dump.

Thay:

```text
Win2016x64_14393
```

bằng profile phù hợp nếu `imageinfo` hoặc `kdbgscan` gợi ý profile khác.

---

## 9. Một số lỗi thường gặp

### Lỗi sai đường dẫn file memory

Thông báo thường gặp:

```text
No such file or directory
```

Kiểm tra lại tên file `.raw` và thư mục đang đứng.

---

### Lỗi sai profile

Nếu chạy không ra dữ liệu hoặc báo lỗi profile, thử lại:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw imageinfo
```

hoặc:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw kdbgscan
```

Sau đó chọn profile khác phù hợp hơn.

---

### Lệnh chạy lâu

Một số plugin có thể chạy lâu với memory dump lớn, ví dụ:

```text
filescan
malfind
yarascan
```

Nên redirect output ra file:

```cmd
volatility_2.6_win64_standalone.exe -f memory.raw --profile=<profile> filescan > filescan.txt
```

---

## 10. Ghi nhớ nhanh

Volatility 2 cần:

```text
-f memory.raw
--profile=<profile>
<plugin>
```

Volatility 3 thường không cần profile:

```text
-f memory.raw
windows.<plugin>
```

Khi dùng Volatility 2, bước quan trọng nhất là xác định đúng `profile`.
