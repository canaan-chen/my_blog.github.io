---
title: "I2C 模块设计|Master Reg 设计"
date: 2020-03-21T20:37:05+08:00
categories: ["一个IC工程师的自我修养"]
tags:  ["I2C 学习", "IC Design", "IC 面试"]
draft: false
---

本章节讲述APB regesiter 模块。register 处于biu和byte_ctrl 之间，它的作用在于收取到来自biu模块的读写信号进行读写操作，之后转换成相应的控制信号对后面模块进行控制，同时byte_ctrl 和bit_ctrl 会产生相应的状态信号反馈给reg模块。

<!--more-->



![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/SharedScreenshot.jpg)



### 1. Register 分类

首先我们先对这次设计的寄存器进行一次分类。

- **预分频寄存器**： 用于存储分频的具体参数。

  ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200321185858.png)

- **控制寄存器**： 用于使能和中断

  ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200321190420.png)

- **数据接收寄存器**：用于接受数据

  ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200321190459.png)

- **数据发送寄存器**：用于传输数据

  ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200321190604.png)

- **状态寄存器**：存储着系统模块的状态，包括应答信号接受状态、数据传输状态等

  ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200321190702.png)

- **命令寄存器**：包含着一系列的指令。

  ![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200321190736.png)



### 2. 整体模块的输入输出口

首先我们定义一下整个master的reg的输入输出。

输入信号：

- 来自biu：`wr_en`, `rd_en`, `byte_en`, `reg_addr`, `ipwdata`。
- 来自byte_ctrl: `done`(一帧完成信号) `i2c_al`(仲裁失败信号)、`rxack`（应答信号）、`i2c_busy`（busy 标志位）、`tip`(传输状态信号)、`rxr_i`(输入数据)

输出信号：

