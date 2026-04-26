# Canon EOS R6 Mark III GPS tagging


## 背景
当相机经过一段时间空闲后，再次使用时一直连不上蓝牙，也无法获取GPS信号（屏幕闪烁）。如果解锁手机，
并把camera connect调到前台，会恢复。但是这太麻烦。我希望从休眠中恢复过来，不用再确认甚至手动设
置（仅在相机拨盘开机时，人工设置并确认蓝牙和GPS连接）。

## 硬件
- 佳能R6 Mark III相机(firmware 1.0.1)
- 苹果手机（iPhone 16, iOS 26）

## 复现过程
1. 手机安装Camera Connect V3
2. 在手机设置中，定位权限设为always，开启精确位置，开启蓝牙，允许后台刷新 
3. 相机拨盘开机 
4. 打开Camera Connect，完成与相机配对 
5. 相机GPS设置为手机GPS，拍摄屏幕显示GPS已连接 
6. 手机锁屏（未关闭Camera Connect），相机保持开机状态，设置自动关机（或者休眠） 
7. 等待一段时间后，相机进入休眠 
8. 按快门键，相机从休眠中恢复，发现无法连接蓝牙，也无法获取GPS信号（屏幕闪烁），非预期现象
9. 解锁手机，打开camera connect调到前台，相机恢复正常，GPS信号恢复。不希望执行此步骤，太麻烦了。

## 进一步详细测试
1. 手机不锁屏，CC前台运行，保持不动；20:55相机开机，GPS闪烁几秒钟后蓝牙和GPS正常，查看GPS时间为当前
时间20:55；关闭自动关机，拨盘在开机，等待2分钟以上，按快门，蓝牙和GPS保持连接，但GPS时间未更新20:55，
但过了几秒钟更新为21:01；相机点击快门恢复到拍照界面，等待到21:15点击快门，蓝牙和GPS保持连接，GPS时间
未更新，仍为21:01；相机点击快门恢复到拍照界面，等到21:23点击快门，蓝牙和GPS保持连接，GPS时间未更新，
仍为21:01；相机点击快门恢复到拍照界面，等到21:32点击快门，蓝牙和GPS保持连接，GPS时间未更新，仍为21:01；
结论：相机不锁屏、CC前台，相机不自动关闭电源，蓝牙能保持连接， 但是GPS没有得到及时更新（至少不稳定）；
2. 手机保持上述状态不变；21:33相机开启1分钟自动关闭电源，点击快门到拍照界面，等待到21:42点击快门，蓝牙
灰GPS闪烁，几秒钟后正常，查看GPS信息时间为21:42；点击快门，等待到21:50点击快门，GPS和蓝牙几秒钟之内恢
复，GPS时间为21:50； 点击快门，等待到21:58点击快门，GPS和蓝牙几秒钟之内恢复，GPS时间为21:58；点击快
门，等待到22:08点击快门，GPS和蓝牙几秒钟之内恢复，GPS时间为22:08；
结论：相机不锁屏、CC前台，相机自动关机。每次按快门后，蓝牙和GPS几秒内恢复，并且GPS时间得到更新；
3. 22:09 CC 前台，锁屏；相机按快门，等待到22:29点击快门，蓝牙和GPS几秒内恢复，GPS时间为22:29；点击
快门，等待到22:42（中途有未接微信来电）点击快门，蓝牙和GPS几秒内恢复，GPS时间为22:42；点击相机快门，等
待到23:12，点击快门，蓝牙和GPS几秒内恢复，GPS时间为23:12；点击相机快门，23:30点击快门，蓝牙和GPS几秒
内恢复，GPS时间为23:30；按快门，等待到23:59分，按快门，蓝牙和GPS几秒内恢复，GPS时间为23:59；23:59重
启cc，保持前台，锁屏，00:13按快门，蓝牙和GPS几秒内恢复，GPS时间为00:13；
结论：相机锁屏，CC前台，相机自动关机。每次按快门后，蓝牙和GPS几秒内恢复，并且GPS时间得到更新；
4. 00:14 重启CC，后台，锁屏，00:20按快门，00:27按快门，蓝牙和GPS几秒内恢复，GPS时间为00:27；00:36
按快门，蓝牙和GPS几秒内恢复，并且GPS时间得到更新。见鬼了
5. 



## 分析
### 查看服务和特征

针对我的硬件，执行以下代码
```python
DEVICE_ADDRESS = "31C8F9D7-6944-5768-FB57-A019297A1254"
async def list_all_characteristics():
print(f"正在连接设备：{DEVICE_ADDRESS}")
    # 连接设备
    async with BleakClient(DEVICE_ADDRESS) as client:
        print("✅ 连接成功！开始遍历所有服务和特征值...\n")
        
        # 遍历设备的所有 GATT 服务
        for service in client.services:
            print(f"[服务] UUID: {service.uuid}")
            print("  └─ 特征值列表：")
            
            # 遍历当前服务下的所有特征值
            for char in service.characteristics:
                # 获取特征值支持的操作（读、写、通知等）
                properties = ", ".join(char.properties)
                print(f"    ✔ 特征值 UUID: {char.uuid}")
                print(f"      支持操作: {properties}\n")

# 运行程序
await list_all_characteristics()
```

