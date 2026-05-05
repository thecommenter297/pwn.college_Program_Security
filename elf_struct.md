**Đây chỉ là bản dịch tiếng Việt bằng AI (có chỉnh sửa) của một cheatsheet viết bằng tiếng Anh.**

**Bài viết gốc**: https://gist.github.com/x0nu11byt3/bcb35c3de461e5fb66173071a2379779

**Tác giả bài viết gốc**: x0nu11byt3

---

## Giới thiệu

Executable and Linkable Format (ELF), là định dạng nhị phân mặc định trên các hệ thống dựa trên Linux.

<img width="680" height="1135" alt="image" src="https://github.com/user-attachments/assets/3be63c5c-a0ad-478c-9909-24f707a88395" />


## Quá trình biên dịch

<img width="690" height="205" alt="image" src="https://github.com/user-attachments/assets/2ea8bb3d-003d-4837-a14a-df8af372388b" />


## Header của file thực thi (Ehdr)

Đây là phần duy nhất của tệp ELF phải nằm ở một vị trí cụ thể (ở đầu tệp ELF).

Nó định nghĩa các thông tin cơ bản, chẳng hạn như số magic của tệp để biết một tệp có phải là ELF hay không.
Nó cũng định nghĩa loại ELF, kiến trúc và một số tùy chọn sẽ liên kết nó với các phần khác của tệp ELF.

Cấu trúc 32-bit:

```c
#define EI_NIDENT (16)

typedef struct
{
  unsigned char	e_ident[EI_NIDENT];	/* Số magic và thông tin khác */
  Elf32_Half	e_type;			/* Loại tệp đối tượng */
  Elf32_Half	e_machine;		/* Kiến trúc */
  Elf32_Word	e_version;		/* Phiên bản tệp đối tượng */
  Elf32_Addr	e_entry;		/* Địa chỉ ảo của điểm vào */
  Elf32_Off	e_phoff;		/* Độ lệch của bảng program header */
  Elf32_Off	e_shoff;		/* Độ lệch của bảng section header */
  Elf32_Word	e_flags;		/* Cờ dành riêng cho bộ xử lý */
  Elf32_Half	e_ehsize;		/* Kích thước header ELF (bytes) */
  Elf32_Half	e_phentsize;		/* Kích thước một entry trong bảng program header */
  Elf32_Half	e_phnum;		/* Số lượng entry trong bảng program header */
  Elf32_Half	e_shentsize;		/* Kích thước một entry trong bảng section header */
  Elf32_Half	e_shnum;		/* Số lượng entry trong bảng section header */
  Elf32_Half	e_shstrndx;		/* Chỉ số của bảng chuỗi tên section */
} Elf32_Ehdr;
```

Cấu trúc 64-bit:

```c
typedef struct
{
  unsigned char	e_ident[EI_NIDENT];	/* Số magic và thông tin khác */
  Elf64_Half	e_type;			/* Loại tệp đối tượng */
  Elf64_Half	e_machine;		/* Kiến trúc */
  Elf64_Word	e_version;		/* Phiên bản tệp đối tượng */
  Elf64_Addr	e_entry;		/* Địa chỉ ảo của điểm vào */
  Elf64_Off	e_phoff;		/* Độ lệch của bảng program header */
  Elf64_Off	e_shoff;		/* Độ lệch của bảng section header */
  Elf64_Word	e_flags;		/* Cờ dành riêng cho bộ xử lý */
  Elf64_Half	e_ehsize;		/* Kích thước header ELF (bytes) */
  Elf64_Half	e_phentsize;		/* Kích thước một entry trong bảng program header */
  Elf64_Half	e_phnum;		/* Số lượng entry trong bảng program header */
  Elf64_Half	e_shentsize;		/* Kích thước một entry trong bảng section header */
  Elf64_Half	e_shnum;		/* Số lượng entry trong bảng section header */
  Elf64_Half	e_shstrndx;		/* Chỉ số của bảng chuỗi tên section */
} Elf64_Ehdr;
```

`EI_NIDENT` là kích thước (tính bằng byte) của mục đầu tiên trong cấu trúc, `e_type`.

Nó là các header magic của ELF và một số thông số kỹ thuật cơ bản của tệp.

Giá trị:

-   `e_ident`: Là một mảng 16 byte xác định đối tượng ELF, luôn bắt đầu bằng `\x7fELF`.
-   `e_type`: Chỉ định loại ELF:
    -   `ET_NONE` (Không xác định): Định dạng ELF không xác định hoặc không được chỉ định.
    -   `ET_EXEC`: (Tệp thực thi): Một tệp thực thi ELF.
    -   `ET_DYN`: (Đối tượng chia sẻ): Một thư viện hoặc một tệp thực thi được liên kết động.
    -   `ET_REL` (Tệp tái định vị): Tệp tái định vị (tệp đối tượng `.o`).
    -   `ET_CORE` (Core dump): Một tệp core dump.
-   `e_machine`: Kiến trúc đích.
-   `e_version`: Phiên bản tệp ELF.
-   `e_entry`: Địa chỉ điểm vào (entry point).
-   `e_phoff`: Độ lệch của Phdr (Program Header Table).
-   `e_shoff`: Độ lệch của Shdr (Section Header Table).
-   `e_flags`: Các cờ dành riêng cho bộ xử lý.
-   `e_ehsize`: Kích thước của Ehdr (tính bằng byte). (Thường là 64 byte trong ELF 64-bit và 52 byte cho 32-bit)
-   `e_phentsize`: Kích thước của một mục Phdr.
-   `e_phnum`: Số lượng mục Phdr.
-   `e_shentsize`: Kích thước của một mục Shdr.
-   `e_shnum`: Số lượng mục Shdr.
-   `e_shstrndx`: Chỉ số của bảng chuỗi tên section (`.shstrtab`, chứa các chuỗi kết thúc bằng null với tên của mỗi section)

Lưu ý: `e_phoff` và `e_shoff` là các độ lệch trong tệp ELF, trong khi `e_entry` là một địa chỉ ảo.

---- Các định nghĩa kiểu cần thiết ----

Định nghĩa `e_type`:

```c
#define ET_NONE		0		/* Không có loại tệp */
#define ET_REL		1		/* Tệp tái định vị */
#define ET_EXEC		2		/* Tệp thực thi */
#define ET_DYN		3		/* Tệp đối tượng chia sẻ */
#define ET_CORE		4		/* Tệp Core */
#define	ET_NUM		5		/* Số lượng các loại đã xác định */
#define ET_LOOS		0xfe00		/* Bắt đầu phạm vi dành riêng cho HĐH */
#define ET_HIOS		0xfeff		/* Kết thúc phạm vi dành riêng cho HĐH */
#define ET_LOPROC	0xff00		/* Bắt đầu phạm vi dành riêng cho bộ xử lý */
#define ET_HIPROC	0xffff		/* Kết thúc phạm vi dành riêng cho bộ xử lý */
```

Định nghĩa `e_machine`:

