@[toc](【Renesas RA6M4开发板之两路PWM驱动】)

# 1.0 PWM 简介
PWM(Pulse Width Modulation , 脉冲宽度调制) 是一种对模拟信号电平进行数字编码的方法，通过不同频率的脉冲使用方波的占空比用来对一个具体模拟信号的电平进行编码，使输出端得到一系列幅值相等的脉冲，用这些脉冲来代替所需要波形的设备。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2231e4949e5542898d127dc27de48775.png)
上图是一个简单的 PWM 原理示意图，假定定时器工作模式为向上计数，当计数值小于阈值时，则输出一种电平状态，比如高电平，当计数值大于阈值时则输出相反的电平状态，比如低电平。当计数值达到最大值是，计数器从0开始重新计数，又回到最初的电平状态。高电平持续时间（脉冲宽度）和周期时间的比值就是占空比，范围为0~100%。上图高电平的持续时间刚好是周期时间的一半，所以占空比为50%。
## 1.1 原理
一个比较常用的pwm控制情景就是用来调节灯或者屏幕的亮度，根据占空比的不同，就可以完成亮度的调节。PWM调节亮度并不是持续发光的，而是在不停地点亮、熄灭屏幕。当亮、灭交替够快时，肉眼就会认为一直在亮。在亮、灭的过程中，灭的状态持续时间越长，屏幕给肉眼的观感就是亮度越低。亮的时间越长，灭的时间就相应减少，屏幕就会变亮。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f84c74fbdd79424c8ab0a668ddf612b1.png)

## 1.2 访问 PWM 设备
应用程序通过 RT-Thread 提供的 PWM 设备管理接口来访问 PWM 设备硬件，相关接口如下所示：

|函数	|描述|
| ------ | ------ |
|[rt_device_find()](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pwm/pwm?id=%E6%9F%A5%E6%89%BE-pwm-%E8%AE%BE%E5%A4%87)	|根据 PWM 设备名称查找设备获取设备句柄|
|[rt_pwm_set()](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pwm/pwm?id=%E8%AE%BE%E7%BD%AE-pwm-%E5%91%A8%E6%9C%9F%E5%92%8C%E8%84%89%E5%86%B2%E5%AE%BD%E5%BA%A6)	|设置 PWM 周期和脉冲宽度|
|[rt_pwm_enable()](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pwm/pwm?id=%E4%BD%BF%E8%83%BD-pwm-%E8%AE%BE%E5%A4%87)	|使能 PWM 设备|
|[rt_pwm_disable()](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/device/pwm/pwm?id=%E5%85%B3%E9%97%AD-pwm-%E8%AE%BE%E5%A4%87%E9%80%9A%E9%81%93)	|关闭 PWM 设备|


# 2. RT-theard配置
## 2.1 硬件需求
> 实现功能：
> 板载LED3（P106）和P107的LED两路PWM驱动。

1、RA6M4开发板
![在这里插入图片描述](https://img-blog.csdnimg.cn/4c5dcda23c6d4afaacb393dc46a7ae51.png)
2、USB下载线，ch340串口和附带4根母母线，**rx---p613;tx---p614**
3、准备LED灯一个，正极接3.3V，负极接P107，板载LED3（P106）不变
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9f9359066fa4774a54c47359a2b42ce.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/8b0fa6e9149248cfa3a7ba4cb9f5aa5e.png)
硬件到此配置完成


