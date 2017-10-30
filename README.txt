/* 关于STM32FLASH的一点修改经验
 * 开发板：正点原子ALIENTEK MiniSTM32 V3.1
 * USB转串口芯片：CH340G
 */

操作顺序: 重点
1. DTR# 输出高电平, RTS# 输出低电平
	(注:此时两个三极管均导通,BOOT0被拉高,配置为“内置ROM”,RESET被拉低，进行复位) 
2. 延时10ms
3. DTR# 输出低电平, RTS# 保持低电平
	(注:DTR#:断开三极管，RESET变高，释放复位, RTS#: BOOT0保持为1,内置ROM模式)
4. 延时100ms让BootLoader程序被执行

具体细节:
1.直接make来编译本程序，若编译不成功，根据提示进行相应修改
2.执行./stm32flash /dev/ttyUSB0读取芯片信息,来测试是否能够
	成功连接, (注：ttyUSB0根据具体设备修改)
3.在Windows上用MCUISP工具烧写程序时，右边会出一些提示信息：
	"DTR电平置低(-3--12V)，复位" 
	"RTS置高(+3--12V),选择进入BootLoader"
	"...延时100毫秒"
	"DTR电平变高(+3--12V)释放复位"
	"RTS维持高"
	"开始连接...,接收到:79" (注：79实际上是0x79 是ACK应应答信号)
	"..."
4.在3中输出的信息中DTR与RTS在真实引脚上是相反的，CH340G上的
	引脚实际上是DTR#与RTS#，所以真实输出的引脚电平要反着理解
5.当执行2不成功时，极有可能是没有正确进入BootLoader,原因是
	serial_poxi.c中的serial_reset串口复位函数没有进行正确配置
	以及在main.c中调用serial_reset函数时,dtr参数没有给对所致.
	如果根据上面的"操作顺序"对代码进行修改后还是不行，那先用minicom
	测试一下是否能成功读取串口打印数据，如果不能，则有可以是串口
	驱动有问题，网上下载对应串口驱动编译插入，再测试能否成功.
6.正确的操作应该是,根据开发板上的USB转串口电路(又名一键下载电路)，
	来正确分析如何操作DTR#与RTS#的电平变化，本例中是以正点原子的板
	载USB转串口电路进行分析，要使芯片进入BootLoader,首先要理解复位
	与BOOT0,BOOT1的关系：
	BOOT1		BOOT0		启动模式		说明
		X				0			Flash				从Flash开始执行程序，用户程序
 		0				1			内置ROM			从ROM开始执行程序，BootLoader固化在内
		1				1			内置SRAM		从SRAM开始执行程序
	板子上已经将BOOT0 BOOT1接了GND,要想进行BootLoader，BOOT1接GND, 
	主要是控制BOOT0和REST,