```c
#define EM_NONE		 0	/* Không có máy nào */
#define EM_M32		 1	/* AT&T WE 32100 */
#define EM_SPARC	 2	/* SUN SPARC */
#define EM_386		 3	/* Intel 80386 */
#define EM_68K		 4	/* Họ Motorola m68k */
#define EM_88K		 5	/* Họ Motorola m88k */
#define EM_IAMCU	 6	/* Intel MCU */
#define EM_860		 7	/* Intel 80860 */
#define EM_MIPS		 8	/* MIPS R3000 big-endian */
#define EM_S370		 9	/* IBM System/370 */
#define EM_MIPS_RS3_LE	10	/* MIPS R3000 little-endian */
				/* dành riêng 11-14 */
#define EM_PARISC	15	/* HPPA */
				/* dành riêng 16 */
#define EM_VPP500	17	/* Fujitsu VPP500 */
#define EM_SPARC32PLUS	18	/* Sun "v8plus" */
#define EM_960		19	/* Intel 80960 */
#define EM_PPC		20	/* PowerPC */
#define EM_PPC64	21	/* PowerPC 64-bit */
#define EM_S390		22	/* IBM S390 */
#define EM_SPU		23	/* IBM SPU/SPC */
				/* dành riêng 24-35 */
#define EM_V800		36	/* Dòng NEC V800 */
#define EM_FR20		37	/* Fujitsu FR20 */
#define EM_RH32		38	/* TRW RH-32 */
#define EM_RCE		39	/* Motorola RCE */
#define EM_ARM		40	/* ARM */
#define EM_FAKE_ALPHA	41	/* Digital Alpha */
#define EM_SH		42	/* Hitachi SH */
#define EM_SPARCV9	43	/* SPARC v9 64-bit */
#define EM_TRICORE	44	/* Siemens Tricore */
#define EM_ARC		45	/* Argonaut RISC Core */
#define EM_H8_300	46	/* Hitachi H8/300 */
#define EM_H8_300H	47	/* Hitachi H8/300H */
#define EM_H8S		48	/* Hitachi H8S */
#define EM_H8_500	49	/* Hitachi H8/500 */
#define EM_IA_64	50	/* Intel Merced */
#define EM_MIPS_X	51	/* Stanford MIPS-X */
#define EM_COLDFIRE	52	/* Motorola Coldfire */
#define EM_68HC12	53	/* Motorola M68HC12 */
#define EM_MMA		54	/* Fujitsu MMA Multimedia Accelerator */
#define EM_PCP		55	/* Siemens PCP */
#define EM_NCPU		56	/* Sony nCPU embeeded RISC */
#define EM_NDR1		57	/* Denso NDR1 microprocessor */
#define EM_STARCORE	58	/* Motorola Start*Core processor */
#define EM_ME16		59	/* Toyota ME16 processor */
#define EM_ST100	60	/* STMicroelectronic ST100 processor */
#define EM_TINYJ	61	/* Advanced Logic Corp. Tinyj emb.fam */
#define EM_X86_64	62	/* Kiến trúc AMD x86-64 */
#define EM_PDSP		63	/* Sony DSP Processor */
#define EM_PDP10	64	/* Digital PDP-10 */
#define EM_PDP11	65	/* Digital PDP-11 */
#define EM_FX66		66	/* Siemens FX66 microcontroller */
#define EM_ST9PLUS	67	/* STMicroelectronics ST9+ 8/16 mc */
#define EM_ST7		68	/* STmicroelectronics ST7 8 bit mc */
#define EM_68HC16	69	/* Motorola MC68HC16 microcontroller */
#define EM_68HC11	70	/* Motorola MC68HC11 microcontroller */
#define EM_68HC08	71	/* Motorola MC68HC08 microcontroller */
#define EM_68HC05	72	/* Motorola MC68HC05 microcontroller */
#define EM_SVX		73	/* Silicon Graphics SVx */
#define EM_ST19		74	/* STMicroelectronics ST19 8 bit mc */
#define EM_VAX		75	/* Digital VAX */
#define EM_CRIS		76	/* Axis Communications 32-bit emb.proc */
#define EM_JAVELIN	77	/* Infineon Technologies 32-bit emb.proc */
#define EM_FIREPATH	78	/* Element 14 64-bit DSP Processor */
#define EM_ZSP		79	/* LSI Logic 16-bit DSP Processor */
#define EM_MMIX		80	/* Donald Knuth's educational 64-bit proc */
#define EM_HUANY	81	/* Harvard University machine-independent object files */
#define EM_PRISM	82	/* SiTera Prism */
#define EM_AVR		83	/* Atmel AVR 8-bit microcontroller */
#define EM_FR30		84	/* Fujitsu FR30 */
#define EM_D10V		85	/* Mitsubishi D10V */
#define EM_D30V		86	/* Mitsubishi D30V */
#define EM_V850		87	/* NEC v850 */
#define EM_M32R		88	/* Mitsubishi M32R */
#define EM_MN10300	89	/* Matsushita MN10300 */
#define EM_MN10200	90	/* Matsushita MN10200 */
#define EM_PJ		91	/* picoJava */
#define EM_OPENRISC	92	/* OpenRISC 32-bit embedded processor */
#define EM_ARC_COMPACT	93	/* ARC International ARCompact */
#define EM_XTENSA	94	/* Tensilica Xtensa Architecture */
#define EM_VIDEOCORE	95	/* Alphamosaic VideoCore */
#define EM_TMM_GPP	96	/* Thompson Multimedia General Purpose Proc */
#define EM_NS32K	97	/* National Semi. 32000 */
#define EM_TPC		98	/* Tenor Network TPC */
#define EM_SNP1K	99	/* Trebia SNP 1000 */
#define EM_ST200	100	/* STMicroelectronics ST200 */
#define EM_IP2K		101	/* Ubicom IP2xxx */
#define EM_MAX		102	/* MAX processor */
#define EM_CR		103	/* National Semi. CompactRISC */
#define EM_F2MC16	104	/* Fujitsu F2MC16 */
#define EM_MSP430	105	/* Texas Instruments msp430 */
#define EM_BLACKFIN	106	/* Analog Devices Blackfin DSP */
#define EM_SE_C33	107	/* Họ Seiko Epson S1C33 */
#define EM_SEP		108	/* Sharp embedded microprocessor */
#define EM_ARCA		109	/* Arca RISC */
#define EM_UNICORE	110	/* PKU-Unity & MPRC Peking Uni. mc series */
#define EM_EXCESS	111	/* eXcess configurable cpu */
#define EM_DXP		112	/* Icera Semi. Deep Execution Processor */
#define EM_ALTERA_NIOS2 113	/* Altera Nios II */
#define EM_CRX		114	/* National Semi. CompactRISC CRX */
#define EM_XGATE	115	/* Motorola XGATE */
#define EM_C166		116	/* Infineon C16x/XC16x */
#define EM_M16C		117	/* Renesas M16C */
#define EM_DSPIC30F	118	/* Microchip Technology dsPIC30F */
#define EM_CE		119	/* Freescale Communication Engine RISC */
#define EM_M32C		120	/* Renesas M32C */
				/* dành riêng 121-130 */
#define EM_TSK3000	131	/* Altium TSK3000 */
#define EM_RS08		132	/* Freescale RS08 */
#define EM_SHARC	133	/* Họ Analog Devices SHARC */
#define EM_ECOG2	134	/* Cyan Technology eCOG2 */
#define EM_SCORE7	135	/* Sunplus S+core7 RISC */
#define EM_DSP24	136	/* New Japan Radio (NJR) 24-bit DSP */
#define EM_VIDEOCORE3	137	/* Broadcom VideoCore III */
#define EM_LATTICEMICO32 138	/* RISC cho Lattice FPGA */
#define EM_SE_C17	139	/* Seiko Epson C17 */
#define EM_TI_C6000	140	/* Texas Instruments TMS320C6000 DSP */
#define EM_TI_C2000	141	/* Texas Instruments TMS320C2000 DSP */
#define EM_TI_C5500	142	/* Texas Instruments TMS320C55x DSP */
#define EM_TI_ARP32	143	/* Texas Instruments App. Specific RISC */
#define EM_TI_PRU	144	/* Texas Instruments Prog. Realtime Unit */
				/* dành riêng 145-159 */
#define EM_MMDSP_PLUS	160	/* STMicroelectronics 64bit VLIW DSP */
#define EM_CYPRESS_M8C	161	/* Cypress M8C */
#define EM_R32C		162	/* Renesas R32C */
#define EM_TRIMEDIA	163	/* NXP Semi. TriMedia */
#define EM_QDSP6	164	/* QUALCOMM DSP6 */
#define EM_8051		165	/* Intel 8051 và các biến thể */
#define EM_STXP7X	166	/* STMicroelectronics STxP7x */
#define EM_NDS32	167	/* Andes Tech. compact code emb. RISC */
#define EM_ECOG1X	168	/* Cyan Technology eCOG1X */
#define EM_MAXQ30	169	/* Dallas Semi. MAXQ30 mc */
#define EM_XIMO16	170	/* New Japan Radio (NJR) 16-bit DSP */
#define EM_MANIK	171	/* M2000 Reconfigurable RISC */
#define EM_CRAYNV2	172	/* Cray NV2 vector architecture */
#define EM_RX		173	/* Renesas RX */
#define EM_METAG	174	/* Imagination Tech. META */
#define EM_MCST_ELBRUS	175	/* MCST Elbrus */
#define EM_ECOG16	176	/* Cyan Technology eCOG16 */
#define EM_CR16		177	/* National Semi. CompactRISC CR16 */
#define EM_ETPU		178	/* Freescale Extended Time Processing Unit */
#define EM_SLE9X	179	/* Infineon Tech. SLE9X */
#define EM_L10M		180	/* Intel L10M */
#define EM_K10M		181	/* Intel K10M */
				/* dành riêng 182 */
#define EM_AARCH64	183	/* ARM AARCH64 */
				/* dành riêng 184 */
#define EM_AVR32	185	/* Amtel 32-bit microprocessor */
#define EM_STM8		186	/* STMicroelectronics STM8 */
#define EM_TILE64	187	/* Tileta TILE64 */
#define EM_TILEPRO	188	/* Tilera TILEPro */
#define EM_MICROBLAZE	189	/* Xilinx MicroBlaze */
#define EM_CUDA		190	/* NVIDIA CUDA */
#define EM_TILEGX	191	/* Tilera TILE-Gx */
#define EM_CLOUDSHIELD	192	/* CloudShield */
#define EM_COREA_1ST	193	/* KIPO-KAIST Core-A thế hệ 1 */
#define EM_COREA_2ND	194	/* KIPO-KAIST Core-A thế hệ 2 */
#define EM_ARC_COMPACT2	195	/* Synopsys ARCompact V2 */
#define EM_OPEN8	196	/* Open8 RISC */
#define EM_RL78		197	/* Renesas RL78 */
#define EM_VIDEOCORE5	198	/* Broadcom VideoCore V */
#define EM_78KOR	199	/* Renesas 78KOR */
#define EM_56800EX	200	/* Freescale 56800EX DSC */
#define EM_BA1		201	/* Beyond BA1 */
#define EM_BA2		202	/* Beyond BA2 */
#define EM_XCORE	203	/* XMOS xCORE */
#define EM_MCHP_PIC	204	/* Microchip 8-bit PIC(r) */
				/* dành riêng 205-209 */
#define EM_KM32		210	/* KM211 KM32 */
#define EM_KMX32	211	/* KM211 KMX32 */
#define EM_EMX16	212	/* KM211 KMX16 */
#define EM_EMX8		213	/* KM211 KMX8 */
#define EM_KVARC	214	/* KM211 KVARC */
#define EM_CDP		215	/* Paneve CDP */
#define EM_COGE		216	/* Cognitive Smart Memory Processor */
#define EM_COOL		217	/* Bluechip CoolEngine */
#define EM_NORC		218	/* Nanoradio Optimized RISC */
#define EM_CSR_KALIMBA	219	/* CSR Kalimba */
#define EM_Z80		220	/* Zilog Z80 */
#define EM_VISIUM	221	/* Controls and Data Services VISIUMcore */
#define EM_FT32		222	/* FTDI Chip FT32 */
#define EM_MOXIE	223	/* Moxie processor */
#define EM_AMDGPU	224	/* AMD GPU */
				/* dành riêng 225-242 */
#define EM_RISCV	243	/* RISC-V */
#define EM_BPF		247	/* Linux BPF -- máy ảo trong kernel */
#define EM_CSKY		252     /* C-SKY */
#define EM_NUM		253
/* Tên cũ/từ đồng nghĩa.  */
#define EM_ARC_A5	EM_ARC_COMPACT
/* Nếu cần gán các giá trị EM_* không chính thức mới,
   vui lòng chọn các số ngẫu nhiên lớn (0x8523, 0xa7f2, v.v.) để giảm thiểu
   khả năng xung đột với các giá trị chính thức hoặc không chính thức khác.  */
#define EM_ALPHA	0x9026
```

Định nghĩa `e_version`:

```c
#define EV_NONE		0		/* Phiên bản ELF không hợp lệ */
#define EV_CURRENT	1		/* Phiên bản hiện tại */
#define EV_NUM		2
```

## Section Headers (Shdr)

Mã và dữ liệu được chia thành các khối liền kề không chồng chéo được gọi là các section (phần).

Nó chỉ là một không gian để lưu trữ dữ liệu hoặc mã, các thông số kỹ thuật của nó nằm trong một section header chỉ định các chi tiết cần thiết như kích thước và độ lệch.

Mỗi section có một section header định nghĩa nó.

Cấu trúc 32-bit:

```c
typedef struct
{
  Elf32_Word	sh_name;		/* Tên section (chỉ số bảng chuỗi) */
  Elf32_Word	sh_type;		/* Loại section */
  Elf32_Word	sh_flags;		/* Cờ section */
  Elf32_Addr	sh_addr;		/* Địa chỉ ảo của section khi thực thi */
  Elf32_Off	sh_offset;		/* Độ lệch của section trong tệp */
  Elf32_Word	sh_size;		/* Kích thước section (bytes) */
  Elf32_Word	sh_link;		/* Liên kết đến section khác */
  Elf32_Word	sh_info;		/* Thông tin bổ sung về section */
  Elf32_Word	sh_addralign;		/* Căn chỉnh section */
  Elf32_Word	sh_entsize;		/* Kích thước entry nếu section chứa bảng */
} Elf32_Shdr;
```

Cấu trúc 64-bit:

```c
typedef struct
{
  Elf64_Word	sh_name;		/* Tên section (chỉ số bảng chuỗi) */
  Elf64_Word	sh_type;		/* Loại section */
  Elf64_Xword	sh_flags;		/* Cờ section */
  Elf64_Addr	sh_addr;		/* Địa chỉ ảo của section khi thực thi */
  Elf64_Off	sh_offset;		/* Độ lệch của section trong tệp */
  Elf64_Xword	sh_size;		/* Kích thước section (bytes) */
  Elf64_Word	sh_link;		/* Liên kết đến section khác */
  Elf64_Word	sh_info;		/* Thông tin bổ sung về section */
  Elf64_Xword	sh_addralign;		/* Căn chỉnh section */
  Elf64_Xword	sh_entsize;		/* Kích thước entry nếu section chứa bảng */
} Elf64_Shdr;
```

Giá trị:
*   `sh_name`: Chỉ số vào bảng chuỗi, nếu bằng 0 có nghĩa là nó không có tên. (`.shstrtab`).
*   `sh_type`: Loại section.
    *   `SHT_NULL`: Mục trong bảng section không được sử dụng.
    *   `SHT_PROGBITS`: Dữ liệu chương trình (Chẳng hạn như lệnh máy hoặc hằng số).
    *   `SHT_SYMTAB`: Bảng ký hiệu. (Bảng ký hiệu tĩnh)
    *   `SHT_STRTAB`: Bảng chuỗi.
    *   `SHT_RELA`: Các mục tái định vị với addend.
    *   `SHT_HASH`: Bảng băm ký hiệu.
    *   `SHT_DYNAMIC`: Thông tin liên kết động.
    *   `SHT_NOTE`: Ghi chú.
    *   `SHT_NOBITS`: Dữ liệu chưa được khởi tạo.
    *   `SHT_REL`: Các mục tái định vị không có addend.
    *   `SHT_SHLIB`: Dành riêng.
    *   `SHT_DYNSYM`: Bảng ký hiệu của trình liên kết động. (Bảng ký hiệu được sử dụng bởi trình liên kết động)
*   `sh_flags`: Mô tả thông tin bổ sung về một section.
    *   `SHF_WRITE`: Có thể ghi vào lúc chạy.
    *   `SHF_ALLOC`: Section sẽ được nạp vào bộ nhớ ảo lúc chạy.
    *   `SHF_EXECINSTR`: Chứa các lệnh có thể thực thi.
*   `sh_addr`: Địa chỉ ảo của section khi thực thi.
*   `sh_offset`: Độ lệch của section trong tệp ELF.
*   `sh_size`: Kích thước của section (tính bằng byte).
*   `sh_link`: Liên kết đến một section khác (Ví dụ: `SHT_SYMTAB`, `SHT_DYNSYM`, hoặc `SHT_DYNAMIC` có một bảng chuỗi liên quan chứa tên tượng trưng cho các ký hiệu được đề cập. Các section tái định vị (loại `SHT_REL` hoặc `SHT_RELA`) được liên kết với một bảng ký hiệu mô tả các ký hiệu liên quan đến việc tái định vị.).
*   `sh_info`: Thông tin bổ sung về section.
*   `sh_addralign`: Căn chỉnh section.
*   `sh_entsize`: Kích thước entry nếu section chứa bảng. (Một số section, chẳng hạn như bảng ký hiệu hoặc bảng tái định vị, chứa một bảng các cấu trúc dữ liệu được xác định rõ (chẳng hạn như `ElfN_Sym` hoặc `ElfN_Rela`). Đối với các section như vậy, trường `sh_entsize` cho biết kích thước tính bằng byte của mỗi entry trong bảng. Khi trường này không được sử dụng, nó được đặt thành không).

