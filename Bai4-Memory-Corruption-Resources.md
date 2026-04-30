# Memory Corruption Resources

---

## Phần 1: Lỗi Bộ Nhớ - Nền Tảng Của Mọi Cuộc Tấn Công

> Cái đoạn lịch sử này lướt qua cũng được

<details>
    <summary>Lịch sử</summary>

### 1. Từ Mã Máy đến "Sức Mạnh Đi Kèm Trách Nhiệm Lớn"

Lịch sử của ngành khoa học máy tính là một chuỗi nỗ lực không ngừng để giao tiếp với máy móc một cách hiệu quả hơn. Ban đầu, các lập trình viên phải "nói chuyện" trực tiếp với CPU bằng **mã máy (machine code)**. Đến năm 1952, Grace Hopper đã đặt nền móng cho các **trình biên dịch (compiler)**, một cuộc cách mạng giúp con người viết mã bằng ngôn ngữ gần gũi hơn. Tuy nhiên, các trình biên dịch đời đầu tạo ra mã kém hiệu quả và bản thân máy tính cũng rất chậm chạp.

Bước ngoặt thực sự đến vào năm 1972, khi **Dennis Ritchie tại Bell Labs tạo ra ngôn ngữ C**. Mục tiêu của C rất rõ ràng và thực dụng:

1.  **Tính di động (Portability):** Có thể chạy trên nhiều kiến trúc phần cứng khác nhau với ít thay đổi.
2.  **Kiểm soát cấp thấp (Low-level Control):** Cung cấp cho lập trình viên quyền truy cập và thao tác trực tiếp với bộ nhớ, thanh ghi, và các tài nguyên hệ thống.

Điểm mấu chốt nằm ở vế thứ hai. C được thiết kế để **ánh xạ gần như trực tiếp xuống Assembly**, không có những tầng trừu tượng hay cơ chế an toàn "bất ngờ" nào khi chạy. Điều này mang lại hiệu năng vô song, nhưng cũng chính là nơi "quyền năng tối thượng" gặp "hiểm họa khôn lường". Triết lý thiết kế này trao toàn bộ trách nhiệm quản lý bộ nhớ cho lập trình viên, và bất kỳ sai sót nào cũng sẽ trở thành một cánh cửa cho kẻ tấn công.

### 2. Di Sản Không An Toàn: Tại Sao Lỗi Bộ Nhớ Vẫn Tồn Tại?

Dù thế giới công nghệ đã đi một chặng đường dài, di sản của C vẫn còn hiện hữu sâu sắc. Hãy nhìn vào dòng thời gian đơn giản hóa này:

*   **Thập niên 70:** C ra đời. Lập trình viên có toàn quyền kiểm soát, đồng nghĩa với việc các lỗ hổng an ninh bắt đầu xuất hiện.
*   **Thập niên 80:** C++ ra đời, tập trung vào thêm tính năng nhưng vẫn kế thừa mô hình quản lý bộ nhớ "nguy hiểm" của C.
*   **Thập niên 90-2000:** Các ngôn ngữ dựa trên Máy ảo (VM) và Just-In-Time (JIT) compilation như Java, Python, JavaScript nổi lên, mang theo cơ chế quản lý bộ nhớ tự động (garbage collection) và kiểm tra biên (bounds checking). Chúng an toàn hơn, nhưng các ngôn ngữ biên dịch truyền thống vẫn chiếm ưu thế trong các tác vụ đòi hỏi hiệu năng đỉnh cao.
*   **Thập niên 2010 đến nay:** Các ngôn ngữ an toàn bộ nhớ thế hệ mới như **Rust** ra đời, cố gắng cân bằng giữa hiệu năng và an toàn. Tuy nhiên, việc thay thế hàng tỷ dòng code C/C++ đã viết là điều không thể.

Hệ quả là một lượng khổng lồ phần mềm cốt lõi của thế giới—từ hệ điều hành (Windows, Linux, macOS), trình duyệt web (Chrome, Firefox), máy chủ web (Nginx, Apache) cho đến các thư viện hệ thống—đều được xây dựng bằng C và C++.

Theo chỉ số TIOBE, C và C++ vẫn liên tục nằm trong top những ngôn ngữ phổ biến nhất. Điều này có nghĩa là các lỗ hổng bộ nhớ không phải là vấn đề của quá khứ, chúng **đang được tạo ra mỗi ngày** trong các phần mềm mới.

### 3. Hạt Mầm Của Lỗ Hổng: "Điều Gì Sẽ Xảy Ra Nếu...?"

Ngay từ năm 1968, trong một bài báo đặt nền móng cho việc cách ly bộ nhớ giữa các tiến trình, Robert Graham và các cộng sự đã nêu ra một câu hỏi mang tính tiên tri (diễn giải):

> *"Điều gì sẽ xảy ra nếu một chương trình cho phép ai đó ghi đè lên vùng nhớ mà họ không được phép?"*

Ngày nay, đây không còn là một câu hỏi giả định. Đối với một nhà nghiên cứu bảo mật hay một hacker, đây chính là **cơ hội**. Câu trả lời cho câu hỏi đó là: "Bạn có thể chiếm quyền điều khiển chương trình đó." Lỗi ghi đè bộ nhớ (Memory Corruption) là gốc rễ của phần lớn các cuộc tấn công khai thác lỗ hổng phần mềm nghiêm trọng nhất.

</details>

---

## Phần 2: Bản Chất Của Lỗ Hổng - Khi Thiết Kế Trở Thành Vũ Khí

Các lỗ hổng bộ nhớ không tự nhiên sinh ra; chúng là hệ quả tất yếu của những quyết định thiết kế ở mức độ ngôn ngữ. Nhìn từ góc độ của một nhà nghiên cứu lỗ hổng, ngôn ngữ C cung cấp cho chúng ta 4 "món quà" tuyệt vời, xuất phát từ chính triết lý tối ưu hóa hiệu năng của nó.

### 1. Sự Tin Tưởng Mù Quáng (Vấn Đề Truy Cập Vượt Biên - Out-of-Bounds)

#### 1.1: OOB READ (RÒ RỈ THÔNG TIN - INFORMATION DISCLOSURE)

OOB Read là tiền đề của mọi cuộc tấn công hiện đại. Nếu không có nó, ASLR sẽ biến exploit của bạn thành một trò chơi may rủi 1/16777216.

**Mã nguồn mô phỏng (Vulnerable Code)**
Hãy tưởng tượng một hệ thống quản lý "ID nhân viên" đơn giản:

```c
// oob_read_example.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void win() {
    printf("Flag: FLAG{pwn_0day_hunter_in_progress}\n");
}

int main() {
    size_t index;
    long secret_key = 0xdeadbeefcafebabe; // Dữ liệu nhạy cảm cần bảo vệ
    long public_data[5] = {1, 2, 3, 4, 5};

    printf("Secret key address: %p\n", &secret_key);
    printf("Public data address: %p\n", public_data);

    printf("Enter index to read public data: ");
    scanf("%lu", &index); 

    // LỖI: Không kiểm tra biên (Bounds Check) của index
    printf("Data at index %lu is: 0x%lx\n", index, public_data[index]);

    return 0;
}
```

**Giải thích dưới lăng kính x64**:
Khi bạn nhập `index`, CPU sẽ tính toán địa chỉ cần đọc theo công thức:
`Address = public_data + (index * sizeof(long))`

*   Nếu `index = 0`: Đọc tại `public_data + 0`.
*   Nếu `index = 5`: Đọc tại `public_data + 40 bytes`. Trên Stack x64, vùng nhớ ngay sau mảng `public_data` chính là nơi lưu trữ biến `secret_key` (tùy vào cách trình biên dịch sắp xếp).

**Mã khai thác (Exploit Code - Python)**

```python
from pwn import *

p = process('./oob_read_example')

# Dựa vào static analysis, ta biết secret_key nằm ngay sau mảng 5 phần tử
# Vậy index 5 hoặc 6 (tùy padding) sẽ trỏ vào secret_key
index_to_leak = 6 

p.sendline(str(index_to_leak).encode())

p.recvuntil(b"is: ")
leaked_value = int(p.recvline().strip(), 16)

log.success(f"Leaked Secret Key: {hex(leaked_value)}")
```

---

#### 1.2: OOB WRITE (THAO TÚNG LUỒNG ĐIỀU KHIỂN)

