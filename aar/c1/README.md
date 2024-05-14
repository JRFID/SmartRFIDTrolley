# 智能RFID推车 JRI-ST-C1 SDK使用说明

### 📖中文文档 | [📖English Document](README-EN.md)

本文是智能RFID推车 JRI-ST-C1（以下简称：推车）SDK的标准的集成指南文档，用以说明推车SDK的使用方法。默认读者已经熟悉Android Studio的基本使用方法，熟悉kotlin的基本语法，并且具有一定的Android编程基础。

耗材柜SDK包含两个`aar`包，分别是位于[aar/core](/aar/core/)中的`jri-manager-core-*.aar`与位于[aar/c1](/aar/c1/)中的`jri-manager-st-c1-*.aar`。开发时请同时导入两个最新版`aar`包到你的项目中。

## 更新记录

### `jri-manager-core-*.aar`更新记录

[点击查看详细更新记录](../core/CHANGE-LOG.md)

### `jri-manager-cb-c1-*.aar`更新记录

#### 1.0.4 📅`2024.03.04`

* 简化初始化逻辑
* 精简sdk开放的方法

[点击查看更多更新记录](CHANGE-LOG.md)

## 引入SDK

### 复制aar

1. 下载`core`文件夹中的最新版本`jri-manager-core-*.aar`文件。
2. 下载`c1`文件夹中的最新版本`jri-manager-st-c1-*.aar`文件。
3. 将上面下载的两个文件复制到你的项目中`libs`文件夹中。

### 添加依赖

```
dependencies {
    implementation files('libs/jri-manager-core-*-release.aar')
    implementation files('libs/jri-manager-st-c1-*-release.aar')
}
```

### 导入包

```
import com.jrfid.manager.trolley.c1.JRIDevicesManager
```

## 功能使用

### 初始化连接设备

初始化连接设备是一个耗时的超过，推荐使用协程来处理。

```
lifecycleScope.launch(Dispatchers.IO) {
    JRIDevicesManager.instance.initDevices()
}
```

### 设备控制

#### IC/ID模块

##### 监听IC/ID卡数据

```
JRIDevicesManager.instance.addOnReceivedICCardDataCallback(object : ReceivedICCardDataCallback {
    override fun onReceived(model: ICCardPacketData) {

            
        }

    })
```

`ICCardPacketData`类说明

```
class ICCardPacketData{

    val icCardNo: ByteArray

    val icCardNoText: String

}
```

| 属性          |     说明          |
| ------------ | ----------------- |
| icCardNo     | IC/ID的卡号字节数组 |
| icCardNoText | IC/ID的卡号字符串   |


#### 抽屉控制

##### 打开抽屉

推车从上往下总共有6层抽屉，分别对应序号1-6。

```
JRIDevicesManager.instance.openTheDoor(drawer:Int)
```

##### 抽屉状态监听

```
JRIDevicesManager.instance.addOnReceivedBasicDataCallback(object : ReceivedBasicDataCallback {

            override fun onDoorOpen(drawer:Int) {

            }


            override fun onDoorClose(drawer:Int) {

            }

        })
```

`ReceivedBasicDataCallback`回调方法说明

| 方法              | 说明       |
| ------------------- |----------|
| fun onDoorOpen(drawer:Int)  | 柜门打开时的回调 |
| fun onDoorClose(drawer:Int) | 柜门关闭时的回调 |

#### 超高频模块控制

##### 开始盘存

```
JRIDevicesManager.instance.startUhfInventory()
```

##### 获取盘存结果

```
JRIDevicesManager.instance.addOnReceivedUhfInventoryDataCallback(object:ReceivedUhfInventoryDataCallback{

            override fun onReceivedTag(tag: UHFTagInfoModel) {

                

            }


            override fun onInventoryEnd() {

                

            }

    })
```

`ReceivedUhfInventoryDataCallback`回调方法说明

