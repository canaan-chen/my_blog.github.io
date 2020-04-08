---
title: "异步fifo的设计"
date: 2020-03-15T17:58:18+08:00
categories: ["一个IC工程师的自我修养"]
tags: ["IC Design","FIFO"]
draft: false
---

这篇文章主要讲解了如何设计一个异步Fifo。

<!--more-->

## 1. 资料

[异步FIFO设计（1）](https://zhuanlan.zhihu.com/p/42991844)

[异步FIFO设计原理及Verliog源代码_网络_deng_d1的博客-CSDN博客](https://blog.csdn.net/deng_d1/article/details/50179677)

## 2. 原理介绍

{% asset_img 1.png %}

### 2.1 使用FIFO的情况

- 两个不同时钟域进行数据传输时可以用FIFO
- 对于不同宽度的数据接口也可以用FIFO， 例如单片机8位输出，而DSP 16位数据输入。

### 2.2 FIFO的相关参数

- FIFO的宽度：表示FIFO进行一次读写操作的数据位。
- FIFO的深度：指的是FIFO能存储多少个N位的数据，N为宽度。
- 满标志：FIFO已满或将要满时由FIFO的状态电路送出的一个信号，以阻止FIFO的写操作继续向FIFO中写数据而造成溢出（overflow）
- 空标志：FIFO已空或将要空时由FIFO的状态电路送出的一个信号，以阻止FIFO的读操作继续从FIFO中读出数据而造成无效数据的读出（underflow）。
- 读时钟：读数据所遵循的时钟。
- 写时钟：写操作所需要遵循的时钟。
- 读指针：指向下一个读出地址。读完后自动加1
- 写指针：指向下一个写入地址，写完后自动加1

### 2.3 空满状态的判断

[FIFO空满判断与地址转换的思考_网络_zyn1347806的博客-CSDN博客](https://blog.csdn.net/zyn1347806/article/details/79626452)

[异步fifo设计（1）](https://www.cnblogs.com/xh13dream/p/9265042.html)

- 用格雷码来作地址判断（多引入一位作位判断，比如设计深度为8，宽度为8的异步FIFO，设计其指针位数为4,即 n+1） 
  - 当最高位和次高位相同时，其余位相同认为是读空
  - 当最高位和次高位不同时，其余位相同认为是写满

### 2.4 读空标志位的产生

[异步FIFO设计(非常详细，图文并茂，值得一看！）](https://www.sohu.com/a/114158723_458015)

FIFO中存在两种指针，rd_ptr 和 wr_ptr。只有两种状态下FIFO才会为空：

- 系统复位，读写指针都被清空。
- 读出速度大于写入速度，读地址赶上写地址。

空标志位的产生要在读时钟域里完成，这样不至于FIFO为空时，而空标志位还没有发生，但是可能发生FIFO里已经有数据了，但是空标志位还是没有被撤销，不过就算在最坏的情况下，空标志撤销也只是滞后3个周期（因为会需要时钟的同步，经过两级触发器）。还有一种情况，就是空标志比较逻辑检测到读地址和写地址相同后，紧接着发生写操作，导致写地址加1，由于同步模块的滞后性，导致没法及时更新写地址，会产生一个虚假的空信号，称作“虚空”。

### 2.5 读满标志位的产生

读满状态可以理解为，读地址超写地址一圈，两个地址仍然在同一个地方，这时候引入指示位来进行区别和判断。

读满状态判断：

1. **最高位相异，因为两个指针速度不同， 写超前于读。**

2. **出去最高位，次高位取反后两者相同。比如：写指针已经走了一圈了跑到3，此时写指针：1110，而读指针第一次走到3，指针数值为：0010。⇒ 发现最高位和次高位均不同，表示满状态。**

3. **相应的，如果最高位和次高位均相同则表示空状态。**

   {% asset_img Untitled.png %}

## 3.代码的实现

这里以一个8x8的fifo作为案例来进行代码的设计，之后再做一个fifo的封装实现自定义化模块。

### 3.1 格雷码转换

需要用Gray Code来实现地址，从而降低亚稳态。

基本的转换关系：

- 二进制转格雷码： 
  - G[ n-1 ] = b[ n - 1 ] （最高位）
  - G [i] = b [ i ] ^ b [i + 1]
- 格雷码转换二进制： 
  - b[n-1] = G [n-1]
  - b [ i ] = G [ i ]^ G [i+1] ^ ......^ G[ n - 1] = G [ i ] ^ b[ i + 1 ]

EDA link: <https://www.edaplayground.com/x/4QrG>

```
//sample of Binary to Gray
module B2G (
		B,   // B input binary
		G    // output Gray
);
input wire [3:0] B;
output reg [3:0] G;
always @(*) begin // 使用always时，要加@
		G [3] = B [3];
		G [2] = B[2] ^ B[3];
		G [1] = B[1] ^ B[2];
		G [0] = B[0] ^ B[1];
end
endmodule

//TestBench
module tb ();
  reg [3:0] B,G;
  reg [2:0] i;
//  reg clk;
  initial begin
    $dumpfile("dump.vcd");// dump waive
    $dumpvars(1, tb);
    B = 0;
    i = 0;
    #10;
    B =1; #10;
   for (i=0; i<=7; i = i+1) begin
		   B = B +1;
			  #10;
   end
  end

  B2G dut(
    .B(B),
    .G(G)
  );
endmodule
```

### 3.2 写模块控制

写模块的功能

- 写信号来的时候，写指针地址加1
- 将wr 指针和rd 指针进行比较，看是否写满了。其中注意的是，满状态要将读指针同步到写时钟里，需要进行同步后再比较。

输入信号：wr_clk, wr_en, wr_rst_n, rd_add_glay

输出信号：wr_add_bin, wr_add_glay

EDA link <https://www.edaplayground.com/x/5565>

```verilog
// Code your design here
`include "B2G.v"
module wr_ctrl (
		input wr_clk,
        input wr_rst_n,
		input wr_en,
 		 input [3:0] rd_add_glay,
 		 output [3:0] wr_add_bin,
 		 output [3:0] wr_add_glay,
		output reg wr_full
);

reg [3:0] wr_add_bin_r;
reg [3:0] wr_add_glay_r;
reg [3:0] rd_add_glay_r1;
reg [3:0] rd_add_glay_r2;
// 地址增加模块

always @ (posedge wr_clk or negedge wr_rst_n or posedge wr_en) begin
	if (wr_en) begin // 所有的操作都是在始能下开始的
	if (!wr_rst_n) begin //复位操作
		wr_add_bin_r <= 0; //地址指针复位
		wr_full <= 0;
	end
    else if (!wr_full) begin //不满情况下
       wr_add_bin_r <= wr_add_bin_r + 1;// 地址加1
      end
end
end

//调用格雷码转换
B2G wr_B2G(
	.B (wr_add_bin_r),
	.G (wr_add_glay_r)
);

//判断满状态，如果格雷码首两位和读地址互异，则满
//同步读地址
always @(posedge wr_clk) begin
	{rd_add_glay_r2,rd_add_glay_r1} <= {rd_add_glay_r1, rd_add_glay} ;
end
  always @(*) begin
    wr_full = (rd_add_glay_r2[3]^wr_add_glay_r[3]) && (rd_add_glay_r2[2]^wr_add_glay_r[2]) && (rd_add_glay_r2[1:0] == wr_add_glay_r [1:0]); // 首两位互异，且后几位相同
  end
assign wr_add_bin = wr_add_bin_r;
assign wr_add_glay = wr_add_glay_r;

endmodule
```

### 3.3 读模块控制

功能代码和写模块相似，只是空标志的判断条件不同，要求两个地址完全相同才可以。

EDA Link:  <https://www.edaplayground.com/x/3tfy>

```verilog
// Code your design here
`include "B2G.v"
module rd_ctrl (
		input rd_clk,
     input rd_rst_n,
		input rd_en,
 		 input [3:0] wr_add_glay,
 		 output [3:0] rd_add_bin,
 		 output [3:0] rd_add_glay,
		output reg rd_empty
);

reg [3:0] rd_add_bin_r;
reg [3:0] rd_add_glay_r;
reg [3:0] wr_add_glay_r1;
reg [3:0] wr_add_glay_r2;
// 地址增加模块

always @ (posedge rd_clk or negedge rd_rst_n or posedge rd_en) begin
	if (rd_en) begin // 所有的操作都是在始能下开始的
	if (!rd_rst_n) begin //复位操作
		rd_add_bin_r <= 0; //地址指针复位
		rd_empty <= 0;
	end
    else if (!rd_empty) begin //不满情况下
       rd_add_bin_r <= rd_add_bin_r + 1;// 地址加1
      end
end
end

//调用格雷码转换
B2G rd_B2G(
	.B (rd_add_bin_r),
	.G (rd_add_glay_r)
);

//判断满状态，如果格雷码首两位和读地址互异，则满
//同步读地址
always @(posedge rd_clk) begin
	{wr_add_glay_r2,wr_add_glay_r1} <= {wr_add_glay_r1, wr_add_glay} ;
end
  always @(*) begin
    rd_empty = wr_add_glay_r2 == rd_add_glay_r ; // 地址相同时，表示读空
  end
assign rd_add_bin = rd_add_bin_r;
assign rd_add_glay = rd_add_glay_r;

endmodule
```

### 3.4 FIFO Mem Ctrl

主体部分会建立一个FIFO的mem，并且输入数据和读出数据。

控制模块功能简介：

- 根据地址将对应的数据存储或者读取
- EDA link <https://www.edaplayground.com/x/bWf>

Bug ： 因为弄错数组的写法，导致出现了bug reg [wordsize : 0] array_name [0 : arraysize]; 其中第二个是0-size，例如需要生成8x8 size的数组，应该为 reg [7:0] mem [0:7] 不是 reg [7:0] mem [2:0] ;

```verilog
module fifo_mem (
		input wr_clk,
		input rd_clk,
  input rst_n,
  input [7:0] wdata,
  input [3:0] wr_add_bin, //写地址
  input [3:0] rd_add_bin, //读地址
		input wr_full,
		input rd_empty,
	output reg [7:0] rdata

);

// 建立一个8x8的reg
  reg [7:0] fifo_mem [0:7]; //建立一个宽度为8深度也为8的mem
reg [2:0] wr_add; //内部3位地址
reg [2:0] rd_add; //内部3位地址
  reg [2:0] i;
  
assign wr_add = wr_add_bin [2:0];
assign rd_add = rd_add_bin [2:0];

// 初始化
  
  always @(negedge rst_n) begin
    if (!rst_n) begin
      for (i=0; i<=7 ; i = i+1) begin
        fifo_mem [i] <= 0;
      end
    end
  end
      
always @(posedge wr_clk) begin
  if (!wr_full) begin
			fifo_mem [wr_add] <= wdata;
  end
end

always @(posedge rd_clk) begin
		if (!rd_empty) 
			rdata <= fifo_mem [rd_add] ;
end


endmodule
```

### 3.5 Top level 各模块集成

EDA link : <https://www.edaplayground.com/x/6Mfp>

```verilog
// Code your design here
`include "fifo_mem.v"
`include "wr_ctrl.v"
`include "rd_ctrl.v"
`include "B2G.v"


module fifo(
	input wr_clk,
	input rd_clk,
	input rst_n,
	input wr_en,
	input rd_en,
	input [7:0] data_i,
  	output reg [7:0] data_o,
  	output reg full, // 用于控制数据的输入
);

wire [3:0] wr_add_glay;
wire [3:0] rd_add_glay;
wire [3:0] wr_add_bin;
wire [3:0] rd_add_bin;
wire empty;
wire wr_rst_n,rd_rst_n;

assign wr_rst_n = rst_n;
assign rd_rst_n = rst_n;

fifo_mem dut(
    .rst_n (rst_n),
    .wr_clk (wr_clk),
    .rd_clk (rd_clk),
    .wdata (data_i),
    .wr_add_bin(wr_add_bin),
    .rd_add_bin(rd_add_bin),
    .wr_full(full),
    .rd_empty(empty),
    .rdata(data_o)
  );

wr_ctrl write(
    .wr_clk(wr_clk),
    .wr_rst_n(wr_rst_n),
    .wr_en(wr_en),
    .rd_add_glay(rd_add_glay),
    .wr_add_bin(wr_add_bin),
    .wr_add_glay(wr_add_glay),
    .wr_full(full)
  );

rd_ctrl read (
		.rd_clk(rd_clk),
    .rd_rst_n(rd_rst_n),
		.rd_en(rd_en),
 		.wr_add_glay(wr_add_glay),
 		.rd_add_bin(rd_add_bin),
 		.rd_add_glay(rd_add_glay),
		.rd_empty(empty)
);

endmodule
```

{% asset_img 3.png Simulation Result %}