OOB Write là thứ cho phép bạn thay đổi những thứ lẽ ra không được phép thay đổi.

**Mã nguồn mô phỏng (Vulnerable Code)**
Đây là lỗi kinh điển trong các hệ thống quản lý phân quyền (Privilege Escalation):

```c
// oob_write_example.c
#include <stdio.h>
#include <stdlib.h>

struct User {
    long id_list[2];
    int is_admin; // Cờ phân quyền: 0 là user, 1 là admin
};

int main() {
    struct User user1;
    user1.is_admin = 0; // Mặc định là user thường
    int index;
    long new_id;

    printf("Before OOB Write: is_admin = %d\n", user1.is_admin);
    printf("Enter index to update ID: ");
    scanf("%d", &index);
    printf("Enter new ID value: ");
    scanf("%ld", &new_id);

    // LỖI: Không kiểm tra index có nằm trong giới hạn [0, 1] hay không
    user1.id_list[index] = new_id;

    if (user1.is_admin != 0) {
        printf("Success! You are now ADMIN.\n");
        // Trong thực tế, đoạn này sẽ mở shell hoặc thực hiện lệnh nhạy cảm
    } else {
        printf("Fail. You are still a normal user.\n");
    }

    return 0;
}
```

**Giải thích cơ chế overwrite**:
Trong bộ nhớ, cấu trúc `User` sẽ được sắp xếp liên tục:
`[ id_list[0] (8 bytes) ] [ id_list[1] (8 bytes) ] [ is_admin (4 bytes) ] [ padding (4 bytes) ]`

Nếu ta nhập `index = 2`, lệnh `user1.id_list[2] = new_id` sẽ ghi vào vùng nhớ ngay sau `id_list[1]`. Vùng nhớ đó chính là biến `is_admin`.

**Mã khai thác (Exploit Code - Python)**

```python
from pwn import *

p = process('./oob_write_example')

# index 0, 1 là ID hợp lệ. Index 2 sẽ đè vào is_admin
p.sendlineafter(b"update ID: ", b"2")

# Ghi giá trị bất kỳ khác 0 vào để trở thành admin
p.sendlineafter(b"new ID value: ", b"1")

print(p.recvall().decode())
```

### 2. Arithmetic Primitives

#### Signedness Bug (Lỗi nhầm lẫn Dấu)

Đây là lỗi xảy ra khi lập trình viên so sánh một số có dấu (signed) với một số không dấu (unsigned), hoặc dùng số có dấu để kiểm tra biên nhưng lại truyền nó vào một hàm nhận số không dấu.

#### Mã nguồn lỗi (Vulnerable Code)
```c
// signedness_bug.c
#include <stdio.h>
#include <string.h>

void exploit_me() {
    char buffer[32];
    int size; // LỖI: Dùng kiểu có dấu (int) thay vì size_t

    printf("Enter size of data to read: ");
    scanf("%d", &size);

    // Bước kiểm tra biên (Bounds Check)
    if (size > 32) { 
        printf("Too big! Max is 32.\n");
        return;
    }

    // read(int fd, void *buf, size_t count)
    // size_t là unsigned long (64-bit)
    printf("Reading %d bytes...\n", size);
    read(0, buffer, size); 
}

int main() { exploit_me(); return 0; }
```

**Phân tích tầng sâu (Assembly x64)**
Kẻ tấn công sẽ nhập `size = -1`.
1.  **Tại câu lệnh `if (size > 32)`:** CPU sử dụng lệnh `jle` hoặc `jg` (so sánh có dấu). Vì `-1 < 32`, chương trình nghĩ rằng đầu vào an toàn và cho đi tiếp.
2.  **Tại hàm `read`:** Tham số thứ 3 (`count`) yêu cầu kiểu `size_t` (unsigned 64-bit). Khi truyền `-1` (dạng hex là `0xffffffff`) vào thanh ghi `RDX`, CPU sẽ thực hiện **Sign-Extension**.
    *   Thanh ghi `EDX` (32-bit) là `0xffffffff`.
    *   Khi mở rộng ra `RDX` (64-bit) để gọi hàm, nó trở thành `0xffffffffffffffff`.
    *   Hàm `read` sẽ cố gắng đọc **18 tỷ tỷ bytes** vào cái buffer 32 bytes $\rightarrow$ Stack Overflow ngay lập tức.

**Tips and Tricks:** Luôn tìm các biến `int` hoặc `short` được dùng làm tham số kích thước trong `malloc`, `memcpy`, `read`, `recv`.

---

#### Integer Overflow (Tràn số nguyên)

Lỗi này xảy ra khi một phép tính vượt quá giá trị tối đa của kiểu dữ liệu và "quay vòng" về giá trị nhỏ.

#### Mã nguồn lỗi (Vulnerable Code)
```c
// integer_overflow.c
#include <stdio.h>
#include <stdlib.h>

void manage_items() {
    unsigned short count;
    printf("How many items? (max 65535): ");
    scanf("%hu", &count);

    // Cấp phát bộ nhớ dựa trên số lượng items
    // Mỗi item nặng 8 bytes (ví dụ: con trỏ)
    size_t total_size = count * 8; 
    
    printf("Allocating %zu bytes for %hu items...\n", total_size, count);
    long *items = (long *)malloc(total_size);

    if (!items) return;

    // Vòng lặp nạp dữ liệu vào các items
    for (int i = 0; i < count; i++) {
        printf("Item %d: ", i);
        scanf("%ld", &items[i]); // OOB Write xảy ra ở đây
    }
}
```

#### Cơ chế "Bẫy bộ nhớ"
1.  **Kẻ tấn công nhập `count = 8193`**:
2.  **Phép tính toán:** `total_size = 8193 * 8 = 65544`.
3.  Tuy nhiên, nếu `total_size` bị ép vào một kiểu dữ liệu nhỏ hơn hoặc xảy ra tràn ở mức thanh ghi:
    *   Giả sử một lỗi logic khiến phép tính bị tràn ở mức 16-bit: `65544` sẽ trở thành `65544 % 65536 = 8`.
4.  `malloc(8)` sẽ cấp phát một vùng nhớ tí hon (chỉ đủ cho 1 item).
5.  Vòng lặp `for` vẫn chạy tới `8193`. Ngay từ item thứ 2 (`i=1`), bạn đã thực hiện một cú **Heap OOB Write** cực mạnh vào các chunk lân cận.

**Tips and Tricks:** Kiểm tra các phép nhân `count * size` hoặc phép cộng `offset + length`. Nếu không có lệnh kiểm tra tràn (`if (result < count)`), đó là lỗ hổng.

---

####  Integer Truncation (Cắt cụt số nguyên)

Xảy ra khi ép kiểu từ một biến lớn (64-bit) xuống một biến nhỏ (16/32-bit), làm mất các bit cao.

#### Mã nguồn lỗi (Vulnerable Code)
```c
// truncation_bug.c
#include <stdio.h>

void process_data(unsigned long long big_size) {
    // big_size là 64-bit (ví dụ: kích thước file)
    unsigned short small_size = (unsigned short)big_size; // LỖI: Cắt cụt

    printf("Big size: %llu, Small size: %hu\n", big_size, small_size);

    if (small_size < 100) {
        char buf[100];
        // Sử dụng giá trị gốc big_size để thực hiện ghi dữ liệu
        // Nhưng logic bảo vệ 'if' lại dựa trên small_size
        memcpy(buf, "A", big_size); 
    }
}
```

**Phân tích chiến thuật**:
1.  Kẻ tấn công gửi `big_size = 65537` (`0x10001` trong hex).
2.  Khi ép kiểu xuống `unsigned short` (16-bit), CPU chỉ lấy 4 số cuối (`0x0001`).
3.  `small_size` bây giờ bằng `1`.
4.  Điều kiện `if (1 < 100)` thỏa mãn.
5.  `memcpy` thực hiện copy **65537 bytes** vào `buf[100]`. Game over.

---

**Mã khai thác minh họa (Python Exploit cho Signedness Bug)**

```python
from pwn import *

# Mục tiêu: Vượt qua check size > 32 bằng cách nhập số âm
# Để thực hiện Stack Overflow

io = process('./signedness_bug')

# -1 được hiểu là 0xffffffff (Max Unsigned Int) khi vào hàm read
payload = b"A" * 72 # Vượt qua buffer (32) + padding + Saved RBP
payload += p64(0x4011d6) # Địa chỉ hàm win() hoặc địa chỉ ROP Gadget

io.sendlineafter(b"read: ", b"-1") # Bypass check size > 32
io.sendline(payload)

io.interactive()
```

