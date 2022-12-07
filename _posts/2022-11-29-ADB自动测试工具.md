---
title: ADB自动测试工具
author: wittykyrie
date: 2022-11-29 09:00:00 +0800
categories: ['ADB']
tags: ['adb','Android']
render_with_liquid: false
---

目的是因为游戏发包时，可能由于更改了SDK导致游戏稳定性变差，会导致Crash或者[ANR](https://developer.android.com/topic/performance/vitals/anr)的出现,因此需要制作一款工具去完成自动打开应用并根据关键信息收集日志的工具。

## ADB需要用的指令

```
adb devices //检查是否有设备进行连接
```

**启动Android App**
```
adb shell am start -n [PACKAGE-NAME]/[ACTIVITY-NAME]
```
其中`PACKAGE-NAME`的参数部分，也就是App的package name，也可以透过以下的指令来进行查看
```
adb shell pm list packages （-s：列出系统应用包名  -3：列出第三方应用包名 -f：列出应用包名及对应的apk名及存放位置  -i：列出应用包名及其安装来源）
```
接着`ACTIVITY-NAME`可以通过以下指令查看，当前运行App的Activity的名称。
```
adb shell dumpsys activity
```
举个例子，假设手机里装上了一款“元气骑士”，我们通过更改上面提到的例子，得到了游戏的包体名称“com.knight.union.bili”，接着我们打开元气骑士的App后，执行`adb shell dumpsys activity`指令，我们在输出的log中寻找包体的关键字，就可以找到如下关键字`com.knight.union.bili/com.chillyroomsdk.bilibili.PrivacyActivity`这个关键字就是我们当前游戏初始画面的Activity了。

所以结合上面的几个代码，我们就可以通过下面这一行的指令顺利的启动我们的App。
```
adb shell am start -n com.knight.union.bili/com.chillyroomsdk.bilibili.PrivacyActivity
```

**获取界面UI控件信息并模拟点击**

可以在Python当中导入Poco库，Poco库当中封装好了一些定位UI的方法,最常用方便的是用Text来进行定位

```python
from poco.drivers.android.uiautomation import AndroidUiautomationPoco

if __name__ == '__main__':
    poco(text="元气骑士").click()
```

>有时候会遇到`devices lineoff`的情况，一般用重启手机设备，或重启adb server的方式可以解决
{: .prompt-warning }

除此之外，网易所提供的[Airtest工具](https://github.com/AirtestProject/Airtest)也提供了一个完整的自动测试环境来协助我们对设备进行UI层面的控制。

**使用Python制作的完整的监控关键词工具**

```python
import datetime
import os
import subprocess
import threading
import json
import sys

LOG_PATH = ""

KEYWORDS = []

STOP_LOGCAT = True

INSTRUCTION = {
    "1": "filter_keywords",
    "2": "stop_filter_keywords",
    "3": "exit"
}


def filter_keywords():
    global STOP_LOGCAT
    STOP_LOGCAT = False
    devices = get_devices()  # 先获取所有连接的设备
    print("开始监控关键字")
    for device in devices:
        t = threading.Thread(target=filter_keyword, args=(device,))
        t.start()


def stop_filter_keywords():
    global STOP_LOGCAT
    if STOP_LOGCAT:
        print("没有正在执行的任务\n")
    else:
        STOP_LOGCAT = True
        print("正在停止关键字监控\n")


def filter_keyword(device):
    print("设备%s关键字监控已开启" % str(device))
    sub = logcat(device)
    with sub:
        for line in sub.stdout:  # 子进程会持续输出日志，对子进程对象.stdout进行循环读取
            for key in KEYWORDS:
                if line.decode("utf-8").find(key) != -1:
                    message = "设备：%s 检测到：%s 日志信息：%s\n" % (device, key, line)
                    path = get_log_path("bugreport")
                    bugreport(device, path, message)
                    print(message)
            if STOP_LOGCAT:
                break
        print("设备%s关键字监控已停止" % str(device))
        sub.kill()


def logcat(device):
    command = "adb logcat -c"
    subprocess.call(command)

    command = "adb -s " + str(device) + " logcat -v time *:W"
    sub = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    return sub


def get_devices():
    command = "adb devices"
    res = os.popen(command).read()
    devices = []
    res = res.split("\n")
    for i in res:
        if i.endswith("device"):
            devices.append(i.split('\t')[0])
    return devices


def bugreport(device, path, message):
    os.chdir(path)
    year = datetime.datetime.now().strftime('%Y')
    month = datetime.datetime.now().strftime('%m')
    day = datetime.datetime.now().strftime('%d')

    command = "adb -s " + str(device) + " bugreport"
    subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, bufsize=-1)
    print("设备：%s 日志路径：%s" % (str(device), path))

    keyword_logcat = open("keyword_logcat%s-%s-%s.txt" % (year, month, day), "a")
    keyword_logcat.write(message)
    keyword_logcat.close()


def get_log_path(tag):
    year = datetime.datetime.now().strftime('%Y')
    month = datetime.datetime.now().strftime('%m')
    day = datetime.datetime.now().strftime('%d')
    path = os.path.join(LOG_PATH, tag, "%s %s %s" % (year, month, day))
    if not os.path.exists(path):
        os.makedirs(path)
    return path


def read_config():
    f = open('SDKTestConfig.json')
    data = json.load(f)
    for i in data['KEYWORDS']:
        KEYWORDS.append(i['word'])

    global LOG_PATH
    LOG_PATH = os.path.join(data['LOG_PATH'])
    print(LOG_PATH)
    return data


def main():
    read_config()
    while True:
        print("-" * 100)
        print("1：开启关键字监控  2：停止关键字监控  3:exit")
        print("-" * 100)
        instruction = str(input("\n请输入要进行的操作号:"))
        print("-" * 100)
        while instruction not in INSTRUCTION.keys():
            instruction = str(input("\n输入无效，请重新输入:"))
        if int(instruction) == 3:
            stop_filter_keywords()
            sys.exit(0)
            # exit()
        eval(INSTRUCTION[str(instruction)] + "()")


if __name__ == '__main__':
    main()

```

以下是Json的配置文件，主要配置关键词以及logcat的输出路径

```json
{
    "KEYWORDS" : [
        {"word" : "ANR"},
        {"word" : "NullPointerException"},
        {"word" : "CRASH"},
        {"word" : "Force Closed"},
        {"word" : "ab"}
    ],

    "LOG_PATH" : "C:\\Users\\Administrator\\Desktop\\SDKTestTool"
}
```

参考文章：
1. [如何使用Adb-command-line启动AndroidApp](https://kkboxsqa.wordpress.com/2014/08/20/%E5%A6%82%E4%BD%95%E9%80%8F%E9%81%8E-adb-command-line-%E6%8C%87%E4%BB%A4%E5%95%9F%E5%8B%95-android-app/)
2. [Android原生App UIPoco库](https://github.com/xiaocong/uiautomator#basic-api-usages)