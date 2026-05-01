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
int size;
scanf("%d", &size);
if (size > 32) return; // Kẻ tấn công nhập -1 để lách điều kiện này
read(0, buffer, size); // read nhận size_t. -1 bị ép kiểu thành 0xffffffffffffffff -> Buffer Overflow!
```

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
3. **Nhảy cóc (Jumping Canary):** Nếu lỗi là vòng lặp OOB bằng index, ta có thể sửa biến index để nhảy qua vùng nhớ Canary, đè thẳng vào `Return Address` mà không chạm vào sợi lông nào của "chim báo bão".

### 4.3. Kỹ nghệ Exploit (Exploit Engineering)
* **Cyclic Values (Mẫu chuỗi De Bruijn):** Thay vì gửi chuỗi `AAAA`, hãy gửi chuỗi tuần hoàn (`aaaaabaaacaaa...`). Khi GDB báo lỗi tại giá trị đặc biệt, ta tra ngược chuỗi để biết chính xác offset đè `RIP` đến từng byte.
* **Dynamic Analysis (GDB):** Biến môi trường lúc runtime sẽ làm xê dịch Stack. Việc tìm offset và địa chỉ phải luôn được đo bằng công cụ debug (GDB + GEF/pwndbg) trong lúc chương trình đang chạy.

---

## Phần 5: Cuộc Chiến Bảng Phân Giải - Chiếm Đoạt GOT

Nếu Stack bị bảo vệ quá nghiêm ngặt (Canary không thể leak), mục tiêu tối thượng chuyển sang **Bảng GOT (Global Offset Table)**.

### 5.1. Liên Kết Động (Lazy Binding) & Bảng GOT
Hầu hết chương trình mượn hàm `puts`, `read` từ thư viện `libc.so`. Khi có ASLR, chương trình không biết địa chỉ hàm nằm ở đâu.
* Khi gọi `puts` lần đầu, luồng lệnh nhảy vào bảng **PLT**, sau đó mượn Dynamic Linker tính toán địa chỉ thật của `puts` trong `libc`, rồi **ghi đè địa chỉ đó vào bảng GOT**.
* Lần gọi sau, chương trình đọc trực tiếp địa chỉ xịn từ GOT.

### 5.2. Vấn Đề Chết Người: GOT Overwrite
Vì Dynamic Linker phải ghi vào GOT trong lúc chạy, **vùng nhớ chứa GOT bắt buộc phải có quyền Đọc & Ghi (Read/Write)**.
Nếu ta có lỗ hổng **Arbitrary Write** (Ghi một giá trị tại địa chỉ tùy ý, VD: Format String `%n`, OOB Write, UAF), ta không cần đánh Stack. Ta chỉ cần ghi địa chỉ hàm `system("/bin/sh")` đè lên địa chỉ `puts@got.plt`.
Khi chương trình ngây thơ gọi `puts("Hello")`, thanh ghi `RIP` sẽ nhảy thẳng vào `system("Hello")`. Game Over.

### 5.3. Phòng thủ bằng RELRO (Relocation Read-Only)
* **Partial RELRO:** Bảng `.got.plt` vẫn cho phép ghi. KHÔNG cản được GOT Overwrite.
* **Full RELRO:** Khi chương trình khởi động, Loader sẽ phân giải sẵn TẤT CẢ các hàm, điền vào GOT, rồi dùng syscall `mprotect` khóa cứng vùng nhớ GOT thành **Chỉ Đọc (Read-Only)**. GOT Overwrite chết đứng. Hacker lúc này phải nhắm vào các mục tiêu khó hơn như `__free_hook`, `__malloc_hook` của libc.

---

## Phần 6: Tư Duy Pwn - Chiến Thuật Phối Hợp Tổng Thể

Trong khai thác thực tế, OOB hay Buffer Overflow chỉ là "công cụ". Kẻ tấn công giỏi là người biết xâu chuỗi chúng thành một quy trình (Exploit Chain):

1. **Rò rỉ thông tin (Info Leak):** Dùng OOB Read / Format String để phá vỡ lớp mù lòa (ASLR, PIE, Canary).
2. **Sắp đặt trận địa bộ nhớ (Memory Orchestration / Heap Feng Shui):** Nếu lỗ hổng nằm trên Heap (vùng nhớ động), ta phải dùng các lệnh `malloc`/`free` để điều hướng bộ nhớ, ép "Mục tiêu" phải nằm sát cạnh "Buffer vũ khí".
3. **Leo thang năng lực (Primitive Escalation):** Biến một lỗi OOB yếu (như Off-by-one, ghi 1 byte Null) ghi đè vào biến `length` của một cấu trúc khác, từ đó tạo ra vũ khí OOB thứ hai mạnh mẽ hơn (Ghi tràn bộ nhớ tùy ý).
4. **Chiếm đoạt Control Flow (Control Flow Hijack):** Tung đòn quyết định. Dùng Arbitrary Write / OOB Write để đè `Return Address`, `GOT`, hoặc `vtable`, ép luồng thực thi đi theo kịch bản đã định (Ret2libc, ROP).