### Tổng kết các mẹo với Lỗi Toán Học:

| Loại lỗi | Đặc điểm nhận dạng | Mục tiêu tấn công |
| :--- | :--- | :--- |
| **Signedness** | Dùng `int` để check `size`, nhưng dùng `size_t` để copy. | Nhập số âm để bypass check. |
| **Overflow** | Phép nhân `n * size` không có kiểm tra tràn. | Nhập số lớn để `malloc` ra vùng nhớ siêu nhỏ. |
| **Truncation** | Ép kiểu (cast) từ `long` sang `int` hoặc `short`. | Nhập số có các bit thấp thỏa mãn điều kiện nhưng giá trị thật rất lớn. |

**Ghi chú quan trọng:** Trong x64, các thanh ghi như `RDI, RSI, RDX` là 64-bit. Khi một lỗi toán học 32-bit xảy ra, hãy quan sát cách CPU mở rộng dấu (**MOVSXD**) hoặc xóa bit cao (**MOV EAX, ...**). Đó là nơi bạn tìm ra "con số kỳ diệu" để kích hoạt OOB.

---

### 3. Bounds Check Elimination

#### Missing Lower Bound (Quên kiểm tra biên dưới)

Lập trình viên thường chỉ sợ người dùng nhập số quá lớn (`index > max`), nhưng họ quên mất rằng số nguyên có thể là **số âm**.

**Mã nguồn lỗi (Vulnerable Code)**
```c
// oob_lower_bound.c
#include <stdio.h>

int main() {
    long secret_data = 0x1337133713371337;
    long public_array[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    int index;

    printf("Enter index to view data: ");
    scanf("%d", &index);

    // LỖI: Chỉ kiểm tra biên trên, quên biên dưới (index < 0)
    if (index >= 10) {
        printf("Out of bounds!\n");
        return 1;
    }

    printf("Data at index %d is: 0x%lx\n", index, public_array[index]);
    return 0;
}
```

**Phân tích chiến thuật**
Trong bộ nhớ Stack x64, các biến được khai báo trước thường nằm ở địa chỉ **cao hơn** các biến khai báo sau. 
*   `secret_data` nằm ở địa chỉ cao.
*   `public_array` nằm ở địa chỉ thấp hơn.
*   Tuy nhiên, các biến cục bộ khác hoặc **Saved RBP / Return Address** của hàm gọi (caller) thường nằm ở địa chỉ **thấp hơn** Stack Frame hiện tại (hoặc cao hơn tùy vào hướng phát triển).

**Kỹ thuật OOB ngược:** Nếu bạn nhập `index = -2`, `index = -5`, CPU sẽ lấy địa chỉ `public_array` trừ đi một khoảng. Điều này cho phép bạn **đọc ngược lên phía trên Stack**, nơi chứa các con trỏ quản lý của hệ điều hành.

---

#### Logic Mismatch (Kiểm tra một đằng, dùng một nẻo)

Đây là lỗi cực kỳ phổ biến trong các trình xử lý giao thức mạng (Network Parsers).

**Mã nguồn lỗi (Vulnerable Code)**
```c
// bounds_check_logic.c
#include <stdio.h>
#include <string.h>

void handle_packet(char *user_data, int user_len) {
    char internal_buffer[64];
    int limit = 64;

    // Bước kiểm tra biên
    if (user_len > limit) {
        printf("Packet too large!\n");
        // LỖI: Cảnh báo nhưng KHÔNG dừng chương trình hoặc không cắt gọn user_len
    }

    // Vẫn sử dụng user_len gốc để thực hiện thao tác bộ nhớ
    memcpy(internal_buffer, user_data, user_len); 
    printf("Packet processed.\n");
}
```

**Note:**
Lập trình viên đôi khi viết code kiểm tra chỉ để in ra log cảnh báo (để debug) mà quên mất rằng luồng thực thi vẫn tiếp tục. Khi bạn thấy một đoạn code có `if (len > max) { log("error"); }` mà không có `return` hoặc `exit`, đó chính là 0-day của bạn.

---

#### TOCTOU (Time-of-Check Time-of-Use) - Race Condition

Đây là kỹ thuật nâng cao, thường dùng để săn lỗi trong **Kernel** hoặc các ứng dụng đa luồng (Multi-threaded).

#### Cơ chế hoạt động:
1.  **Thread A (Check):** Kiểm tra biến `index = 5`. Thấy `5 < 10` (Hợp lệ).
2.  **Thread B (Attack):** Ngay sau khi Thread A check xong nhưng chưa kịp dùng, Thread B nhảy vào ghi đè giá trị `index = 9999` vào bộ nhớ.
3.  **Thread A (Use):** Tiếp tục thực hiện lệnh `array[index] = value` với giá trị `index` đã bị sửa thành `9999`.

**Kết quả:** Vượt qua mọi rào cản kiểm tra biên bằng cách tận dụng độ trễ của CPU.

---

#### Advanced: Bounds Check Elimination (BCE) của Compiler

Đây là đỉnh cao của việc "trình biên dịch hại lập trình viên". Để tối ưu tốc độ, các trình biên dịch hiện đại (như LLVM/Clang) sẽ cố gắng đoán xem một lệnh kiểm tra biên có thừa hay không.

Nếu trình biên dịch "nghĩ" rằng một biến `i` luôn nhỏ hơn 10 dựa trên các đoạn code phía trên, nó sẽ **tự động xóa bỏ** câu lệnh `if (i < 10)`. Nếu bạn tìm ra cách đánh lừa logic phân tích của trình biên dịch (thông qua các phép toán phức tạp), bạn sẽ có một cú OOB mà khi đọc mã nguồn C thì thấy có vẻ an toàn, nhưng trong file Binary thì lệnh check đã biến mất.

---

#### Mã khai thác minh họa (Python - OOB Read ngược)

```python
from pwn import *

# Mục tiêu: Đọc Saved RIP để leak địa chỉ libc
# Bằng cách sử dụng index âm trong mảng trên Stack

io = process('./oob_lower_bound')

# Giả sử qua debug GDB, ta biết Return Address nằm cách mảng 
# về phía địa chỉ thấp (index âm) là 8 đơn vị (mỗi đơn vị 8 bytes)
index_to_leak = -8 

io.sendlineafter(b"view data: ", str(index_to_leak).encode())

io.recvuntil(b"is: ")
leak = int(io.recvline().strip(), 16)

log.info(f"Leaked Address (RIP): {hex(leak)}")

# Tính toán base address của chương trình
# base = leak - offset_tu_rip_den_base
```

### Tổng kết tư duy săn lỗi Bounds Check:

| Chiến thuật | Cách tìm | Mục tiêu |
| :--- | :--- | :--- |
| **OOB Ngược** | Tìm các index không check `< 0`. | Đọc/Ghi các dữ liệu quản lý nằm trước mảng. |
| **Logic Mismatch** | Tìm các câu lệnh `if` check biên nhưng không có `return` thoát hàm. | Thực hiện Buffer Overflow dù đã có check. |
| **TOCTOU** | Tìm các biến biên được lưu trong bộ nhớ dùng chung (Shared Memory/Global). | Dùng đa luồng để sửa giá trị giữa bước Check và Use. |

**Tips and Tricks:** Khi đọc code, hãy luôn đặt câu hỏi: *"Điều kiện này có thể bị sai không? Nếu tôi là CPU, tôi có thể bỏ qua lệnh này không?"*. 

---

### 2. Sự Trộn Lẫn Chết Người Giữa Dữ Liệu và Luồng Điều Khiển

Trên lý thuyết, dữ liệu người dùng (Non-control data) không bao giờ được phép chi phối luồng thực thi (Control flow) của chương trình. Tuy nhiên, kiến trúc Von Neumann và cấu trúc của Ngăn xếp (Stack) lại trộn lẫn tất cả chúng vào cùng một nơi.

