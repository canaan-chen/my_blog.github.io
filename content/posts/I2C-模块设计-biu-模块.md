---
title: "I2C 模块设计| biu 模块"
date: 2020-03-17T20:36:24+08:00
categories: ["一个IC工程师的自我修养"]
tags: ["I2C 学习", "IC Design", "IC 面试"]
draft: false
---





下方结构图为I2C接口中的master模块示意图，这个章节主要用于讲述其中的biu模块。根据结构图可知，biu模块存在的地址在于APB interface 和APB register 之间，它的目的在于将复杂的AHB 接口信号转换为单周期的读写信号，这样方便对后面的reg进行操作。

<!--more-->

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/1584102093634.png)

```verilog
module apb_i2c_biu #(
	parameter ADDR_SLICE_LHS = 5, // addres [4:0] 
	parameter APB_DATA_WIDTH = 32
)
(
// signals connect to APB module 
	input							pclk,
	input							presetn, // reset
	input 							psel, // select signal
	input	[ADDR_SLICE_LHS-1 : 0] 	paddr, // 5bit addres
	input							pwrite,
	input 							penable,
	input	[APB_DATA_WIDTH-1 : 0]	pwdata,
	output	reg [APB_DATA_WIDTH-1 : 0] prdata,
//signals connect to register module
	input	[15:0]					iprdata, // ?
	output							wr_en,
									rd_en,
	output	[ADDR_SLICE_LHS-3:0]	reg_addr, // ahb addres searching
	output	[3:0]					byte_en,
	output	reg	[31:0]				ipwdata
);
```



### 写操作

当AHB实现写操作时，要求片选信号（psel）为高，使能信号（penable）为高，写信号（pwrite）为高，当满足这三个条件时，biu信号输出写使能（wr_en）高，来驱动后面的reg。当ahb数据到来时可直接存到输出reg（ipwdata）中。

- 写操作代码：

  ```verilog
  assign wr_en = psel & penable & pwrite; // 写信号的要求
  
  always @(pwdata) begin
  	ipwdata = 32'b0;
  	ipwdata [APB_DATA_WIDTH-1:0] = pwdata[APB_DATA_WIDTH-1:0]; //apb总线上的数据每次发生变化，就将数据传输到输出reg中
  end
  
  ```



### 读操作

实现读操作时，要求在AHB读操作信号（pwrite 为低时）提前将reg中的数据传到输出数据缓存区prdata中。

- 关于读信号的判断

  biu 输出读信号（rd_en）的条件为：pwrite 为低，psel为高，penable 为高。但是为了保证在读操作实行时提前将数据准备好，rd_en 在penable为低便使能，具体代码如下：

  ```verilog
  assign rd_en = psel & !penable & !pwrite; // 读信号要提前产生，因为要求把数据提前放到APB总线上
  
  // 数据传输过程
  always @ (posedge pclk or negedge presetn) begin
  	if(presetn == 1'b0) begin
  		prdata <= {APB_DATA_WIDTH{1'b0}}; //apb 读寄存器中的数据清零
  	end
  	else begin
  		if (rd_en) begin
  			prdata <= {16'b0,iprdata}; // 将读入的数据保存到apb reg中
  		end
  	end
  end
  ```

  

### 具体的代码链接如下：

[Biu Ctrl Code](https://github.com/canaan-chen/I2C-IP-design/blob/master/apb_i2c_biu.v)