- 发送给biu ：`iprdata`(读出数据，传输给apb）
- 发送给byte_ctrl：`core_en`(片选信号)、`ien_en`(中断功能使能信号)、`prer`(分频数预设置，传输给byte_ctrl)、`txr`(存储来自apb的数据，并且用于传输给下一级)、`sta`（启动标志位）、`sto`（停止标志位）、`rd`（读信号）、`wr`（写信号）、`ack`（响应信号）、`irq_flag_d`(中断标志位) 



```verilog
module ahb_i2c_reg #(
	parameter ADDR_SLICE_LHS = 5, // 5位输入地址
	parameter APB_DATA_WIDTH = 32 // 32 位数据宽度
)
(
	input 							pclk,
	input							prestn,
	// biu 模块连接信号
	input							wr_en, // 来自biu的写信号
	input							rd_en, // 来自biu的读信号
	input	[3:0]					byte_en,
	input	[ADDR_SLICE_LHS-3:0]	reg_addr,// 3位地址
	input	[APB_DATA_WIDTH-1:0]	ipwdata, //来自biu的写数据
	output	reg [15:0]				iprdata, //输出给biu的读数据
	
	input							done,
	input							i2c_al, //仲裁失败信号
	input							rxack, //应答信号
	input							i2c_busy, //busy 标志位
	input							tip,
	input	[7:0]					rxr_i,
	
	// 输出控制信号，全都时reg形式，表示控制信号都来自D触发器
	output	reg	[15:0]				prer,
	output	reg						corn_en,
	output	reg						ien_en,
	output	reg	[7:0]				txr,
	output	reg						sta,
	output	reg						sto,
	output	reg						rd,
	output	reg						wr,
	output	reg						ack,
	output	reg						irq_flag_d

);
```



### 3.寄存器地址的分配

#### 3.1 寄存器地址分配

因为APB原地址分配方式为00h, 04h, 08h, 0Ch, 10h和14h， 这种八位的地址通过biu，只取低三位的方式已经转换为3位的地址addr，通过下面的代码实现对对应寄存器的选取。

```verilog
assign i2c_cpr_en = (reg_addr == 3'h0) ? 1'b1 : 1'b0; //预分频寄存器
assign i2c_ctr_en = (reg_addr == 3'h1) ? 1'b1 : 1'b0; // 控制寄存器
assign i2c_rdr_en = (reg_addr == 3'h2) ? 1'b1 :	1'b0; //接受寄存器
assign i2c_sr_en  = (reg_addr == 3'h3) ? 1'b1 : 1'b0; //状态寄存器
assign i2c_tdr_en = (reg_addr == 3'h4) ? 1'b1 : 1'b0; //发送数据寄存器
assign i2c_cr_en  = (reg_addr == 3'h5) ? 1'b1 : 1'b0; //命令寄存器
```



#### 3.2 Biu 读写信号转换

将biu 的读写信号转换为寄存器的读写信号。其中i2c_sr_rd， 是状态寄存器的读信号转换，因为该寄存器只能在读的时候清状态，写的时候不可以清除状态。

```verilog
assign i2c_cpr_wr =  i2c_cpr_en & wr_en;
assign i2c_ctr_wr =	i2c_ctr_en & wr_en;
assign i2c_rdr_wr = i2c_rdr_en & wr_en;
assign i2c_sr_wr  = i2c_sr_en & wr_en;
assign i2c_tdr_wr = i2c_tdr_en & wr_en;
assign i2c_cr_wr  = i2c_cr_en & wr_en;

assign i2c_sr_rd = i2c_sr_en & rd_en;// 读操作信号
```



### 4. 预分频寄存器的设计

预分频寄存器须在该寄存器的写信号`i2c_cpr_wr`为高时，并且片选信号为低时进行赋值，这是因为预分频必须在接口运作前先完成设定，设定方式为将输入数据`ipwdata`进行赋值。同时`prer`保存在`i2c_cpr_reg` 中。

```verilog
always @ (posedge pclk or negedge prestn) begin
	if (!prestn) begin
		prer <= 16'h0000;
	end else begin
		prer <= prer_pre;
	end
end

assign prer_pre = i2c_cpr_wr & ~core_en ? ipwdata[15:0] : prer; //如果写操作地址选中，并且未使能时，则将APB 上的数据写入。
// 设定core_en的目的是，当模块未使能时，要求设定好预分频寄存器，若使能后，则预分频寄存器不能改变。
assign i2c_cpr_reg = {prer};
```



### 5. 控制寄存器

![](https://image-1301586523.cos.ap-shanghai.myqcloud.com/20200321190420.png)

控制寄存器提供两个信号，一个是片选使能信号，另一个是中断使能信号，而这两个信号通过`ipwdata`的第八位和第七位来实现，即`ipwdata[7],ipwdata[6]`。

```verilog
always @ (posedge pclk or negedge prestn) begin
	if (!prestn) begin
		core_en <= 1'b0;
		ien_en	<= 1'b0;
	end else begin
		core_en <= core_en_pre;
		ien_en	<= ien_en_pre;
	end
end

assign core_en_pre = i2c_ctr_wr ? ipwdata[7] : core_en; // 第七位来赋值
assign ien_en_pre  = i2c_ctr_wr ? ipwdata[6] : ien_en;

assign i2c_ctr_reg = {core_en,ien_en,6'b0};
```



### 6. 发送寄存器

发送寄存器会将收到的`ipwdata`数据保存发送给下一级，因为传输过程中实际只用到八位传播，所以最后`txr`中存储的仅为`ipwdata`中的低八位。

```verilog
always @ (posedge pclk or negedge prestn) begin // txr 数据发送寄存器
	if(!prestn) begin
		txr <= 8'b0;
	end else begin
		txr <= txr_pre;
	end
end

assign txr_pre = i2c_tdr_wr ? ipwdata[7:0] : txr; //是否要输入16位数据？
assign i2c_tdr_reg = {8'b0, txr};
```



### 7. 接收寄存器

接受寄存器功能和发送寄存器类似，负责存储读取到的数据`rxr_i`。其中`done`信号表示一帧信号已经完成传输，当`done`信号为高时，才可以进行数据存储。

```verilog
always @ ( posedge pclk or negedge prestn) begin
	if (!prestn) begin
		rxr <= 8'b0;
	end else begin
		rxr <= rxr_pre;
	end
end

assign rxr_pre = done ? rxr_i : rxr; //将byte control模块读取到的数据输入，用于给apb模块读取
assign i2c_rdr_reg = {8'h0,rxr};
```



### 8. 控制寄存器

控制寄存器中含有较多的输出控制信号：`sta`（启动标志位）、`sto`（停止标志位）、`rd`（读信号）、`wr`（写信号）、`ack`（响应信号）。

当done信号传过来时，表示一帧数据发送完毕，控制信号都清零，这时候如果要赋值进行控制，则统一根据输入的`ipwdata[7:4]`来进行（选7到4是因为spec规定的，个人设计可以进行修改），`iack` 信号作为备用。

```verilog
always @ (posedge pclk or negedge prestn) begin
	if (!prestn) begin
		sta <= 1'b0;
		sto	<= 1'b0;
		rd	<= 1'b0;
		wr	<= 1'b0;
		ack	<= 1'b0;
		iack <= 1'b0;
	end else begin
		sta <= sta_pre;
		sto	<= sto_pre;
		rd	<= rd_pre;
		wr	<= wr_pre;
		ack	<= ack_pre; 
		iack <= iack_pre;
	end
end

assign sta_pre = done ? 1'b0 : i2c_cr_wr & ien_en ? ipwdata[7] : sta ; //如果done信号穿过来，表示已经完成，则start标志清零。实现物理清零操作
assign sto_pre = done ? 1'b0 : i2c_cr_wr & ien_en ? ipwdata[6] : sto ; 
assign rd_pre = done ? 1'b0 : i2c_cr_wr & ien_en ? ipwdata[5] : rd ;
assign wr_pre = done ? 1'b0 : i2c_cr_wr & ien_en ? ipwdata[4] : wr ;
assign ack_pre = i2c_cr_wr ? ipwdata[3] : 1'b0;
assign iack_pre = i2c_cr_wr ? ipwdata [0] : 1'b0;
```

### 9. 状态寄存器

状态寄存器用于接收状态信息，该状态寄存器只允许读而不可以写。总共这几个状态：`done`(一帧完成信号) `i2c_al`(仲裁失败信号)、`rxack`（应答信号）、`i2c_busy`（busy 标志位），以及发出中断状态`irq_flag_d`。

其中中断状态，如果对该寄存器进行读操作时，会将中断寄存器清零，不会实现中断。相应的，中断发生条件为中断flag 为1 并且中断信号使能为1。

```verilog
always @ (posedge pclk or negedge prestn) begin
	if (!prestn) begin
		rxack_d			<= 1'b0;
		i2c_busy_d		<= 1'b0;
		al_d			<= 1'b0;
		tip_d			<= 1'b0;
		irq_flag_d		<= 1'b0;
	end else begin
		rxack_d 	<= 	rxack;
		i2c_busy_d	<=	i2c_busy;
		al_d		<= 	i2c_al | (al_d & ~sta); // i2c_al 是仲裁失败信号
		tip_d		<= (wr | rd);		
		irq_flag_d <= irq_flag_pre;
	end
end

assign irq_flag_pre = i2c_sr_rd ? 1'b0 : irq_flag & ien_en ? 1'b1 : irq_flag_d ; // 读的时候会把flag 给清掉
assign i2c_cr_reg = {8'b0
					,rack_d
					,i2c_busy_d
					,al_d
					,tip_d
					,irq_flag_d
					};
assign irq_flag = (done | i2c_al); //当完成一帧数据的传输或者发生总裁错位的时候产生中断标志位。
```

### 10. 检测寄存器情况

为了方便检测寄存器里的数据情况，通过以下的组合逻辑，在选中地址之后对访问的寄存器的数据进行存储。

```verilog
always @ (*) begin
	iprdata = {32{1'b0}};
	if (i2c_cpr_en == 1'b1) iprdata [15:0] = i2c_cpr_reg;
	if (i2c_ctr_en == 1'b1) iprdata [15:0] = i2c_ctr_reg;
	if (i2c_rdr_en == 1'b1) iprdata [15:0] = i2c_rdr_reg;
	if (i2c_sr_en == 1'b1) iprdata [15:0] = i2c_sr_reg;
	if (i2c_tdr_en == 1'b1) iprdata [15:0] = i2c_tdr_reg;
	if (i2c_cr_en == 1'b1) iprdata [15:0] = i2c_cr_reg;
end
```