Trên kiến trúc x64, khi một hàm được gọi, Stack Frame của nó thường được sắp xếp như sau (từ địa chỉ cao xuống địa chỉ thấp):
*   **Return Address (8 bytes):** Địa chỉ lệnh tiếp theo (`RIP` sẽ nhảy tới đây sau khi hàm gọi lệnh `ret`).
*   **Saved RBP (8 bytes):** Base pointer của hàm gọi trước đó.
*   **Các biến cục bộ (Local Variables & Buffers).**

Mọi thứ nằm sát cạnh nhau. Nếu một biến cục bộ bị ghi đè vượt giới hạn (Buffer Overflow), dữ liệu của kẻ tấn công sẽ tràn dần lên địa chỉ cao hơn, ghi đè lên `Saved RBP` và quan trọng nhất là **Return Address**. 

Khi hàm kết thúc, lệnh `ret` được thực thi. Lệnh này đơn giản là `pop rip` – lấy 8 bytes tại đỉnh stack hiện tại và nhét vào thanh ghi lệnh `RIP`. Nếu chúng ta đè Return Address bằng một địa chỉ tùy ý, chương trình sẽ ngoan ngoãn nhảy đến đó. Lịch sử đã chứng minh mức độ tàn phá của cơ chế này bằng sự kiện Morris Worm năm 1988 (làm sập toàn bộ Internet bằng việc khai thác hàm `gets` trong sendmail/fingerd). Ngày nay, thao tác hijack `RIP` này là bước đầu tiên để thiết lập một chuỗi ROP (Return-Oriented Programming) tinh vi nhằm qua mặt các lớp bảo vệ của hệ điều hành.

### 3. Vấn Đề Dữ Liệu Đóng Vai Trò Metadata (In-band Signaling)

C xử lý chuỗi (string) theo một cách rất đặc thù: Chuỗi không có kích thước cố định, mà được kết thúc bằng một byte Null (`\0`). Tại đây, byte Null đóng vai trò là một Metadata (siêu dữ liệu báo hiệu kết thúc) nhưng lại nằm xen kẽ trực tiếp vào Data (Dữ liệu).

Lỗ hổng logic và rò rỉ bộ nhớ bắt nguồn từ đây:
*   **Thiếu Null-byte (Overread / Information Leak):** Hãy tưởng tượng một buffer có kích thước 16 bytes và bạn nhập chính xác 16 ký tự 'A' (không còn chỗ cho `\0`). Khi một hàm như `printf` hay `puts` in chuỗi này ra, nó sẽ không dừng lại ở byte thứ 16. Nó sẽ tiếp tục đọc và in các byte nằm liền kề trên Stack cho đến khi vô tình gặp một byte `\0` nào đó. Vùng dữ liệu liền kề này thường chứa các con trỏ (địa chỉ của thư viện `libc`, địa chỉ của Stack, hoặc Stack Canary). Việc cố ý tạo ra chuỗi mất Null-byte là kỹ thuật sống còn để thu thập **Info Leak**, dùng để vượt qua cơ chế ngẫu nhiên hóa không gian địa chỉ (ASLR / PIE) của hệ thống hiện đại.
*   **Dư thừa Null-byte (Truncation):** Ngược lại, nếu kẻ tấn công khéo léo chèn `\0` vào giữa payload, các hàm xử lý chuỗi sẽ dừng lại sớm hơn dự kiến, dẫn đến các sai lệch nghiêm trọng trong logic xác thực hoặc tính toán kích thước.

### 4. Ký Ức Của Ngăn Xếp (Uninitialized Data)

C không dọn dẹp bộ nhớ thay bạn. Nếu bạn khai báo một mảng nhưng không khởi tạo nó:
```c
void my_function() {
    char my_variable[32];
    // my_variable đang chứa gì?
}
```
Nó sẽ chứa toàn bộ "rác" còn sót lại của các Stack Frame trước đó. Khi một hàm thực thi xong, bộ nhớ stack của nó chỉ đơn giản là được "trả lại" bằng cách cộng dồn thanh ghi `RSP` đi lên, nhưng giá trị vật lý trên RAM không hề bị xóa.

Nếu một hàm trước đó vừa thực hiện các tác vụ mã hóa, hoặc gọi vào các hàm của libc (để lại các địa chỉ libc trên stack), biến `my_variable` chưa khởi tạo ở hàm hiện tại sẽ vô tình ôm trọn các dữ liệu nhạy cảm này. Nếu lập trình viên vô tình in hoặc trả về giá trị của `my_variable`, họ lại một lần nữa dâng tặng cho kẻ tấn công một lỗ hổng Information Leak hoàn hảo mà không cần phải ghi đè bất cứ một byte nào.

*(Lưu ý: Mặc dù đôi khi lập trình viên có ý định làm sạch bộ nhớ bằng hàm `memset`, các trình biên dịch hiện đại với cờ tối ưu hóa như `-O2` có thể sẽ tự động loại bỏ các lệnh `memset` này nếu nó phân tích thấy biến đó không được sử dụng ở phía dưới, vô tình biến code an toàn thành code lỗi).*

---

## Phần 3: Smashing the Stack - "Đập Vỡ" Ngăn Xếp

"Smashing the Stack for Fun and Profit" (Aleph One, 1996) là bài viết kinh điển đã định hình toàn bộ ngành khai thác lỗ hổng nhị phân. Dù các kỹ thuật đã phát triển phức tạp hơn rất nhiều từ năm 1996, nguyên lý cơ bản của việc ghi đè ngăn xếp (Stack-based Buffer Overflow) vẫn giữ nguyên bản chất. 

Chúng ta hãy mổ xẻ một ví dụ để hiểu rõ cách bộ nhớ bị lây nhiễm.

### 1. Phân Tích Hiện Trường: Ngăn Xếp 64-bit Trong Thực Tế

Hãy xem xét đoạn mã bị lỗi sau:

```c
11 char * quote(char *s) {
12    char output[16];
13    sprintf(output, "\"%s\"", s); // Lỗi ở đây!
14    return output;
15 }
```

Trước khi dữ liệu người dùng (`s`) được bơm vào, chúng ta cần hình dung trạng thái vật lý của Stack trong hàm `quote()`. Trên kiến trúc x64 (Linux), Stack phát triển từ địa chỉ cao xuống địa chỉ thấp, nhưng các thao tác ghi buffer (như `sprintf`) lại ghi từ địa chỉ thấp lên địa chỉ cao. Do đó, buffer luôn hướng thẳng vào các dữ liệu điều khiển:

```text
[High Address]
...
(Các dữ liệu của tiến trình cha, biến môi trường, đối số hàm main)
...[Stack Frame của main()]
+-------------------------+
| Return Address của main | (Trỏ về __libc_start_main)
+-------------------------+
...
[Stack Frame của print_quoted()]
+--------------------------------+
| Return Address về print_quoted | (Trỏ về lệnh tiếp theo trong main)
+--------------------------------+
| Saved RBP                      |
+--------------------------------+
...
[Stack Frame của quote()]
+--------------------------------+
| Return Address về quote        | (Trỏ về lệnh tiếp theo trong print_quoted)  <--- MỤC TIÊU CỐT LÕI
+--------------------------------+
| Saved RBP                      |
+--------------------------------+
| output buffer (16 bytes)       | <--- NƠI BẮT ĐẦU GHI (Địa chỉ thấp)
+--------------------------------+
[Low Address]
```

**Nguyên nhân gốc rễ ở đây là sự kết hợp của hai yếu tố:**
1. Lập trình viên sử dụng hàm `sprintf` – một hàm xử lý chuỗi không kiểm tra độ dài.
2. Bản thân hàm `sprintf` chỉ nhận con trỏ của `output` mà không có bất kỳ thông tin nào về kích thước vật lý (16 bytes) của mảng đó. `sprintf` sẽ sao chép cho đến khi gặp Null-byte ở mảng nguồn `s`.

### 2. Sự Lây Nhiễm Bộ Nhớ (Memory Corruption)

Khi dữ liệu `s` (do kẻ tấn công kiểm soát) vượt quá 16 bytes (cộng thêm 2 bytes dấu ngoặc kép `""`), chuỗi ký tự sẽ bắt đầu tràn khỏi mảng `output`. Quá trình "đập vỡ" diễn ra theo trình tự từ dưới lên trên (trong sơ đồ trên):