| 方法                               | 说明                                         |
| --------------------------------- |--------------------------------------------|
| fun onReceivedTag(UHFTagInfoModel) | 盘存到标签回调，标签详细信息见下方`UHFTagInfoModel`类说明。 |
| fun onInventoryEnd()               | 本次盘存结束回调                                   |

`UHFTagInfoModel`类说明

```
class UHFTagInfoModel {

    val freq: Int

    val pc: ByteArray

    val epc: ByteArray

    val rssi: Int

    fun getFreqText(): String

    fun getEpcText(): String

}
```

| 属性/方法                      | 说明                                                             |
| --------------------------- | ------------------------------------------------------------------ |
| freq                      | 盘存到标签的工作频率参数，对应的频率见下方《工作频率参数对应表》 |
| pc                        | 标签的pc字节数据                                                 |
| epc                       | 标签epc字节数据                                                  |
| rssi                      | 盘存到标签的rssi值                                               |
| fun getFreqText(): String | 盘存到标签的工作频率字符串，包含单位：MHz。 |
| fun getEpcText(): String  | epc字符串 |

##### 工作频率参数对应表

| 频率参数 | 对应频点 | 频率参数 | 对应频点 |
| --------------- | --------------- | --------------- | --------------- |
| 0(0x00)       | 865.00 MHz    | 30(0x1E)      | 913.50 MHz    |
| 1(0x01)       | 865.50 MHz    | 31(0x1F)      | 914.00 MHz    |
| 2(0x02)       | 866.00 MHz    | 32(0x20)      | 914.50 MHz    |
| 3(0x03)       | 866.50 MHz    | 33(0x21)      | 915.00 MHz    |
| 4(0x04)       | 867.00 MHz    | 34(0x22)      | 915.50 MHz    |
| 5(0x05)       | 867.50 MHz    | 35(0x23)      | 916.00 MHz    |
| 6(0x06)       | 868.00 MHz    | 36(0x24)      | 916.50 MHz    |
| 7(0x07)       | 902.00 MHz    | 37(0x25)      | 917.00 MHz    |
| 8(0x08)       | 902.50 MHz    | 38(0x26)      | 917.50 MHz    |
| 9(0x09)       | 903.00 MHz    | 39(0x27)      | 918.00 MHz    |
| 10(0x0A)      | 903.50 MHz    | 40(0x28)      | 918.50 MHz    |
| 11(0x0B)      | 904.00 MHz    | 41(0x29)      | 919.00 MHz    |
| 12(0x0C)      | 904.50 MHz    | 42(0x2A)      | 919.50 MHz    |
| 13(0x0D)      | 905.00 MHz    | 43(0x2B)      | 920.00 MHz    |
| 14(0x0E)      | 905.50 MHz    | 44(0x2C)      | 920.50 MHz    |
| 15(0x0F)      | 906.00 MHz    | 45(0x2D)      | 921.00 MHz    |
| 16(0x10)      | 906.50 MHz    | 46(0x2E)      | 921.50 MHz    |
| 17(0x11)      | 907.00 MHz    | 47(0x2F)      | 922.00 MHz    |
| 18(0x12)      | 907.50 MHz    | 48(0x30)      | 922.50 MHz    |
| 19(0x13)      | 908.00 MHz    | 49(0x31)      | 923.00 MHz    |
| 20(0x14)      | 908.50 MHz    | 50(0x32)      | 923.50 MHz    |
| 21(0x15)      | 909.00 MHz    | 51(0x33)      | 924.00 MHz    |
| 22(0x16)      | 909.50 MHz    | 52(0x34)      | 924.50 MHz    |
| 23(0x17)      | 910.00 MHz    | 53(0x35)      | 925.00 MHz    |
| 24(0x18)      | 910.50 MHz    | 54(0x36)      | 925.50 MHz    |
| 25(0x19)      | 911.00 MHz    | 55(0x37)      | 926.00 MHz    |
| 26(0x1A)      | 911.50 MHz    | 56(0x38)      | 926.50 MHz    |
| 27(0x1B)      | 912.00 MHz    | 57(0x39)      | 927.00 MHz    |
| 28(0x1C)      | 912.50 MHz    | 58(0x3A)      | 927.50 MHz    |
| 29(0x1D)      | 913.00 MHz    | 59(0x3B)      | 928.00 MHz    |