得到结果：
```text
正在连接设备：31C8F9D7-6944-5768-FB57-A019297A1254
✅ 连接成功！开始遍历所有服务和特征值...

[服务] UUID: 0000180a-0000-1000-8000-00805f9b34fb
└─ 特征值列表：
    ✔ 特征值 UUID: 00002a29-0000-1000-8000-00805f9b34fb
    支持操作: read

    ✔ 特征值 UUID: 00002a24-0000-1000-8000-00805f9b34fb
      支持操作: read

    ✔ 特征值 UUID: 00002a26-0000-1000-8000-00805f9b34fb
      支持操作: read

    ✔ 特征值 UUID: 00002a28-0000-1000-8000-00805f9b34fb
      支持操作: read

    ✔ 特征值 UUID: 00002a25-0000-1000-8000-00805f9b34fb
      支持操作: read

[服务] UUID: 00010000-0000-1000-0000-d8492fffa821
└─ 特征值列表：
    ✔ 特征值 UUID: 00010005-0000-1000-0000-d8492fffa821
    支持操作: read

    ✔ 特征值 UUID: 0001000a-0000-1000-0000-d8492fffa821
      支持操作: write, write-without-response

    ✔ 特征值 UUID: 0001000b-0000-1000-0000-d8492fffa821
      支持操作: read

    ✔ 特征值 UUID: 00010006-0000-1000-0000-d8492fffa821
      支持操作: write, indicate, write-without-response

    ✔ 特征值 UUID: 0001000c-0000-1000-0000-d8492fffa821
      支持操作: indicate, read

[服务] UUID: 00020000-0000-1000-0000-d8492fffa821
└─ 特征值列表：
    ✔ 特征值 UUID: 00020001-0000-1000-0000-d8492fffa821
    支持操作: read

    ✔ 特征值 UUID: 00020002-0000-1000-0000-d8492fffa821
      支持操作: write, notify, write-without-response

    ✔ 特征值 UUID: 00020003-0000-1000-0000-d8492fffa821
      支持操作: indicate, read

    ✔ 特征值 UUID: 00020004-0000-1000-0000-d8492fffa821
      支持操作: read

    ✔ 特征值 UUID: 00020005-0000-1000-0000-d8492fffa821
      支持操作: read

    ✔ 特征值 UUID: 00020006-0000-1000-0000-d8492fffa821
      支持操作: read

[服务] UUID: 00030000-0000-1000-0000-d8492fffa821
└─ 特征值列表：
✔ 特征值 UUID: 00030001-0000-1000-0000-d8492fffa821
支持操作: notify, read

    ✔ 特征值 UUID: 00030002-0000-1000-0000-d8492fffa821
      支持操作: notify

    ✔ 特征值 UUID: 00030010-0000-1000-0000-d8492fffa821
      支持操作: write, write-without-response

    ✔ 特征值 UUID: 00030011-0000-1000-0000-d8492fffa821
      支持操作: notify, read

    ✔ 特征值 UUID: 00030020-0000-1000-0000-d8492fffa821
      支持操作: write, write-without-response

    ✔ 特征值 UUID: 00030021-0000-1000-0000-d8492fffa821
      支持操作: notify, read

    ✔ 特征值 UUID: 00030030-0000-1000-0000-d8492fffa821
      支持操作: write, write-without-response

    ✔ 特征值 UUID: 00030031-0000-1000-0000-d8492fffa821
      支持操作: notify, read

[服务] UUID: 00040000-0000-1000-0000-d8492fffa821
└─ 特征值列表：
✔ 特征值 UUID: 00040001-0000-1000-0000-d8492fffa821
支持操作: read

    ✔ 特征值 UUID: 00040002-0000-1000-0000-d8492fffa821
      支持操作: write, write-without-response

    ✔ 特征值 UUID: 00040003-0000-1000-0000-d8492fffa821
      支持操作: indicate, read
```

### 抓包分析
#### 参考信息
1. https://iandouglasscott.com/2017/09/04/reverse-engineering-the-canon-t7i-s-bluetooth-work-in-progress/
2. https://iandouglasscott.com/2018/07/04/canon-dslr-bluetooth-remote-protocol/
3. https://github.com/ids1024/cannon-bluetooth-remote
4. https://github.com/3bl3gamer/canon-bluetooth-control
5. https://github.com/gkoh/furble/issues/189
6. https://github.com/iebyt/cbremote
7. https://github.com/pklaus/canoremote
8. https://github.com/RReverser/eos-remote-web
9. https://github.com/ArthurFDLR/BR-M5
10. https://github.com/Jerroder/remote-lens