1.  **Tràn qua biến cục bộ:** Nếu có các biến cục bộ khác nằm trên `output`, chúng sẽ bị đè đầu tiên. Kẻ tấn công có thể thay đổi các cờ (flags), bộ đếm vòng lặp, hoặc các con trỏ trạng thái để thay đổi luồng logic của hàm hiện tại.
2.  **Đè lên Saved RBP:** Tiếp tục tràn, kẻ tấn công ghi đè địa chỉ Base Pointer cũ. Việc này tạo tiền đề cho kỹ thuật **Stack Pivoting** (chuyển hướng cả Stack Frame hiện tại sang một vùng nhớ khác do kẻ tấn công kiểm soát).
3.  **Chiếm đoạt Return Address (Ultimate Power):** Đây là mục tiêu tối thượng. Bằng cách ghi đè chính xác 8 bytes của Return Address, khi hàm `quote` kết thúc và gọi lệnh `ret`, CPU sẽ nhảy đến địa chỉ mà kẻ tấn công đã đặt sẵn thay vì quay lại hàm `print_quoted`.

### 3. Sức Mạnh Của Memory Corruption

Khi đã có khả năng ghi đè bộ nhớ, tùy vào "chất lượng" của lỗ hổng và mục tiêu trên Stack, chúng ta có thể đạt được những gì?

*   **Ghi đè Dữ liệu phi điều khiển (Non-control data):** Có vẻ nhàm chán, nhưng nếu biến bị đè là một biến kiểm tra quyền (`isAdmin`), một điều kiện nhảy (`win variable`), kích thước bộ đệm cho một lệnh `read` tiếp theo, hoặc mã PIN, kẻ tấn công có thể thay đổi hoàn toàn cục diện mà không cần động đến các con trỏ hàm.
*   **Arbitrary Read/Write qua Con trỏ (Pointer Overwrite):** Nếu phía trên buffer là một biến kiểu con trỏ (ví dụ: `char *dest`), việc đè lên con trỏ này bằng một địa chỉ khác (VD: địa chỉ bảng GOT của hàm `puts`) sẽ khiến thao tác ghi dữ liệu tiếp theo của chương trình bị "trượt" vào bảng GOT, tạo ra lỗ hổng **Arbitrary Write**.
*   **Redirect Execution (Chiếm đoạt Control Flow):** Như đã phân tích, ghi đè Return Address (hoặc Function Pointers) cho phép chúng ta điều khiển thanh ghi `RIP`.

### 4. Từ Việc Chiếm RIP Đến Việc Thực Thi Mã Lệnh

Sau khi ghi đè thành công Return Address, chúng ta có thể làm gì?

**Mức độ 1: Return-to-Win (Ret2Win)**
Nếu chương trình tự nó chứa sẵn một hàm "tặng điểm" (ví dụ: `void win() { system("/bin/sh"); }`), chúng ta chỉ cần đè Return Address bằng địa chỉ của hàm `win()`. Đây là dạng bài tập cơ bản nhất.

**Mức độ 2: Thực thi Shellcode (Ret2Shellcode)**
Nếu vùng nhớ Stack (hoặc Heap) được cấp quyền Thực thi (Executable - thiếu cơ chế bảo vệ NX/DEP), chúng ta có thể:
1. Chèn chính mã máy độc hại (Shellcode - thường là các lệnh gọi syscall `execve`) vào chính buffer chúng ta gửi lên.
2. Ghi đè Return Address sao cho nó trỏ thẳng lại về đầu buffer của chúng ta.
Lệnh `ret` sẽ nhảy vào vùng dữ liệu đó và CPU sẽ coi dữ liệu (Data) là mã lệnh (Instruction) và thực thi.

**Mức độ 3: ROP (Return-Oriented Programming)**
Trên các hệ điều hành hiện đại, Stack luôn bị khóa quyền thực thi (NX - No-eXecute). Kẻ tấn công không thể đưa mã mới vào. Thay vào đó, họ phải **"tái chế" lại các mã lệnh đã có sẵn** trong chương trình và các thư viện (như `libc`).

Trên kiến trúc x64, tham số đầu tiên của một hàm không được truyền qua Stack mà nằm ở thanh ghi `RDI`. Để gọi `system("/bin/sh")`, chúng ta không thể chỉ nhảy đến hàm `system`. Chúng ta cần đưa chuỗi `/bin/sh` vào `RDI` trước. 

Để làm điều này, kẻ tấn công xâu chuỗi (chain) các mảnh mã ngắn (gọi là **Gadgets**) kết thúc bằng lệnh `ret`. Ví dụ, một gadget phổ biến là `pop rdi; ret`. Bằng cách thiết kế Stack chứa địa chỉ của gadget này, tiếp theo là giá trị cần đưa vào `RDI` (địa chỉ của chuỗi "/bin/sh"), và cuối cùng là địa chỉ của hàm `system`, chúng ta có thể thực thi các tác vụ vô cùng phức tạp hoàn toàn chỉ thông qua việc ghi đè Ngăn xếp.

---

## Phần 4: Cội Nguồn Của Sự Sụp Đổ - Phân Tích Logic Gây Lỗi

Hầu hết các lập trình viên đều biết "Buffer Overflow là gì". Tuy nhiên, trong môi trường săn lỗ hổng chuyên nghiệp hay các phần mềm hệ thống phức tạp, bạn hiếm khi gặp một lỗi `strcpy` hay `gets` lộ liễu. Kẻ tấn công thường nhắm vào **khoảng trống ngữ nghĩa (semantic gap)** giữa những gì lập trình viên nghĩ ngôn ngữ C sẽ làm, và những gì CPU thực sự thực thi ở tầng Assembly. 

Dưới đây là 4 tác nhân cốt lõi gây ra Memory Corruption.

### 1. Classic Buffer Overflow & Kích Thước Ảo Ảnh

Lỗi cơ bản nhất xuất phát từ việc C không tự động theo dõi kích thước mảng. 
```c
char small_buffer[16];
read(0, small_buffer, 128); // read() không hề biết small_buffer chỉ chứa được 16 bytes
```
Tuy nhiên, trên kiến trúc x64, mọi thứ không chỉ đơn giản là đếm byte. Do yêu cầu về **Data Alignment (Căn chỉnh dữ liệu)** để tối ưu tốc độ đọc/ghi của CPU, trình biên dịch thường thêm các byte đệm (padding) vào giữa các biến cục bộ hoặc các trường (fields) trong một `struct` sao cho chúng nằm trên ranh giới 8-byte hoặc 16-byte.

Một lỗi overflow nhỏ (vài byte) có thể không chạm tới `Saved RBP` hay `Return Address` ở cuối Stack, nhưng nó có thể âm thầm ghi đè vào các byte padding, và nguy hiểm hơn là tràn vào các biến cục bộ liền kề phía dưới (ví dụ: các cờ boolean phân quyền, hoặc các con trỏ trạng thái). Việc kiểm soát các biến cục bộ này thông qua một overflow "nửa vời" thường là chìa khóa để leo thang đặc quyền (Privilege Escalation) mà không làm crash chương trình.

### 2. Signedness Mixups (Lỗi Cú Pháp Dấu) - Cú Lừa Của Two's Complement

Đây là một trong những dạng lỗi "thơm" nhất để săn 0-day trong các giao thức mạng hoặc parsing file. Các hàm hệ thống cốt lõi (`read`, `memcpy`, `strncpy`) luôn yêu cầu tham số kích thước (tham số thứ 3) phải là một số nguyên dương không dấu – kiểu `size_t` (trên x64, `size_t` là một số nguyên không dấu 64-bit, truyền qua thanh ghi `RDX`).

Nhưng các lập trình viên thường có thói quen dùng `int` (số nguyên có dấu 32-bit) để lưu độ dài.

```c
int size;
char buf[16];
scanf("%i", &size);         // Lập trình viên nhập vào -1 (0xffffffff)
if (size > 16) exit(1);     // Kiểm tra xem size có quá lớn không?
read(0, buf, size);         // BOOM!
```

**Dưới lăng kính Assembly x64, điều gì đã xảy ra?**