#### 指静脉模块控制

##### 指静脉使用流程推荐

###### 录入指静脉

分3次获取用户指静脉特征值 -> 合成指静脉特征值模版 -> 保存特征值模版

###### 验证指静脉

导入特征值模版到算法库 -> 记录返回的ID并于用户绑定 -> 获取指静脉特征值 -> 验证特征值是否存在

##### 是否放置手指

调用后立即返回`Boolean`类型结果，`true`表示有手指放置到指静脉模块上，反之则没有。

```
val result:Boolean = JRIDevicesManager.instance.checkFingerIn()
```

##### 是否移开手指

调用后立即返回`Boolean`类型结果，`true`表示有没有手指放置到指静脉模块上，反之则有。

```
val result:Boolean = JRIDevicesManager.instance.checkFingerOut()
```

##### 等待手指放置

该操作为阻塞操作需放置到协程中调用，调用后会一直阻塞程序，检测到手指放置到指静脉模块上后会释放。

```
lifecycleScope.launch {
    JRIDevicesManager.instance.waitingFingerIn()
 }
```

##### 等待手指移开

该操作为阻塞操作需放置到协程中调用，调用后会一直阻塞程序，检测到手指移开后会释放。

```
lifecycleScope.launch {
    JRIDevicesManager.instance.waitingFingerOut()
}
```

##### 获取指静脉特征值

实时获取，只有当手指正确放置到指静脉模块上时调用后会立即返回`String`类型的指静脉特征值，其他情况下返回`null`。

```
val result:String? = JRIDevicesManager.instance.getFingerVeinChara()
```

阻塞获取，需放置到协程中调用，调用后会一直阻塞程序，直到获取到指静脉特征值后返回，如果设置的超时时间内未获取到指静脉特征值则返回`null`。

```
lifecycleScope.launch {
    val result: String? = JRIDevicesManager.instance.getFingerVeinChara(timeoutMillis = 1000)
}
```

##### 获取指静脉特征值模板

将分3次获取的指静脉特征值融合成一个特征值模版，融合成功返回`String`类型的特征值模版，融合失败则返回`null`。

```
val result: String? = JRIDevicesManager.instance.createFingerVeinTemp("", "", "")
```

##### 持续获取指静脉特征值:boom:

通过`kotlin`中`Flow`的形式持续获取指静脉特征值，可以根据返回的`MODEL_TYPE`反馈给用户做相应的操作。

```
lifecycleScope.launch {
    JRIDevicesManager.instance.getFingerVeinCharaFlow()
        .flowOn(Dispatchers.IO)
        .collect {
            if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_IN) {
                LogUtils.d("请放置手指")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_OUT) {
                LogUtils.d("请移开手指")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_CHARA_DATA) {
                LogUtils.d("指静脉特征值为：${it.data}")
            }
        }
}
```

##### 持续获取指静脉特征值模版:boom:

通过`kotlin`中`Flow`的形式持续获取指静脉特征值，可以根据返回的`MODEL_TYPE`反馈给用户做相应的操作。

```
lifecycleScope.launch {
    JRIDevicesManager.instance.getFingerVeinTempFlow()
        .flowOn(Dispatchers.IO)
        .collect {
            if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_IN) {
                 LogUtils.d("请放置手指")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_OUT) {
                 LogUtils.d("请移开手指")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_TEMP_DATA) {
                 LogUtils.d("指静脉特征值模版为：${it.data}")
            }
         }
}
```
