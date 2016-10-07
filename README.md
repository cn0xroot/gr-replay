#Wireless Hacking With SDR And GnuRadio

###0x01 信号捕获
市面上常见的无线遥控工作的频段，通常工作在315Mhz、433Mhz，也有少数的会采用868Mhz.915Mhz这几个频点。
我们可以用电视棒、HackRF、BladeRF等SDR硬件来确定遥控的工作频率：
打开软件按下遥控器后，能在瀑布图上看到明显的反应：
`osmocom_fft -F -f 433e6 -s 4e6`

![](http://image.3001.net/images/20160906/14731396517189.png)

`gqrx`

![](http://image.3001.net/images/20160906/14731399552822.png)

无线遥控中心频率：433870000

###0x02 录制信号
SDR软件通常支持录制信号，可将遥控的信号保存为wav音频文件或者以.cfile、.raw格式保存。

这里用gnuradio-companion流图来实现信号录制以及信号重放。

<pre>
wget http://www.0xroot.cn/SDR/signal-record.grc
gnuradio-companion signal-record.grc
</pre>

![](http://image.3001.net/images/20160906/14731408275359.png)

左侧osmocom Source模块调用SDR硬件，我们设置其中心频率为433.874MHz，采样率为2M:

![](http://image.3001.net/images/20160906/14731410493905.png)

右侧上边 QT GUI Sink模块将捕获到的信号在瀑布图上展示出来，右侧下边的File Sink将录制到的信号保存为/tmp/key.raw文件：

![](http://image.3001.net/images/20160906/14731412508422.png)

执行流图，按下遥控前：

![](http://image.3001.net/images/20160906/14731414379973.png)

按下遥控：

![](http://image.3001.net/images/20160906/14731416435960.png)

转到/tmp 缓存目录：

![](http://image.3001.net/images/20160906/14731418105787.png)

###0x03 信号重放
接下来再用gnuradio-companion写个信号重放的流图：

<pre>
wget http://www.0xroot.cn/SDR/signal-replay.grc
gnuradio-companion signal-replay.grc
</pre>

![](http://image.3001.net/images/20160906/14731422579853.png)

左侧File Source调用捕获到的key.raw信号文件,osmocom Sink调用HackRF、BladeRF将信号发射出去，与此同时QT GUI Time Sink、QT GUI Frequency Sink模块分别在屏幕上显示时间轴（时间域）、频率幅度（频率域），执行流图：
![](http://image.3001.net/images/20160906/14731427257276.png)

bingo!

### 0x04 信号分析

`inspectrum key.raw`

![](http://image.3001.net/images/20160907/14732280661074.png)

信号分析&转码细节参考：
[如何使用SDR+inspectrum逆向分析无线遥控信号](https://github.com/cn0xroot/cn0xroot.github.io/tree/master/SDR/signal-analysis)
一文。

![](http://image.3001.net/images/20160907/1473228141351.png)

<pre>
s = ''
a = [0.333033, 0.326189, 0.0332124, 0.388094, 0.326704, 0.0154539, 0.322883, 0.0270275, 0.0150091, 0.443235, 0.362946, 0.027745, 0.430879, 0.443824, 0.0277048, 0.330736, 0.0290668, 0.0133217, 0.376686, 0.0123277, 0.00931546, 0.446231, 0.397617, 0.0162406, 0.447861, 0.0050071, 0.0109479, 0.389289, 0.0271959, 0.0138626, 0.32109, 0.0268736, 0.0129828, 0.401142, 0.326009, 0.0303488, 0.379368, 0.0229494, 0.0134011, 0.318115, 0.346288, 0.017666, 0.333818, 0.326769, 0.0141554, 0.341832, 0.0291055, 0.0153984, 0.446665, 0.399975, 0.024566, 0.316297, 0.0159851, 0.010876, 0.428384, 0.444201, 0.0214323, 0.376211, 0.00628675, 0.0105036, 0.44565, 0.0195615, 0.012549, 0.445242, 0.366523, 0.0225733, 0.324775, 0.0192127, 0.0134437, 0.318991, 0.381386, 0.0149852, 0.00882163, 0.447015]
for i in a:
	if i > 0.1:
		s +='1'
	else:
		s +='0'
print s		
		</pre>
<pre>
python test.py 
 11011010011011010010011010010010011010011011010011010011010010011010011001
</pre> 
 
![](http://image.3001.net/images/20160907/14732283689846.png)

<pre>
pip install bitstring
</pre>

<pre>
python
import bitstring

bitstring.BitArray(bin='11011010011011010010011010010010011010011011010011010011010010011010011001').tobytes()
</pre>		

![](http://image.3001.net/images/20160928/14750512374959.png)
[Automated RF/SDR Signal Analysis [Reverse Engineering]](https://github.com/tresacton/dspectrum)
<pre>
Payload : formatted hexcode
\x36\x9b\x49\xa4\x9a\x6d\x34\xd2\x69\x9
</pre>
thanks for tresacton‘s help [GitHub](https://github.com/tresacton/dspectrum/issues/1)
### 0x05 Hacking The world with watch
德州仪器生产的EZ430 Chronos手表由于采用了MSP430芯片，该芯片支持发射1GHz以下频率的无线信号,覆盖市面上各种常见的无线遥控频率（315MHz、433MHz、868MHz、915MHz）:
![](http://image.3001.net/images/20160907/14732312105191.png)
#### 5.1 开发环境搭建
到 TI德州仪器官网下载：(需注册账号)
CCS studio (Code Composer Studio ):[http://processors.wiki.ti.com/index.php/Download_CCS](http://processors.wiki.ti.com/index.php/Download_CCS)

FET-Pro430-Lite程序:[http://www.elprotronic.com/download.html](http://www.elprotronic.com/download.html)

SmartRF Studio : [http://www.ti.com.cn/tool/cn/smartrftm-studio](http://www.ti.com.cn/tool/cn/smartrftm-studio)

以及GitHub上面的 miChronos项目代码：[http://github.com/jackokring/miChronos](http://github.com/jackokring/miChronos)

百度网盘：[https://pan.baidu.com/s/1hsse2Ni](https://pan.baidu.com/s/1hsse2Ni)

windows 7如果不是Service Pack 1 则需下载安装Windows 7 和 Windows Server 2008 R2 Service Pack 1 (KB976932)补丁，否则无法安装 Code Composer Studio
下载地址：[https://www.microsoft.com/zh-cn/download/confirmation.aspx?id=5842](https://www.microsoft.com/zh-cn/download/confirmation.aspx?id=5842)

![](http://image.3001.net/images/20160907/14732297441519.png)

![](http://image.3001.net/images/20160907/14732320781614.png)

### 0x06 refer

[Michael Ossmann: Software Defined Radio with HackRF, Lesson 11: Replay YouTuBe https://www.youtube.com/watch?v=CyYteFiIozM ](Michael Ossmann: Software Defined Radio with HackRF, Lesson 11: Replay YouTuBe https://www.youtube.com/watch?v=CyYteFiIozM)

[TI eZ430-Chronos Hacking quickstart http://timgray.blogspot.jp/2012/12/ti-ez430-chronos-hacking-quickstart.html](TI eZ430-Chronos Hacking quickstart http://timgray.blogspot.jp/2012/12/ti-ez430-chronos-hacking-quickstart.html)



[The hackable watch: a wearable MSP430 MCU  http://www.itopen.it/the-hackable-watch-a-wearable-msp430-mcu/](The hackable watch: a wearable MSP430 MCU  http://www.itopen.it/the-hackable-watch-a-wearable-msp430-mcu/)

[You can ring my bell! Adventures in sub-GHz RF land...  http://adamsblog.aperturelabs.com/2013/03/you-can-ring-my-bell-adventures-in-sub.html?m=1](You can ring my bell! Adventures in sub-GHz RF land...  http://adamsblog.aperturelabs.com/2013/03/you-can-ring-my-bell-adventures-in-sub.html?m=1)

[TI EZ430 Chronos watch, quick guide / tutorial to hacking the firmware  https://www.youtube.com/watch?v=20dVNyJ8fYw&feature=youtu.be](TI EZ430 Chronos watch, quick guide / tutorial to hacking the firmware  https://www.youtube.com/watch?v=20dVNyJ8fYw&feature=youtu.be)

###Author:[雪碧0xroot](http://www.0xroot.cn) @[漏洞盒子安全团队 VULBOX Security Team](https://www.vulbox.com/)  