1.  **Kiểm tra điều kiện:** Khi so sánh `size > 16`, CPU sử dụng lệnh `cmp eax, 16`. Vì `size` được khai báo là `int` (có dấu), trình biên dịch sinh ra lệnh nhảy **`jge` (Jump if Greater or Equal - so sánh có dấu)**. Số `-1` hoàn toàn nhỏ hơn `16`, nên nó vượt qua cửa ải kiểm tra một cách dễ dàng.
2.  **Mở rộng dấu (Sign-Extension):** Trước khi gọi `read()`, tham số `size` (32-bit `eax`) phải được nạp vào thanh ghi `rdx` (64-bit) để tuân thủ calling convention. Trình biên dịch sẽ dùng lệnh **`movsxd rdx, eax` (Move with Sign-Extension)**. Nó thấy `eax` có bit dấu là 1 (vì là số âm), nên nó nhồi toàn bộ 32 bit cao của `rdx` thành 1.
    *   Giá trị ban đầu: `eax = 0xffffffff` (-1)
    *   Sau khi chuyển hóa: `rdx = 0xffffffffffffffff` (Max giá trị của uint64_t)
3.  **Thảm họa thực thi:** Hàm `read` nhận được một số siêu khổng lồ (vài Exabyte) chứ không phải -1. Nó sẽ liên tục đọc dữ liệu từ nguồn vào bộ nhớ cho đến khi tràn toàn bộ Stack hoặc gặp lỗi Segmentation Fault, tạo ra một cú Arbitrary Write vô tận.

*(Lưu ý đối trọng: Nếu `size` được khai báo là `unsigned int`, trình biên dịch sẽ dùng lệnh nhảy `jae` (Jump if Above or Equal - không dấu), và `-1` tức `0xffffffff` sẽ ngay lập tức lớn hơn 16, chặn đứng cuộc tấn công).*

### 3. Integer Overflows & Sự Sụp Đổ Của Toán Học

Lỗ hổng này xảy ra khi các phép tính số học vượt qua ngưỡng giới hạn của kiểu dữ liệu. Xét đoạn mã tính toán cấp phát động (rất hay gặp trong các trình parse định dạng file):

```c
unsigned int size;
scanf("%i", &size);
char *buf = alloca(size + 1); // Cấp phát trên stack dựa trên size + 1
int n = read(0, buf, size);
```

Ngay cả khi dùng `unsigned int`, lỗ hổng vẫn tồn tại. Giá trị lớn nhất của `unsigned int` 32-bit là `0xffffffff` (4,294,967,295). Điều gì xảy ra nếu kẻ tấn công nhập chính xác giá trị `0xffffffff` vào `size`?

1.  **Tràn số (Wrap-around):** Phép tính `size + 1` trở thành `0xffffffff + 1`. Ở cấp độ thanh ghi 32-bit, giá trị này quay vòng về `0`.
2.  **Cấp phát tàng hình:** Hàm `alloca()` (cấp phát bộ nhớ trực tiếp trên Stack bằng cách đẩy thanh ghi `RSP` xuống) sẽ thực thi `sub rsp, 0`. Nói cách khác, **không có một byte nào được cấp phát thêm**. Con trỏ `buf` trỏ ngay vào đỉnh Stack hiện tại (chứa nguyên các biến cũ hoặc Return Address).
3.  **Ghi đè:** Lệnh `read(0, buf, size)` sau đó lại dùng biến `size` ban đầu (`0xffffffff`). Nó sẽ ghi 4GB dữ liệu trực tiếp vào đỉnh Stack, cày nát toàn bộ Stack Frame phía trên nó.

*(Trong thực tế săn 0-day, kỹ thuật này thường xuyên được áp dụng với bộ nhớ Heap: `malloc(count * sizeof(struct))`. Một phép nhân tràn số sẽ dẫn đến việc cấp phát một chunk Heap cực nhỏ, nhưng vòng lặp ghi dữ liệu phía sau lại dựa trên `count` cực lớn, gây ra Heap-based Buffer Overflow cực kỳ uy lực).*

### 4. Off-by-one Errors (OBOE) & Hiệu Ứng Cánh Bướm

Off-by-one là những lỗi ranh giới, thường sinh ra từ các vòng lặp lệch 1 index:
```c
int a[3] = { 1, 2, 3 };
for (int i = 0; i <= 3; i++) a[i] = 0; // Kẻ sát nhân tĩnh lặng
```
Hoặc kinh điển nhất là **Off-By-One Null Byte**, khi các hàm xử lý chuỗi (như `strcpy`, `strncat`) tự động nhồi thêm một byte `\0` vào vị trí cuối cùng của buffer mà lập trình viên không lường trước.

Nhìn qua, việc đè **đúng 1 byte** có vẻ vô hại. Làm sao bạn có thể chiếm được `RIP` khi bạn cần tới 8 byte để ghi đè Return Address? Câu trả lời nằm ở kiến trúc hoạt động của `RBP` (Base Pointer) và kỹ thuật **Stack Pivoting (Bẻ nhánh ngăn xếp)**.

Hãy nhớ lại cấu trúc Stack Frame: ngay bên trên mảng `a` chính là `Saved RBP` (8 bytes), rồi mới tới `Return Address`. Việc tràn 1 byte `\0` sẽ vô tình ghi đè đè lên **Least Significant Byte (Byte thấp nhất - LSB)** của `Saved RBP`.

Trên x64, địa chỉ bộ nhớ là Little Endian. Nếu `Saved RBP` ban đầu đang trỏ đến Stack Frame của hàm gọi (ví dụ: `0x7fffffffe120`), một byte `\0` ghi lấn sang sẽ biến nó thành `0x7fffffffe100`. 
Đột nhiên, Base Pointer của hàm cha bị kéo lệch khỏi vị trí gốc, trượt sâu vào bên trong các biến cục bộ (rất có thể là buffer mà kẻ tấn công đang kiểm soát).

Khi hàm cha thực thi xong, chuỗi lệnh Epilogue kinh điển sẽ diễn ra:
```assembly
leave   ; Tương đương: mov rsp, rbp; pop rbp
ret     ; Tương đương: pop rip
```
Bởi vì `RBP` đã bị bóp méo, lệnh `mov rsp, rbp` sẽ ném thanh ghi đỉnh ngăn xếp (`RSP`) sang vùng nhớ độc hại của kẻ tấn công. Lệnh `ret` sau đó sẽ ung dung bốc giá trị giả mạo mà kẻ tấn công cấy sẵn tại đó để tống vào `RIP`. 
Chỉ với 1 byte `\0`, một chiếc cánh bướm nhỏ đã tạo ra cơn bão cướp toàn quyền điều khiển chương trình. Đây là một trong những kỹ thuật Pwn thanh lịch và tinh xảo nhất.




Dưới đây là phần tiếp theo, được tổng hợp từ các slide "Stack Canaries", "Causes of Disclosure" và "Tips and Tricks". Phần này đi sâu vào cuộc chiến giữa các cơ chế phòng thủ hiện đại của trình biên dịch và các nghệ thuật rò rỉ bộ nhớ (Information Leak) – chìa khóa sống còn của mọi cuộc khai thác thực tế trên x64.

---

## Phần 5: Lớp Phòng Thủ Đầu Tiên & Nghệ Thuật Rò Rỉ Dữ Liệu

Khi các cuộc tấn công ghi đè Stack trở nên quá phổ biến, giới bảo mật không thể chỉ ngồi chờ lập trình viên viết code an toàn hơn. Họ quyết định thay đổi ngay từ tầng Trình biên dịch (Compiler). Năm 1998, khái niệm **StackGuard** (hay Stack Canary) ra đời, tạo ra một rào cản vật lý bảo vệ các dữ liệu điều khiển nhạy cảm. 

Nhưng trong thế giới khai thác lỗ hổng, mọi rào cản đều đi kèm với một chìa khóa để mở nó.

### 1. Stack Canary Trên Kiến Trúc x64: "Cơ Chế Báo Động"

Ý tưởng của Stack Canary rất đơn giản: Đặt một giá trị ngẫu nhiên (Canary) nằm giữa các biến cục bộ và `Saved RBP` / `Return Address`. Nếu một cuộc tấn công Buffer Overflow xảy ra từ dưới lên, kẻ tấn công buộc phải "cày" qua giá trị Canary này trước khi chạm tới được `RIP`.

Dưới góc nhìn Assembly x64 (Linux), cơ chế này được nhúng trực tiếp vào Prologue và Epilogue của hàm:

**Khi vào hàm (Prologue):**
```assembly
mov    rax, QWORD PTR fs:0x28  ; Lấy giá trị Canary 64-bit ngẫu nhiên từ TLS (Thread Local Storage)
mov    QWORD PTR [rbp-0x8], rax ; Đặt nó ngay phía trên các biến cục bộ (sát RBP)
xor    eax, eax                ; Xóa thanh ghi để tránh rò rỉ Canary vô tình
```

