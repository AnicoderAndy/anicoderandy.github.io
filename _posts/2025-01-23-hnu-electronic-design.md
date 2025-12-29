---
layout: post
title:  "HNU 2024 级 电路与电子学 模型机综合设计实验报告"
date:   2025-01-23 22:36:00 +0800
categories: study
---

## 前言
HNU 信院的电子电路感觉是挑战性很大的一门课，综合设计当中也会遇到很多课上不会讲的知识，网上关于 Quartus II 软件的相关资料也是少之又少。这学期的课程，本人也是在各种跌跌撞撞中查阅资料、向 AI 提问的曲折过程中，最终才实现了模型机的综合设计。这篇博客希望能给以后的同学当作参考，但也希望大家不要抄袭，独立完成之后获得的成就感是无论如何都无法取代的：完成每一个元件的代码和波形测试、把每一个自己编写的元件连接在一起、用简陋的指令集实现一个程序——这其中每一个过程对我来说都是新奇且具有挑战性的。

祝大家顺利完成综合设计。

## 一、设计目的
以“培养现代数字系统设计能力”为目标，贯彻以 CPU 设计为核心，以层次化、模块化设计方法为抓手的组织思路，培养设计与实现数字系统的能力。

完整、连贯地运用课程所学知识，熟练掌握现代 EDA 工具基本使用方法，为后续课程学习和今后从事相关工作打下良好的基础或做下一些铺垫。

## 二、设计内容
1. 按照给定的数据通路、数据格式和指令系统，使用 EDA 工具设计一台用硬连线逻辑控制的简易计算机；
2. 用所设计的计算机控制模拟通道对外界模拟输入信号进行采样、离散和处理，并显示在数码显示管上。

## 三、详细设计
### 3.1 模型机的设计思路
1. 根据自上到下的设计方式，了解模型机整体的设计思路，根据整体原理设计下层结构功能。
2. 利用分块设计，使用 Verilog HDL 语言对每个模块进行设计，通过功能波形仿真和时序波形仿真检查功能是否实现，分析元件性能表现。
3. 根据自下至上的原则，依据整体架构对封装好的各板块进行连接，每连接一个板块进行一次仿真验证，确保无误后再进行下一个组件的连接，直到整个模型机设计完毕。

