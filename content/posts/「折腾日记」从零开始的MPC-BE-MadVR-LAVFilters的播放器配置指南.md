---
title: 「折腾日记」从零开始的MPC-BE+MadVR+LAVFilters的播放器配置指南
tags:
  - MadVR
  - 播放器
keywords:
  - 折腾日记
  - MadVR
  - LAVFilters
  - MPC-BE
  - 播放器
  - 视频渲染
  - 视频解码
  - 音频解码
  - 字幕渲染
categories:
  - 折腾日记
  - 长文
date: 2025-07-20 11:30:39
description: Windows平台下基于MPC-BE+MadVR+LAVFilters的播放器详细配置指南，小白也能轻松看懂。或许是目前最优观影方案？
summary: Windows平台下基于MPC-BE+MadVR+LAVFilters的播放器详细配置指南，小白也能轻松看懂。或许是目前最优观影方案？
draft: false
---
<font size=5>***长文注意⚠️***</font>

## 前言

### 岚的声明
<font size=4>本文为**岚**的**折腾日记**，它只是因为我**想**而写下的，**不具备**权威性，我也**无法**保证正确，**仅供参考**</font>

### 本文参考
- **[万年冷冻库的播放器教程](https://lysandria1985.blogspot.com/)**
- **[维基百科](https://en.wikipedia.org)**
- 以及一些论坛

### 一些碎碎念
Potplayer 被广泛推荐，但它因使用开源项目代码（如 FFmpeg）却闭源发布，被认为可能违反 GPL 协议，且部分版本包含推广内容，引起了一些争议，故本文不推荐
开源的MPC-BE且功能同样强大，且积极维护
当然MPC-BE并非唯一选择，也可以选择mpv，它更加强大，且跨平台，具体参见[mpv_lazy](https://github.com/hooke007/MPV_lazy)
本文主要介绍基于MPC-BE的配置，我并非音视频相关专业人士，主要提供一个参考，不会去深入讲解 ~~(因为我也不会)~~，有需要请自行查阅相关资料

[Video Helper](https://www.videohelp.com/)是一个很实用的网站，多数音视频相关的软件都可以在这里找到甚至溯源

<font size=5>***内容太多啦，脑袋晕晕的(っ °Д °;)っ，可能会有错误，欢迎指正***</font>

## 安装
- [MPC-BE](https://github.com/Aleksoid1978/MPC-BE) —— 播放器本体
- 也可以选择[K-Lite Codec Pack](https://www.codecguide.com/)，这是一个 **MPC-HC(非MPC-BE)** 懒人整合包，包含MPC-HC、LAVFilters、madVR等
- [MadVR](http://madshi.net/madVR.zip) —— 视频渲染
    + 下载后解压，运行**install.bat**即可
- madshi目前对MadVR提供了[Test Build](http://madshi.net/madVRhdrMeasure208.zip)，包含更专业的配置项目，尤其是HDR部分，普通用户可选性安装
    + madVRhdrMeasure(Test Build)版本过期时会在播放器提示更新，获取更新的方式为当前版本号+1，例如：`http://madshi.net/madVRhdrMeasure208.zip` ，过期后更新为 `http://madshi.net/madVRhdrMeasure209.zip`
- [LAVFilters](https://github.com/Nevcairiel/LAVFilters) —— 分离器/解码器
- [xy-VSFilter](https://github.com/pinterf/xy-VSFilter) —— 字幕渲染器
    + 下载后解压，运行**Install_XySubFilter.bat**即可

## 配置
### 前言：
除了给出推荐配置外，还会尽可能告诉你每个配置项的含义，可能会有些啰唆(。・ω・。)
### MPC-BE
- `右键`-`选项`-`视频`
    - **视频渲染器**，选择**MadVR**

- `字幕`
    - **字幕**，选择**XySubFilter**

- `滤镜`
    - `内置滤镜`：把**源滤镜**、**视频解码器**、**音频解码器**下的所有滤镜取消勾选，以保证不使用播放器自带的解码器
    - `拓展滤镜`：添加滤镜，在弹出的窗口中选择**LAV Splitter**、**LAV Splitter Source**、**LAV Video Decoder**以及**LAV Audio Decoder**，并确保勾选
    - `拓展滤镜`：将**LAV Video Decoder**和**LAV Audio Decoder**的优先级改为“首选”

### LAVFilters
#### LAV Video Configuration
- 建议配置如下：
    ![LAV Video](https://pica.zhimg.com/v2-72e349f4722e88f5f08eb58a06ccc9ce.webp)
- **Settings**：
    + **Threads for Multi-Threading(多线程)**，选择**Auto**即可
    + **Use Stream Aspect Ratio(使用流宽高比)**，选择 **Indeterminate(Auto)** 即可
    + **Settings for Interlaced Video Streams(交错视频流设置)**：
        - **Field Ordering(场序)**，选择**Auto**：注意Auto可能误判老式DVD，错误的场序会导致画面出现跳帧、画面撕裂等，如果遇到判断错误的情况可以尝试手动启用**top-field-first(顶部优先)**或**bottom-field-first(底部优先)**
        - **Deinterlace Mode(反交错模式)**选择**Auto**即可，如果遇到判断错误的情况可以尝试手动启用
            + 选项解读：
                - **Auto**：使用帧标志(Field Flags)判断隔行/逐行
                - **Aggressive**：强制处理所有**疑似**隔行扫描的帧(可能误判)
                - **Force**：**无视帧标记**，全部按隔行处理（老片源救星）
                - **Disabled**：将所有帧都按逐行扫描处理
    + 右下角的**Software Deinterlace**，是软件反交错，显卡正常的情况下不建议启用
    + 概念解读：
        - **逐行扫描**：将每一帧从左至右、由上至下，逐一的将所有的像素显示出来
        - **隔行扫描**：将一帧图像的奇数行像素及偶数行像素分开，分成为两个场域(field)。轮流扫描奇数行所构成的场及偶数行所构成的场域
        - 现在新式的显示设备大多都是使用**逐行扫描**，在这些新型的显示设备上直接播放**隔行信号**会产生严重的闪烁现象，而且因**隔行信号**两行只有一行有影像另一行则是全黑的，所以亮度看起来会减少一半。由于有上述这些问题，所有使用**逐行扫描**的新式显示设备都需要有 **反交错(Deinterlace)** 的功能
- **Hardware Acceleration(硬件加速)**：
    + **Hardware Decoder to use**，建议**D3D11**，旁边的**Resolution**和**Codecs for HW Decoding**全部勾选(启用所有支持的格式和分辨率)，**Device to use**选择你的高性能显卡
- **Output Format(输出格式)**，**全部勾选**
- 其余选项**默认**即可
- Format分页维持**默认**即可(通常不需要手动启用/禁用特定格式)

#### LAV Audio Configuration
- 建议配置如下：
    ![LAV Audio](https://picx.zhimg.com/v2-f914889caef69fbe165588aa6b8a0b51.webp)
    ![LAV Audio](https://pic1.zhimg.com/v2-32682241e0e6accf9f07138d84a3eb12.webp)
- **Dynamic Range Compression(动态范围压缩)**，可以动态调整音频输出幅值，避免音量忽大忽小/破音，但会损失原声动态细节，一般保持**关闭**即可
- **Bitstream (S/PDIF, HDMI)(比特流)**：
    + **注意**：Bitstream(比特流)需音频设备**原生支持解码**（如功放显示"DTS-HD MA"），若不支持启用格式将导致**无声**
    + 如果你是笔记本扬声器/耳机，此项**保证禁用**，因为笔记本扬声器不支持Bitstream(比特流)输出
    + 如果你有通过 HDMI 连接支持高级音频解码的 AV 功放 或 多声道音箱系统/通过光纤/同轴 (S/PDIF) 连接支持基础解码的 Soundbar 或功放，可以根据**具体设备的支持情况**来选择。例如：直接连接电视时，通常需要确认电视支持的比特流格式（查看电视说明书或音频输出设置），或更简单地选择 PCM 输出让电视解码
    + 配置解读：
        - **Formats**：这里列出了 LAV Audio 可以尝试进行Bitstream(比特流)输出的具体格式。你需要勾选你希望 LAV 尝试传递的格式
        - **Force max DTS+ID rate (not recommended)**：强制最大DTS码率，**不建议使用**
        - **Fallback to PCM if Bitsreaming is not supported**：如果Bitstream(比特流)输出不被支持，则回退到PCM输出，建议**勾选**
    + 概念解读：
        - **Bitstream(比特流)**：将原始、未经解码的音频数据直接从播放器传递给你的外部音频设备(如声卡等)，这需要接收端设备具备相应的解码能力
        - **PCM(脉冲编码调制)**：播放器（LAV Audio）先将音频编码数据解码成原始的、未压缩的多声道 PCM 数据，再输出给音频设备。这是最通用的方式，所有设备都支持 PCM 输出
- **Optins**：
    + **Auto A/V Sync correction(自动音视频同步校正)**，自动音视频同步校正，**勾选**
    + **Convert Output to Standard Channel Layouts(将输出转换为标准声道布局)**，用于HDMI输出PCM时，声道未达到标准(5.1/6.1/7.1)时自动添加静音声道补齐，一般有下游混音器(例如Win自带的WASAPI)则**不需要勾选**，如果下游设备严格要求标准布局（某些老旧设备），则**需要勾选**，对于大多数现代设备（通过 HDMI 连接功放/声卡），都**不需要勾选**
        - **WASAPI**：是Windows管理音频流的底层接口，所有应用的音频数据最终都会通过 WASAPI 驱动到声卡或数字输出设备
    + **Expand Mono to Stereo(将单声道扩展为立体声)**，一般保持**关闭**即可，以避免对单声道源做不必要处理
    + **Expand 6.1 to 7.1(将6.1声道扩展为7.1声道)**，一般保持**关闭**即可，同上
    + **Use Legacy Channel Layout(使用旧版声道布局)**，一般保持**关闭**即可，除非设备不支持新版声道标准
- **Audio Delay(音频延迟)**，基本用不上，保持**关闭**即可
- **Output Format(输出格式)**，**全部勾选**
- **Mixing**页，用于多声道到目标声道的混音
    + 配置解读：
        - **Output Speaker Configuration(输出扬声器配置)**：可选`Stereo`、`4.0`、`6.1`、`5.1`、`7.1`、`mono`，建议**与音频设备完全匹配**
        - **Center Mix Level(中心声道混音级别)**：控制中心声道混入时的音量，一般是**人声/对白**，按需微调
        - **Surround Mix Level(环绕声道混音级别)**：控制环绕声道混入时的音量，一般是**音乐/环境**，按需微调
        - **LFE Mix Level(LFE声道混音级别)**：控制低音声道混入时的音量，一般是**低音**，按需微调
        - **Don't mix Stereo sources(不混音立体声源)**：勾选后，立体声源将不会混音，而是直接输出，建议**启用**，以避免二次处理
        - **Normalize Matrix(矩阵标准化)**：勾选后，将对音频进行全局衰减，使其更安静(音量降低)，以确保音量单音频文件音量一致。虽然不会发生削波，但会导致不同源格式之间的音量不一致，如：立体声会比5.1声道更响亮，但播放单个文件时的音量不会改变，**通常关闭**，由下游均衡器或功放处理
            + 若需全局响度一致，可在播放软件或系统级均衡器中另行处理；LAV 的 Normalize Matrix 更多用于特殊兼容性或录制场景
        - **Clipping Prevention(防止剪切)**：勾选后，保证声道叠加导致的削波失真（爆音），建议**启用**
        - **Matrix Encoding(矩阵编码)**：矩阵编码格式，自行选择

### MadVR
接下来是重头戏——MadVR的配置，MadVR的配置项非常多
有关HDR的内容或许会补充(挖坑.png)，因为HDR(含Test Bulid)的配置项非常多，且复杂，我也没有观看HDR的需求
~~而且我也没有hdr显示器，所以只能靠口胡ο(=•ω＜=)ρ⌒☆（被打~~
screenshot(截图)我用不上，~~相信你也用不上~~，就不占篇幅了
~~写的我快崩溃了X﹏X~~
- **如何进入MadVR配置页面：**
    - 在MPC-BE中，右键-选项-视频-视频渲染器，**属性**，点击**Edit Settings**，即可进入MadVR配置页面
#### devices(设备)
![Devices](https://pica.zhimg.com/v2-d3a8fc02f2a23ef30fc8993701ebde4d.webp)
- 这里是选择你的**设备类别**以供MadVR识别
- **Receiver, Processor, Switch**：信号处理/切换设备，本身不是显示器，而是驱动以下显示终端的前端
- **Digital Monitor / TV**：LCD/LED/OLED/QLED的显示器/电视
- **Digital Projector**：数字投影仪
- **CRT projector**：CRT投影仪
- **CRT monitor ITV**：CRT显示器
- **unknown**：以上之外的设备
- 现代显示器一般是**Digital Monitor / TV**
#### 展开显示器配置
- **properties(属性)**：
    ![Properties](https://pica.zhimg.com/v2-f13cd83ae5e3ebd945cdbb8b26565639.webp)
    + **output level(输出级别)**：一般电脑选择**PC levels(0-255)**，电视选择**TV levels(16-235)**，但还是推荐确认一下你的显示器的输出级别，如果非这两个选项可以选择**custom levels(自定义级别)**，自行填写
    + **bitdepth(位深)**：选择**auto**，自动匹配即可，bitdepth是指显示器**支持**的颜色深度，常见的有8bit、10bit、12bit等，位深越高，颜色越丰富
    + **3D format(3D格式)**：一般选择**auto**，自动匹配即可
- **calibration(校准)**：
    ![Calibration](https://pic4.zhimg.com/v2-7305e323c3beeab966c6e948696fa1a1.webp)
    - **disable calibration for this display(禁用此显示器的校准)**：勾选后，MadVR将不会对显示器进行校准
    - **this display is already calibrated(此显示器已经校准)**：勾选后，MadVR将不会对显示器进行校准，而是直接使用外部校准结果
        + **gamut**与**gamma**按照校准显示器时的参数填写即可
    - **by using yCMS(使用yCMS)**：勾选后，MadVR将使用yCMS进行校准
    - **by using external 3DLUT files(使用外部3DLUT文件)**：勾选后，MadVR将使用外部3DLUT文件进行校准，需要自行准备3DLUT文件
    - 一般而言，如果显示器**进行过专业校准**，则勾选**this display is already calibrated**，**没有经过校准**则勾选或**不确定色域**则勾选**disable calibration for this display**
- **display modes(显示模式)**：
    ![Display Modes](https://pic4.zhimg.com/v2-9c749242ddfede1be7a1bd332d0971a0.webp)
    - **switch to matching display mode(切换到匹配的显示模式)**：勾选后，MadVR将在符合条件时自动切换到匹配的显示模式
        + **when playback starts(播放开始时)**：勾选后，MadVR将在播放开始时切换到匹配的显示模式
        + **when media player goes fullscreen(全屏时)**：勾选后，MadVR将在全屏时切换到匹配的显示模式
    - **restore original display mode(恢复原始显示模式)**：勾选后，MadVR将在符合条件时恢复原始显示模式
        + **when media player is closed(播放器关闭时)**：勾选后，MadVR将在播放器关闭时恢复原始显示模式
        + **when media player leaves fullscreen(退出全屏时)**：勾选后，MadVR将在退出全屏时恢复原始显示模式
    - **list all display modes MadVR switch to(列出MadVR切换到的所有显示模式)**：如果要使用MadVR的显示模式切换功能，此项必须填屏幕支持的显示模式(如：1080p144、1920x1080p60)，否则自动切换不生效，可在**Windows设置-系统—屏幕-高级显示设置-显示器x的显示适配器属性-列出所有显示模式**中查看
    - **treat 25p movies as 24p(将25p电影视为24p)**：勾选后，MadVR将把25p电影视为24p，需要Reclock或VideoClock支持
    - **hack Direct3D to make 24.000Hz and 60.000Hz work(将24.000Hz和60.000Hz的显示模式视为24.000Hz和60.000Hz)**：勾选后，通过绕过 Direct3D 的默认限制，强制系统将显示器刷新率切换至精确的 24.000Hz 或 60.000Hz，而非系统可能锁定的近似值（如 23.976Hz 或 59.940Hz），会导致 **presentation queues(显示队列)** 填不满，增加掉帧风险，依个人喜好选择
    - 分页**custom modes(自定义模式)**，是用于自定义显示模式的，也是madshi提供的**Reclock**替代方案，如果你对显示模式有特殊需求，可以自行修改，请阅读[madshi的文档](http://madvr.com/crt/CustomResTutorial.html)
- **color & gamma(颜色与伽马)**：
    ![Color & Gamma](https://pic1.zhimg.com/v2-04c79992a172b330b2abcb4c5775f9d7.webp)
    - 这一页主要修改伽马，默认值会根据你在 **calibration(校准)** 的设置变化：
        + **disable calibration** ⟶ **gamma**：默认值**2.2**
        + 如果有设置，则与你设置的伽马一致
        + 一般来说，设置为`2.2`，然后根据环境光线动态调整，较暗环境下，伽马可以调高至`2.4`或更高，较亮环境下，伽马可以调低至`2.0`或更低
    - 上方的颜色配置，一般不需要去改，有需要再去调整
#### processing(处理)
- **deinterlacing(反交错)**：
    ![Deinterlacing](https://pic4.zhimg.com/v2-1b4376dd75588779c41a15a3930dfa7c.webp)
- 由于LAV已经提供反交错，madVR获得的信号是LAV处理过的，所以这里直接保持**默认**即可，配置翻译：
    - **Automatically activate deinterlacing when needed (在需要时自动激活反交错)**
        + **if in doubt, activate deinterlacing (若有疑问，激活反交错)**
        + **if in doubt, deactivate deinterlacing (若有疑问，停用反交错)**
    - **disable automatic source type detection (禁用自动源类型检测)**，不同模式反交错算法不同
        + **force film mode (强制电影模式 - 适用于电影/24fps内容)**
        + **force video mode (强制视频模式 - 适用于视频/隔行内容)**
    - **only look at pixels in the frame center (仅检测画面中心的像素 - 适用于广播电视，对动画效果不佳)**，如果会看**电视台**播放的节目的话，建议勾选此选项，如果是看DVD原盘的话，则建议取消勾选
- **artifact removal(去除伪影)**：
    ![Artifact Removal](https://pica.zhimg.com/v2-4aa6596a7c9f5e6372eb630502c0268e.webp)
    - **reduce banding artifacts (减少带状伪影/去色带)**
        + **default debanding strength (默认去带强度)**
        + **strength during fade in/out (淡入/淡出时的强度)**
        + 概念解读：
            - **banding artifacts(带状伪影)**：即**色带**，是一种色彩显示不准确的问题，通常出现在低色深环境(4~8bit)。现在大多数片源(1080p)都是8bit，或多或少含有色带，视频解码时YUV ⟶ RGB，还会产生新的色带
        + 推荐**default debanding strength**设置为`low`，**strength during fade in/out**设置为`medium/high`，强度越高减少色带的效果越明显，但会产生噪点。淡入/淡出时色带会很明显，所以设置为`medium/high`
    - **reduce ringing artifacts (减少环状伪影)**
        + **reduce dark halos around bright edges, too(同时减少亮边缘周围的暗晕)**
            - 避免用于线条锐利的内容（动漫/字幕/工程图），**reduce dark halos**本质是抑制亮边周围的暗区过冲，但动漫的硬边缘（如发丝）会被误判为振铃导致细节丢失
        + 概念解读：
            - **ringing artifacts(环状伪影)**：即**振铃效应**，是一种出现在信号快速转换时，附加在转换边缘上导致失真的信号。而在图像或影像上，振铃效应会导致出现在边缘附近的环带或像是“鬼影”的环状伪影，不恰当的缩放算法、锐化等操作都会导致这种伪影
        + 推荐当观察到 **ringing artifacts(环状伪影)** 时，再启用此选项
    - **reduce comprssion artifacts (减少压缩伪影)**，旁边的项目用来控制强度和质量
        + 该功能专门用于修复视频压缩产生的瑕疵（如色带、蚊式噪声、块效应等），开启后能显著去除网格式噪块，但也会略微模糊细节，推荐当需要时，再启用此选项
        + **process chroma channels, too(同时处理色度通道)**，修复色度通道的伪影（如彩色色带/色块），性能开销较大，一般不启用，在色度伪影肉眼可见且GPU性能冗余时启用
        + **activate only if it comes for free(as part of NGU sharp)(仅在NGU锐化的启用时激活)**，若使用NGU时，强烈建议启用，可以减少NGU的伪影
    - **reduce random noise (减少随机噪声)**，旁边的项目用于控制强度
        + 此项用于减少随机噪声，但会破坏细节，一般不启用，在噪声严重时启用
        + **process chroma channels, too(同时处理色度通道)**，同上
- **image enhancements(图像增强)**：
    + 这一页主要针对影片锐利度不足进行增强，影响原始帧，其处理在放大算法之前，该页面的配置项为锐化算法，不论哪种效果都十分显著，但噪声与瑕疵也会被放大，一般不启用，在需要时启用。锐化算法这里不做解释，自行Google
- **zoom contorl(缩放控制)**：
    + 根据黑边检测、分辨率变化与字幕位置，自动调整缩放或裁切策略，保证内容充满屏幕并保持字幕可见，自行按需调整，一般不需要启用，配置翻译：
    + **disable scaling if image size changes by only ... pixels(如果图像大小仅变化...像素则禁用缩放)**，控制片源与目标分辨率差异低于几个像素时，不进行缩放，而添加黑边代替，避免频繁微调影响流畅性
    + **move subtitles(移动字幕)**，强制设置字幕的位置是画面底部或是影片可视范围内
    + **automatically detect hard coded black bars(自动检测硬编码黑边)**，自动检测硬编码黑边，并非比例不一致添加的黑边，而是影片本身编码时添加的黑边
    + **if black bars change pick one zoom factor**，黑边尺寸变化时，选择不裁剪画面内容的缩放比例、砍除25%黑边、砍除50%黑边、砍除75%黑边、砍除全部黑边
    + **if black bars change quickly back and forth**，如果黑边尺寸变化频繁
        - 当低于2/5/15/45秒时，不切换缩放比例
        - 设定通用的缩放比例
    + **notify media player about cropped black bars**，控制每隔多久通知一次播放器黑边尺寸变化
    + **always shift the image**，总是切齐画面底部/顶部
    + **keep bars visible if they contain subtitles**，如果上面有字幕的话保留黑边（字幕出现后保留黑边5秒、15秒、45秒、3分、10分、30分、90分或永远）
    + **clearup image borders by cropping**，裁切黑边以及其边缘指定像素
    + **if there are big black bars**，如果黑边大小超过范围，直接裁切黑边一定尺寸并放大
    + **zoom small black bars away**，如果黑边小范围，直接裁切黑边并放大
    + **crop black bars**，裁切黑边以及其边缘指定像素，会影响到profile的判定（如果有设置的话）
#### scaling algorithms(缩放算法)
- **chroma upscaling(色度升频)**、**image downscaling(图像降频)**、**image upscaling(图像升频)**，这三个选项是色度放大与图像的放大、缩小算法的配置，配置页注明了**sharpness(锐度)**、**hide source artifacts(隐藏源伪影)**、**ringing(振铃效应)**、**other artifacts(其他伪影)**，其中绿色为效果，红色为失真，不同算法效果、与性能开销不同，请自行权衡
- 推荐：
    - **色度升频**算法优先推荐使用**NGU + Sharp**；性能不足的话，尝试**NGU + Anti-Alias/soft**；其次是**super-xbr + anti-alias**以及**Bilinear + anti-alias**；最次**Cubic(Bicubic 75) + anti-alias**。**anti-ringing filter**能开就开，**SuperRes filter**自行权衡
    - **图像升频**同样优先推荐**NGU Sharp**；性能不足的话尝试**NGU Anti-Alias/soft**；其次是**Cubic(Bicubic 100)**、**Lanczos**等，使用这几种时，**anti-ringing filter**能开就开，而 **scale in sigmoidal light(对数光)** 的降低锯齿效果需要自己测试。**Bilinear**和**DXVA2**效果最次，仅作无法负荷其他算法时使用。使用NGU系列时只需要指定 **luma doubling(亮度解析度翻倍)** 和 **luma quadrupling(亮度解析度四倍)** 的强度，其余 **let madVR decide(让madVR决定)** 即可。
    - **图像降频**算法优先推荐使用**SSIM**，1D是基于Bicubic的，2D是基于Jinc的，2D效果更好，但性能开销更大；性能不足的话，尝试**Cubic(Bicubic 150)**。**scale in linear light(线性光)** 与 **anti-ringing filter(抗振铃)** 能开就开，抗振铃的`strict（soft）`跟`relaxed`，前者是AR滤镜预设强度，后者是降低AR滤镜强度，换得一些锐利度的提升；**activate anti-bloating filter(抗膨胀滤镜)**，它会抑制因过度锐化导致的高频信号'膨胀'或'白边'伪影，是否启用取决于个人喜好
    - 每个设置项的`low`、`medium`、`high`、`very high`均是越高性能开销越大，但效果越好，请自行权衡。
    - 关于**图片升频**，现在的madVR版本，使用NGU系列时只需要指定 **luma doubling(亮度倍增)** 和 **luma quadrupling(亮度四倍)** 的强度，其余 **let madVR decide(让madVR决定)** 即可
    - 关于**always - supersampling(强制超取样)**，选择此项的话即使在源分辨率与目标分辨率一致时，也会强制使用升频算法，而且效果肉眼基本看不出，但性能开销很大，请自行权衡
- **upscaling refinement(升频优化)**，它的副作用相对前面的 **image enhancements(图像增强)** 要小，效果差不多，同样一般不启用，在需要时启用，缩放算法自行Google
#### rendering(渲染)
- **general settings(通用设置)**：
    ![general settings](https://pic2.zhimg.com/v2-0a83e7428c78b137fc4f52ffcbec8b84.webp)
    + **delay playback start until render queue is full(延迟播放直到渲染队列满)**，避免播放器卡顿
        - **delay playback start after seeking, too(在快进后也延迟播放)**，避免播放器卡顿
    + **enable windowed overlay(启用窗口化覆盖模式)**，启用窗口化覆盖模式(游戏的无边框窗口模式)，能绕过部分桌面合成步骤，降低延迟和资源占用，和下面的全屏独占模式二选一
    + **enable automatic fullscreen exclusive mode(启用自动全屏独占模式)**，启用全屏独占模式，能使渲染器直接控制显示输出，绕过DWM，和上面的窗口化覆盖模式二选一，推荐此项
    + **disable desktop composition(禁用窗口管理器)(仅Win7)**，禁用DWM，获得类似全屏独占模式的效果，但渲染能获得更直接的显示权限，Windows 8及以后版本**不支持**
        - **only when media player is in fullscreen mode(仅在播放器全屏时启用)**
    + **use Direct3D 11 for presentation(使用Direct3D 11进行呈现)**，使用Direct3D 11 API进行呈现，性能比Direct3D 9好
        - **present a frame for every VSync(每个垂直同步呈现一帧)**，即常说的垂直同步，这要求视频帧率必须能被显示器刷新率整除(或配置上文的**display modes**)，否则画面可能卡顿
    + **use a separate device for presentation(使用单独的设备进行呈现)**，让 madVR 为“呈现”(Presentation) 操作（将最终处理好的帧发送到显示器）使用一个独立的 Direct3D 设备（和上下文），这能减少对视频解码的干扰
    + **use a separate device for DXVA processing(使用单独的设备进行DXVA处理)**，让 madVR 为 DXVA 处理（视频解码）使用一个独立的 Direct3D 设备（和上下文），也是隔离策略
    + 剩下的滑块**CPU queue size(CPU队列大小)**、**GPU queue size(GPU队列大小)**，控制渲染队列的大小，滑块越大，渲染队列越大，能减少卡顿，但会占用更多RAM，请自行权衡
- **windowed mode(窗口模式)**：
    ![windowed mode](https://picx.zhimg.com/v2-1d7df8d8a10b143cf1c8afcd53101b3c.webp)
    + **present several frames in advance(提前呈现几帧)**，提前呈现几帧以减少卡顿，建议资源允许时启用
    + **how many video frames shall be presented in advance(提前呈现多少帧视频)**，如果启用提前帧，会显示此项，按照GPU性能自行调整，越高减少卡顿的效果越好，但RAM占用越大，请自行权衡。如果使用**smooth motion(平滑运动)**，建议能拉多高就拉多高，也要**相应地增加CPU/GPU队列以填充缓冲区**
    + **how many backbuffers shall be used(使用多少个后台缓冲区)**，如果未启用提前帧，会显示此项，按照RAM自行调整，越高效果越好，但性能开销越大，请自行权衡
    + **when and how shall the GPU be flushed(何时以及如何刷新GPU)**：
        - GPU 刷新（Flush）的作用：GPU 任务以命令队列形式提交，刷新会强制 GPU 立即执行队列中的所有任务，确保后续操作基于最新结果。频繁刷新会导致性能下降，画面卡顿，对于大多数设备**默认方案就是最好的选择**，但某些设备可能需要调整刷新策略
        - **after intermediate render steps(在中间渲染步骤后)**
        - **after last render step(在最后渲染步骤后)**
        - **after copy to backbuffer(在复制到后台缓冲区后)**
        - **after D3D presentation(在DXGI呈现后)**
- **exclusive mode(独占模式)**：
    ![exclusive mode](https://pic3.zhimg.com/v2-fd7406337cd32d6a5047928871c1569a.webp)
    - **show seek bar(显示进度条)**，启用后，在播放器全屏时，会显示一个MadVR提供的进度条
    - **delay switch to exclusive mode by 3 seconds(延迟3秒切换到独占模式)**，不勾选则立即切换到独占模式
    - **其余配置项同*windowed mode(窗口模式)***
- **stereo 3D(立体3D)**：
    - 这一部分是有播放3D视频需求时才需要配置的，一般全部取消勾选
    - **enable stereo 3D playback(启用立体3D播放)**，启用立体3D播放，需要配合立体3D视频播放
    - **when playing 2d content(播放2D内容时)**，控制播放2D内容时是否禁用3D支持
    - **when playing 3d content(播放3D内容时)**，控制播放3D内容时是否禁用3D支持
    - **restore os stereo 3D settings when meida player closed(在媒体播放器关闭时恢复操作系统立体3D设置)**
- **smooth motion(平滑运动)**：
    - 这个功能(FRC)能够将任何源帧率转换为任何目标刷新率，在**性能足够**的情况下，可以达到不使用**Reclock**达到无卡顿/无重复的效果
    - **FRC**实现原理不是补帧，按madshi的说法：“FRC 模拟了一个具有无限刷新率的显示器，每个视频帧都会在时间戳要求的精确时刻显示。因此，运动的流畅度取决于**正确的时间戳**。如果时间戳（或音频时钟）存在抖动，播放也会出现抖动... 完全启用 Reclock 并进行音频重采样仍然是可行的，这不会对 madVR 的 FRC 算法造成损害。” 因此它不具有补帧算法的副作用（如伪影、肥皂剧效应），运动相对自然。代价是轻微的清晰度损失（动态模糊感略有增加）和 GPU 使用率升高（损失程度：低帧率->高刷新率 < 高帧率->低刷新率）
    - 与**补帧插件**建议**二选一**
    - **建议**：如果追求绝对流畅无 judder 且能接受轻微的清晰度损失和 GPU 开销增加，勾选**enable smooth motion**并选择 **...or if the display refresh rate is an exact multiple of the video frame rate** ，希望控制GPU开销，选择**only if there would be motion judder without it...**，always不推荐
    - **enable smooth motion frame rate conversion(启用平滑运动帧率转换)**
        + **only if there would be motion judder without it...(只有在没有它的情况下才会出现运动抖动时才启用)**，仅在视频帧率不能整除显示器刷新率，产生可见抖动时启用
        + **...or if the display refresh rate is an exact multiple of the video frame rate(或者如果显示刷新率是视频帧率的精确倍数)**，它在上一条件的基础上拓展，在显示刷新率是视频帧率的精确倍数时也会启用
        + **always(总是)**，无论当前视频帧率和显示器刷新率的关系如何（无论是否整数倍，无论是否有抖动风险），都强制启用
- **dithering(抖动)**：
    - **None**，不应用任何抖动处理（可能导致色带问题）**十分不建议**
    - **Random Dithering(随机抖动)**，噪声明显，速度快，无抖动图案，极低能耗
    - **Ordered Dithering(有序抖动)**，错误扩散的轻量替代方案，噪声较少，中等抖动图案，低能耗
    - **Error Diffusion - option 1(错误扩散 - 选项1)**，中等噪声，无抖动图案(需要DX11 GPU)
    - **Error Diffusion - option 2(错误扩散 - 选项2)**，低噪声，中等抖动图案(需要DX11 GPU)
    - **options**：
        + **use colored noise(使用彩色噪声)**，低亮度噪声，会增加色度噪声，优化人眼感知，减少可见噪点
        + **change dither for every frame(为每一帧更改抖动)**，隐藏抖动图案，避免固定图案导致的静态闪烁，但主观上增加噪声感
    - **推荐**：如果 `GPU支持DX11`，优先选择**Error Diffusion - option 1**并勾选**use colored noise** 和**change dither for every frame**。这是效果和性能的良好平衡。追求极致选 **option 2**。性能不足选 **Ordered Dithering**
- **trade quality for performance(以质量为代价换取性能)**：
    - 这一页所有项目都是**牺牲质量换性能**，如果性能足够，**一个也不建议勾选！！**
    - **optimize subtitle quality performance instead of quality(优先以性能优化字幕渲染)**，降低字幕渲染精度以节省资源
    - **use DXVA chroma upscaling when doing native DXVA decoding(原生DXVA解码时使用DXVA色度升频)**，用硬件加速色度处理（可能产生锯齿）
    - **use DXVA chroma upscaling when doing DXVA deinterlacing(DXVA反交错时使用DXVA色度升频)**，硬件加速反交错+色度升频（画质损失明显）
    - **don't use linear light for dithering(抖动处理不使用线性光空间)**，关闭线性光抖动（暗部细节减少）
    - **don't analyze gradient angles for debanding(去色带时不分析渐变角度)**，简化梯度计算（可能导致带状伪影残留）
    - **don't rerender frames when fade in/out is detected(检测到淡入/淡出时不重新渲染帧)**，跳过透明度变化的帧处理（过渡不自然）
    - **scale chroma separately, if it saves performance(若可提升性能则单独处理色度通道)**，色度/亮度分离处理（可能色彩错位）
    - **compromise on HDR tone & gamut mapping accuracy(降低HDR色调 & 色域映射精度)**，简化 HDR 动态转换（色彩准确性下降）
    - **compromise on HDR luminance channel quality(降低HDR亮度通道质量)**，压缩高光细节（亮部层次减少）
    - **don't measure HDR frame peak luminance(不测量HDR帧峰值亮度)**，禁用逐帧亮度分析（局部过曝风险）
    - **lose BTB and WTW if it improves performance(若可提升性能则牺牲超黑/超白细节 (BTB/WTW))**，裁剪 <0 或 >255 灰阶（丢失极端暗/亮部）
    - **don't use linear light for smooth motion frame blending(平滑插帧不使用线性光混合)**，简化帧混合计算（运动流畅度下降）
    - **use 10bit chroma buffer instead of 16bit(使用10bit色度缓冲代替16bit)**，降低色度数据精度（色彩过渡轻微断层）
    - **use 10bit image buffer instead of 16bit(使用10bit图像缓冲代替16bit)**，全局降低处理精度（所有画质影响）
    - **run custom pixel shaders in video levels instead of PC levels(在视频电平（16-235）而非PC电平（0-255）运行自定义像素着色器)**，简化色域转换（暗部发灰/亮部过曝）
    - **强烈不建议使用的选项，但决定权在你**：
        + **use half frame rate for DXVA deinterlacing(DXVA反交错使用半帧率)**，帧数减半（运动卡顿明显，仅兼容老式隔行视频）
        + **trust DXVA color & levels conversion(任DXVA的色彩与电平转换)**，绕过 madVR 色彩管理（色偏/灰阶错误高发）

## 尾声
至此，播放器的配置就完成了，如果还有其他问题，请善用搜索引擎，或者翻翻madshi的帖子[更新日志](https://forum.doom9.org/showthread.php?t=146228)，[Test Build(HDR)](https://www.avsforum.com/threads/improving-madvr-hdr-to-sdr-mapping-for-projector-no-support-questions.2954506/#replies)
祝你有个愉快的观影体验！

## 常见问题 Q&A
1. Q：笔记本/非独显直连/只有核显，播放时画面卡顿怎么办？
    - A：这是因为渲染的压力全部分给了核显与CPU。解决方法(按顺序尝试):
        1. 是否有独显，若有尝试b/c；若没有，尝试降低/关闭绝大部分madVR配置
        2. 在独显驱动中使播放器优先使用独显
        3. 若尝试b仍卡顿，尝试降低madVR配置：
            1. 优先**降低&nbsp;*色度升频*&nbsp;配置**，此项性能开销较大，而且降低强度也不会有明显画质损失
            2. 其次是 **降低&nbsp;*图像升频*&nbsp;或&nbsp;*图像降频*&nbsp;配置** ，此项性能开销较大，而且降低强度会有明显画质损失，因此建议在降低色度升频强度后尝试
            3. 最后尝关闭**smooth motion**，此项性能开销较大，而且关闭后可能产生抖动
            4. 尝试以上后若仍无改善，考虑逐次尝试开启**trade quality for performance**中的选项

2. Q：播放时画面卡顿甚至黑屏，但CPU占用率不高，GPU占用率很低甚至为0%，但温度很高，风扇狂转，怎么办？
    - A：倘若显卡驱动正常，则可能是因为显卡过热停机/降频，尝试Q.1中的方法，然后重启设备；播放时观察显卡占用率，推荐保证占用低于80%，否则显卡过热风险较高，也会影响显卡寿命

3. Q：没有声音，怎么办？
    - A：确认是否在设备不支持的情况下启用了**Bitstream**，若确认，尝试关闭**Bitstream**或勾选**Fallback to PCM if Bitsreaming is not supported**；若仍有问题，尝试更换音频输出设备