**Khi thoát hàm (Epilogue):**
```assembly
mov    rcx, QWORD PTR [rbp-0x8] ; Lấy lại Canary trên Stack
xor    rcx, QWORD PTR fs:0x28   ; So sánh với giá trị gốc trong TLS
je     safe_exit                ; Nếu bằng nhau (Canary nguyên vẹn) -> Thoát bình thường
call   __stack_chk_fail         ; NẾU BỊ ĐÈ: Gây Crash (SIGABRT) ngay lập tức
```

Để ngăn chặn các hàm xử lý chuỗi (như `puts`, `printf`) vô tình in ra giá trị này, Canary trên x64 Linux được thiết kế **luôn bắt đầu bằng một byte Null (`0x00`)** ở LSB (Least Significant Byte - nằm ở địa chỉ thấp nhất theo kiến trúc Little Endian). Kẻ tấn công không thể dễ dàng đoán được 7 bytes ngẫu nhiên còn lại. Tuy nhiên, nếu chúng ta có thể làm lộ giá trị này, Canary sẽ hoàn toàn vô tác dụng.

### 2. Sự Rò Rỉ (Information Disclosure) – Vũ Khí Tối Thượng

Trong các bài toán khai thác hiện đại với sự hiện diện của Canary và ASLR (Address Space Layout Randomization), việc có được một lỗ hổng Memory Corruption (Arbitrary Write) là chưa đủ. Bạn bị mù hoàn toàn về không gian bộ nhớ. **Rò rỉ dữ liệu (Info Leak)** là điều kiện tiên quyết để giải quyết bài toán mù này. 

Dưới đây là các lỗ hổng logic tạo ra sự rò rỉ:

*   **Lỗ Hổng Thiếu Null-Byte (Termination Problems):** 
    Như đã phân tích, chuỗi C cần `\0` để kết thúc. Nhờ thiết kế Little Endian, byte `\0` của Canary nằm sát ngay trên buffer của chúng ta. Nếu ta nhập một chuỗi vừa khít chạm vào Canary và ghi đè đúng **1 byte Null** của nó thành một ký tự khác (ví dụ 'A'), hàm `puts(buffer)` sẽ in chuỗi của ta, trôi tuột qua ranh giới, và in luôn 7 bytes Canary còn lại. (Tất nhiên, trước khi hàm kết thúc, ta phải khéo léo hoàn trả lại byte Null này để Canary không bị báo lỗi).
*   **Buffer Overread (Đọc Vượt Biên):** 
    Xảy ra khi chương trình cung cấp cho người dùng quyền kiểm soát số lượng byte được đọc ra (ví dụ: `write(1, buffer, size)` với `size` bị thao túng). Lỗ hổng này quét qua toàn bộ cấu trúc Stack, cho phép kẻ tấn công thu hoạch không chỉ Canary, mà còn cả `Saved RBP` (để tính toán base của Stack) và `Return Address` (để tính toán base address của libc và Program, xuyên thủng PIE/ASLR).
*   **Dữ Liệu Chưa Khởi Tạo (Uninitialized Data):** 
    Kẻ tấn công có thể lợi dụng việc một hàm trước đó gọi vào thư viện để lại các con trỏ `libc` trên Stack. Nếu hàm tiếp theo có một mảng không khởi tạo và lại đẩy mảng đó ra màn hình, kẻ tấn công dễ dàng thu thập được các mảng địa chỉ nhạy cảm mà không cần bất kỳ thao tác tràn bộ đệm nào.

### 3. Các Phương Pháp "Lách" Qua Stack Canary

Có ba kỹ thuật phổ biến để biến Canary thành vật trang trí trong các cuộc khai thác thực tế:

1.  **Leak Canary:** Sử dụng các lỗ hổng Disclosure nêu trên. Một khi thu thập được Canary hiện tại, trong payload tràn bộ đệm tiếp theo, kẻ tấn công chỉ cần "xây" lại Stack bằng cách nhét chính giá trị Canary đó vào đúng vị trí cũ, và tiếp tục đè lên `RIP`.
2.  **Byte-by-Byte Brute-Force (Tấn công các tiến trình phân nhánh):**
    Trong các mô hình máy chủ web cổ điển hoặc các dịch vụ mạng gọi hàm `fork()` để xử lý client mới, tiến trình con sẽ thừa kế **chính xác** bản sao bộ nhớ của tiến trình cha, bao gồm cả Canary. Nếu ta ghi đè sai Canary, tiến trình con crash, nhưng tiến trình cha không hề hấn gì và sẽ sinh ra một tiến trình con mới (với Canary không đổi).
    Thay vì đoán 8 bytes cùng lúc (bất khả thi), ta đè 1 byte. Nếu crash $\rightarrow$ sai. Nếu chương trình trả về bình thường $\rightarrow$ byte đó đúng. Tiếp tục tịnh tiến, trên x64 ta chỉ mất tối đa $256 \times 7 = 1792$ lần thử để mò ra toàn bộ 8 bytes Canary. 
3.  **Nhảy Cóc (Jumping the Canary) bằng Arbitrary Write:**
    Nếu lỗ hổng không phải là ghi đè tuyến tính (Linear Overflow) mà là một vòng lặp sử dụng chỉ số mảng (Index Manipulation):
    ```c
    for (i = 0; i < 128; i++) read(0, buf+i, 1);
    ```
    Kẻ tấn công có thể ghi đè biến đếm `i` nằm trước Canary. Ví dụ, thiết lập `i` nhảy thẳng từ offset của biến cục bộ sang offset của `Return Address`, hoàn toàn bỏ qua vùng nhớ chứa Canary, khiến đoạn mã kiểm tra (Epilogue) vẫn nghĩ bộ nhớ hoàn toàn nguyên vẹn.

### 4. Kỹ Nghệ Phát Triển Mã Khai Thác (Exploit Engineering)

Đến lúc này, chúng ta cần sự chính xác tuyệt đối. Việc đoán "khoảng 100 chữ A là tràn" không có chỗ trong phát triển 0-day. Cấu trúc Stack thay đổi tùy thuộc vào cách trình biên dịch chèn padding. 

Để tính toán khoảng cách (offset) từ vị trí bắt đầu kiểm soát đầu vào đến mục tiêu (Canary, RBP, hoặc RIP), các kỹ sư bảo mật sử dụng hai phương pháp kết hợp:

*   **Dynamic Analysis (GDB & Ký ức động):**
    Viết exploit mà không debug giống như lái xe nhắm mắt. Việc sử dụng GDB (thường gắn kèm các công cụ hỗ trợ như GEF/pwndbg) cho phép đặt Breakpoint ngay trước lệnh `ret` hoặc `cmp` của Canary. Lệnh `vmmap` giúp xác nhận các vùng nhớ có quyền thực thi/đọc ghi, lệnh `x/gx rsp` giúp soi rõ cấu trúc Stack tại thời điểm crash. Việc chạy thử trên môi trường tĩnh sẽ khác xa lúc runtime do tác động của biến môi trường (Environment Variables) làm xô lệch Stack. Do đó, offset phải luôn được đo lường thực tế trên bộ nhớ đang chạy.
*   **Cyclic Values (Mẫu chuỗi De Bruijn):**
    Thay vì gửi một chuỗi toàn chữ 'A' và bối rối không biết mình đang ở vị trí nào khi thanh ghi `RIP` crash với giá trị `0x4141414141414141`, ta gửi một chuỗi tuần hoàn đặc biệt mà mỗi nhóm 4 hoặc 8 bytes là hoàn toàn duy nhất (Ví dụ: `aaaaabaaacaaadaaa...`). Khi GDB báo lỗi Segmentation Fault tại RIP = `0x61616164` (daaa), ta chỉ cần tra ngược giá trị này trong mẫu thuật toán để biết chính xác offset là bao nhiêu byte. Đây là quy chuẩn bắt buộc để căn chỉnh Stack chuẩn xác đến từng byte trong mọi cuộc tấn công.

---

## Phần 6: Cuộc Chiến Bảng Phân Giải - Chiếm Đoạt GOT (Global Offset Table)