### 豆包自动帮忙总结了以下信息（正确性待验证）：
|           扫描 UUID           | 社区逆向确认功能	| 来源|
|:---------------------------:|:--------:|:------:|
| 00030000-xxxx-d8492fffa821	 | 相机状态 / 唤醒 / 保活	| Remote Lens|
|       00030010-xxxx	        | 唤醒指令写入	| Remote Lens|
|       00030011-xxxx	        | 休眠 / 唤醒状态通知	| Remote Lens|
| 00040000-xxxx-d8492fffa821	 | GPS 控制服务	| Furble #189|
|       00040002-xxxx	        | GPS 数据写入	| Furble #189| 

#### 我自己抓包
使用packetlogger抓包，导出为BTSnoop格式，使用wireshark分析。
请指导一下如何分析抓包结果，先学习下如何分析，我当前用的是python bleak库，怎么把抓的包和bleak API对应起来？

##### wireshark 增加Opcode、handle、value三列

##### 为什么 0x02/0x03/0x04/0x08 没有 Handle
这 4 个是面向 “整个连接 / 一段范围” 的操作，不是针对单个特征（单个 Handle），所以天生不带 Attribute Handle

##### Handle ↔ UUID 怎么对应？
- 一个 UUID（特征） → 对应 唯一一个 Handle
- 0x05 包 = 相机直接告诉你：这个 Handle 对应 这个 UUID （短）
- 0x05 里的短 UUID = 蓝牙标准特征的缩写，套固定公式 0000xxxx-0000-1000-8000-00805f9b34fb = 完整 UUID
- 

##### Opcode和bleak API对应（from 豆包）
| Opcode(十六进制) | 操作名称 | 消息类型 | 配对Opcode | 对应Bleak API / 行为 | Handle 规则 | Value 规则 | 相机核心作用 |
| :---: | --- | --- | --- | --- | --- | --- | --- |
| 0x01 | Error Response | 响应 | 所有请求 | 无主动API，Bleak抛出异常 | 有（指向出错的句柄） | 无（仅错误码） | 连接/读写失败报错 |
| 0x02 | Exchange MTU Request | 请求 | 0x03 | 无主动API，Bleak自动执行 | 无（全局连接操作） | 无（仅MTU参数） | 协商蓝牙数据长度 |
| 0x03 | Exchange MTU Response | 响应 | 0x02 | 无任何Bleak API（系统自动处理） | 无（全局连接操作） | 无（仅MTU参数） | 相机回复MTU协商结果 |
| 0x04 | Find Information Request | 请求 | 0x05 | read_char_by_uuid() 内部调用 | 无（范围查询） | 无（仅查询参数） | 查询Handle与UUID的映射关系 |
| 0x05 | Find Information Response | 响应 | 0x04 | 无主动API，是read_char_by_uuid()的内部返回数据 | 有（映射的目标句柄） | 无（返回UUID映射表） | 相机返回：Handle ↔ UUID 对应关系 |
| 0x08 | Read By Type Request | 请求 | 0x09 | read_gatt_char() 主动调用 | 无（范围读取） | 无（仅读取参数） | 按类型读取相机信息/状态 |
| 0x09 | Read By Type Response | 响应 | 0x08 | 无主动API，是read_gatt_char()的返回结果 | 有（读取的句柄） | 有（读取到的数据） | 相机返回设备名、状态信息 |
| 0x0a | Read Request | 请求 | 0x0b | read_gatt_char() 主动调用 | 有（目标特征句柄） | 无（仅指定读哪个） | 读取GPS、相机状态单值 |
| 0x0b | Read Response | 响应 | 0x0a | 无主动API，是read_gatt_char()的返回结果 | 无 | 有（读取的真实数据） | 相机返回GPS数据/状态值 |
| 0x12 | Write Request | 请求 | 0x13 | write_gatt_char() 主动调用 | 有（目标特征句柄） | 有（写入的指令/数据） | 发唤醒、GPS、控制指令 |
| 0x13 | Write Response | 响应 | 0x12 | 无主动API，是write_gatt_char()的回执 | 无 | 无（仅代表写入成功） | 相机确认写入指令完成 |
| 0x1d | Handle Value Indication | 主动推送 | 0x1e | start_notify() 回调函数触发 | 有（推送的特征句柄） | 有（GPS/状态数据） | 相机主动发GPS、唤醒确认 |
| 0x1e | Handle Value Confirmation | 响应 | 0x1d | 无主动API，Bleak自动回复 | 无 | 无（仅确认收到） | 手机回执：已收到相机推送 |


###### start_notify()和包opcode对应关系
start_notify()调用，对应的Opcode是0x12，向2902写入的Value是0x01（订阅通知）或者0x02（订阅指示）或者0x00（取消订阅）。相机收到这个写
请求后，会在对应特征发生变化时，主动发送0x1d的Handle Value Indication，携带新的数据。手机收到这个推送后，执行回调函数，并且会自动回复
0x1e的 Handle Value Confirmation，告诉相机已经收到数据了。
即
opcode 0x12(2902): start_notify()调用
opcode 0x1d: 对方主动推送数据，触发start_notify()指定的回调函数