### 3.2 模型机的整体架构
参见 [AI 麥爾的项目](https://gitee.com/aimaier4869/vspm-ts/tree/495eeb24f2079ef7cca248ca2a889c9dcc109cee)。
![模型机的架构图](https://s2.loli.net/2025/01/09/wkvC2bAxjPhydNl.jpg)

### 3.3 各模块的具体实现
#### 3.3.1. 指令译码器

![ins_decode 元件图](https://s2.loli.net/2025/01/09/Z286XkdyWjhgGEF.png)

接口设计：见上图。模块输入为使能信号 en 和机器码 ir[3..0]，输出指令信号 mova, movb, movc, movd, add, sub, jmp, jg, in1, out1, movi, halt 将会接入控制信号发生逻辑上。

| 输入 en | 输入 Ir[3:0] | mova | movb | movc | movd | add | sub | jmp | jg  | in1 | out1 | movi | halt |
|:------:|:---------:|:------:|:------:|:------:|:------:|:-----:|:-----:|:-----:|:-----:|:-----:|:------:|:------:|:------:|
| 0    | xxxx    | 0    | 0    | 0    | 0    | 0   | 0   | 0   | 0   | 0   | 0    | 0    | 0    |
| 1    | 0100    | 1    | 0    | 0    | 0    | 0   | 0   | 0   | 0   | 0   | 0    | 0    | 0    |
| 1    | 0101    | 0    | 1    | 0    | 0    | 0   | 0   | 0   | 0   | 0   | 0    | 0    | 0    |
| 1    | 0110    | 0    | 0    | 1    | 0    | 0   | 0   | 0   | 0   | 0   | 0    | 0    | 0    |
| 1    | 0111    | 0    | 0    | 0    | 1    | 0   | 0   | 0   | 0   | 0   | 0    | 0    | 0    |
| 1    | 1000    | 0    | 0    | 0    | 0    | 1   | 0   | 0   | 0   | 0   | 0    | 0    | 0    |
| 1    | 1001    | 0    | 0    | 0    | 0    | 0   | 1   | 0   | 0   | 0   | 0    | 0    | 0    |
| 1    | 1010    | 0    | 0    | 0    | 0    | 0   | 0   | 1   | 0   | 0   | 0    | 0    | 0    |
| 1    | 1011    | 0    | 0    | 0    | 0    | 0   | 0   | 0   | 1   | 0   | 0    | 0    | 0    |
| 1    | 1100    | 0    | 0    | 0    | 0    | 0   | 0   | 0   | 0   | 1   | 0    | 0    | 0    |
| 1    | 1101    | 0    | 0    | 0    | 0    | 0   | 0   | 0   | 0   | 0   | 1    | 0    | 0    |
| 1    | 1110    | 0    | 0    | 0    | 0    | 0   | 0   | 0   | 0   | 0   | 0    | 1    | 0    |
| 1    | 1111    | 0    | 0    | 0    | 0    | 0   | 0   | 0   | 0   | 0   | 0    | 0    | 1    |
| 1    | 其它组合 | 0    | 0    | 0    | 0    | 0   | 0   | 0   | 0   | 0   | 0    | 0    | 0    |

功能实现：当译码器的使能信号为 1 时，根据传入的机器码将对应端口输出为 1，其他端口输出 0。具体功能见上表。

Verilog HDL 实现：

```verilog
module ins_decode(
	input [3:0] ir,
	input en,
	output reg mova,
	output reg movb,
	output reg movc,
	output reg movd,
	output reg add,
	output reg sub,
	output reg jmp,
	output reg jg,
	output reg in1,
	output reg out1,
	output reg movi,
	output reg halt
);

always @(ir, en) begin
	mova = 1'b0;
	movb = 1'b0;
	movc = 1'b0;
	movd = 1'b0;
	add = 1'b0;
	sub = 1'b0;
	jmp = 1'b0;
	jg = 1'b0;
	in1 = 1'b0;
	out1 = 1'b0;
	movi = 1'b0;
	halt = 1'b0;
	
	if (en == 1) begin
		if (ir[3:0] == 4'b0100) mova = 1;
		else if (ir[3:0] == 4'b0101) movb = 1;
		else if (ir[3:0] == 4'b0110) movc = 1;
		else if (ir[3:0] == 4'b0111) movd = 1;
		else if (ir[3:0] == 4'b1000) add = 1;
		else if (ir[3:0] == 4'b1001) sub = 1;
		else if (ir[3:0] == 4'b1010) jmp = 1;
		else if (ir[3:0] == 4'b1011) jg = 1;
		else if (ir[3:0] == 4'b1100) in1 = 1;
		else if (ir[3:0] == 4'b1101) out1 = 1;
		else if (ir[3:0] == 4'b1110) movi = 1;
		else if (ir[3:0] == 4'b1111) halt = 1;
	end
end

endmodule
```

#### 3.3.2. 算术单元
![AU 元件图](https://s2.loli.net/2025/01/09/m9WjbXlpOoQrTyg.png)

接口设计：见上图。共 4 个输入端口，包括来自通用寄存器的数据来源 a[7..0], b[7..0]，以及来在控制信号发生逻辑的 ac[3..0] 指令码和使能信号 au_en。输出端口 t[7..0] 表示计算结果，gf 表示 sub 运算得到的标识。

| au_en | ac[3..0]            | t[7..0]                  | gf                            |
|:-------:|:---------------------:|:--------------------------:|:-------------------------------:|
| 1     | 1000                | t = a + b               | 不影响                       |
| 1     | 1001                | t = b - a               | IF(b > a), THEN gf = 1, ELSE gf = 0 |
| 1     | 0100、0101、1101    | t = a                   | 不影响                       |
| 1     | 其它组合            | t = 8’hZZ               | 不影响                       |
| 0     | XXXX                | t = 8’hZZ               | 不影响                       |

功能实现：根据输入的指令码，实现加法运算、减法运算、直接经过三种功能。具体功能见上表。

Verilog HDL 实现：

```verilog
module au(
	input au_en,
	input [3:0] ac,
	input [7:0] a,
	input [7:0] b,
	output reg [7:0] t,
	output reg gf
);

always @(au_en, ac, a, b) begin
	gf = 1'b0;
	t = 8'b0000_0000;
	if (au_en == 1'b0) begin
		t = 8'hZZ;
	end
	else begin
		if (ac == 4'b1000) t = a + b;
		else if (ac == 4'b1001) begin
			t = b + (~a) + 8'b0000_0001;
			if ((t[7] == 0) & (t[0] | t[1] | t[2] | t[3] | t[4] | t[5] | t[6]) == 1) gf = 1;
			else gf = 0;
		end
		else if (ac == 4'b0100 || ac == 4'b0101 || ac == 4'b1101) t = a;
		else t = 8'hZZ;
	end
end

endmodule
```
 
#### 3.3.3. 八重 3-1 多路复用器
![八重 MUX3-1 元件图](https://s2.loli.net/2025/01/09/EpmaR98cwLVyFsB.png)

接口设计：见上图。八重 3-1 多路复用器有 3 个八位输入 a[7:0]、b[7:0]、c[7:0]，1 个输出 y[7:0]，称之为八重，s[1:0] 选择将哪个输入传至输出。

| S[1..0] | Y |
|:---------:|:---:|
| 00      | a |
| 01      | b |
| 10      | c |
| 其他    | a |

功能实现：见上表。

Verilog HDL 实现：

```verilog
module mux3_1(
	input [7:0] a,
	input [7:0] b,
	input [7:0] c,
	input [1:0] s,
	output reg [7:0] y
);
always @(*) begin
	if (s == 2'b01) y = b;
	else if (s == 2'b10) y = c;
	else y = a;
end
endmodule
```

#### 3.3.4. 八重 2-1 多路复用器
![八重 MUX2-1 元件图](https://s2.loli.net/2025/01/09/thH7KDOeXM9wdjT.png)

接口设计：见上图。输入包含选择信号 s 和两个 8 位输入 a[7..0], b[7..0]。输出 y[7..0] 由 s 选择。

| S | Y |
|:-:|:-:|
| 0 | a |
| 1 | b |

功能实现：见上表。

Verilog HDL 实现：

```verilog
module mux2_1(
	input [7:0] a,
	input [7:0] b,
	input s,
	output reg [7:0] y
);
always @(*) begin
	if (s == 1'b0) y = a;
	else y = b;
end
endmodule
```
 
#### 3.3.5. 控制信号产生逻辑
![con_signal 元件图](https://s2.loli.net/2025/01/09/YAq8PJWcZas75Sm.png)

接口设计：见上图。包含 15 个输入端口，其中 12 个与指令译码器的输出端口对应，另外 g 由状态寄存器输出，ir[7..0] 为输入指令，sm 信号控制当前为读取还是执行周期。输出包含 16 个控制信号。

功能实现：控制信号产生逻辑接受指令译码器的输出，在 sm 以及状态 g 的配合下产生每个模块需要的控制信号。

Verilog HDL 实现：

```verilog
module con_signal(
	input mova,
	input movb,
	input movc,
	input movd,
	input add,
	input sub,
	input jmp,
	input jg,
	input g,
	input in1,
	input out1,
	input movi,
	input halt,
	input [7:0] ir,
	input sm,
	output reg sm_en,
	output reg ir_ld,
	output reg ram_re,
	output reg ram_wr,
	output reg pc_ld,
	output reg pc_in,
	output reg [1:0] reg_sr,
	output reg [1:0] reg_dr,
	output reg reg_we,
	output reg [1:0] s,
	output reg au_en,
	output reg [3:0] au_ac,
	output reg gf_en,
	output reg in_en,
	output reg out_en,
	output reg mux_s
);

always @(*) begin
	sm_en = ~halt;
	ir_ld = ~sm;
	ram_re = (~sm) | movc | movi;
	ram_wr = movb;
	pc_ld = jmp | (jg & g);
	pc_in = movi | (~sm);
	reg_sr = ir[1:0];
	reg_dr = ir[3:2];
	reg_we = movi | mova | movc | movd | sub | add | in1;
	if (movb) s = 2'b10;
	else if (movc) s = 2'b01;
	else s = 2'b00;
	au_en = mova | movb | add | out1 | sub;
	au_ac = ir[7:4];
	gf_en = sub;
	in_en = in1;
	out_en = out1;
	mux_s = mova | movi | add | sub | in1 | movc;
end

endmodule
```
 
#### 3.3.6. 状态机 SM
![SM 元件图](https://s2.loli.net/2025/01/10/SWL9rfqUcoOiC2u.png)

接口设计：见上图。输入时钟 clk 和使能信号 sm_en，输出 sm 指示当前周期信息。

|   CLK  | SM_EN |           功能          |      备注     |
|:------:|:-----:|:-----------------------:|:-------------:|
| 下降沿 |   1   | SM $\Leftarrow$ SM 取反 | SM 初始值为 0 |

功能实现：见上表。SM 为 0 是读取指令周期；SM 为 1 是执行指令周期。

Verilog HDL 实现：

```verilog
module sm(
	input clk,
	input sm_en,
	output sm
);

reg state = 1'b0;

always @(negedge clk) begin
	if (sm_en == 1'b1) state <= ~state;
end

assign sm = state;

endmodule
```

#### 3.3.7. 指令寄存器 IR
![IR 元件图](https://s2.loli.net/2025/01/10/TP9xwLiRHljU1bN.png)

接口设计：见上图。输入时钟 clk，控制信号 ld_ir 和指令 a[7..0]。输出指令 x[7..0]。

|   CLK  | Ld_ir |           功能          |      备注     |
|:------:|:-----:|:-----------------------:|:-------------:|
| 下降沿 |   1   | a[7:0] 写入 x[7:0] | X 初始值为 00000000 |

功能实现：见上表。当时钟处在下降沿且控制信号为 1 时将总线数据写入寄存器。

Verilog HDL 实现：

```verilog
module ir(
	input clk,
	input ld_ir,
	input [7:0] a,
	output reg [7:0] x = 8'b0000_0000
);

always @(negedge clk) begin
	if (ld_ir == 1'b1) x <= a;
end

endmodule
```
 
#### 3.3.8. 状态寄存器 PSW
![PSW 元件图](https://s2.loli.net/2025/01/10/abXYNhWTMcmsfBJ.png)

接口设计：见上图。输入时钟 clk，使能信号 g_en 和状态信号 g。输出状态信号 gf。

|   CLK  | g_en |           功能          |      备注     |
|:------:|:-----:|:-----------------------:|:-------------:|
| 下降沿 |   1   | g 写入 gf | gf 初始值为 0 |

功能实现：见上表。本模型机 PSW 用来存放 SUB 指令执行结果的状态标志。

Verilog HDL 实现：

```verilog
module psw(
	input clk,
	input g,
	input g_en,
	output reg gf = 1'b0
);

always @(negedge clk) begin
	if (g_en == 1'b1) gf <= g;
end

endmodule
```

#### 3.3.9. 指令计数器 PC
![PC 元件图](https://s2.loli.net/2025/01/10/AjerZhnEOyLFfIu.png)

接口设计：见上图。输入时钟 clk，控制信号 ld_pc 和 in_pc，以及装载数据 a[7..0]。输出当前地址 c[7..0]。

|   CLK  | in_pc |           功能          |      备注     |
|:------:|:-----:|:-----------------------:|:-------------:|
| 下降沿 |   1   | c[7..0] 中数据自加 1 | PC 初始值为 00000000（第一条指令在 RAM 中的存放地址） |
| 下降沿 |   0   | a[7..0] 写入 c[7..0] |   |

功能实现：见上表。根据不同的输入信号使寄存器中的地址自增 1 或是装载新的地址。

Verilog HDL 实现：

```verilog
module pc(
	input ld_pc,
	input in_pc,
	input clk,
	input [7:0] a,
	output reg [7:0] c = 8'h00
);

always @(negedge clk) begin
	if (in_pc == 1'b1 && ld_pc == 1'b0) c <= c + 8'h01;
	else if (in_pc == 1'b0 && ld_pc == 1'b1) c <= a;
end

endmodule
```
 
#### 3.3.10. 通用寄存器组
![reg_group 元件图](https://s2.loli.net/2025/01/10/lPoINJ3qSvbMTXU.png)

接口设计：见上图。输入时钟 clk，控制信号 sr[1..0] 和 dr[1..0]，以及装载数据 i[7..0]。根据控制信号输出当前寄存器的值 s[7..0], d[7..0]。

| 操作 | CLK        | WE  | 功能                                                                                     |
|:------:|:------------:|:------:|:------------------------------------------------------------------------------------------:|
| 读   |            |      | 根据 SR[1..0] 的值从 R0、R1、R2、R3 中选择一个寄存器的值从 S 口输出。根据 DR[1..0] 的值从 R0、R1、R2、R3 中选择一个寄存器的值从 D 口输出 |
| 写   | 下降沿   | 1    | 控制信号 WE 为 1，根据 DR[1..0] 的值，在 CLK 下降沿将外部输入 A 写入 R0、R1、R2、R3 中的一个寄存器。  |

功能实现：见上表。根据 sr 和 dr 输出对应寄存器的存储值（组合逻辑）。在时钟下降沿将 i 端口的数据写入 dr 对应的寄存器（时序逻辑）。

Verilog HDL 实现：

```verilog
module reg_group(
	input we,
	input clk,
	input [1:0] sr,
	input [1:0] dr,
	input [7:0] i,
	output reg [7:0] s,
	output reg [7:0] d
);

reg [7:0] r0 = 8'h01;
reg [7:0] r1 = 8'h00;
reg [7:0] r2 = 8'h00;
reg [7:0] r3 = 8'h07;

always @(negedge clk) begin
	case ({we, dr})
		3'b100: r0 <= i;
		3'b101: r1 <= i;
		3'b110: r2 <= i;
		3'b111: r3 <= i;
	endcase
end

always @(*) begin
	case(sr)
		2'b00: s = r0;
		2'b01: s = r1;
		2'b10: s = r2;
		2'b11: s = r3;
	endcase
	
	case(dr)
		2'b00: d = r0;
		2'b01: d = r1;
		2'b10: d = r2;
		2'b11: d = r3;
	endcase
end

endmodule
```
 
#### 3.3.11. 随机存取存储器 RAM
![RAM 元件图](https://s2.loli.net/2025/01/10/tkiB6ZmdCRT7qwO.png)

接口设计：见上图。输入地址 address[] 和时钟 inclock，控制信号 we 和 outenab。

功能实现：当 we 和 outenab 均为 0 时，dio 端口保持高阻态。当 we 为 1，outenab 为 0 时，在时钟上升沿将把 address 处的数据从 dio 输出。当 we 为 0，outenab 为 1 时，在时钟上升沿将 dio 端口处的数据写入 address 地址。

## 四、系统测试
### 4.1 测试环境
开发软件：	Quartus II Version 9.0 Build 235 06/17/2009 SJ web Edition

操作系统：	Windows 11 家庭中文版 24H2

FPGA 型号：	Cyclone II EP2C5T144C8

### 4.2 测试代码
#### 代码一
该代码系作业提供的示范代码。

| RAM 地址 | 汇编指令        | 机器码（RAM 存放内容） | 说明                                        |
|:------:|:-----------:|:-------------:|:-----------------------------------------:|
| 0      | JMP         | 1010 0011     | R3 的初始值为 07h，指令执行完，PC=07h                 |
| 1      |             | 0001 0001     | R0 的初始值为 01h，RAM 的 01 地址单元存放 11h          |
| 7      | IN R1       | 1100 0100     | 外部引脚输入 01h，指令执行完，R1=01h                   |
| 8      | MOVA R2, R1 | 0100 1001     | 指令执行完，R2=01h                              |
| 9      | MOVC R1, M  | 0110 0100     | RAM 的 01 单元内容 11h 赋给 R1，指令执行完，R1=11h      |
| 10     | SUB R1, R2  | 1001 0110     | 指令执行完，R1=10h 且 G=1                        |
| 11     | MOVD R3, PC | 0111 1100     | 取指后 PC 自加 1，此时 PC=12，那么 R3=12=0Ch         |
| 12     | ADD R3, R1  | 1000 1101     | 指令执行完，R3=1Ch=28                           |
| 13     | JG          | 1011 0011     | G=1，满足跳转条件，执行跳转，PC=28                     |
| 28     | MOVB M, R3  | 0101 0011     | 将 R3 中的数据写入 RAM                           |
| 29     | MOVC R1, M  | 0110 0100     | 将 RAM 的数据写入 R1，两条指令执行完，R1=R3=1Ch          |
| 30     | MOVI #9     | 1110 0000     | 此为两字节指令，指令执行完后，R0=09H                     |
| 31     |             | 0000 1001     |                                           |
| 32     | SUB R0, R1  | 1001 0001     | 指令执行完，R0=EDh 且 G=0                        |
| 33     | JG          | 1011 0011     | G=0，不满足跳转条件，不执行跳转                         |
| 34     | OUT R0      | 1101 0000     | 指令执行完，输出引脚为 EDh                           |
| 35     | HALT        | 1111 0000     | 停机指令                                      |
| 36     | JMP         | 1010 0011     | 此指令在停机指令后用于检测停机是否正确，正确的停机指令应该是停机后面的指令不执行  |

#### 代码二
以下代码实现了一个 $a\times b$ 的乘法功能。其中乘数 $a$ 从外部读入，乘数$b$ 存储在 `RAM[2]` 位置。要求乘数 $a$ 以有符号数补码形式输入，乘数 $b$ 为非负整数。该代码会持续循环运行，每次根据读入的数据更新乘法结果，持续输出。若输入数字小于 $0$ 则停机。
R3 的初始值设为 `0d07`。

|      地址（Dec）     |       汇编指令     |       机器码     |          解释（Dec）        |
|:--------------------:|:------------------:|:----------------:|:---------------------------:|
|           0          |         JMP        |     1010 0011    |            Goto 07          |
|           1          |                    |     0000 0000    |     RAM[1] 用来存运算次数    |
|           2          |                    |     0000 0100    |       RAM[2] 用来存乘数 b     |
|           3          |                    |     0000 0000    |       RAM[3] 用来存结果      |
|           7          |        IN R2       |     1100 1000    |           R2=Input          |
|           8          |       MOVI #2      |     1110 0000    |             R0=2            |
|           9          |                    |     0000 0010    |                             |
|           10         |      MOVC R1, M    |     0110 0100    |          R1=RAM[2]=b        |
|           11         |       MOVI #1      |     1110 0000    |             R0=1            |
|           12         |                    |     0000 0001    |                             |
|           13         |      MOVB M, R1    |     0101 0001    |           RAM[1]=b          |
|           14         |      SUB R0, R1    |     1001 0001    |        R0=1-R1, G=1>R1      |
|           15         |       MOVI #36     |     1110 0000    |             R0=36           |
|           16         |                    |     0010 0100    |                             |
|           17         |     MOVA R3, R0    |     0100 1100    |           R3=R0=36          |
|           18         |          JG        |     1011 0011    |        If (G) Goto 36       |
|           19         |       MOVI #3      |     1110 0000    |             R0=3            |
|           20         |                    |     0000 0011    |                             |
|           21         |      MOVC R1, M    |     0110 0100    |       将结果先复制给 R1      |
|           22         |      ADD R1, R2    |     1000 0110    |           R1=R1+R2          |
|           23         |      MOVB M, R1    |     0101 0001    |           RAM[3]=R1         |
|           24         |       MOVI #1      |     1110 0000    |        RAM[1]=RAM[1]-1      |
|           25         |                    |     0000 0001    |                             |
|           26         |      MOVC R1, M    |     0110 0100    |                             |
|           27         |      SUB R1, R0    |     1001 0100    |                             |
|           28         |      MOVB M, R1    |     0101 0001    |                             |
|           29         |       MOVI #18     |     1110 0000    |             R0=18           |
|           30         |                    |     0001 0010    |                             |
|           31         |     MOVD R3, PC    |     0111 1100    |             R3=32           |
|           32         |      SUB R3, R0    |     1001 1100    |          R3=32-18=14        |
|           33         |       MOVI #1      |     1110 0000    |             R0=1            |
|           34         |                    |     0000 0001    |                             |
|           35         |         JMP        |     1010 0011    |            Goto 14          |
|           36         |       MOVI #0      |     1110 0000    |             R0=0            |
|           37         |                    |     0000 0000    |                             |
|           38         |      SUB R0, R2    |     1001 0010    |       R0=R0-R2, G=R0>R2     |
|           39         |       MOVI #57     |     1110 0000    |             R0=57           |
|           40         |                    |     0011 1001    |                             |
|           41         |     MOVA R3, R0    |     0100 1100    |           R3=R0=57          |
|           42         |          JG        |     1011 0011    |        If (G) Goto 57       |
|           43         |       MOVI #3      |     1110 0000    |             R0=3            |
|           44         |                    |     0000 0011    |                             |
|           45         |      MOVC R1, M    |     0110 0100    |           R1=RAM[3]         |
|           46         |        OUT R1      |     1101 0001    |         Output RAM[3]       |
|           47         |       MOVI #7      |     1110 0000    |             R0=7            |
|           48         |                    |     0000 0111    |                             |
|           49         |     MOVA R3, R0    |     0100 1100    |            R3=R0=7          |
|           50         |       MOVI #0      |     1110 0000    |             R2=0            |
|           51         |                    |     0000 0000    |                             |
|           52         |     MOVA R2, R0    |     0100 1000    |                             |
|           53         |       MOVI #3      |     1110 0000    |          RAM[3]=R2=0        |
|           54         |                    |     0000 0011    |                             |
|           55         |      MOVB M, R2    |     0101 0010    |                             |
|           56         |         JMP        |     1010 0011    |            Goto 7           |
|           57         |         HALT       |     1111 0000    |             Quit            |
|           58         |        OUT R3      |     1101 0011    |       测试是否正常停机        |

### 4.3 测试结果
代码一功能仿真（输入 0x01）：根据代码，最终应当输出 0xED，波形符合预期，正常停机

![代码 1 功能仿真波形](https://s2.loli.net/2025/01/10/UZAf1e49jgL7a6h.png)

代码一时序仿真（输入 0x01）：

![代码 1 时序仿真波形](https://s2.loli.net/2025/01/10/fJ9O52LajVuQsSl.png)

代码二功能仿真（先输入 2，后输入 -1）：输出 4 符合预期，寄存器中间数值均符合预期，输入 -1 时正常停机

![代码 2 功能仿真波形-I](https://s2.loli.net/2025/01/10/nDvVuwyhdsBFOCL.png)

![代码 2 功能仿真波形-II](https://s2.loli.net/2025/01/10/iRJejMxdX24Bgty.png)

代码二时序仿真（先输入 5，后输入 -3）：

![代码 2 功能时序波形-I](https://s2.loli.net/2025/01/10/EfRbho5INYZFG4A.png)

![代码 2 功能时序波形-II](https://s2.loli.net/2025/01/10/nvdQCYygIwVM3DG.png)

### 4.4 模型机性能分析
![时序性能](https://s2.loli.net/2025/01/10/peHJT65mksDlRP3.png)

![编译情况](https://s2.loli.net/2025/01/10/jiDu394oskdPrQA.png)

![综合性能](https://s2.loli.net/2025/01/10/erQVhIbN5En7T2f.png)

性能表现良好，时钟周期为 23.35ns，warning 为 13 条。

## 五、实验总结、必得体会及建议
### 5.1 从需要掌握的理论、遇到的困难、解决的办法以及经验教训等方面进行总结。
本模型机需要掌握数字电路的基础原理，需要学会使用 Quartus II 软件编写具有对应功能的模块与元件，熟悉计算机数据和信号流动的过程以及模型机的基本结构，掌握如何组装并设计简易的模型机，了解如何在 FPGA 实验板上验证功能。

在实验过程中，本人遇到了编写测试代码的困难，是由于对汇编编程的不熟悉导致的，在了解和学习汇编的基本方法后顺利完成了代码。在测试代码时，还发现部分指令的执行和指导书中的功能描述有细微差别，在后续修改了元件代码实现了功能统一，在解决这一困难的过程中认识到了加强小模块事先测试的重要性。