Trong các module trước, chúng ta đã khai thác thông qua Stack. Nhưng nếu Stack được bảo vệ nghiêm ngặt (Canary không thể leak) hoặc lỗ hổng chúng ta có không phải là tràn bộ đệm tuyến tính mà là một lỗ hổng **Arbitrary Write (Ghi tùy ý tại một địa chỉ bất kỳ)**, mục tiêu béo bở nhất trong không gian bộ nhớ của chương trình chính là **Bảng GOT (Global Offset Table)**.

### 1. Nguồn Gốc Của Bảng GOT: Liên Kết Động (Dynamic Linking)

Hầu hết các chương trình C không tự mình chứa các hàm như `printf`, `read`, hay `puts`. Chúng mượn các hàm này từ một thư viện động chung (ví dụ: `libc.so`). Lợi ích là tiết kiệm dung lượng ổ cứng và RAM, vì hàng ngàn tiến trình có thể dùng chung một `libc` trong bộ nhớ. Tuy nhiên, điều này tạo ra một bài toán: Khi lập trình viên gọi `puts()`, trình biên dịch không hề biết địa chỉ thực sự của hàm `puts` nằm ở đâu trong `libc` tại thời điểm chạy (đặc biệt là khi có ASLR).

Hệ điều hành giải quyết bài toán này bằng cơ chế **Lazy Binding (Liên kết trễ)**. Nó dựa vào sự phối hợp của hai bảng nằm trong bộ nhớ của chương trình:
1.  **PLT (Procedure Linkage Table - Bảng Liên kết Thủ tục):** Chứa những "mã nhảy" (stub). Đây là nơi chương trình gọi đến thay vì gọi thẳng hàm. Mã này nằm trong phân vùng Code (`.text`) và có quyền Thực thi.
2.  **GOT (Global Offset Table - Bảng Phân giải Địa chỉ):** Một mảng chứa địa chỉ thực của các hàm. Ban đầu nó chứa địa chỉ "ảo", sau khi phân giải nó sẽ chứa địa chỉ thật trong `libc`.

### 2. Lazy Binding Hoạt Động Như Thế Nào (Dưới góc nhìn x64)?

Cơ chế Lazy Binding được thiết kế để tối ưu thời gian khởi động: Hàm nào thực sự được gọi thì mới phân giải địa chỉ. Hãy phân tích vòng đời của lệnh `puts("hello")` trong mã máy x64:

**Lần gọi `puts()` ĐẦU TIÊN (Trạng thái Pre-Resolution):**

Chương trình gọi đến `puts@plt`. Tại đây có một chuỗi lệnh nhảy gián tiếp.
```assembly
0x555555555030 <puts@plt>:
  jmp QWORD PTR [rip+0x2fca]  ; Nhảy đến địa chỉ đang được lưu trong bảng GOT của puts (puts@got.plt)
```
Tuy nhiên, vì đây là lần gọi đầu tiên, bảng GOT chưa có địa chỉ thật của `puts` trong `libc`. Thay vào đó, hệ điều hành cố tình để GOT trỏ về **chính dòng lệnh tiếp theo bên trong PLT**:

```assembly
0x555555558000 <puts@got.plt>: 0x555555555036 ; (Trỏ ngược lại vào puts@plt + 6)
```
Kết quả là, luồng thực thi vòng ngược lại vào PLT và tiếp tục chạy:
```assembly
0x555555555036 <puts@plt+6>:
  push 0x0                    ; Đẩy ID (reloc_index) của hàm puts lên Stack
  jmp  0x555555555020         ; Nhảy vào "Phù thủy" (PLT0 / Dynamic Linker)
```
Đoạn mã "Phù thủy" (Dynamic Linker/Resolver của hệ điều hành, thường là `_dl_runtime_resolve`) sẽ tính toán địa chỉ thực tế của `puts` trong bộ nhớ `libc`. Sau đó, **nó ghi đè địa chỉ thật này vào `puts@got.plt`** rồi nhảy đến hàm `puts`.

**Lần gọi `puts()` THỨ HAI (Trạng thái Post-Resolution):**

```assembly
0x555555555030 <puts@plt>:
  jmp QWORD PTR[rip+0x2fca]  ; Vẫn đọc địa chỉ từ GOT
```
Nhưng lúc này, GOT không còn trỏ về PLT nữa:
```assembly
0x555555558000 <puts@got.plt>: 0x00007ffff7a91da0 ; (Địa chỉ xịn của puts trong libc!)
```
Thanh ghi `RIP` nhảy trực tiếp vào `libc`. Rất tối ưu.

### 3. Vấn Đề Chết Người: Bảng GOT Phải Cho Phép Ghi (Writable)

Bởi vì Dynamic Linker cần ghi địa chỉ thật vào GOT *sau khi* chương trình đã khởi chạy, phân vùng chứa GOT (thường gọi là vùng `.got.plt`) **bắt buộc phải có quyền Đọc và Ghi (Read/Write)**.

Hãy soi vùng nhớ này bằng GDB:
```assembly
gdb> info proc mappings
Start addr          End addr            Perms 
0x555555557000      0x555555558000      rw-p   <-- Vùng nhớ chứa GOT!
```

Đây là gót chân Achilles của chương trình. GOT thực chất chỉ là một chuỗi các con trỏ hàm (Function Pointers) tập trung tại một nơi dễ đoán.

**Kịch bản khai thác (GOT Overwrite):**
Nếu kẻ tấn công có lỗ hổng **Arbitrary Write** (như Lỗi Format String `%n`, Out-of-Bounds Write với index lớn, hoặc Use-After-Free trên Heap), họ hoàn toàn không cần động đến Stack.
Họ chỉ cần ghi đè một giá trị (ví dụ: địa chỉ của hàm `system("/bin/sh")` hoặc một chuỗi ROP chain) vào vị trí của `puts@got.plt`. 
Khi luồng thực thi của chương trình tiếp tục và gọi `puts("Bất cứ thứ gì")` một cách ngây thơ, luồng thực thi của `RIP` sẽ nhảy thẳng vào cái bẫy `system` của kẻ tấn công. Quyền điều khiển rơi vào tay kẻ tấn công ngay lập tức.

### 4. Mitigation: RELRO (Relocation Read-Only)

Để vá lỗ hổng chí mạng của kiến trúc Lazy Binding, các nhà phát triển hệ điều hành đã đưa ra cơ chế bảo vệ **RELRO**. Nó có hai mức độ:

*   **Partial RELRO:** Vẫn duy trì Lazy Binding. Bảng `.got.plt` vẫn có quyền Ghi. Cơ chế này chỉ bảo vệ một số phân vùng biến tĩnh khác (`.got`, `.dynamic`), nên nó **không cản được tấn công GOT Overwrite**. (Đây là mặc định trên các hệ thống Linux đời cũ hoặc được biên dịch với tùy chọn lỏng lẻo).
*   **Full RELRO:** Chấm dứt hoàn toàn sự lười biếng.
    *   Khi chương trình khởi động, trước khi hàm `main()` được chạy, Dynamic Linker sẽ lùng sục toàn bộ bảng `.dynamic`, phân giải **tất cả** các hàm ngoại lai và điền sẵn địa chỉ thật của chúng vào GOT.
    *   Ngay sau đó, Loader dùng syscall `mprotect` để khóa vùng nhớ GOT, **hủy bỏ quyền Ghi (Write)** và biến nó thành Chỉ Đọc (Read-Only).

Kiểm tra lại bằng GDB khi có Full RELRO:
```assembly
gdb> vmmap 0x555555557fd0
Start addr          End addr            Perms
0x0000555555557000  0x0000555555558000  r--p   <-- Chỉ Đọc!
```

Nếu một phần mềm được biên dịch với cơ chế Full RELRO, cánh cửa GOT Overwrite sẽ đóng sập lại. Một Arbitrary Write vào GOT sẽ ngay lập tức gây ra lỗi Segmentation Fault. Lúc này, nhà nghiên cứu lỗ hổng sẽ phải chuyển hướng Arbitrary Write sang những mục tiêu khó nhằn hơn (ví dụ: các con trỏ hàm nội bộ của `libc` như `__free_hook`, `__malloc_hook`, TLS data, hoặc vdtors trong C++). Cuộc chiến mèo vờn chuột giữa Kẻ Tấn Công và Trình Biên Dịch vẫn tiếp tục ở một cấp độ tinh xảo hơn.
