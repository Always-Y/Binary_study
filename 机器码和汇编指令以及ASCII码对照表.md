# 机器码和汇编指令以及ASCII码对照表

机器码，汇编指令以及ASCII码对照表

## 参数说明

`reg8:`8位寄存器
`reg16:`16位寄存器
`mem8:`8位内存数值
`mem16:` 16位内存数值
`immed8:`8位立即数值
`immed16:` 16位立即数值
`immed32:`32位立即数值
`segReg:`16位段寄存器

## 机器码，汇编指令以及ASCII码对照

| 十进制 | 机器码 | 汇编指令                 | ASCII字符                   | 字符含义     |
| ------ | ------ | ------------------------ | --------------------------- | ------------ |
| 0      | 0x00   | ADD reg8/mem8,reg8       | NUL(null)                   | 空字符       |
| 1      | 0x01   | ADD reg16/mem16,reg16    | SOH(start of headline)      | 标题开始     |
| 2      | 0x02   | ADD reg8,reg8/mem8       | STX (start of text)         | 正文开始     |
| 3      | 0x03   | ADD reg16,reg16/mem16    | ETX (end of text)           | 正文结束     |
| 4      | 0x04   | ADD AL,immed8            | EOT (end of transmission)   | 传输结束     |
| 5      | 0x05   | ADD AX,immed16           | ENQ (enquiry)               | 请求         |
| 6      | 0x06   | PUSH es                  | ACK (acknowledge)           | 收到通知     |
| 7      | 0x07   | POP es                   | BEL (bell)                  | 响铃         |
| 8      | 0x08   | OR reg8/mem8,reg8        | BS (backspace)              | 退格         |
| 9      | 0x09   | OR reg16/mem16,reg16     | HT (horizontal tab)         | 水平制表符   |
| 10     | 0x0A   | OR reg8,reg8/mem8        | LF (NL line feed, new line) | 换行键       |
| 11     | 0x0B   | OR reg16,reg16/mem16     | VT (vertical tab)           | 垂直制表符   |
| 12     | 0x0C   | OR al,immed8             | FF (NP form feed, new page) | 换页键       |
| 13     | 0x0D   | OR ax,immed16            | CR (carriage return)        | 回车键       |
| 14     | 0x0E   | PUSH cs                  | SO (shift out)              | 不用切换     |
| 15     | 0x0F   | Not used                 | SI (shift in)               | 启用切换     |
| 16     | 0x10   | ADC reg8/mem8,reg8       | DLE (data link escape)      | 数据链路转义 |
| 17     | 0x11   | ADC reg16/mem16,reg16    | DC1 (device control 1)      | 设备控制1    |
| 18     | 0x12   | ADC reg8,reg8/mem8       | DC2 (device control 2)      | 设备控制2    |
| 19     | 0x13   | ADC reg16,reg16/mem16    | DC3 (device control 3)      | 设备控制3    |
| 20     | 0x14   | ADC al,immed8            | DC4 (device control 4)      | 设备控制4    |
| 21     | 0x15   | ADC ax,immed16           | NAK (negative acknowledge)  | 拒绝接收     |
| 22     | 0x16   | PUSH ss                  | SYN (synchronous idle)      | 同步空闲     |
| 23     | 0x17   | POP ss                   | ETB (end of trans. block)   | 结束传输块   |
| 24     | 0x18   | SBB reg8/mem8,reg8       | CAN (cancel)                | 取消         |
| 25     | 0x19   | SBB reg16/mem16,reg16    | EM (end of medium)          | 媒介结束     |
| 26     | 0x1A   | SBB reg8,reg8/mem8       | SUB (substitute)            | 代替         |
| 27     | 0x1B   | SBB reg16,reg16/mem16    | ESC (escape)                | 换码(溢出)   |
| 28     | 0x1C   | SBB al,immed8            | FS (file separator)         | 文件分隔符   |
| 29     | 0x1D   | SBB ax,immed16           | GS (group separator)        | 分组符       |
| 30     | 0x1E   | PUSH ds                  | RS (record separator)       | 记录分隔符   |
| 31     | 0x1F   | POP ds                   | US (unit separator)         | 单元分隔符   |
| 32     | 0x20   | AND reg8/mem8,reg8       | (space)                     | 空格         |
| 33     | 0x21   | AND reg16/mem16,reg16    | !                           | 叹号         |
| 34     | 0x22   | AND reg8,reg8/mem8       | “                           | 双引号       |
| 35     | 0x23   | AND reg16,reg16/mem16    | #                           | 井号         |
| 36     | 0x24   | AND al,immed8            | $                           | 美元符       |
| 37     | 0x25   | AND ax,immed16           | %                           | 百分号       |
| 38     | 0x26   | Segment override         | &                           | 和号         |
| 39     | 0x27   | DAA                      | ‘                           | 闭单引号     |
| 40     | 0x28   | SUB reg8/mem8,reg8       | (                           | 开括号       |
| 41     | 0x29   | SUB reg16/mem16,reg16    | )                           | 闭括号       |
| 42     | 0x2A   | SUB reg8,reg8/mem8       | *                           | 星号         |
| 43     | 0x2B   | SUB reg16,reg16/mem16    | +                           | 加号         |
| 44     | 0x2C   | SUB al,immed8            | ,                           | 逗号         |
| 45     | 0x2D   | SUB ax,immed16           | -                           | 减号/破折号  |
| 46     | 0x2E   | Segment override         | .                           | 句号         |
| 47     | 0x2F   | DAS                      | /                           | 斜杠         |
| 48     | 0x30   | XOR reg8/mem8,reg8       | 0                           | 字符0        |
| 49     | 0x31   | XOR reg16/mem16,reg16    | 1                           | 字符1        |
| 50     | 0x32   | XOR reg8,reg8/mem8       | 2                           | 字符2        |
| 51     | 0x33   | XOR reg16,reg16/mem16    | 3                           | 字符3        |
| 52     | 0x34   | XOR al,immed8            | 4                           | 字符4        |
| 53     | 0x35   | XOR ax,immed16           | 5                           | 字符5        |
| 54     | 0x36   | Segment override         | 6                           | 字符6        |
| 55     | 0x37   | AAA                      | 7                           | 字符7        |
| 56     | 0x38   | CMP reg8/mem8,reg8       | 8                           | 字符8        |
| 57     | 0x39   | CMP reg16/mem16,reg16    | 9                           | 字符9        |
| 58     | 0x3A   | CMP reg8,reg8/mem8       | :                           | 冒号         |
| 59     | 0x3B   | CMP reg16,reg16/mem16    | ;                           | 分号         |
| 60     | 0x3C   | CMP al,immed8            | <                           | 小于         |
| 61     | 0x3D   | CMP ax,immed16           | =                           | 等号         |
| 62     | 0x3E   | Segment override         | >                           | 大于         |
| 63     | 0x3F   | AAS                      | ?                           | 问号         |
| 64     | 0x40   | INC ax                   | @                           | 电子邮件符号 |
| 65     | 0x41   | INC cx                   | A                           | 大写字母A    |
| 66     | 0x42   | INC dx                   | B                           | 大写字母B    |
| 67     | 0x43   | INC bx                   | C                           | 大写字母C    |
| 68     | 0x44   | INC sp                   | D                           | 大写字母D    |
| 69     | 0x45   | INC bp                   | E                           | 大写字母E    |
| 70     | 0x46   | INC si                   | F                           | 大写字母F    |
| 71     | 0x47   | INC di                   | G                           | 大写字母G    |
| 72     | 0x48   | DEC ax                   | H                           | 大写字母H    |
| 73     | 0x49   | DEC cx                   | I                           | 大写字母I    |
| 74     | 0x4A   | DEC dx                   | J                           | 大写字母J    |
| 75     | 0x4B   | DEC bx                   | K                           | 大写字母K    |
| 76     | 0x4C   | DEC sp                   | L                           | 大写字母L    |
| 77     | 0x4D   | DEC bp                   | M                           | 大写字母M    |
| 78     | 0x4E   | DEC si                   | N                           | 大写字母N    |
| 79     | 0x4F   | DEC di                   | O                           | 大写字母O    |
| 80     | 0x50   | PUSH ax                  | P                           | 大写字母P    |
| 81     | 0x51   | PUSH cx                  | Q                           | 大写字母Q    |
| 82     | 0x52   | PUSH dx                  | R                           | 大写字母R    |
| 83     | 0x53   | PUSH bx                  | S                           | 大写字母S    |
| 84     | 0x54   | PUSH sp                  | T                           | 大写字母T    |
| 85     | 0x55   | PUSH bp                  | U                           | 大写字母U    |
| 86     | 0x56   | PUSH si                  | V                           | 大写字母V    |
| 87     | 0x57   | PUSH di                  | W                           | 大写字母W    |
| 88     | 0x58   | POP ax                   | X                           | 大写字母X    |
| 89     | 0x59   | POP cx                   | Y                           | 大写字母Y    |
| 90     | 0x5A   | POP dx                   | Z                           | 大写字母Z    |
| 91     | 0x5B   | POP bx                   | [                           | 开方括号     |
| 92     | 0x5C   | POP sp                   | \                           | 反斜杠       |
| 93     | 0x5D   | POP bp                   | ]                           | 闭方括号     |
| 94     | 0x5E   | POP si                   | ^                           | 脱字符       |
| 95     | 0x5F   | POP di                   | _                           | 下划线       |
| 96     | 0x60   | PUSHA                    | `                           | 开单引号     |
| 97     | 0x61   | POPA                     | a                           | 小写字母a    |
| 98     | 0x62   | BOUND reg16/mem16,reg16  | b                           | 小写字母b    |
| 99     | 0x63   | Not used                 | c                           | 小写字母c    |
| 100    | 0x64   | Not used                 | d                           | 小写字母d    |
| 101    | 0x65   | Not used                 | e                           | 小写字母e    |
| 102    | 0x66   | Not used                 | f                           | 小写字母f    |
| 103    | 0x67   | Not used                 | g                           | 小写字母g    |
| 104    | 0x68   | PUSH immed16             | h                           | 小写字母h    |
| 105    | 0x69   | IMUL reg16/mem16,immed16 | i                           | 小写字母i    |
| 106    | 0x6A   | PUSH immed8              | j                           | 小写字母j    |
| 107    | 0x6B   | IMUL reg8/mem8,immed8    | k                           | 小写字母k    |
| 108    | 0x6C   | INSB                     | l                           | 小写字母l    |
| 109    | 0x6D   | INSW                     | m                           | 小写字母m    |
| 110    | 0x6E   | OUTSB                    | n                           | 小写字母n    |
| 111    | 0x6F   | OUTSW                    | o                           | 小写字母o    |
| 112    | 0x70   | JO immed8                | p                           | 小写字母p    |
| 113    | 0x71   | JNO immed8               | q                           | 小写字母q    |
| 114    | 0x72   | JB immed8                | r                           | 小写字母r    |
| 115    | 0x73   | JNB immed8               | s                           | 小写字母s    |
| 116    | 0x74   | JZ immed8                | t                           | 小写字母t    |
| 117    | 0x75   | JNZ immed8               | u                           | 小写字母u    |
| 118    | 0x76   | JBE immed8               | v                           | 小写字母v    |
| 119    | 0x77   | JA immed8                | w                           | 小写字母w    |
| 120    | 0x78   | JS immed8                | x                           | 小写字母x    |
| 121    | 0x79   | JNS immed8               | y                           | 小写字母y    |
| 122    | 0x7A   | JP immed8                | z                           | 小写字母z    |
| 123    | 0x7B   | JNP immed8               | {                           | 开花括号     |
| 124    | 0x7C   | JL immed8                | \|                          |              |
| 125    | 0x7D   | JNL immed8               | }                           | 闭花括号     |
| 126    | 0x7E   | JLE immed8               | ~                           | 波浪号       |
| 127    | 0x7F   | JG immed8                | DEL (delete)                | 删除         |
| 128    | 0x80   | Table2 reg8              |                             |              |
| 129    | 0x81   | Table2 reg16             |                             |              |
| 130    | 0x82   | Table2 reg8              |                             |              |
| 131    | 0x83   | Table2 reg8, reg16       |                             |              |
| 132    | 0x84   | TEST reg8/mem8,reg8      |                             |              |
| 133    | 0x85   | TEST reg16/mem16,reg16   |                             |              |
| 134    | 0x86   | XCHG reg8,reg8           |                             |              |
| 135    | 0x87   | XCHG reg16,reg16         |                             |              |
| 136    | 0x88   | MOV reg8/mem8,reg8       |                             |              |
| 137    | 0x89   | MOV reg16/mem16,reg16    |                             |              |
| 138    | 0x8A   | MOV reg8,reg8/mem8       |                             |              |
| 139    | 0x8B   | MOV reg16,reg16/mem16    |                             |              |
| 140    | 0x8C   | MOV reg16/mem16,segReg   |                             |              |
| 141    | 0x8D   | LEA reg16,reg16/mem16    |                             |              |
| 142    | 0x8E   | MOV segReg,reg16/mem16   |                             |              |
| 143    | 0x8F   | POP reg16/mem16          |                             |              |
| 144    | 0x90   | NOP                      |                             |              |
| 145    | 0x91   | XCHG ax,cx               |                             |              |
| 146    | 0x92   | XCHG ax,dx               |                             |              |
| 147    | 0x93   | XCHG ax,bx               |                             |              |
| 148    | 0x94   | XCHG ax,sp               |                             |              |
| 149    | 0x95   | XCHG ax,bp               |                             |              |
| 150    | 0x96   | XCHG ax,si               |                             |              |
| 151    | 0x97   | XCHG ax,di               |                             |              |
| 152    | 0x98   | CBW 99CWD                |                             |              |
| 154    | 0x9A   | CALL immed32             |                             |              |
| 155    | 0x9B   | WAIT                     |                             |              |
| 156    | 0x9C   | PUSHF                    |                             |              |
| 157    | 0x9D   | POPF                     |                             |              |
| 158    | 0x9E   | SAHF                     |                             |              |
| 159    | 0x9F   | LAHF                     |                             |              |
| 160    | 0xA0   | MOV al,[mem8]            |                             |              |
| 161    | 0xA1   | MOV ax,[mem16]           |                             |              |
| 162    | 0xA2   | MOV [mem8],al            |                             |              |
| 163    | 0xA3   | MOV [mem16],ax           |                             |              |
| 164    | 0xA4   | MOVSB                    |                             |              |
| 165    | 0xA5   | MOVSW                    |                             |              |
| 166    | 0xA6   | CMPSB                    |                             |              |
| 167    | 0xA7   | CMPSW                    |                             |              |
| 168    | 0xA8   | TEST al,[mem8]           |                             |              |
| 169    | 0xA9   | TEST ax,[mem16]          |                             |              |
| 170    | 0xAA   | STOSB                    |                             |              |
| 171    | 0xAB   | STOSW                    |                             |              |
| 172    | 0xAC   | LODSB                    |                             |              |
| 173    | 0xAD   | LODSW                    |                             |              |
| 174    | 0xAE   | SCASB                    |                             |              |
| 175    | 0xAF   | SCASW                    |                             |              |
| 176    | 0xB0   | MOV al,immed8            |                             |              |
| 177    | 0xB1   | MOV cl,immed8            |                             |              |
| 178    | 0xB2   | MOV dl,immed8            |                             |              |
| 179    | 0xB3   | MOV bl,immed8            |                             |              |
| 180    | 0xB4   | MOV ah,immed8            |                             |              |
| 181    | 0xB5   | MOV ch,immed8            |                             |              |
| 182    | 0xB6   | MOV dh,immed8            |                             |              |
| 183    | 0xB7   | MOV bh,immed8            |                             |              |
| 184    | 0xB8   | MOV ax,immed16           |                             |              |
| 185    | 0xB9   | MOV cx,immed16           |                             |              |
| 186    | 0xBA   | MOV dx,immed16           |                             |              |
| 187    | 0xBB   | MOV bx,immed16           |                             |              |
| 188    | 0xBC   | MOV sp,immed16           |                             |              |
| 189    | 0xBD   | MOV bp,immed16           |                             |              |
| 190    | 0xBE   | MOV si,immed16           |                             |              |
| 191    | 0xBF   | MOV di,immed16           |                             |              |
| 192    | 0xC0   | Table1 reg8              |                             |              |
| 193    | 0xC1   | Table1 reg8, reg16       |                             |              |
| 194    | 0xC2   | RET immed16              |                             |              |
| 195    | 0xC3   | RET                      |                             |              |
| 196    | 0xC4   | LES reg16/mem16,mem16    |                             |              |
| 197    | 0xC5   | LDS reg16/mem16,mem16    |                             |              |
| 198    | 0xC6   | MOV reg8/mem8,immed8     |                             |              |
| 199    | 0xC7   | MOV reg16/mem16,immed16  |                             |              |
| 200    | 0xC8   | ENTER immed16, immed8    |                             |              |
| 201    | 0xC9   | LEAVE                    |                             |              |
| 202    | 0xCA   | RET immed16              |                             |              |
| 203    | 0xCB   | RET                      |                             |              |
| 204    | 0xCC   | INT 3                    |                             |              |
| 205    | 0xCD   | INT immed8               |                             |              |
| 206    | 0xCE   | INTO                     |                             |              |
| 207    | 0xCF   | IRET                     |                             |              |
| 208    | 0xD0   | Table1 reg8              |                             |              |
| 209    | 0xD1   | Table1 reg16             |                             |              |
| 210    | 0xD2   | Table1 reg8              |                             |              |
| 211    | 0xD3   | Table1 reg16             |                             |              |
| 212    | 0xD4   | AAM                      |                             |              |
| 213    | 0xD5   | AAD                      |                             |              |
| 214    | 0xD6   | Not used                 |                             |              |
| 215    | 0xD7   | XLAT [bx]                |                             |              |
| 216    | 0xD8   | ESC immed8               |                             |              |
| 217    | 0xD9   | ESC immed8               |                             |              |
| 218    | 0xDA   | ESC immed8               |                             |              |
| 219    | 0xDB   | ESC immed8               |                             |              |
| 220    | 0xDC   | ESC immed8               |                             |              |
| 221    | 0xDD   | ESC immed8               |                             |              |
| 222    | 0xDE   | ESC immed8               |                             |              |
| 223    | 0xDF   | ESC immed8               |                             |              |
| 224    | 0xE0   | LOOPNE immed8            |                             |              |
| 225    | 0xE1   | LOOPE immed8             |                             |              |
| 226    | 0xE2   | LOOP immed8              |                             |              |
| 227    | 0xE3   | JCXZ immed8              |                             |              |
| 228    | 0xE4   | IN al,immed8             |                             |              |
| 229    | 0xE5   | IN ax,immed16            |                             |              |
| 230    | 0xE6   | OUT al,immed8            |                             |              |
| 231    | 0xE7   | OUT ax,immed16           |                             |              |
| 232    | 0xE8   | CALL immed16             |                             |              |
| 233    | 0xE9   | JMP immed16              |                             |              |
| 234    | 0xEA   | JMP immed32              |                             |              |
| 235    | 0xEB   | JMP immed8               |                             |              |
| 236    | 0xEC   | IN al,dx                 |                             |              |
| 237    | 0xED   | IN ax,dx                 |                             |              |
| 238    | 0xEE   | OUT al,dx                |                             |              |
| 239    | 0xEF   | OUT ax,dx                |                             |              |
| 240    | 0xF0   | LOCK                     |                             |              |
| 241    | 0xF1   | Not used                 |                             |              |
| 242    | 0xF2   | REPNE                    |                             |              |
| 243    | 0xF3   | REP                      |                             |              |
| 244    | 0xF4   | HLT                      |                             |              |
| 245    | 0xF5   | CMC                      |                             |              |
| 246    | 0xF6   | Table3 reg8              |                             |              |
| 247    | 0xF7   | Table3 reg16             |                             |              |
| 248    | 0xF8   | CLC                      |                             |              |
| 249    | 0xF9   | STC                      |                             |              |
| 250    | 0xFA   | CLI                      |                             |              |
| 251    | 0xFB   | STI                      |                             |              |
| 252    | 0xFC   | CLD                      |                             |              |
| 253    | 0xFD   | STD                      |                             |              |
| 254    | 0xFE   | Table4 reg8              |                             |              |
| 255    | 0xFF   | Table4 reg16             |                             |              |