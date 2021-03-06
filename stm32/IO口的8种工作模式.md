STM32的I/O口的8种工作模式

2019年7月23日

18:37

浮空,顾名思义就是浮在空中,上面用绳子一拉就上去了,下面用绳子一拉就沉下去了. 

开漏,就等于输出口接了个NPN三极管,并且只接了e,b. c极 是开路的,你可以接一个电阻到3.3V,也可以接一个电阻到5V,这样,在输出1的时候,就可以是5V电压,也可以是3.3V电压了.但是不接电阻上拉的时候,这个输出高就不能实现了. 

推挽,就是有推有拉,任何时候IO口的电平都是确定的,不需要外接上拉或者下拉电阻. 

 

 

（1）GPIO_Mode_AIN 模拟输入 

（2）GPIO_Mode_IN_FLOATING 浮空输入 

（3）GPIO_Mode_IPD 下拉输入 

（4）GPIO_Mode_IPU 上拉输入 

（5）GPIO_Mode_Out_OD 开漏输出 

（6）GPIO_Mode_Out_PP 推挽输出 

（7）GPIO_Mode_AF_OD 复用开漏输出 

（8）GPIO_Mode_AF_PP 复用推挽输出 

 

推挽电路是两个参数相同的三极管或MOSFET,以推挽方式存在于电路中,各负责正负半周的波形放大任务,电路工作时，两只对称的功率开关管每次只有一个导通，所以导通损耗小、效率高。输出既可以向负载灌电流，也可以从负载抽取电流。推拉式输出级既提高电路的负载能力，又提高开关速度。 

 

开漏输出:输出端相当于三极管的集电极. 要得到高电平状态需要上拉电阻才行. 适合于做电流型的驱动,其吸收电流的能力相对强(一般20ma以内). 

开漏形式的电路有以下几个特点： 

\1. 利用外部电路的驱动能力，减少IC内部的驱动。当IC内部MOSFET导通时，驱动电流是从外部的VCC流经R pull-up ，MOSFET到GND。IC内部仅需很下的栅极驱动电流。 

\2. 一般来说，开漏是用来连接不同电平的器件，匹配电平用的，因为开漏引脚不连接外部的上拉电阻时，只能输出低电平，如果需要同时具备输出高电平的功能，则需要接上拉电阻，很好的一个优点是通过改变上拉电源的电压，便可以改变传输电平。比如加上上拉电阻就可以提供TTL/CMOS电平输出等。（上拉电阻的阻值决定了逻辑电平转换的沿的速度 。阻值越大，速度越低功耗越小，所以负载电阻的选择要兼顾功耗和速度。） 

\3. OPEN-DRAIN提供了灵活的输出方式，但是也有其弱点，就是带来上升沿的延时。因为上升沿是通过外接上拉无源电阻对负载充电，所以当电阻选择小时延时就小，但功耗大；反之延时大功耗小。所以如果对延时有要求，则建议用下降沿输出。 

\4. 可以将多个开漏输出的Pin，连接到一条线上。通过一只上拉电阻，在不增加任何器件的情况下，形成“与逻辑”关系。这也是I2C，SMBus等总线判断总线占用状态的原理。补充：什么是“线与”？： 

在一个结点(线)上, 连接一个上拉电阻到电源 VCC 或 VDD 和 n 个 NPN 或 NMOS 晶体管的集电极 C 或漏极 D, 这些晶体管的发射极 E 或源极 S 都接到地线上, 只要有一个晶体管饱和, 这个结点(线)就被拉到地线电平上. 因为这些晶体管的基极注入电流(NPN)或栅极加上高电平(NMOS),晶体管就会饱和, 所以这些基极或栅极对这个结点(线)的关系是或非 NOR 逻辑. 如果这个结点后面加一个反相器, 就是或 OR 逻辑. 

其实可以简单的理解为：在所有引脚连在一起时，外接一上拉电阻，如果有一个引脚输出为逻辑0，相当于接地，与之并联的回路“相当于被一根导线短路”，所以外电路逻辑电平便为0，只有都为高电平时，与的结果才为逻辑1。 

 

由于浮空输入一般多用于外部按键输入，结合图上的输入部分电路，我理解为浮空输入状态下，IO的电平状态是不确定的，完全由外部输入决定，如果在该引脚悬空的情况下，读取该端口的电平是不确定的。 

上拉输入/下拉输入/模拟输入：这几个概念很好理解，从字面便能轻易读懂。 

复用开漏输出、复用推挽输出：可以理解为GPIO口被用作第二功能时的配置情况（即并非作为通用IO口使用） 

最后总结下使用情况： 

在STM32中选用IO模式 

（1） 浮空输入_IN_FLOATING ——浮空输入，可以做KEY识别，RX1 

（2）带上拉输入_IPU——IO内部上拉电阻输入 

（3）带下拉输入_IPD—— IO内部下拉电阻输入 

（4） 模拟输入_AIN ——应用ADC模拟输入，或者低功耗下省电 

（5）开漏输出_OUT_OD ——IO输出0接GND，IO输出1，悬空，需要外接上拉电阻，才能实现输出高电平。当输出为1时，IO口的状态由上拉电阻拉高电平，但由于是开漏输出模式，这样IO口也就可以由外部电路改变为低电平或不变。可以读IO输入电平变化，实现C51的IO双向功能 

（6）推挽输出_OUT_PP ——IO输出0-接GND， IO输出1 -接VCC，读输入值是未知的 

（7）复用功能的推挽输出_AF_PP ——片内外设功能（I2C的SCL,SDA） 

（8）复用功能的开漏输出_AF_OD——片内外设功能（TX1,MOSI,MISO.SCK.SS） 

STM32设置实例： 

（1）模拟I2C使用开漏输出_OUT_OD，接上拉电阻，能够正确输出0和1；读值时先GPIO_SetBits(GPIOB, GPIO_Pin_0)；拉高，然后可以读IO的值；使用GPIO_ReadInputDataBit(GPIOB,GPIO_Pin_0)； 

（2）如果是无上拉电阻，IO默认是高电平；需要读取IO的值，可以使用带上拉输入_IPU和浮空输入_IN_FLOATING和开漏输出_OUT_OD；