Tất cả các section header định nghĩa các section đều được chứa trong bảng section header.

Để nạp và thực thi một tệp nhị phân trong một tiến trình, bạn cần một cách tổ chức mã và dữ liệu khác trong tệp nhị phân. Vì lý do này, các tệp thực thi ELF chỉ định một tổ chức logic khác, được gọi là các segment, được sử dụng tại thời điểm thực thi (trái ngược với các section, được sử dụng tại thời điểm liên kết).

Các section là tùy chọn, chúng chỉ là siêu dữ liệu cho các trình gỡ lỗi. Các program header mới là thứ quyết định cách một tệp nhị phân ELF được nạp vào bộ nhớ.

Do đó, các section header không được nạp vào bộ nhớ.

---- Định nghĩa kiểu ----

Định nghĩa `sh_type`:

```c
#define SHT_NULL	  0		/* Mục header section không sử dụng */
#define SHT_PROGBITS	  1		/* Dữ liệu chương trình */
#define SHT_SYMTAB	  2		/* Bảng ký hiệu */
#define SHT_STRTAB	  3		/* Bảng chuỗi */
#define SHT_RELA	  4		/* Các entry tái định vị có addend */
#define SHT_HASH	  5		/* Bảng băm ký hiệu */
#define SHT_DYNAMIC	  6		/* Thông tin liên kết động */
#define SHT_NOTE	  7		/* Ghi chú */
#define SHT_NOBITS	  8		/* Không gian chương trình không có dữ liệu (bss) */
#define SHT_REL		  9		/* Các entry tái định vị, không có addend */
#define SHT_SHLIB	  10		/* Dành riêng */
#define SHT_DYNSYM	  11		/* Bảng ký hiệu của trình liên kết động */
#define SHT_INIT_ARRAY	  14		/* Mảng các hàm khởi tạo */
#define SHT_FINI_ARRAY	  15		/* Mảng các hàm hủy */
#define SHT_PREINIT_ARRAY 16		/* Mảng các hàm tiền khởi tạo */
#define SHT_GROUP	  17		/* Nhóm section */
#define SHT_SYMTAB_SHNDX  18		/* Chỉ số section mở rộng */
#define	SHT_NUM		  19		/* Số lượng các loại được định nghĩa.  */
#define SHT_LOOS	  0x60000000	/* Bắt đầu dành riêng cho HĐH.  */
#define SHT_GNU_ATTRIBUTES 0x6ffffff5	/* Thuộc tính đối tượng.  */
#define SHT_GNU_HASH	  0x6ffffff6	/* Bảng băm kiểu GNU.  */
#define SHT_GNU_LIBLIST	  0x6ffffff7	/* Danh sách thư viện prelink */
#define SHT_CHECKSUM	  0x6ffffff8	/* Checksum cho nội dung DSO.  */
#define SHT_LOSUNW	  0x6ffffffa	/* Giới hạn dưới dành riêng cho Sun.  */
#define SHT_SUNW_move	  0x6ffffffa
#define SHT_SUNW_COMDAT   0x6ffffffb
#define SHT_SUNW_syminfo  0x6ffffffc
#define SHT_GNU_verdef	  0x6ffffffd	/* Section định nghĩa phiên bản.  */
#define SHT_GNU_verneed	  0x6ffffffe	/* Section yêu cầu phiên bản.  */
#define SHT_GNU_versym	  0x6fffffff	/* Bảng ký hiệu phiên bản.  */
#define SHT_HISUNW	  0x6fffffff	/* Giới hạn trên dành riêng cho Sun.  */
#define SHT_HIOS	  0x6fffffff	/* Kết thúc loại dành riêng cho HĐH */
#define SHT_LOPROC	  0x70000000	/* Bắt đầu dành riêng cho bộ xử lý */
#define SHT_HIPROC	  0x7fffffff	/* Kết thúc dành riêng cho bộ xử lý */
#define SHT_LOUSER	  0x80000000	/* Bắt đầu dành riêng cho ứng dụng */
#define SHT_HIUSER	  0x8fffffff	/* Kết thúc dành riêng cho ứng dụng */
```

Định nghĩa `sh_flags`:

