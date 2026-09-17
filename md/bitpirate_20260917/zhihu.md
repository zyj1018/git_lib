# ESP32 Bit Pirate：几十块钱，把一块开发板变成“能听懂十几种协议”的多面手

5861 Star · MIT 开源固件 · 一块 ESP32-S3 板就能嗅探 I2C / SPI / UART / CAN，也能玩红外、RFID 与 Sub-GHz；浏览器一键烧录，手机连上 Wi-Fi 就能敲命令

![官方演示动图截帧：Bit Pirate 的命令行界面在不同模式下工作（官方 README 演示素材）](https://cdn.jsdelivr.net/gh/zyj1018/git_lib@master/diy-lab/img/bitpirate_20260917/demo_f1.jpg)

> 官方演示动图截帧：Bit Pirate 的命令行界面在不同模式下工作（官方 README 演示素材）

**机器人的舵机忽然不动了。**你换过电机、换过电源、重烧了三次程序，它还是不动。这时候老手不急着拆零件，而是掏出一块小板子，夹在信号线上，让电脑把总线上跑的数据一句一句念出来——十有八九，问题就藏在几行从来没人看过的字节里。

今天要介绍的就是这么一块"翻译板"：**ESP32 Bit Pirate**，一个把普通 ESP32-S3 开发板变成多协议调试工具的 MIT 开源固件。它灵感来自老牌硬件工具 Bus Pirate，但把整套界面搬进了浏览器，连驱动都省了。

## 先看它会什么

**【官方已实现】** **下面三件事在当前固件里都能用，不是画饼：**

**01　把看不见的电信号念成文字。**总线模式覆盖 I2C、SPI、UART 与半双工 UART、1-Wire、2-Wire、3-Wire、JTAG（含 SWD）与 CAN；I2C 支持扫描地址、dump、读写 EEPROM、甚至从机模式，SPI 支持 EEPROM、Flash、SD 卡与从机模式。再直白一点：传感器找不到、Flash 读不出、舵机总线不回话，都可以先让它把数据抓出来看。

**02　无线那一摊也管。**蓝牙（BLE HID、扫描、嗅探）、Wi-Fi 与以太网（嗅探、nmap、netcat）、Sub-GHz（分析、录制、回放）、RFID（读、写、复制）、RF24 收发，都在官方模式清单里。红外更夸张，官方写明支持 80 种以上的红外协议，能当中文遥控器的"万能遥控"用。

**03　三种操作方式，随场景换。**手机/电脑浏览器里的 Web CLI、USB 串口终端、以及 M5 Cardputer 上的单机键盘屏；同一个命令体系可以随便切换，还能用 Python 脚本把一串操作写成自动化流程。

## 一分钟判断：你先别急着买任何东西

**【本文核对】** **以下口径来自官方仓库的 README、Wiki 与发布记录，本文没有独立复现（我手边没有 ESP32-S3 板子），请当作"入场前的地图"看。**

**复现难度**：★★☆☆☆——烧录即用、不用焊接；但如果想玩红外、RFID、Sub-GHz，需要自己接线接模块，难度会往上跳一档。

**首次跑通**：约 10 分钟——插板、浏览器烧录、打开终端敲一个 help。

**预算**：40–70 元起（一块 ≥8MB Flash 的 ESP32-S3 开发板），软件 0 元；把无线模块配齐大约 250–450 元。

**前置条件**：一块 ESP32-S3 开发板 + 一根数据线 + 一个 Chrome / Edge 浏览器（Web 烧录与 Web 终端都靠它）。

**适合谁**：玩单片机、机器人、舵机与传感器总线的人；喜欢拆机、想亲手把信号看明白的人。**不适合谁**：只想点两下按钮看结果、不想碰命令行的人——它毕竟是个工具，不是玩具。

## 它为什么值得玩

值得记住的一句话是：**它把"排查问题"从玄学变成了证据。**以前遇到一个不回话的传感器，你能做的只有换线、换板、换传感器，全靠感觉；有了它，总线上有没有应答、地址对不对、数据帧长什么样，屏幕上直接读。

另一个乐趣是：它不是一个买来就封死的成品盒子，而是一份你也能改的固件。官方 Wiki 专门写了一节"怎么加一条新命令"，把贡献流程拆得很细——这对学生的价值，往往比工具本身更大：你用的是别人的工具，改的是自己的作品。

## 它是怎么工作的：信号 → 板子 → 你的浏览器

![本文自绘：Bit Pirate 的输入—处理—输出链路（依据官方模式清单与界面说明整理）](https://cdn.jsdelivr.net/gh/zyj1018/git_lib@master/diy-lab/img/bitpirate_20260917/system_diagram.png)

> 本文自绘：Bit Pirate 的输入—处理—输出链路（依据官方模式清单与界面说明整理）

按"输入—处理—输出"拆开看，逻辑非常清楚：

**输入**：被测设备的总线电平（I2C/SPI/UART…）或无线信号（红外、RFID、Sub-GHz、蓝牙、Wi-Fi）。

**处理**：ESP32-S3 里的固件按你选中的"模式"去解析这些信号，把电平变成协议层的数据帧；Wi-Fi 模式下板子自己起热点或接入你的局域网，把命令界面暴露出来。

**输出**：浏览器或串口终端里的文字结果，以及具体的动作——dump 一段 EEPROM、回放一段遥控信号、给 JTAG 设备挂上调试器。

**【本文核对】** 官方把"同一套命令、三种入口"写得很清楚：Web 界面免线缆、手机平板都能开，适合快速测试；串口界面更快、适合长时间高频交互；Cardputer 的单机模式则完全脱离电脑。

![官方 README 配图：手机浏览器里打开的 Bit Pirate Web CLI（免装任何驱动）](https://cdn.jsdelivr.net/gh/zyj1018/git_lib@master/diy-lab/img/bitpirate_20260917/presentation_mobile.png)

> 官方 README 配图：手机浏览器里打开的 Bit Pirate Web CLI（免装任何驱动）

## 硬件与软件清单、成本口径

![本文自绘：入门成本口径（2026-09 国内电商估算，官方未发布整机 BOM 与定价）](https://cdn.jsdelivr.net/gh/zyj1018/git_lib@master/diy-lab/img/bitpirate_20260917/bom_card.png)

> 本文自绘：入门成本口径（2026-09 国内电商估算，官方未发布整机 BOM 与定价）

官方支持的机型清单很长，从一块裸开发板到带屏带键盘的整机都行：

• ESP32-S3 DevKit（20+ 可用 GPIO，最便宜的入门路线）

• M5 Cardputer / Cardputer ADV、M5 Stick C Plus 2、M5 Stick S3、M5 StampS3、M5 AtomS3 Lite

• LILYGO T-Embed / T-Embed CC1101、Seeed Xiao ESP32-S3、Heltec LoRa 32 V4 等

按官方说明：**任何 ESP32-S3 板子都能烧，前提是至少 8MB Flash；但默认引脚映射不一定和你的板子一致，需要自己核对。**

![官方支持机型：ESP32-S3 DevKit（README 支持设备清单配图）](https://cdn.jsdelivr.net/gh/zyj1018/git_lib@master/diy-lab/img/bitpirate_20260917/s3-devkit.jpg)

> 官方支持机型：ESP32-S3 DevKit（README 支持设备清单配图）

![官方支持机型：M5 Cardputer——带屏幕和键盘，可以脱离电脑单机操作](https://cdn.jsdelivr.net/gh/zyj1018/git_lib@master/diy-lab/img/bitpirate_20260917/cardputer.jpg)

> 官方支持机型：M5 Cardputer——带屏幕和键盘，可以脱离电脑单机操作

软件侧一分钱不用花：固件是 MIT 许可，烧录用官网的浏览器一键烧录器，终端用官网的 Web Serial 工具，写脚本还有在线的 Python Lab。

## 最短复现路线（今天就能做）

**第 1 步　把板子连上电脑。**找一块 ESP32-S3 板（≥8MB Flash），用一根能传数据的 USB 线接上。**成功标志**：系统里能看到串口设备（Windows 设备管理器出现新端口，Linux 下能看到 /dev/ttyACM 开头的设备）。

**卡住先看什么**：换个 USB 口、换根线——很多"烧不进去"其实是线只能充电。

**第 2 步　浏览器一键烧录固件。**用 Chrome 或 Edge 打开官方 Web Flasher，按提示选中你的板子型号，点烧录。**成功标志**：页面提示写入完成，板子重启后出现对应机型的现象（屏幕点亮或指示灯变化）。

**卡住先看什么**：M5 系列机型官方也支持用 M5Burner 烧录，换条路走。

**第 3 步　打开命令行。**用官方 Web Serial 终端（或任意串口终端软件）连接板子，敲一个 help。**成功标志**：命令行给出可用的模式与命令列表，敲 mode 能看到模式清单。

**第 4 步　先玩最安全的一档。**在默认的 HiZ 模式或 DIO 模式下读一读 GPIO 电平，用手碰一碰、接一根杜邦线到 3.3V，看读数跳变。**成功标志**：读数随接线变化，说明"输入—处理—输出"整条链通了。

**第 5 步　扫一条真实的总线。**接一个手边的 I2C 器件（比如常见的 OLED 屏，地址 0x3C），进 I2C 模式执行扫描。**成功标志**：扫描结果里出现器件的地址，而且反复扫都稳定。

到这一步，你就有了一个能对着真实硬件"问话"的工具，后面所有高级玩法都是在这套命令上加模块。

## 新手最常卡住的三个点

**现象：点烧录没反应，页面找不到设备。****先判断**：浏览器和线材——Web 烧录依赖浏览器的串口能力，且必须用能传数据的线。**正确做法**：先换 Chrome/Edge，再换数据线与 USB 口；仍不行就用 M5Burner 或 PlatformIO 走本地烧录。

**现象：串口一打开全是乱码。****先判断**：多半是连上的瞬间正好撞上板子上电输出，或者你用的终端工具参数不对。**正确做法**：关掉终端重新连，必要时先按一下复位键；优先用官方 Web Serial 终端，它和固件是配套的。

**现象：I2C 扫描什么都扫不到。****先判断**：先怀疑接线和供电，再怀疑器件——SDA/SCL 有没有接反、有没有共地、器件是不是 3.3V 逻辑。**正确做法**：断电重接一遍，确认器件单独供电时也能工作，再回来扫描。

## 三条魔改方向（初级 / 中级 / 高级）

**【魔改设想】** **下面三条都不是官方宣传的功能，而是在官方模式之上能长出来的玩法。**

**初级｜给桌面机器人做一次"体检"。**用 DIO 模式量一量舵机信号线的电平变化，用 I2C 模式扫一遍传感器总线，再把结果截图贴进实验报告——比"我换了个传感器就好了"专业得多。

**中级｜把家里遥控器"学"下来。**配一个红外收发或 CC1101 模块，录制自家遥控器的信号并回放，顺手理解什么叫编码与协议（只对自己的设备做，别去碰别人的信号）。

**高级｜做成自动化测试台。**用官方 Python 脚本接口把"上电—扫描—读寄存器—dump—记录日志"串成一条流水线，接进班级或实验室的板测流程，让每次都出同一份体检报告。

![社区 Dock 扩展板照片（官方 README 收录的第三方硬件，可复用原版 Bus Pirate 适配器）](https://cdn.jsdelivr.net/gh/zyj1018/git_lib@master/diy-lab/img/bitpirate_20260917/bit_pirate_dock_board.jpeg)

> 社区 Dock 扩展板照片（官方 README 收录的第三方硬件，可复用原版 Bus Pirate 适配器）

## 边界、许可证与安全

**开源协议：**MIT（仓库根目录 LICENSE，本文核对）。也就是说你可以自用、修改、二次分发，只要保留版权声明；官方同时说明，文档与视觉素材可自由用于文章与视频。

**版本边界：**仓库当前默认分支是 pioarduino；老的 main 分支被官方标注为 legacy（已停止开发，仅作历史留存）。跟着老教程走容易踩坑，认准官网的烧录入口最稳。

**电压边界：**官方红字警告——只能接 3.3V 或 5V 的器件，接错电压可能直接烧掉你的板子；接线前断电，接完再上电。

**合法边界：**官方写明本固件仅用于教育、诊断与互联互通测试，不要用它去探测、干扰没有授权的设备；Sub-GHz、Wi-Fi、蓝牙相关的发射功能在国内受无线电管理相关规定约束，请只在自己的设备上做实验。

**没复现的部分要说清楚：**本文没有独立复现——手边没有 ESP32-S3 板子。上文的功能、命令与界面描述来自官方 README / Wiki / 发布记录，成本是电商估算，不是实测账单。

## 官方来源与教程

**仓库（MIT）：**[https://github.com/geo-tp/ESP32-Bit-Pirate](https://github.com/geo-tp/ESP32-Bit-Pirate)　5861 Star / 486 Fork，2026-09-16 仍在更新，最新版 v1.7

**官网与一键烧录：**[https://geo-tp.github.io/ESP32-Bit-Pirate/](https://geo-tp.github.io/ESP32-Bit-Pirate/)

**Wiki 文档（每种模式的命令）：**[https://github.com/geo-tp/ESP32-Bit-Pirate/wiki](https://github.com/geo-tp/ESP32-Bit-Pirate/wiki)

**脚本仓库：**[https://github.com/geo-tp/ESP32-Bit-Pirate-Scripts](https://github.com/geo-tp/ESP32-Bit-Pirate-Scripts)　**Sub-GHz 扩展：**[https://github.com/geo-tp/ESP32-Bus-Expander](https://github.com/geo-tp/ESP32-Bus-Expander)

**社区 Dock 扩展板：**[https://github.com/AndreiVladescu/ESP32-Bit-Pirate-Dock](https://github.com/AndreiVladescu/ESP32-Bit-Pirate-Dock)

**热度证据（本文核对）：**项目 2026-06-05 首次登上 Hacker News 拿到 211 分，2026-09-06 以 "Hardware Hacking Kit with Web Tools" 再次上榜拿到 121 分、44 条讨论：[https://news.ycombinator.com/item?id=49587465](https://news.ycombinator.com/item?id=49587465)；国内它也上过知乎的「GitHub 今日热榜」（2026-08-01，第 5 位）。

你更想看我们下一篇做哪一件？

**方向 A**：拿它去抓一台舵机机器人的总线数据，把"为什么不听话"当场抓出来

**方向 B**：用它把自家遥控器的红外 / Sub-GHz 信号学下来，顺手做一个万能遥控

留言说一句你选哪个就行——下一期我们按票数多的那条动手。