## 2.2 软件配置
Renesas RA6M4开发板环境配置参照：[【基于 RT-Thread Studio的CPK-RA6M4 开发板环境搭建】](https://blog.csdn.net/vor234/article/details/125634313)
1、新建项目RA6M4-pwm工程
![在这里插入图片描述](https://img-blog.csdnimg.cn/674569cca175498e9c7323f58701c3bd.png)

2、查阅RA6M4硬件资源，相关资料，在RT-theard Setting 硬件下开启PWM，使能pwm8
![在这里插入图片描述](https://img-blog.csdnimg.cn/5bec936e93b347fe9e9b0bbcaa91ec56.png)
pdf文档第21章pwm
![在这里插入图片描述](https://img-blog.csdnimg.cn/bd2c69206b5a41a0bd5352ddcc5aa8e8.png)
需要使能pwm8
![在这里插入图片描述](https://img-blog.csdnimg.cn/9348dac7b4e84f6d9c257aaa65af7942.png)

3、打开RA Smart Congigurator，在Stacks中New Stack添加r_gpt
![在这里插入图片描述](https://img-blog.csdnimg.cn/f71cdf8fc9fc4372bce12c7095779492.png)

4、在Property的Module的General中选Channel8，Pins选择P107和P106
![在这里插入图片描述](https://img-blog.csdnimg.cn/0e852cea24ee44788ad6b5f794a67470.png)

5、然后Generate Project Content 同步更新刚刚配置的文件

图形化配置已经完成，接下来配置相关代码
# 3. 代码分析
1、修改`hal_entry.c`函数，屏蔽LED3普通GPIO输出

```cpp
/*
 * Copyright (c) 2006-2021, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author        Notes
 * 2021-10-10     Sherman       first version
 * 2021-11-03     Sherman       Add icu_sample
 */

#include <rtthread.h>
#include "hal_data.h"
#include <rtdevice.h>

//#define LED3_PIN    BSP_IO_PORT_01_PIN_06
#define USER_INPUT  "P105"

void hal_entry(void)
{
    rt_kprintf("\nHello RT-Thread!\n");

    while (1)
    {
//        rt_pin_write(LED3_PIN, PIN_HIGH);
//        rt_thread_mdelay(500);
//        rt_pin_write(LED3_PIN, PIN_LOW);
        rt_thread_mdelay(500);
    }
}

void irq_callback_test(void *args)
{
    rt_kprintf("\n IRQ00 triggered \n");
}

void icu_sample(void)
{
    /* init */
    rt_uint32_t pin = rt_pin_get(USER_INPUT);
    rt_kprintf("\n pin number : 0x%04X \n", pin);
    rt_err_t err = rt_pin_attach_irq(pin, PIN_IRQ_MODE_RISING, irq_callback_test, RT_NULL);
    if(RT_EOK != err)
    {
        rt_kprintf("\n attach irq failed. \n");
    }
    err = rt_pin_irq_enable(pin, PIN_IRQ_ENABLE);
    if(RT_EOK != err)
    {
        rt_kprintf("\n enable irq failed. \n");
    }
}
MSH_CMD_EXPORT(icu_sample, icu sample);

```

在src文件下新建pwmled.c和pwmled.h文件，其他不变。
![在这里插入图片描述](https://img-blog.csdnimg.cn/aaa79402d2e847e18c7fb76030825986.png)


`pwmled.c`
```cpp
/*
 * Copyright (c) 2006-2021, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2022-07-11     Asus       the first version
 */
/*
 * 程序清单：这是一个 PWM 设备使用例程
 * 例程导出了 pwm_led_sample 命令到控制终端
 * 命令调用格式：pwm_led_sample
 * 程序功能：通过 PWM 设备控制 LED 灯的亮度，可以看到LED不停的由暗变到亮，然后又从亮变到暗。
*/

#include <rtthread.h>
#include <rtdevice.h>
#define PWM_DEV_NAME        "pwm8"  /* PWM设备名称 */
#define PWM_DEV_CHANNEL      0   /* PWM通道 */
struct rt_device_pwm *pwm_dev;      /* PWM设备句柄 */
//static int pwm_led_sample(int argc, char *argv[])
int pwm_led_sample(void)
{
    rt_uint32_t period, pulse, dir;
    period = 500000;    /* 周期为0.5ms，单位为纳秒us */
    dir = 1;            /* PWM脉冲宽度值的增减方向 */
    pulse = 0;          /* PWM脉冲宽度值，单位为纳秒ns */
    /* 查找设备 */
    pwm_dev = (struct rt_device_pwm *)rt_device_find(PWM_DEV_NAME);
    if (pwm_dev == RT_NULL)
    {
        rt_kprintf("pwm sample run failed! can't find %s device!\n", PWM_DEV_NAME);
        return RT_ERROR;
    }
    /* 设置PWM周期和脉冲宽度默认值 */
    rt_pwm_set(pwm_dev, PWM_DEV_CHANNEL, period, pulse);
    rt_pwm_set(pwm_dev, 1, period, pulse);
    /* 使能设备 */
    rt_pwm_enable(pwm_dev, PWM_DEV_CHANNEL);
    rt_pwm_enable(pwm_dev, 1);
    while (1)
    {
        rt_thread_mdelay(50);
        if (dir)
        {
            pulse += 5000;      /* 从0值开始每次增加5000ns */
        }
        else
        {
            pulse -= 5000;      /* 从最大值开始每次减少5000ns */
        }
        if (pulse >= period)
        {
            dir = 0;
        }
        if (0 == pulse)
        {
            dir = 1;
        }
        /* 设置PWM周期和脉冲宽度 */
        rt_pwm_set(pwm_dev, PWM_DEV_CHANNEL, period, pulse);
        rt_pwm_set(pwm_dev, 1, period, abs(period-pulse));
    }
}
/* 导出到 msh 命令列表中 */
MSH_CMD_EXPORT(pwm_led_sample, pwm sample);

```
`pwmled.h`

```cpp
/*
 * Copyright (c) 2006-2021, RT-Thread Development Team
 *
 * SPDX-License-Identifier: Apache-2.0
 *
 * Change Logs:
 * Date           Author       Notes
 * 2022-07-11     Asus       the first version
 */
#ifndef PWMLED_H_
#define PWMLED_H_

int pwm_led_sample(void);

#endif /* PWMLED_H_ */

```
**保存完是灰色，没有保存是蓝色。**
`pwm_led_sample`导 出 到 msh 命 令 列 表 中，实现pwm8的两路输出


# 4. 下载验证
1、编译重构
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a67aae083ce403692b643b84771024b.png)

编译成功

2、下载程序
![在这里插入图片描述](https://img-blog.csdnimg.cn/d5cdde684a414d76af483aa14a445c29.png)
下载成功

3、CMD串口调试

![在这里插入图片描述](https://img-blog.csdnimg.cn/181227ee2ed64ef2801477ece50cf41c.png)
然后板载复位，输入：`pwm_led_sample`
![在这里插入图片描述](https://img-blog.csdnimg.cn/3615450cf919443997e9e46759ac1ca8.png)

效果如下
![请添加图片描述](https://img-blog.csdnimg.cn/37f99084c179459dbaa0273a5358d9af.gif)



这样我们就可以天马行空啦!
![请添加图片描述](https://img-blog.csdnimg.cn/92099d4d054b4b2cbd39b95719739a90.gif)

参考文献；
[【基于 RT-Thread Studio的CPK-RA6M4 开发板环境搭建】](https://blog.csdn.net/vor234/article/details/125634313)
[【开发板评测】Renesas RA6M4开发板之PWM呼吸灯](https://club.rt-thread.org/ask/article/606a6ea62d8845fc.html "【开发板评测】Renesas RA6M4开发板之PWM呼吸灯")
pwm端口是成对存在的，一共有10对