```c
#define SHF_WRITE	     (1 << 0)	/* Có thể ghi */
#define SHF_ALLOC	     (1 << 1)	/* Chiếm bộ nhớ trong quá trình thực thi */
#define SHF_EXECINSTR	     (1 << 2)	/* Có thể thực thi */
#define SHF_MERGE	     (1 << 4)	/* Có thể được hợp nhất */
#define SHF_STRINGS	     (1 << 5)	/* Chứa các chuỗi kết thúc bằng null */
#define SHF_INFO_LINK	     (1 << 6)	/* `sh_info' chứa chỉ số SHT */
#define SHF_LINK_ORDER	     (1 << 7)	/* Giữ nguyên thứ tự sau khi kết hợp */
#define SHF_OS_NONCONFORMING (1 << 8)	/* Yêu cầu xử lý dành riêng cho HĐH không chuẩn */
#define SHF_GROUP	     (1 << 9)	/* Section là thành viên của một nhóm.  */
#define SHF_TLS		     (1 << 10)	/* Section chứa dữ liệu cục bộ của luồng.  */
#define SHF_COMPRESSED	     (1 << 11)	/* Section với dữ liệu nén. */
#define SHF_MASKOS	     0x0ff00000	/* Dành riêng cho HĐH.  */
#define SHF_MASKPROC	     0xf0000000	/* Dành riêng cho bộ xử lý */
#define SHF_ORDERED	     (1 << 30)	/* Yêu cầu sắp xếp đặc biệt (Solaris).  */
#define SHF_EXCLUDE	     (1U << 31)	/* Section bị loại trừ trừ khi được tham chiếu hoặc cấp phát (Solaris).*/
```

## Các Section (Phần)

Mục đầu tiên trong bảng section header của mọi tệp ELF được tiêu chuẩn ELF định nghĩa là một mục NULL. Loại của mục này là `SHT_NULL`, và tất cả các trường trong section header đều được đặt về không.

Các Section:
*   `.init`: Mã thực thi thực hiện các tác vụ khởi tạo và cần chạy trước bất kỳ mã nào khác trong tệp nhị phân được thực thi (Do đó nó có cờ `SHF_EXECINSTR`). Hệ thống thực thi mã trong section `.init` trước khi chuyển quyền kiểm soát đến điểm vào chính của tệp nhị phân.
*   `.fini`: Ngược lại với `.init`, nó có mã thực thi phải chạy sau khi chương trình chính hoàn thành.
*   `.text`: Là nơi chứa mã chính của chương trình (Do đó nó có cờ `SHF_EXECINSTR`), nó là `SHT_PROGBITS` vì nó có mã do người dùng định nghĩa.
*   `.bss`: Nó chứa dữ liệu chưa được khởi tạo (Loại `SHT_NOBITS`). Nó không chiếm dung lượng trên đĩa để tránh tốn dung lượng, sau đó tất cả dữ liệu thường được khởi tạo thành không tại thời điểm chạy. Nó có thể ghi.
*   `.data`: Dữ liệu được khởi tạo của chương trình, nó có thể ghi. (Loại `SHT_PROGBITS`).
*   `.rodata`: Dữ liệu chỉ đọc, chẳng hạn như các chuỗi được sử dụng bởi mã, nếu dữ liệu cần có thể ghi thì `.data` được sử dụng thay thế. Dữ liệu được đặt ở đây có thể là các chuỗi được mã hóa cứng được sử dụng cho một `printf`.
*   `.plt`: Viết tắt của Procedure Linkage Table (Bảng Liên kết Thủ tục). Nó là mã được sử dụng cho mục đích liên kết động giúp gọi các hàm bên ngoài từ các thư viện chia sẻ với sự trợ giúp của GOT (Global Offset Table).
*   `.got.plt`: Nó là một bảng nơi các địa chỉ đã được giải quyết từ các hàm bên ngoài được lưu trữ. Nó mặc định có thể ghi vì Lazy Binding (Liên kết lười) được sử dụng theo mặc định. (Trừ khi Relocation Read-Only được sử dụng hoặc biến môi trường LD_BIND_NOW được xuất để giải quyết tất cả các hàm được nhập vào lúc khởi tạo chương trình).
*   `.rel.*`: Chứa thông tin về cách các phần của một đối tượng ELF hoặc hình ảnh tiến trình cần được sửa chữa hoặc sửa đổi tại thời điểm liên kết hoặc thời gian chạy (Loại `SHT_REL`).
*   `.rela.*`: Chứa thông tin về cách các phần của một đối tượng ELF hoặc hình ảnh tiến trình cần được sửa chữa hoặc sửa đổi tại thời điểm liên kết hoặc thời gian chạy (với addend) (Loại `SHT_RELA`).
*   `.dynamic`: Các cấu trúc và đối tượng liên kết động. Chứa một bảng các cấu trúc `ElfN_Dyn`. Cũng chứa các con trỏ đến các thông tin quan trọng khác được yêu cầu bởi trình liên kết động (ví dụ: bảng chuỗi động, bảng ký hiệu động, section `.got.plt`, và section tái định vị động được trỏ đến bởi các thẻ loại `DT_STRTAB`, `DT_SYMTAB`, `DT_PLTGOT`, và `DT_RELA`, tương ứng).
*   `.init_array`: Chứa một mảng các con trỏ đến các hàm để sử dụng làm hàm khởi tạo (mỗi hàm này được gọi lần lượt khi tệp nhị phân được khởi tạo). Trong `gcc`, bạn có thể đánh dấu các hàm trong tệp nguồn C của mình là hàm khởi tạo bằng cách trang trí chúng bằng `__attribute__((constructor))`. Theo mặc định, có một mục trong `.init_array` để thực thi `frame_dummy`.
*   `.fini_array`: Chứa một mảng các con trỏ đến các hàm để sử dụng làm hàm hủy.
*   `.shstrtab`: Đơn giản là một mảng các chuỗi kết thúc bằng NULL chứa tên của tất cả các section trong tệp nhị phân.
*   `.symtab`: Chứa một bảng ký hiệu, là một bảng các cấu trúc `ElfN_Sym`, mỗi cấu trúc liên kết một tên tượng trưng với một đoạn mã hoặc dữ liệu ở nơi khác trong tệp nhị phân, chẳng hạn như một hàm hoặc biến.
*   `.strtab`: Chứa các chuỗi chứa các tên tượng trưng. Các chuỗi này được trỏ đến bởi các cấu trúc `ElfN_Sym`.
*   `.dynsym`: Giống như `.symtab` nhưng chứa các ký hiệu cần thiết cho liên kết động thay vì liên kết tĩnh.
*   `.dynstr`: Giống như `.strtab` nhưng chứa các chuỗi cần thiết cho liên kết động thay vì liên kết tĩnh.
*   `.interp`: Chuỗi nhúng RTLD (Run-Time Linker).
*   `.rel.dyn`: Bảng tái định vị biến toàn cục.
*   `.rel.plt`: Bảng tái định vị hàm.

Các section của phiên bản `gcc` cũ hơn:
*   `.ctors`: Tương đương với `.init_array` được tạo ra bởi các phiên bản `gcc` cũ hơn.
*   `.dtors`: Tương đương với `.fini_array` được tạo ra bởi các phiên bản `gcc` cũ hơn.

## Program Headers (Phdr)

Bảng program header cung cấp một cái nhìn theo segment của tệp nhị phân, trái ngược với cái nhìn theo section được cung cấp bởi bảng section header. Cái nhìn theo section của một tệp nhị phân ELF chỉ dành cho mục đích liên kết tĩnh.

Ngược lại, cái nhìn theo segment được hệ điều hành và trình liên kết động sử dụng khi nạp một tệp ELF vào một tiến trình để thực thi nhằm xác định vị trí mã và dữ liệu liên quan và quyết định những gì sẽ được nạp vào bộ nhớ ảo.

Các segment cung cấp một cái nhìn thực thi, chúng chỉ cần thiết cho các tệp ELF có thể thực thi chứ không phải cho các tệp không thể thực thi như các đối tượng tái định vị.

Cấu trúc 32-bit:

```c
typedef struct
{
  Elf32_Word	p_type;			/* Loại segment */
  Elf32_Off	p_offset;		/* Độ lệch của segment trong tệp */
  Elf32_Addr	p_vaddr;		/* Địa chỉ ảo của segment */
  Elf32_Addr	p_paddr;		/* Địa chỉ vật lý của segment */
  Elf32_Word	p_filesz;		/* Kích thước segment trong tệp */
  Elf32_Word	p_memsz;		/* Kích thước segment trong bộ nhớ */
  Elf32_Word	p_flags;		/* Cờ của segment */
  Elf32_Word	p_align;		/* Căn chỉnh của segment */
} Elf32_Phdr;
```

Cấu trúc 64-bit:

```c
typedef struct
{
  Elf64_Word	p_type;			/* Loại segment */
  Elf64_Word	p_flags;		/* Cờ của segment */
  Elf64_Off	p_offset;		/* Độ lệch của segment trong tệp */
  Elf64_Addr	p_vaddr;		/* Địa chỉ ảo của segment */
  Elf64_Addr	p_paddr;		/* Địa chỉ vật lý của segment */
  Elf64_Xword	p_filesz;		/* Kích thước segment trong tệp */
  Elf64_Xword	p_memsz;		/* Kích thước segment trong bộ nhớ */
  Elf64_Xword	p_align;		/* Căn chỉnh của segment */
} Elf64_Phdr;
```

Giá trị:
*   `p_type`: Loại của segment.
    *   `PT_NULL`: Mục trong bảng Program Header không được sử dụng (thường là mục đầu tiên của Bảng Program Header).
    *   `PT_LOAD`: Segment chương trình có thể nạp.
    *   `PT_DYNAMIC`: Thông tin liên kết động (giữ section `.dynamic`).
    *   `PT_INTERP`: Trình thông dịch chương trình (giữ section `.interp`).
    *   `PT_GNU_EH_FRAME`: Đây là một hàng đợi đã được sắp xếp được sử dụng bởi trình biên dịch GNU C (gcc). Nó lưu trữ các trình xử lý ngoại lệ. Vì vậy, khi có sự cố xảy ra, nó có thể sử dụng vùng này để xử lý chính xác.
    *   `PT_GNU_STACK`: Header này được sử dụng để lưu trữ thông tin về ngăn xếp.
*   `p_flags`: Các cờ xác định quyền của segment trong bộ nhớ.
    *   `PF_X`: Segment có thể thực thi.
    *   `PF_W`: Segment có thể ghi.
    *   `PF_R`: Segment có thể đọc.
*   `p_offset`: Độ lệch từ đầu tệp ELF đến segment.
*   `p_vaddr`: Địa chỉ ảo của segment (đối với các segment có thể nạp, `p_vaddr` phải bằng `p_offset`, modulo kích thước trang (thường là 4.096 byte)).
*   `p_paddr`: Địa chỉ vật lý của segment (trên một số hệ thống, có thể sử dụng trường `p_paddr` để chỉ định địa chỉ trong bộ nhớ vật lý để nạp segment. Trên các hệ điều hành hiện đại như Linux, trường này không được sử dụng và được đặt thành không vì chúng thực thi tất cả các tệp nhị phân trong bộ nhớ ảo).
*   `p_filesz`: Kích thước của segment trên đĩa (tính bằng byte).
*   `p_memsz`: Kích thước của segment trong bộ nhớ (tính bằng byte). (một số section chỉ cho biết nhu cầu cấp phát một số byte trong bộ nhớ nhưng thực tế không chiếm các byte này trong tệp nhị phân, chẳng hạn như `.bss`).
*   `p_align`: Căn chỉnh segment (tương tự như trường `sh_addralign` trong một section header).

---- định nghĩa kiểu ----

`p_type` định nghĩa:

```c
#define	PT_NULL		0		/* Mục bảng Program header không sử dụng */
#define PT_LOAD		1		/* Segment chương trình có thể nạp */
#define PT_DYNAMIC	2		/* Thông tin liên kết động */
#define PT_INTERP	3		/* Trình thông dịch chương trình */
#define PT_NOTE		4		/* Thông tin phụ trợ */
#define PT_SHLIB	5		/* Dành riêng */
#define PT_PHDR		6		/* Mục cho chính bảng header */
#define PT_TLS		7		/* Segment lưu trữ cục bộ của luồng */
#define	PT_NUM		8		/* Số lượng các loại được định nghĩa */
#define PT_LOOS		0x60000000	/* Bắt đầu dành riêng cho HĐH */
#define PT_GNU_EH_FRAME	0x6474e550	/* Segment .eh_frame_hdr của GCC */
#define PT_GNU_STACK	0x6474e551	/* Cho biết khả năng thực thi của ngăn xếp */
#define PT_GNU_RELRO	0x6474e552	/* Chỉ đọc sau khi tái định vị */
#define PT_LOSUNW	0x6ffffffa
#define PT_SUNWBSS	0x6ffffffa	/* Segment dành riêng cho Sun */
#define PT_SUNWSTACK	0x6ffffffb	/* Segment ngăn xếp */
#define PT_HISUNW	0x6fffffff
#define PT_HIOS		0x6fffffff	/* Kết thúc dành riêng cho HĐH */
#define PT_LOPROC	0x70000000	/* Bắt đầu dành riêng cho bộ xử lý */
#define PT_HIPROC	0x7fffffff	/* Kết thúc dành riêng cho bộ xử lý */
```

`p_flags` định nghĩa:

```c
#define PF_X		(1 << 0)	/* Segment có thể thực thi */
#define PF_W		(1 << 1)	/* Segment có thể ghi */
#define PF_R		(1 << 2)	/* Segment có thể đọc */
#define PF_MASKOS	0x0ff00000	/* Dành riêng cho HĐH */
#define PF_MASKPROC	0xf0000000	/* Dành riêng cho bộ xử lý */
```

## Các Segment

Phân chia các segment / section:

-   Text Segment
    -   `.text`
    -   `.rodata`
    -   `.hash`
    -   `.dynsym`
    -   `.dynstr`
    -   `.plt`
    -   `.rel.got`
-   Data segment
    -   `.data`
    -   `.dynamic`
    -   `.got.plt`
    -   `.bss`

## Các ký hiệu (Symbols)

Các ký hiệu là một tham chiếu tượng trưng đến một số loại dữ liệu hoặc mã như một biến toàn cục hoặc một hàm.

Cấu trúc 32-bit:

```c
typedef struct
{
  Elf32_Word	st_name;		/* Tên ký hiệu (chỉ số bảng chuỗi) */
  Elf32_Addr	st_value;		/* Giá trị ký hiệu */
  Elf32_Word	st_size;		/* Kích thước ký hiệu */
  unsigned char	st_info;		/* Loại và ràng buộc của ký hiệu */
  unsigned char	st_other;		/* Khả năng hiển thị của ký hiệu */
  Elf32_Section	st_shndx;		/* Chỉ số section */
} Elf32_Sym;
```

Cấu trúc 64-bit:

```c
typedef struct
{
  Elf64_Word	st_name;		/* Tên ký hiệu (chỉ số bảng chuỗi) */
  unsigned char	st_info;		/* Loại và ràng buộc của ký hiệu */
  unsigned char st_other;		/* Khả năng hiển thị của ký hiệu */
  Elf64_Section	st_shndx;		/* Chỉ số section */
  Elf64_Addr	st_value;		/* Giá trị ký hiệu */
  Elf64_Xword	st_size;		/* Kích thước ký hiệu */
} Elf64_Sym;
```

Giá trị:
*   `st_name`: Tên ký hiệu.
*   `st_info`: Loại và ràng buộc của ký hiệu. Được tính toán bằng các macro.
*   `st_other`: Khả năng hiển thị của ký hiệu.
    *   `STV_DEFAULT`: Đối với các ký hiệu có khả năng hiển thị mặc định, thuộc tính của nó được chỉ định bởi loại ràng buộc của ký hiệu.
    *   `STV_PROTECTED`: Ký hiệu có thể được nhìn thấy bởi các đối tượng khác, nhưng không thể bị chiếm quyền ưu tiên.
    *   `STV_HIDDEN`: Ký hiệu không thể được nhìn thấy bởi các đối tượng khác.
    *   `STV_INTERNAL`: Khả năng hiển thị của ký hiệu được dành riêng.
*   `st_shndx`: Chỉ số section.
*   `st_value`: Giá trị ký hiệu.
*   `st_size`: Kích thước ký hiệu.

Giá trị `st_info`:

*   `st_bind`: Ràng buộc ký hiệu.
    *   `STB_LOCAL`: Các ký hiệu cục bộ không hiển thị bên ngoài tệp đối tượng chứa định nghĩa của chúng, chẳng hạn như một hàm được khai báo là static.
    *   `STB_GLOBAL`: Các ký hiệu toàn cục có thể nhìn thấy bởi tất cả các tệp đối tượng đang được kết hợp.
    *   `STB_WEAK`: Tương tự như ràng buộc toàn cục, nhưng có độ ưu tiên thấp hơn, nghĩa là ràng buộc yếu và có thể bị ghi đè bởi một ký hiệu khác (có cùng tên) không được đánh dấu là `STB_WEAK`.
*   `st_type`: Loại ký hiệu.
    *   `STT_NOTYPE`: Loại ký hiệu không được xác định.
    *   `STT_FUNC`: Ký hiệu được liên kết với một hàm hoặc mã thực thi khác.
    *   `STT_OBJECT`: Ký hiệu được liên kết với một đối tượng dữ liệu.
    *   `STT_SECTION`: Ký hiệu là một section.

Macro:

*   `ELFN_ST_BIND(st_info)`: Lấy giá trị `st_bind` từ `st_info`.
*   `ELFN_ST_TYPE(st_info)`: Lấy giá trị `st_type` từ `st_info`.
*   `ELFN_ST_INFO(st_bind, st_type)`: Lấy giá trị `st_info` từ `st_type` và `st_bind`.



---- định nghĩa kiểu ----

macro `st_info`:

```c
#define ELF32_ST_BIND(val)		(((unsigned char) (val)) >> 4)
#define ELF32_ST_TYPE(val)		((val) & 0xf)
#define ELF32_ST_INFO(bind, type)	(((bind) << 4) + ((type) & 0xf))

