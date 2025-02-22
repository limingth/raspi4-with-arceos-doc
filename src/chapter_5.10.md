# 实验一：ArceOS的 I2C 驱动实现

飞腾派的每个MIO均可单独当做UART/I2C。端口功能的选择，可以通过配置creg_mio_func_sel寄存器来实现，配置为00选择I2C，配置为01选择UART。

 MIO寄存器基地址：

| Name | Offset |
|-------|-------|
| MIO0 | 0x000_2801_4000 |
| MIO1 | 0x000_2801_6000 |
| MIO2 | 0x000_2801_8000 | 
| MIO3 | 0x000_2801_A000 | 
| MIO4 | 0x000_2801_C000 | 
| MIO5 | 0x000_2801_E000 | 
| MIO6 | 0x000_2802_0000 | 
| MIO7 | 0x000_2802_2000 |
| MIO8 | 0x000_2802_4000 | 
| MIO9 | 0x000_2802_6000 | 
| MIO10 | 0x000_2802_8000  |
| MIO11 | 0x000_2802_A000  |
| MIO12 | 0x000_2802_C000  | 
| MIO13 | 0x000_2802_E000  | 
| MIO14 | 0x000_2803_0000 | 
| MIO15 | 0x000_2803_2000 | 

![image](https://github.com/chenlongos/raspi4-with-arceos-doc/assets/83756052/ad8433b3-5cd2-4780-8664-01c86312e702)


I2C 操作说明：

1. 模式配置：

   配置MIO寄存器creg_mio_func_sel为0x00，选择I2C模式

2. 配置为master：

   1. 配置寄存器0x6c（IC_ENABLE）为0；
  
   2. 写寄存器0x00（IC_CON），配置主从、speed、设备地址宽度。
  
   3. 将设备地址写入寄存器0x04（IC_TAR）；

   4. 使能控制器，配置寄存器0x6c（IC_ENABLE）为1。
  
3. 发送数据：

   1. 判断发送FIFO不满：读0x70(IC_STATUS)地址，判断bit[1]为1时，即发送 FIFO 不满。

   2. 发送写数据命令：向0x10（IC_DATA_CMD）的bit[7:0]写入数据，向bit[8]写入0。

   3. 支持写入多字节数据，重复1、2步骤即可。

   4. 写入最后一个字节数据时要加上停止信号，即除了向0x10（IC_DATA_CMD）的bit[7:0]写数据，bit[8]写0表示写以外，向bit[9]写1表示停止。
  
4. 接收数据：

   1. 发送读数据命令：向0x10（IC_DATA_CMD）bit[8]写1，表示命令为读操作。

   2. 判断接收FIFO不空：读0x70(IC_STATUS)地址，判断bit[3]为1时，即接收 FIFO 不空。

   3. 读取数据：读0x10（IC_DATA_CMD）地址。

   4. 支持读多字节数据，重复前三步即可。

   5. 读最后一个字节数据时要加上停止信号，即除了向0x10（IC_DATA_CMD）的bit[8]仍写1表示读以外，向bit[9]写1表示停止。
  
5. I2C 部分寄存器：
  
 | Name | Offset | Description |
|-------|-------|-------|
| IC_CON | 0x00 | I2C控制寄存器 |
| IC_TAR | 0x04 | I2C主机地址寄存器 |
| IC_DATA_CMD | 0x10 | I2C数据寄存器 |
| IC_ENABLE | 0x6C | I2C使能寄存器 |
| IC_STATUS |  0x70 | I2C状态寄存器 |

tip：可参考本文档的树莓派串口部分。

