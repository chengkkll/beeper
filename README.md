
# 提示器通信协议

## 目录
- [提示器通信协议](#提示器通信协议)
  - [目录](#目录)
  - [1. 通讯接口规格](#1-通讯接口规格)
    - [1.1 串口接口](#11-串口接口)
    - [1.2 网络接口](#12-网络接口)
  - [2. 协议描述](#2-协议描述)
  - [3. 数据的格式](#3-数据的格式)
    - [1. 命令格式](#1-命令格式)
      - [Cmd 说明](#cmd-说明)
      - [子命令码说明](#子命令码说明)
      - [CRC 代码示例](#crc-代码示例)
  - [4.操作命令详细描述](#4操作命令详细描述)
    - [1. 心跳命令](#1-心跳命令)
    - [4. 呼叫命令](#4-呼叫命令)
    - [5. 下发数据](#5-下发数据)
    - [6. 清除OLED显示](#6-清除oled显示)
    - [8. 显示图片](#8-显示图片)
    - [9. 停止呼叫](#9-停止呼叫)

## 1. 通讯接口规格

### 1.1 串口接口

提示器主板通过TYPE-C接口与上位机(PC)串行通讯，按上位机(PC)的命令要求完成相应操作。串行通讯接口的数据帧为一个起始位，8个数据位，一个停止位，无校验位，缺省波特率115200。

### 1.2 网络接口
提示器主板可以通过wifi或以太网与上位机进行TCP/IP通信，通信命令与串口通信命令一致。

## 2. 协议描述

通讯过程由上位机发送命令给主板，主板将将命令通过无线转发给提示器终端，然后提示器终端将命令执行结果状态和数据返回给提示器主板并上传给上位机。

上位机发送过程如下：
| 上位机     | 数据传递方向 | 提示器主板 | 数据传递方向 | 提示器终端 |
| :--------- | :----------: | :--------: | :----------: | :--------: |
| 命令数据块 |      →       |            |      →       |            |

> 说明：上位机发送完命令数据块后，切换到接收模式，等待接收Demo板的响应数据块。


Demo板发送过程如下：
| 提示器终端 | 数据传递方向 | 提示器主板 | 数据传递方向 | 上位机 |
| :--------- | :----------: | :--------: | :----------: | :----: |
| 响应数据块 |      →       |            |      →       |        |


## 3. 数据的格式

### 1. 命令格式

|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | State | Data[] |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :----: | :---: | :---: |

命令数据块各部分说明如下：

|        | 长度  |                                    说明                                    |
| :----- | :---: | :------------------------------------------------------------------------: |
| SOH    | 1字节 |                               起始符（68H）                                |
| Len    | 2字节 | 整个报文的ID（从Cmd到Data（包含Cmd与Data）的长度） 高字节在前 ，低字节在后 |
| IOF    | 1字节 |                               识别符（68H）                                |
| Cmd    | 1字节 |                           [Cmd 说明](#cmd-说明)                            |
| SubCmd | 1字节 |                       [子命令码 说明](#子命令码说明)                       |
| Key    | 2字节 |                    下行时带的命令标识，上行时需要带回。                    |
| Addr   | 2字节 |                    Demo板地址为2字节BCD码（0-65535）。                     |
| Num    | 2字节 |              批量传输数据时，表示首位数据在整个数据块中的编号              |
| State  | 1字节 | 批量传输数据时，传输完成标志位，0：传输为完成，1：传输完成，此帧为最后一帧 |
| Data   | N字节 |                用户数据（具体命令具体数据）最大长度80 字节                 |
| CRC    | 2字节 |               从SOH到Data（包含SOH与Data）所有字节的crc16。                |
| EOT    | 1字节 |                               结束符（16H）                                |

#### Cmd 说明

| 比特 |   名称   |                        描述                         |
| :--- | :------: | :-------------------------------------------------: |
| 7:7  | 传输方向 | 0 = 上位机发出的下行报文； 1 = Demo板发出的上行报文 |
| 6:6  |  异常码  |                命令出错（上行报文）                 |
| 5:0  |   命令   |                       命令码                        |

#### 子命令码说明
     
| Cmd [bit5:0] | SubCmd [bit7:0] |   类型    |                                                说明                                                 |
| :----------- | :-------------: | :-------: | :-------------------------------------------------------------------------------------------------: |
| 17H          |      0x01       | 发送/回答 |                                      心跳，下位机返回电量信息                                       |
| 17H          |      0x02       | 发送/回答 |                                              注册命令                                               |
| 17H          |      0x03       | 发送/回答 |                                              激活命令                                               |
| 17H          |      0x04       | 发送/回答 |                                              呼叫命令                                               |
| 17H          |      0x05       | 发送/回答 | 下发字符data[0]序号，data[1]X轴坐标，data[2]Y轴坐标,data[3]数据下发是否完成，data[4]...oled显示数据 |
| 17H          |      0x06       | 发送/回答 |                                              清除显示                                               |
| 17H          |      0x07       | 发送/回答 | 下发图片data[0]序号，data[1]X轴坐标，data[2]Y轴坐标,data[3]数据下发是否完成，data[4]...oled显示数据 |
| 17H          |      0x08       | 发送/回答 |                                            显示下发数据                                             |
| 17H          |      0x09       | 发送/回答 |                                            停止呼叫命令                                             |

#### CRC 代码示例

C语言代码：
```c
unsigned short MainWindow::CRC16_CCITT( char *puchMsg, uint16_t usDataLen)
{
       uint16_t len = usDataLen;
       unsigned int crc=0;
       unsigned char i;
       while( len-- )
       {
           crc ^=  (unsigned char)*puchMsg;
           for (i=0;i<8;i++)
           {
               if (crc&0x0001)
               {
                 crc = crc >> 1;
                 crc = crc ^ 0x8408;
               }else
               {
                crc = crc >> 1;
               }
            }
            puchMsg++;
         }
       return crc;
}
```

c# 代码：
```c#
static private byte[] CRC16(IENumerable<byte> data)
{
    int len = data.Count();
    ushort crc = 0;

    if (len > 0)
    {
        for (int index = 0; index < len; index++)
        {
            crc ^= data.ElementAt(index);
            for (byte j = 0; j < 8; j++)
            {
                crc = (crc & 1) != 0 ? (ushort)((crc >> 1) ^ 0x8408) : (ushort)(crc >> 1);
            }
        }
    }
    byte hi = (byte)((crc & 0xFF00) >> 8);  //高位置
    byte lo = (byte)(crc & 0x00FF);         //低位置

    return new byte[] { hi, lo };
}
```

## 4.操作命令详细描述

### 1. 心跳命令

* 命令说明
  
> 心跳命令是在没有通讯时定时由上位机发起，下位机将电量信息上传给上位机。

* 命令格式 (16进制示例)
  
|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | 00 0d |  68   |  17   |   01   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Addr参数表示设备地址
> 
> Data[]参数解析：
> | 参数 | 长度  | 说明  |
> | :--- | :---: | :---: |
> | Data | 4字节 |  全0  |

* 命令应答 (16进制示例)
  
|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state | Data[] |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :----: | :---: | :---: |
|  68   | XX XX |  68   |  97   |   01   | XX XX | XX XX | 00 00 |  01   |   XX   | XX XX |  16   |

> Data[]参数解析： 不管Data返回多少数据，只取高位1个字节并转换成10进制作为电量
>
> | 参数 | 长度  |      说明      |
> | :--- | :---: | :------------: |
> | Data | 1字节 | 电量信息0-100% |

### 4. 呼叫命令

* 命令说明
  
> 呼叫提示器终端

* 业务场景
  
> 需要先调用下发数据命令，发送一些文本信息，然后调用呼叫命令，让设备震动并显示文本。让使用者查看信息，并引导去几号窗口

* 命令格式 (16进制示例)
  
|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | 00 0d |  68   |  17   |   04   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析：
> 
> | 参数 | 长度  | 说明  |
> | :--- | :---: | :---: |
> | Data | 4字节 |  全0  |

* 命令应答 (16进制示例)

|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | XX XX |  68   |  97   |   04   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析： 
>
> | 参数 | 长度  |      说明      |
> | :--- | :---: | :------------: |
> | Data | 4字节 | 全0 |

### 5. 下发数据

* 命令说明
  
> 下发数据给提示器终端

* 命令格式 (16进制示例)
  
|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | XX XX |  68   |  17   |   05   | XX XX | XX XX | XX XX |  XX   | XX .. .. .. | XX XX |  16   |

> Num 表示当前是第几个包
> 
> State 表示当前是否为最后一个包，0=未完成；1=完成（最后一帧）
> 
> Data[]参数解析：使用实际长度，最多不能超过80字节。
> | 参数 | 长度  | 说明  |
> | :--- | :---: | :---: |
> | Data | 80字节 |  下发字符data[0]序号，data[1]X轴坐标，data[2]Y轴坐标,data[3]数据下发是否完成，data[4]...oled显示数据  |

* 命令应答 (16进制示例)

|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | XX XX |  68   |  97   |   05   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析： 
>
> | 参数 | 长度  |      说明      |
> | :--- | :---: | :------------: |
> | Data | 4字节 | 全0 |

### 6. 清除OLED显示

* 命令说明
  
> 清除OLED显示的内容，使设备不显示

* 命令格式 (16进制示例)
  
|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | 00 0d |  68   |  17   |   06   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析：
> | 参数 | 长度  | 说明  |
> | :--- | :---: | :---: |
> | Data | 4字节 | 全0  |

* 命令应答 (16进制示例)

|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | XX XX |  68   |  97   |   06   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析： 
>
> | 参数 | 长度  |      说明      |
> | :--- | :---: | :------------: |
> | Data | 4字节 | 全0 |

### 8. 显示图片

* 命令说明
  
> 显示图片

* 命令格式 (16进制示例)
  
|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | 00 0d |  68   |  17   |   08   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析：
> | 参数 | 长度  | 说明  |
> | :--- | :---: | :---: |
> | Data | 4字节 | 全0  |

* 命令应答 (16进制示例)

|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | XX XX |  68   |  97   |   08   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析： 
>
> | 参数 | 长度  |      说明      |
> | :--- | :---: | :------------: |
> | Data | 4字节 | 全0 |

### 9. 停止呼叫

* 命令说明
  
> 停止呼叫提示器终端

* 命令格式 (16进制示例)
  
|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | 00 0d |  68   |  17   |   09   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析：
> | 参数 | 长度  | 说明  |
> | :--- | :---: | :---: |
> | Data | 4字节 | 全0  |

* 命令应答 (16进制示例)

|  SOH  |  Len  |  IOF  |  Cmd  | SubCmd |  Key  | Addr  |  Num  | state |   Data[]    |  CRC  |  EOT  |
| :---: | :---: | :---: | :---: | :----: | :---: | :---: | :---: | :---: | :---------: | :---: | :---: |
|  68   | XX XX |  68   |  97   |   09   | XX XX | XX XX | 00 00 |  01   | 00 00 00 00 | XX XX |  16   |

> Data[]参数解析： 
>
> | 参数 | 长度  |      说明      |
> | :--- | :---: | :------------: |
> | Data | 4字节 | 全0 |