#define ELF64_ST_BIND(val)		ELF32_ST_BIND (val)
#define ELF64_ST_TYPE(val)		ELF32_ST_TYPE (val)
#define ELF64_ST_INFO(bind, type)	ELF32_ST_INFO ((bind), (type))```
```

định nghĩa `st_bind`:

```c
#define STB_LOCAL	0		/* Ký hiệu cục bộ */
#define STB_GLOBAL	1		/* Ký hiệu toàn cục */
#define STB_WEAK	2		/* Ký hiệu yếu */
#define	STB_NUM		3		/* Số lượng các loại đã được định nghĩa.  */
#define STB_LOOS	10		/* Bắt đầu của các định nghĩa dành riêng cho HĐH */
#define STB_GNU_UNIQUE	10		/* Ký hiệu duy nhất.  */
#define STB_HIOS	12		/* Kết thúc của các định nghĩa dành riêng cho HĐH */
#define STB_LOPROC	13		/* Bắt đầu của các định nghĩa dành riêng cho bộ xử lý */
#define STB_HIPROC	15		/* Kết thúc của các định nghĩa dành riêng cho bộ xử lý */
```

định nghĩa `st_type`:

```c
#define STT_NOTYPE	0		/* Loại ký hiệu không xác định */
#define STT_OBJECT	1		/* Ký hiệu là một đối tượng dữ liệu */
#define STT_FUNC	2		/* Ký hiệu là một đối tượng mã */
#define STT_SECTION	3		/* Ký hiệu liên quan đến một section */
#define STT_FILE	4		/* Tên của ký hiệu là tên tệp */
#define STT_COMMON	5		/* Ký hiệu là một đối tượng dữ liệu chung */
#define STT_TLS		6		/* Ký hiệu là đối tượng dữ liệu cục bộ của luồng*/
#define	STT_NUM		7		/* Số lượng các loại đã được định nghĩa.  */
#define STT_LOOS	10		/* Bắt đầu của các định nghĩa dành riêng cho HĐH */
#define STT_GNU_IFUNC	10		/* Ký hiệu là đối tượng mã gián tiếp */
#define STT_HIOS	12		/* Kết thúc của các định nghĩa dành riêng cho HĐH */
#define STT_LOPROC	13		/* Bắt đầu của các định nghĩa dành riêng cho bộ xử lý */
#define STT_HIPROC	15		/* Kết thúc của các định nghĩa dành riêng cho bộ xử lý */
```

## Liên kết động (Dynamic Linking)

<img width="500" height="562" alt="image" src="https://github.com/user-attachments/assets/7c7e34ab-4f43-4f23-83d6-2f6241447a73" />

Liên kết động là quá trình chúng ta giải quyết các hàm từ các thư viện bên ngoài (đối tượng chia sẻ).

Theo mặc định, liên kết lười (lazy binding) được sử dụng, tức là giải quyết các hàm tại thời điểm chúng được gọi lần đầu tiên, ở các lần gọi tiếp theo, địa chỉ sẽ được lưu trong GOT (Global Offset Table). Khi đó, mục nhập PLT chỉ cần nhảy đến địa chỉ chứa trong mục nhập GOT của hàm đó.

<img width="736" height="581" alt="image" src="https://github.com/user-attachments/assets/293183b9-1be7-436c-ac7b-2eaa45b92153" />


Chúng ta có thể tránh liên kết lười bằng cách sử dụng biến môi trường `LD_BIND_NOW`, hoặc sử dụng `RELRO` (Relocation Read-Only).

Khi một hàm bên ngoài được gọi từ mã, thay vì gọi hàm thực, mục nhập PLT cho hàm đó sẽ được gọi.

PLT là mã sử dụng GOT để nhảy và giải quyết các hàm bên ngoài với sự trợ giúp của trình liên kết.

Có một sự tái định vị cần thiết cho `fgets` sẽ được giải quyết bởi trình liên kết, vì địa chỉ được giải quyết phải được ghi vào đâu đó, trong giá trị offset, nó trỏ đến mục nhập GOT cho `fgets()`. Sau đó, trình liên kết một khi hàm được giải quyết sẽ ghi địa chỉ đó vào đó.

```assembly
Offset       Info        Type         SymValue     SymName
...
0804a000   00000107  R_386_JUMP_SLOT   00000000    fgets
...
```

`0x0804a000` là mục nhập GOT cho `fgets()`.

Khi một hàm như `fgets` được gọi lần đầu tiên:

```assembly
objdump -d ./prog
...
8048481: e8 da fe ff ff    call 0x8048360 <fgets@plt>
...
```

`fgets@plt` được gọi.

Mục nhập PLT:

```assembly
...
08048360 <fgets@plt>:
/* Một lệnh jmp vào GOT */
8048360:  ff 25 00 a0 04 08   jmp *0x804a000
8048366:  68 00 00 00 00      push $0x0
804836b:  e9 e0 ff ff ff      jmp  0x8048350 <_init+0x34>
...
```

Trong lệnh đầu tiên, nó thực hiện một bước nhảy gián tiếp đến địa chỉ chứa trong mục nhập GOT cho `fgets`.

Địa chỉ chứa trong GOT tại thời điểm đó là lệnh tiếp theo của `jmp` đó, vì vậy lệnh `push 0x0` được thực thi, lệnh này đẩy chỉ số trong GOT nơi `fgets` được định vị vào ngăn xếp, lưu ý rằng 3 mục nhập đầu tiên được dành riêng, vì vậy thực tế nó sẽ là mục thứ 4.

Các mục nhập GOT được dành riêng:

-   `GOT[0]`: Chứa một địa chỉ trỏ đến segment động của tệp thực thi, được trình liên kết động sử dụng để trích xuất thông tin liên quan đến liên kết động.
-   `GOT[1]`: Chứa địa chỉ của cấu trúc `link_map` được trình liên kết động sử dụng để giải quyết các ký hiệu.
-   `GOT[2]`: Chứa địa chỉ đến hàm `_dl_runtime_resolve()` của trình liên kết động để giải quyết địa chỉ ký hiệu thực tế cho hàm thư viện chia sẻ.

Lệnh cuối cùng trong đoạn mã PLT của `fgets()` là `jmp 0x8048350`. Địa chỉ này trỏ đến mục nhập PLT đầu tiên trong mọi tệp thực thi, được gọi là PLT-0.

```assembly
8048350: ff 35 f8 9f 04 08      pushl  0x8049ff8
8048356: ff 25 fc 9f 04 08      jmp   *0x8049ffc
804835c: 00 00                  add    %al,(%eax)
```

Lệnh `pushl` đầu tiên đẩy địa chỉ của mục nhập GOT thứ hai, `GOT[1]`, vào ngăn xếp, như đã lưu ý trước đó, chứa địa chỉ của cấu trúc `link_map`.

Lệnh `jmp *0x8049ffc` thực hiện một bước nhảy gián tiếp vào mục nhập GOT thứ ba, `GOT[2]`, chứa địa chỉ đến hàm `_dl_runtime_resolve()` của trình liên kết động, do đó chuyển quyền điều khiển cho trình liên kết động và giải quyết địa chỉ cho `fgets()`. Sau khi `fgets()` đã được giải quyết, tất cả các lần gọi trong tương lai đến mục nhập PLT cho `fgets()` sẽ dẫn đến một bước nhảy đến chính mã `fgets()`, thay vì trỏ lại vào PLT và trải qua quá trình liên kết lười một lần nữa.

*Liên kết tĩnh:*

<img width="441" height="257" alt="image" src="https://github.com/user-attachments/assets/de1c35e9-2ec8-4c93-b593-6888d8239dfe" />


*Liên kết động:*

<img width="506" height="257" alt="image" src="https://github.com/user-attachments/assets/a815e14a-fd72-4479-bb47-811bf9e728ac" />


## Dynamic (Động)

Cấu trúc 32-bit:

```c
typedef struct
{
  Elf32_Sword	d_tag;			/* Loại mục nhập động */
  union
    {
      Elf32_Word d_val;			/* Giá trị số nguyên */
      Elf32_Addr d_ptr;			/* Giá trị địa chỉ */
    } d_un;
} Elf32_Dyn;
```

Cấu trúc 64-bit:

```c
typedef struct
{
  Elf64_Sxword	d_tag;			/* Loại mục nhập động */
  union
    {
      Elf64_Xword d_val;		/* Giá trị số nguyên */
      Elf64_Addr d_ptr;			/* Giá trị địa chỉ */
    } d_un;
} Elf64_Dyn;
```

Giá trị:
*   `d_tag`: Chứa một thẻ (tag).
    *   `DT_NEEDED`: Giữ độ lệch trong bảng chuỗi đến tên của một thư viện chia sẻ cần thiết.
    *   `DT_SYMTAB`: Chứa địa chỉ của bảng ký hiệu động còn được biết đến với tên section là `.dynsym`.
    *   `DT_HASH`: Giữ địa chỉ của bảng băm ký hiệu, còn được biết đến với tên section là `.hash` (hoặc đôi khi được đặt tên là `.gnu.hash`).
    *   `DT_STRTAB`: Giữ địa chỉ của bảng chuỗi ký hiệu, còn được biết đến với tên section là `.dynstr`.
    *   `DT_PLTGOT`: Giữ địa chỉ của bảng offset toàn cục.
*   `d_val`: Giữ một giá trị số nguyên có nhiều cách diễn giải khác nhau, ví dụ như là kích thước của một mục tái định vị.
*   `d_ptr`: Giữ một địa chỉ bộ nhớ ảo có thể trỏ đến các vị trí khác nhau mà trình liên kết cần; một ví dụ điển hình là địa chỉ đến bảng ký hiệu cho thẻ `d_tag` là `DT_SYMTAB`.

---- định nghĩa kiểu ----

định nghĩa `d_tag`:

```c
#define DT_NULL		0		/* Đánh dấu kết thúc của section động */
#define DT_NEEDED	1		/* Tên của thư viện cần thiết */
#define DT_PLTRELSZ	2		/* Kích thước (bytes) của các reloc PLT */
#define DT_PLTGOT	3		/* Giá trị do bộ xử lý định nghĩa */
#define DT_HASH		4		/* Địa chỉ của bảng băm ký hiệu */
#define DT_STRTAB	5		/* Địa chỉ của bảng chuỗi */
#define DT_SYMTAB	6		/* Địa chỉ của bảng ký hiệu */
#define DT_RELA		7		/* Địa chỉ của các reloc Rela */
#define DT_RELASZ	8		/* Tổng kích thước của các reloc Rela */
#define DT_RELAENT	9		/* Kích thước của một reloc Rela */
#define DT_STRSZ	10		/* Kích thước của bảng chuỗi */
#define DT_SYMENT	11		/* Kích thước của một mục nhập bảng ký hiệu */
#define DT_INIT		12		/* Địa chỉ của hàm init */
#define DT_FINI		13		/* Địa chỉ của hàm kết thúc */
#define DT_SONAME	14		/* Tên của đối tượng chia sẻ */
#define DT_RPATH	15		/* Đường dẫn tìm kiếm thư viện (không dùng nữa) */
#define DT_SYMBOLIC	16		/* Bắt đầu tìm kiếm ký hiệu tại đây */
#define DT_REL		17		/* Địa chỉ của các reloc Rel */
#define DT_RELSZ	18		/* Tổng kích thước của các reloc Rel */
#define DT_RELENT	19		/* Kích thước của một reloc Rel */
#define DT_PLTREL	20		/* Loại reloc trong PLT */
#define DT_DEBUG	21		/* Dành cho gỡ lỗi; không xác định */
#define DT_TEXTREL	22		/* Reloc có thể sửa đổi .text */
#define DT_JMPREL	23		/* Địa chỉ của các reloc PLT */
#define	DT_BIND_NOW	24		/* Xử lý các reloc của đối tượng */
#define	DT_INIT_ARRAY	25		/* Mảng chứa địa chỉ của các hàm init */
#define	DT_FINI_ARRAY	26		/* Mảng chứa địa chỉ của các hàm fini */
#define	DT_INIT_ARRAYSZ	27		/* Kích thước (bytes) của DT_INIT_ARRAY */
#define	DT_FINI_ARRAYSZ	28		/* Kích thước (bytes) của DT_FINI_ARRAY */
#define DT_RUNPATH	29		/* Đường dẫn tìm kiếm thư viện */
#define DT_FLAGS	30		/* Cờ cho đối tượng đang được nạp */
#define DT_ENCODING	32		/* Bắt đầu của phạm vi được mã hóa */
#define DT_PREINIT_ARRAY 32		/* Mảng chứa địa chỉ của các hàm preinit*/
#define DT_PREINIT_ARRAYSZ 33		/* kích thước (bytes) của DT_PREINIT_ARRAY */
#define DT_SYMTAB_SHNDX	34		/* Địa chỉ của section SYMTAB_SHNDX */
#define	DT_NUM		35		/* Số lượng đã sử dụng */
#define DT_LOOS		0x6000000d	/* Bắt đầu của các định nghĩa dành riêng cho HĐH */
#define DT_HIOS		0x6ffff000	/* Kết thúc của các định nghĩa dành riêng cho HĐH */
#define DT_LOPROC	0x70000000	/* Bắt đầu của các định nghĩa dành riêng cho bộ xử lý */
#define DT_HIPROC	0x7fffffff	/* Kết thúc của các định nghĩa dành riêng cho bộ xử lý */
#define	DT_PROCNUM	DT_MIPS_NUM	/* Được sử dụng nhiều nhất bởi bất kỳ bộ xử lý nào */
```

## Tái định vị (Relocation)

Tái định vị là quá trình kết nối các tham chiếu tượng trưng với các định nghĩa tượng trưng. Các tệp có thể tái định vị phải có thông tin mô tả cách sửa đổi nội dung section của chúng, do đó cho phép các tệp thực thi và đối tượng chia sẻ chứa đúng thông tin cho hình ảnh chương trình của một tiến trình. Các mục tái định vị chính là những dữ liệu này.

Cấu trúc Rel 32-bit:

```c
typedef struct
{
  Elf32_Addr	r_offset;		/* Địa chỉ */
  Elf32_Word	r_info;			/* Loại tái định vị và chỉ số ký hiệu */
} Elf32_Rel;
```

Cấu trúc Rel 64-bit:

```c
typedef struct
{
  Elf64_Addr	r_offset;		/* Địa chỉ */
  Elf64_Xword	r_info;			/* Loại tái định vị và chỉ số ký hiệu */
} Elf64_Rel;
```

Cấu trúc Rela 32-bit:

```c
typedef struct
{
  Elf32_Addr	r_offset;		/* Địa chỉ */
  Elf32_Word	r_info;			/* Loại tái định vị và chỉ số ký hiệu */
  Elf32_Sword	r_addend;		/* Số hạng cộng thêm */
} Elf32_Rela;
```

Cấu trúc Rela 64-bit:

```c
typedef struct
{
  Elf64_Addr	r_offset;		/* Địa chỉ */
  Elf64_Xword	r_info;			/* Loại tái định vị và chỉ số ký hiệu */
  Elf64_Sxword	r_addend;		/* Số hạng cộng thêm */
} Elf64_Rela;
```

Giá trị:
*   `r_offset`: Trỏ đến vị trí yêu cầu hành động tái định vị.
    *   Đối với các tệp nhị phân loại `ET_REL`, giá trị này biểu thị một độ lệch trong một section header, nơi việc tái định vị phải diễn ra.
    *   Đối với các tệp nhị phân loại `ET_EXEC`, giá trị này biểu thị một địa chỉ ảo bị ảnh hưởng bởi một sự tái định vị.
*   `r_info`: Cung cấp cả chỉ số bảng ký hiệu mà việc tái định vị phải được thực hiện theo và loại tái định vị cần áp dụng.
*   `r_addend`: Chỉ định một số hạng cộng thêm không đổi được sử dụng để tính toán giá trị được lưu trữ trong trường có thể tái định vị.

*Các loại tái định vị x86 (x86 Relocation types):*

<img width="618" height="720" alt="image" src="https://github.com/user-attachments/assets/b30639a5-362a-45f5-9de5-ddc4a78536bc" />

*Các loại tái định vị x86_64 (x86_64 Relocation types):*

<img width="574" height="773" alt="image" src="https://github.com/user-attachments/assets/c84243c3-0200-4e3b-b36a-647c17acb79d" />


Giá trị:
*   `A`: Đây có nghĩa là số hạng cộng thêm (addend) được sử dụng để tính toán giá trị của trường có thể tái định vị.
*   `B`: Đây có nghĩa là địa chỉ cơ sở tại đó một đối tượng chia sẻ đã được nạp vào bộ nhớ trong quá trình thực thi. Nói chung, một tệp đối tượng chia sẻ được xây dựng với địa chỉ ảo cơ sở là 0, nhưng địa chỉ thực thi sẽ khác.
*   `G`: Đây có nghĩa là độ lệch vào bảng offset toàn cục tại đó địa chỉ của ký hiệu của mục tái định vị sẽ tồn tại trong quá trình thực thi.
*   `GOT`: Đây có nghĩa là địa chỉ của bảng offset toàn cục.
*   `L`: Đây có nghĩa là vị trí (độ lệch section hoặc địa chỉ) của mục nhập bảng liên kết thủ tục cho một ký hiệu. Một mục nhập bảng liên kết thủ tục chuyển hướng một lời gọi hàm đến đích thích hợp. Trình liên kết xây dựng bảng liên kết thủ tục ban đầu và trình liên kết động sửa đổi các mục nhập trong quá trình thực thi.
*   `P`: Đây có nghĩa là vị trí (độ lệch section hoặc địa chỉ) của đơn vị lưu trữ đang được tái định vị (được tính toán bằng `r_offset`).
*   `S`: Đây có nghĩa là giá trị của ký hiệu có chỉ số nằm trong mục tái định vị.

Hậu tố tái định vị chung:

*   `_NONE`: Mục bị bỏ qua.
*   `_64`: Giá trị tái định vị qword.
*   `_32`: Giá trị tái định vị dword.
*   `_16`: Giá trị tái định vị word.
*   `_8`: Giá trị tái định vị byte.
*   `_PC`: Tương đối với bộ đếm chương trình.
*   `_GOT`: Tương đối với GOT.
*   `_PLT`: Tương đối với PLT (Bảng liên kết thủ tục).
*   `_COPY`: Giá trị được sao chép trực tiếp từ đối tượng chia sẻ tại thời điểm nạp.
*   `_GLOB_DAT`: Biến toàn cục.
*   `_JMP_SLOT`: Mục nhập PLT.
*   `_RELATIVE`: Tương đối với địa chỉ cơ sở của hình ảnh chương trình.
*   `_GOTOFF`: Địa chỉ tuyệt đối trong GOT.
*   `_GOTPC`: Độ lệch GOT tương đối với bộ đếm chương trình.

<img width="877" height="731" alt="image" src="https://github.com/user-attachments/assets/6de6f44c-5570-4e22-955b-ab799c7670a4" />

Các Section:

*   `.rel.bss`: Chứa tất cả các reloc `R_386_COPY`.
*   `.rel.plt`: Chứa tất cả các reloc `R_386_JMP_SLOT`, những reloc này sửa đổi nửa đầu của các phần tử GOT.
*   `.rel.got`: Chứa tất cả các reloc `R_386_GLOB_DATA`, những reloc này sửa đổi nửa sau của các phần tử GOT.
*   `.rel.data`: Chứa tất cả các reloc `R_386_32` và `R_386_RELATIVE`.
*   `.rela.dyn`: Chứa các tái định vị động cho các biến.
*   `.rela.plt`: Chứa các tái định vị động cho các hàm.

## Các tệp nhị phân đã được lược bỏ (Stripped binaries)

Các tệp nhị phân đã được lược bỏ là những tệp mà các ký hiệu của chúng đã bị xóa.

Các ký hiệu nói chung không cần thiết cho trình nạp để nạp một tệp thực thi ELF, ngoại trừ các ký hiệu liên kết động.

Chúng thường được sử dụng cho mục đích gỡ lỗi, và chúng làm cho nhiệm vụ kỹ thuật đảo ngược dễ dàng hơn vì chúng cung cấp tên hàm và rất nhiều thông tin về cấu trúc tệp ELF.

Tuy nhiên, vì các ký hiệu động vẫn còn, bạn có thể xem các hàm được nhập từ các thư viện bên ngoài như glibc.

## Sự khác biệt giữa các đối tượng ELF 32-bit và 64-bit

Sự khác biệt chính là:

*   Trong header ELF, `e_machine` thay đổi.
*   Kích thước của các giá trị trong tệp ELF cũng thay đổi.

## Sections so với Segments

Các segment được chia thành các section, mỗi section có một tiện ích cho tệp ELF.

Bản thân các section không hữu ích trong thời gian chạy, vì vậy chúng chỉ hữu ích trong thời gian liên kết.

Các segment được sử dụng để tạo một khối bộ nhớ, với một số quyền cụ thể và lưu trữ một số nội dung ở đó.

Trái ngược với các định dạng tệp khác, các tệp ELF bao gồm các section và segment. Như đã đề cập trước đó, các section thu thập tất cả thông tin cần thiết để liên kết một tệp đối tượng nhất định và xây dựng một tệp thực thi, trong khi các Program Header chia tệp thực thi thành các segment với các thuộc tính khác nhau, cuối cùng sẽ được nạp vào bộ nhớ.

Để hiểu mối quan hệ giữa các Section và Segment, chúng ta có thể hình dung các segment như một công cụ giúp cuộc sống của trình nạp linux dễ dàng hơn, vì chúng nhóm các section theo thuộc tính vào các segment đơn lẻ để làm cho quá trình nạp tệp thực thi hiệu quả hơn, thay vì nạp từng section riêng lẻ vào bộ nhớ. Sơ đồ sau đây cố gắng minh họa khái niệm này:

<img width="612" height="628" alt="image" src="https://github.com/user-attachments/assets/3bb927ac-9214-42fd-a37a-f4838368dd04" />


## Tệp ELF được nạp trong bộ nhớ so với Tệp ELF

Các tệp ELF trên đĩa chỉ là một định dạng xác định cách nạp nó vào bộ nhớ để hoạt động tốt.

Trên đĩa, nó chỉ định một số thông tin không cần thiết như `.symtab`, `.strtab`, chúng không được sử dụng trong thời gian chạy và chỉ có ở đó cho mục đích gỡ lỗi.

Kích thước trong bộ nhớ thường khác với trên đĩa, ví dụ, ai đó có thể định nghĩa các biến chưa được khởi tạo (được lưu trữ tại bss). Trên đĩa, bạn chỉ cần chỉ định kích thước của nó mà không chiếm không gian đó. Sau khi được nạp vào bộ nhớ, bạn phải lấp đầy không gian đó bằng cách nào đó, ví dụ như bằng các số không, vì vậy khi nạp, dung lượng lưu trữ cần thiết để cấp phát ELF sẽ tăng lên.

Tổng quan cơ bản:

Tệp ELF trên đĩa:

<div>
<img src="https://i.imgur.com/qPYlh7B.png" width="500"/>
</div>

ELF được nạp trong bộ nhớ:

<img width="400" height="599" alt="image" src="https://github.com/user-attachments/assets/d2398f11-0e84-4104-99fe-dd7af01df3d6" />


## Sự khác biệt giữa các đối tượng ELF

#### Tệp đối tượng (Object Files)

Tệp đối tượng là các tệp có thể tái định vị, chúng được sử dụng để liên kết chúng với các tệp đối tượng khác.

Nó cung cấp thông tin cho trình liên kết để, khi đến lúc liên kết nó với phần còn lại của các tệp đối tượng, cho phép tái định vị và làm cho nó dễ dàng hơn.

Nội dung của tệp đối tượng khác với các tệp ELF khác như `ET_EXEC` và `ET_DYN`.

Nó thường có các section `.rela.text` và `.rela.eh_frame`.

Vì nó chưa phải là một tệp ELF hoàn chỉnh, chưa có section cụ thể nào được tạo ra, do đó bạn sẽ chỉ tìm thấy các section mã và dữ liệu thông thường, và các ký hiệu.

#### Tệp thực thi liên kết tĩnh (Statically-linked executable files)

Tệp thực thi là những tệp không phụ thuộc vào các thư viện bên ngoài, do đó không có sự tái định vị nào đang chờ xử lý đối với chúng vì chúng có thể được nạp mà không cần các đối tượng bên ngoài.

Chúng không cần `.dynamic` hoặc segment Dynamic, chúng không cần GOT hoặc PLT vì các lời gọi hàm được thực hiện trực tiếp đến địa chỉ hàm và không có bất kỳ trung gian nào.

Sau đó, trong loại tệp ELF này, bạn sẽ tìm thấy các section mã và dữ liệu thông thường, và các ký hiệu (có thể bị xóa).

Vì chúng là tĩnh, nếu chúng sử dụng các hàm libc, tổng kích thước sẽ dài đáng kể.

#### Tệp thực thi liên kết động (Dynamically-linked executable files)

Chúng vẫn là các tệp thực thi, nhưng vì chúng được liên kết động nên chúng là PIC (Process Independent Code - Mã độc lập với tiến trình).

Chúng cần GOT và PLT làm trung gian để sử dụng các hàm bên ngoài từ các thư viện chia sẻ như `printf()`.

Trong loại tệp thực thi này, bạn thường sẽ tìm thấy các section mã và dữ liệu thông thường, GOT, PLT, các section ký hiệu liên kết động như `.dynsym` và `.dynstr` (Cũng như các ký hiệu tĩnh không cần thiết).

Bạn cũng sẽ tìm thấy section `.dynamic`, rất quan trọng cho việc liên kết động, và `.rela.dyn`, `.rela.plt`.

#### Thư viện chia sẻ (Shared libraries)

Chúng được nạp vào bộ nhớ của một tiến trình để cung cấp các hàm cho tệp thực thi sẽ sử dụng chúng.

Chúng tương tự như các tệp thực thi được liên kết động, nhưng không giống nhau.

Ở đây không có segment `PT_INTERP`, vì thư viện chia sẻ không được nạp bởi kernel mà bởi trình liên kết.

Ngoài ra, các hàm cục bộ cũng được bao gồm trong `.dynsym` (Không chỉ trong `.symtab`), và `__libc_start_main` không được nhập.

Cấu trúc còn lại hầu hết giống như các tệp thực thi được liên kết động.

## Từng bước nạp ELF cho mỗi loại đối tượng, ASLR và PIC/PIE

### Tệp tái định vị (Relocatable files)

Chúng không được cho là sẽ được nạp vì một số quá trình tái định vị đang chờ xử lý để tạo ra một tệp thực thi hoạt động đầy đủ trước tiên.

### Tệp thực thi liên kết tĩnh

Đầu tiên, khi chúng ta quyết định chạy một tệp thực thi, kernel sẽ thiết lập một tiến trình và cấp cho nó một không gian bộ nhớ ảo, một ngăn xếp, v.v.

Ngăn xếp cho không gian địa chỉ của tiến trình đó được thiết lập theo một cách rất cụ thể để truyền thông tin cho trình liên kết động. Thiết lập và sắp xếp thông tin đặc biệt này được gọi là auxiliary vector hoặc auxv.

<img width="122" height="130" alt="image" src="https://github.com/user-attachments/assets/e71adb00-e9ec-4b83-8972-832dff2d4a72" />


Cấu trúc:

```c
typedef struct
{
    uint64_t a_type;
    union
    {
        uint64_t a_val;
    } a_un;
} Elf64_auxv_t;
```
Loại Auxv:

```c
/* Giá trị hợp lệ cho a_type (loại entry).  */
#define AT_NULL         0               /* Kết thúc vector */
#define AT_IGNORE       1               /* Entry nên được bỏ qua */
#define AT_EXECFD       2               /* Mô tả tệp của chương trình */
#define AT_PHDR         3               /* Program headers cho chương trình */
#define AT_PHENT        4               /* Kích thước của entry program header */
#define AT_PHNUM        5               /* Số lượng program headers */
#define AT_PAGESZ       6               /* Kích thước trang hệ thống */
#define AT_BASE         7               /* Địa chỉ cơ sở của trình thông dịch */
#define AT_FLAGS        8               /* Cờ */
#define AT_ENTRY        9               /* Điểm vào của chương trình */
#define AT_NOTELF       10              /* Chương trình không phải là ELF */
#define AT_UID          11              /* uid thực */
#define AT_EUID         12              /* uid hiệu lực */
#define AT_GID          13              /* gid thực */
#define AT_EGID         14              /* gid hiệu lực */
#define AT_CLKTCK       17              /* Tần số của times() */
/* Con trỏ đến trang hệ thống toàn cục được sử dụng cho các lời gọi hệ thống và những thứ hay ho khác.  */
#define AT_SYSINFO      32
#define AT_SYSINFO_EHDR 33
```

Auxiliary vector là một cấu trúc đặc biệt dùng để truyền thông tin trực tiếp từ kernel đến chương trình mới chạy. Nó chứa thông tin cụ thể của hệ thống có thể được yêu cầu, chẳng hạn như kích thước mặc định của một trang bộ nhớ ảo trên hệ thống hoặc các khả năng của phần cứng; đó là các tính năng cụ thể mà kernel đã xác định phần cứng cơ bản có mà các chương trình không gian người dùng có thể tận dụng.

Sau đó, hệ điều hành ánh xạ một trình thông dịch vào bộ nhớ ảo của tiến trình (Thường là `ld-linux.so`). Sau đó đọc mã của trình thông dịch và khởi động nó từ điểm vào của nó. Trình thông dịch có thể được lấy từ section `.interp` trong tệp ELF.

Trình thông dịch nạp tệp nhị phân, và trao quyền kiểm soát cho điểm vào của tệp nhị phân.

Tóm tắt:

-   Kernel ánh xạ chương trình vào bộ nhớ (và vDSO);
-   Kernel thiết lập ngăn xếp và các thanh ghi (truyền thông tin như các đối số và biến môi trường) và gọi điểm vào của chương trình chính.
-   Tệp thực thi được nạp tại một địa chỉ cố định và không cần tái định vị.

### Tệp thực thi liên kết động

Đầu tiên, khi chúng ta quyết định chạy một tệp thực thi, kernel sẽ thiết lập một tiến trình và cấp cho nó một không gian bộ nhớ ảo, một ngăn xếp, v.v.

Ngăn xếp cho không gian địa chỉ của tiến trình đó được thiết lập theo một cách rất cụ thể để truyền thông tin cho trình liên kết động. Cấu hình và sắp xếp thông tin đặc biệt này được gọi là auxiliary vector hoặc auxv.

<img width="122" height="130" alt="image" src="https://github.com/user-attachments/assets/8556f36a-efb1-444d-a7cb-fa7d0078dc10" />

<img width="721" height="517" alt="image" src="https://github.com/user-attachments/assets/9d4c6121-7ddd-40d8-97d7-730b5a2243ee" />

Giao diện mẫu:

<img width="394" height="445" alt="image" src="https://github.com/user-attachments/assets/574bf131-12b1-44c4-afd5-94851fc5f801" />


Cấu trúc:

```c
typedef struct
{
    uint64_t a_type;
    union
    {
        uint64_t a_val;
    } a_un;
} Elf64_auxv_t;
```
Loại Auxv:

```c
/* Giá trị hợp lệ cho a_type (loại entry).  */
#define AT_NULL         0               /* Kết thúc vector */
#define AT_IGNORE       1               /* Entry nên được bỏ qua */
#define AT_EXECFD       2               /* Mô tả tệp của chương trình */
#define AT_PHDR         3               /* Program headers cho chương trình */
#define AT_PHENT        4               /* Kích thước của entry program header */
#define AT_PHNUM        5               /* Số lượng program headers */
#define AT_PAGESZ       6               /* Kích thước trang hệ thống */
#define AT_BASE         7               /* Địa chỉ cơ sở của trình thông dịch */
#define AT_FLAGS        8               /* Cờ */
#define AT_ENTRY        9               /* Điểm vào của chương trình */
#define AT_NOTELF       10              /* Chương trình không phải là ELF */
#define AT_UID          11              /* uid thực */
#define AT_EUID         12              /* uid hiệu lực */
#define AT_GID          13              /* gid thực */
#define AT_EGID         14              /* gid hiệu lực */
#define AT_CLKTCK       17              /* Tần số của times() */
/* Con trỏ đến trang hệ thống toàn cục được sử dụng cho các lời gọi hệ thống và những thứ hay ho khác.  */
#define AT_SYSINFO      32
#define AT_SYSINFO_EHDR 33
```

Auxiliary vector là một cấu trúc đặc biệt dùng để truyền thông tin trực tiếp từ kernel đến chương trình mới chạy. Nó chứa thông tin cụ thể của hệ thống có thể được yêu cầu, chẳng hạn như kích thước mặc định của một trang bộ nhớ ảo trên hệ thống hoặc các khả năng của phần cứng; đó là các tính năng cụ thể mà kernel đã xác định phần cứng cơ bản có mà các chương trình không gian người dùng có thể tận dụng.

Sau khi mã chương trình đã được nạp vào bộ nhớ như đã mô tả trước đó, trình xử lý ELF cũng nạp chương trình thông dịch ELF vào bộ nhớ bằng `load_elf_interp()`. Quá trình này tương tự như quá trình nạp chương trình gốc: mã kiểm tra thông tin định dạng trong header ELF, đọc program header ELF, ánh xạ tất cả các segment `PT_LOAD` từ tệp vào bộ nhớ của chương trình mới và để lại không gian cho segment `BSS` của trình thông dịch. Trình thông dịch có thể được lấy từ section `.interp` trong tệp ELF.

Địa chỉ bắt đầu thực thi của chương trình cũng được đặt thành điểm vào của trình thông dịch, thay vì của chính chương trình. Khi lời gọi hệ thống `execve()` hoàn tất, quá trình thực thi sẽ bắt đầu với trình thông dịch ELF, trình này sẽ đảm nhận việc đáp ứng các yêu cầu liên kết của chương trình từ không gian người dùng - tìm và nạp các thư viện chia sẻ mà chương trình phụ thuộc vào, và giải quyết các ký hiệu chưa được định nghĩa của chương trình thành các định nghĩa chính xác trong các thư viện đó. Sau khi quá trình liên kết này hoàn tất (dựa trên sự hiểu biết sâu sắc hơn về định dạng ELF so với kernel), trình thông dịch có thể bắt đầu thực thi chính chương trình mới, tại địa chỉ đã được ghi trước đó trong giá trị phụ trợ `AT_ENTRY`.

Chúng tôi đã đề cập trước đó rằng các lời gọi hệ thống rất chậm và các hệ thống hiện đại có các cơ chế để tránh chi phí gọi một bẫy đến bộ xử lý.

Trong Linux, điều này được thực hiện bằng một mẹo nhỏ giữa trình nạp động và kernel, tất cả được giao tiếp bằng cấu trúc `AUXV`. Kernel thực sự thêm một thư viện chia sẻ nhỏ vào không gian địa chỉ của mọi tiến trình mới được tạo, thư viện này chứa một hàm thực hiện các lời gọi hệ thống cho bạn. Vẻ đẹp của hệ thống này là nếu phần cứng cơ bản hỗ trợ một cơ chế lời gọi hệ thống nhanh, kernel (là người tạo ra thư viện) có thể sử dụng nó, nếu không, nó có thể sử dụng sơ đồ cũ là tạo ra một bẫy. Thư viện này có tên là linux-gate.so.1, được gọi như vậy vì nó là một cổng vào các hoạt động bên trong của kernel.

Khi kernel khởi động trình liên kết động, nó sẽ thêm một mục vào auxv có tên là AT_SYSINFO_EHDR, là địa chỉ trong bộ nhớ mà thư viện kernel đặc biệt đó tồn tại. Khi trình liên kết động khởi động, nó có thể tìm con trỏ AT_SYSINFO_EHDR, và nếu tìm thấy, hãy nạp thư viện đó cho chương trình. Chương trình không biết thư viện này tồn tại; đây là một thỏa thuận riêng tư giữa trình liên kết động và kernel.

Trình thông dịch nạp tệp nhị phân và phân tích cú pháp nó để tìm ra những thư viện nào mà tệp nhị phân cần, và ánh xạ chúng bằng mmap hoặc các tùy chọn tương tự, sau đó thực hiện bất kỳ sự tái định vị cuối cùng cần thiết nào trong các section mã của tệp nhị phân để điền vào các địa chỉ chính xác cho các tham chiếu đến các thư viện động.

Trình liên kết động sẽ nhảy đến địa chỉ điểm vào như được cung cấp trong tệp nhị phân ELF.

Điểm vào là hàm `_start` trong tệp nhị phân. Tại thời điểm này, chúng ta có thể thấy trong quá trình giải mã một số giá trị được đẩy vào ngăn xếp. Giá trị đầu tiên là địa chỉ của hàm `__libc_csu_fini`, một giá trị khác là địa chỉ của `__libc_csu_init` và cuối cùng là địa chỉ của hàm `main()`. Sau đó, giá trị hàm `__libc_start_main` được gọi.

Ở giai đoạn này, chúng ta có thể thấy rằng hàm `__libc_start_main` sẽ nhận khá nhiều tham số đầu vào trên ngăn xếp. Đầu tiên, nó sẽ có quyền truy cập vào các đối số chương trình, biến môi trường và auxiliary vector từ kernel. Sau đó, hàm khởi tạo sẽ đẩy lên ngăn xếp các địa chỉ cho các hàm để xử lý `init`, `fini`, và cuối cùng là địa chỉ của chính hàm `main()`.

Giá trị cuối cùng được đẩy vào ngăn xếp cho `__libc_start_main` là hàm khởi tạo `__libc_csu_init`. Nếu chúng ta theo dõi chuỗi lời gọi từ `__libc_csu_init`, chúng ta có thể thấy nó thực hiện một số thiết lập và sau đó gọi hàm `_init` trong tệp thực thi. Hàm `_init` cuối cùng gọi một số hàm có tên là `__do_global_ctors_aux`, `frame_dummy` và `call_gmon_start`.

Sau khi `__libc_start_main` hoàn thành với lời gọi `_init`, cuối cùng nó gọi hàm `main()`. Hãy nhớ rằng nó đã thiết lập ngăn xếp ban đầu với các đối số và con trỏ môi trường từ kernel; đây là cách `main` nhận được các đối số `argc`, `argv[]`, `envp[]` của nó. Tiến trình bây giờ chạy và giai đoạn thiết lập đã hoàn tất.

Cuối cùng, gọi các hàm kết thúc và gọi `exit()` với giá trị trả về từ `main()`.

Công việc tiếp theo của trình liên kết sẽ là giải quyết bằng liên kết lười tất cả các hàm thư viện khi chúng được gọi.
Sử dụng các ký hiệu của thư viện và các ký hiệu động từ tệp thực thi của bạn, và các tái định vị cho GOT, việc liên kết động sẽ được thực hiện thành công.

Tóm tắt:

-   xác định vị trí và ánh xạ tất cả các phụ thuộc (cũng như đối tượng chia sẻ được chỉ định trong LD_PRELOAD);

-   tái định vị các tệp.

Đây là một cái nhìn tổng quan ở mức độ rất cao theo như tôi hiểu:

-   kernel khởi tạo tiến trình:

    -   nó ánh xạ chương trình chính, các segment của trình thông dịch (trình liên kết động) và vDSO vào không gian địa chỉ ảo;

    -   nó thiết lập ngăn xếp (truyền các đối số, môi trường) và gọi điểm vào của trình liên kết động;

-   trình liên kết động nạp các đối tượng ELF khác nhau và liên kết chúng lại với nhau

    -   nó tự tái định vị (!);

    -   nó tìm và nạp các thư viện cần thiết;

    -   nó thực hiện các tái định vị (liên kết các đối tượng ELF);

    -   nó gọi các hàm khởi tạo của các đối tượng chia sẻ;

-   Các hàm đó được chỉ định trong các mục DT_INIT và DT_INIT_ARRAY của các đối tượng ELF.

    -   nó gọi điểm vào của chương trình chính;
    -   Điểm vào của chương trình chính được tìm thấy trong mục AT_ENTRY của auxiliary vector: nó đã được kernel khởi tạo từ trường e_entry của header ELF.

    -   sau đó tệp thực thi tự khởi tạo.

### Thư viện chia sẻ

Như đã giải thích trước đó, chúng được nạp vào không gian bộ nhớ của tiến trình, và trình liên kết thực hiện công việc liên kết động.

## Các đối tượng và hàm thông thường

-   `frame_dummy`: Hàm này nằm trong section `.init`. Nó được định nghĩa là `void frame_dummy ( void )` và toàn bộ mục đích tồn tại của nó là gọi `__register_frame_info_bases` có các đối số.
-   `_start`: Đây là nơi `e_entry` trỏ đến, và là mã đầu tiên được thực thi.
-   `_init`: Trình nạp động thực thi hàm (INIT) trước khi quyền kiểm soát được chuyển đến hàm `_start` và thực thi hàm (FINI) ngay trước khi quyền kiểm soát được trả lại cho kernel của HĐH. Hàm `_init` là hàm mặc định được sử dụng cho thẻ (INIT). Nó gọi một số hàm như `__gmon_start__`, `frame_dummy`, `__do_global_ctors_aux`.
-   `_fini`: Trình nạp động thực thi hàm (FINI) ngay trước khi quyền kiểm soát được trả lại cho kernel của HĐH.
-   `.init`: Mã sẽ được thực thi khi chương trình bắt đầu.
-   `.fini`: Mã sẽ được thực thi khi chương trình kết thúc.
-   `.init_array`: Mảng các con trỏ để sử dụng làm hàm khởi tạo.
-   `.fini_array`: Mảng các con trỏ để sử dụng làm hàm hủy.
-   `__libc_start_main`: Các hàm Libc thiết lập một số thứ và gọi `main()`.
-   `deregister_tm_clones`: Bộ nhớ giao dịch nhằm mục đích làm cho việc lập trình với các luồng trở nên đơn giản hơn. Nó là một giải pháp thay thế cho đồng bộ hóa dựa trên khóa. Các thủ tục này hủy bỏ và thiết lập, tương ứng, một bảng được sử dụng bởi thư viện (libitm) hỗ trợ các hàm này.
-   `register_tm_clones`: Bộ nhớ giao dịch nhằm mục đích làm cho việc lập trình với các luồng trở nên đơn giản hơn. Nó là một giải pháp thay thế cho đồng bộ hóa dựa trên khóa. Các thủ tục này hủy bỏ và thiết lập, tương ứng, một bảng được sử dụng bởi thư viện (libitm) hỗ trợ các hàm này.
-   `__register_frame_info_bases`:
-   `__stack_chk_fail`: Hàm bảo vệ chống tràn bộ đệm ngăn xếp (Stack Smashing Protector).
-   `__do_global_dtors_aux`: Chạy tất cả các hàm hủy toàn cục khi thoát khỏi chương trình trên các hệ thống không có `.fini_array`.
-   `__do_global_dtors_aux_fini_array_entry` và `__init_array_end`: Các mục này đánh dấu điểm cuối và điểm bắt đầu của section `.fini_array`, chứa các con trỏ đến tất cả các hàm kết thúc ở cấp độ chương trình.
-   `__frame_dummy_init_array_entry` và `__init_array_start`: Các mục này đánh dấu điểm cuối và điểm bắt đầu của section `.init_array`, chứa các con trỏ đến tất cả các hàm khởi tạo ở cấp độ chương trình.
-   `__libc_csu_init`: Các hàm này chạy bất kỳ hàm khởi tạo cấp độ chương trình nào (giống như các hàm khởi tạo cho toàn bộ chương trình của bạn).
-   `__libc_csu_fini`: Các hàm này chạy bất kỳ hàm kết thúc cấp độ chương trình nào (giống như các hàm hủy cho toàn bộ chương trình của bạn).
-   `main`: Đối với các chương trình được liên kết với libc, đây là thư viện mặc định được gọi bởi `__libc_start_main` và là nơi mã tùy chỉnh của người dùng đầu tiên được thực thi.
-   `.eh_frame`: Các tính năng gỡ lỗi dựa trên DWARF như gỡ rối ngăn xếp (stack unwinding).

Tóm tắt:

-   `_start` gọi hàm `__libc_start_main` của libc;
-   `__libc_start_main` gọi hàm `__libc_csu_init` của tệp thực thi (phần được liên kết tĩnh của libc);
-   `__libc_csu_init` gọi các hàm khởi tạo của tệp thực thi (và các khởi tạo khác);
-   `__libc_start_main` gọi hàm `main()` của tệp thực thi;
-   `__libc_start_main` gọi hàm `exit()` của tệp thực thi.

<img width="741" height="635" alt="image" src="https://github.com/user-attachments/assets/f7ab2794-c6f7-4ba6-a607-e67cfb114473" />

## FAQ (Câu hỏi thường gặp)

### Tại sao chúng ta cần các section?

Các section chỉ có ở đó để làm cho công việc của trình liên kết dễ dàng hơn. Ví dụ, khi bạn, trong một lần tái định vị muốn chỉ định một lần tái định vị cho các tệp `ET_REL`, bạn chỉ định độ lệch trong section đó.

### Trình biên dịch tạo ra các tệp thực thi liên kết động (DT_NEEDED) như thế nào?

Khi trình biên dịch biên dịch cho một tệp thực thi liên kết động, thay vì biên dịch nó thành một thư viện .a và liên kết nó một cách tĩnh, nó tạo ra trong section `.dynamic` được chỉ định bởi `DT_NEEDED` một chuỗi với tên thư viện (Ví dụ: libc.so.6).

Khi tệp nhị phân được thực thi trên một hệ thống khác, trình thông dịch cố gắng tìm thư viện đó theo tên và nạp nó vào bộ nhớ để bắt đầu quá trình liên kết động.

### Khi sử dụng các tệp thực thi PIC/PIE, làm thế nào để các địa chỉ được vá để thêm độ lệch?

-- CẦN LÀM --

### Sự khác biệt giữa .got, .plt.got, .plt và .got.plt là gì?

`.got` dùng cho các tái định vị liên quan đến các 'biến' toàn cục trong khi `.got.plt` là một section phụ trợ để hoạt động cùng với `.plt` khi giải quyết các địa chỉ tuyệt đối của các thủ tục.

### Không gian mmap nằm ở đâu?

-- CẦN LÀM --

### ld được nạp ở đâu?

-- CẦN LÀM --

### Các thư viện cần thiết được nạp ở đâu?

-- CẦN LÀM --

### Sự khác biệt giữa Rel và Rela là gì?

Rel được sử dụng trong các hệ thống 32-bit, thay vào đó, Rela được sử dụng trong các hệ thống 64-bit.

Rela, có một addend, Rel thì không.

### Địa chỉ tiến trình được chọn như thế nào?

-- CẦN LÀM --

### Căn chỉnh hoạt động như thế nào?

-- CẦN LÀM --

### Các segment khác được bao gồm trong các segment PT_LOAD như thế nào?

-- CẦN LÀM --

### Điều gì xảy ra nếu chúng ta bao gồm nhiều hơn một thư viện chia sẻ?

-- CẦN LÀM --

### Điều gì xảy ra nếu A (chương trình) sử dụng libc, cũng nhập B (thư viện) cũng sử dụng libc?

-- CẦN LÀM --

### Khi a() (cục bộ) gọi b() (libc) và b() gọi c() (cũng là libc), c() có được nhập vào .dynsym không?

-- CẦN LÀM --

## Tài liệu tham khảo

-   [Practical Linux Binary Analysis: Build Your Own Linux Tools for Binary Instrumentation, Analysis, and Disassembly](https://www.amazon.es/Practical-Binary-Analysis-Instrumentation-Disassembly/dp/1593279124) của Dennis Andriesse

-   [Learning Linux Binary Analysis](https://www.amazon.es/Learning-Linux-Binary-Analysis-elfmaster/dp/1782167102) của Ryan "elfmaster" O'Neill

-   [https://web.stanford.edu/~ouster/cgi-bin/cs140-winter13/pintos/specs/sysv-abi-update.html/ch4.eheader.html](https://web.stanford.edu/~ouster/cgi-bin/cs140-winter13/pintos/specs/sysv-abi-update.html/ch4.eheader.html)

-   [https://hydrasky.com/malware-analysis/elf-file-chapter-2-relocation-and-dynamic-linking/](https://hydrasky.com/malware-analysis/elf-file-chapter-2-relocation-and-dynamic-linking/)

-   [https://www.intezer.com/blog/research/executable-linkable-format-101-part1-sections-segments/](https://www.intezer.com/blog/research/executable-linkable-format-101-part1-sections-segments/)

-   [https://man7.org/linux/man-pages/man8/ld.so.8.html](https://man7.org/linux/man-pages/man8/ld.so.8.html)

-   [https://lwn.net/Articles/631631/](https://lwn.net/Articles/631631/)

-   [https://codywu2010.wordpress.com/2014/11/29/about-elf-pie-pic-and-else/](https://codywu2010.wordpress.com/2014/11/29/about-elf-pie-pic-and-else/)

-   [https://www.it-swarm-es.tech/es/c/que-funciones-agrega-gcc-al-linux-elf/822753373/](https://www.it-swarm-es.tech/es/c/que-funciones-agrega-gcc-al-linux-elf/822753373/)

-   [https://gcc.gnu.org/onlinedocs/gccint/Initialization.html](https://gcc.gnu.org/onlinedocs/gccint/Initialization.html)

-   [https://gcc.gnu.org/wiki/TransactionalMemory](https://gcc.gnu.org/wiki/TransactionalMemory)

-   [http://pmarlier.free.fr/gcc-tm-tut.html](http://pmarlier.free.fr/gcc-tm-tut.html)

-   [https://github.com/gcc-mirror/gcc/blob/master/libgcc/crtstuff.c](https://github.com/gcc-mirror/gcc/blob/master/libgcc/crtstuff.c)

-   [https://www.bottomupcs.com/starting_a_process.xhtml](https://www.bottomupcs.com/starting_a_process.xhtml)

-   [https://www.gabriel.urdhr.fr/2015/01/22/elf-linking/](https://www.gabriel.urdhr.fr/2015/01/22/elf-linking/)

-   [https://web.archive.org/web/20191210114310/http://dbp-consulting.com/tutorials/debugging/linuxProgramStartup.html](https://web.archive.org/web/20191210114310/http://dbp-consulting.com/tutorials/debugging/linuxProgramStartup.html)

-   `/usr/include/elf.h`

-   Man page `ELF(5)`
