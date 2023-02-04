# BRAM_HDMI_TEST
将数据从FPGA的BRAM输出到HDMI接口的测试例程

在顶层模块top_hdmi_test下主要包含了两个模块，分别是负责生成图像数据的img_data_generator和hdmi接口的顶层模块hdmi_top，工作机制很简单：即hdmi_top根据行场同步信号的时序发出数据请求，img_data_generator接受到请求信号后在下一个时钟周期将存储在BRAM中的下一个图像像素数据发送给hdmi接口，实现显示功能。

接下来将对img_data_generator和hdmi_top这两个模块的接口和主要内容进行分析：

## 1. img_data_generator——数据生成模块

### 1.1 简要分析

模块头代码如下
````
module img_data_generator
(
    input sys_clk,
    input sys_reset_n,
    input data_req,
    output [15:0] data_img
);
````
上述代码中：
（1）时钟sys_clk与hdmi中行场信号的频率保持一致；采用1080P格式，其频率为148.5MHz，来源于锁相环IP核；
（2）复位sys_reset_n与顶层模块的复位一致，并与物理按键连接；
（3）data_req为从hdmi_top中接受到的数据请求信号；
（4）data_img为16位的RGB565的图像数据，在data_req的后一拍有效；其数据来源于img_data_generator模块中提前例化的BRAM模块；

由于BRAM大小的限制，没有足够空间存储1080P格式的图像（1920宽 * 1080高 * 16位宽），因此在实际操作中，将图像缩放5倍，即384宽 * 216高 * 16位大小的图像数据存储在BRAM中，并在屏幕中显示时，在宽和高两个方向将图像重复5次，即显示25个重复的图像。为实现这一目标需对BRAM的操作地址进行计算，具体见img_data_generator.v文件。

### 1.2 仿真波形

为验证数据生成模块有效性，工程中可见tb_img_data_generator.v仿真文件来检测img_data_generator的波形。

在仿真文件中，模拟了1080P的hdmi时序的信号，包括行场同步信号、行场有效信号、数据有效信号等，具体先见如下波形图，再作分析。

![2](https://user-images.githubusercontent.com/95362898/216741189-f7d5f588-1780-4470-bc6f-40bdcec4424b.PNG)

![1](https://user-images.githubusercontent.com/95362898/216741172-60775984-5bb4-4996-bb60-4171c2d7957c.PNG)







