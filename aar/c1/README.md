# 智能RFID推车 JRI-ST-C1 SDK使用说明

### 📖中文文档 | [📖English Document](README-EN.md)

本文是智能RFID推车 JRI-ST-C1（以下简称：推车）SDK的标准的集成指南文档，用以说明推车SDK的使用方法。默认读者已经熟悉Android Studio的基本使用方法，熟悉kotlin的基本语法，并且具有一定的Android编程基础。

耗材柜SDK包含两个`aar`包，分别是位于[aar/core](/aar/core/)中的`jri-manager-core-*.aar`与位于[aar/c1](/aar/c1/)中的`jri-manager-st-c1-*.aar`。开发时请同时导入两个最新版`aar`包到你的项目中。

## 更新记录

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

| 参数                          | 说明       |
|-----------------------------|----------|
| drawer:Int | 需盘存的抽屉序号 |

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
| fun onDoorOpen(drawer:Int)  | 抽屉打开时的回调 |
| fun onDoorClose(drawer:Int) | 抽屉关闭时的回调 |

#### 超高频模块控制

##### 开始盘存

```
JRIDevicesManager.instance.startUhfInventory(drawer:Int)
```

| 参数                          | 说明       |
|-----------------------------|----------|
| drawer:Int | 需盘存的抽屉序号 |

##### 获取盘存结果

```
JRIDevicesManager.instance.addOnReceivedUhfInventoryDataCallback(object:ReceivedUhfInventoryDataCallback{

            override fun onReceivedTag(tag: InventoryTag) {
                
            }

            override fun onInventoryTagEnd() {

            }

            override fun onInventoryFailure(inventoryFailure: InventoryFailure) {
                
            }

    })
```

`ReceivedUhfInventoryDataCallback`回调方法说明

| 方法                                                | 说明                                  |
|---------------------------------------------------|-------------------------------------|
| fun onReceivedTag(tag: InventoryTag)              | 盘存到标签回调，标签详细信息见下方`InventoryTag`类说明。 |
| fun onInventoryEnd()                              | 本次盘存结束回调                            |
| fun onInventoryFailure(failure: InventoryFailure) | 本次盘存错误回调                            |

`InventoryTag`类说明

```
class InventoryTag {

    val pc: String

    val epc: String

    val rssi: Int
    
}
```

| 属性/方法                      | 说明                                                           |
| --------------------------- | ---------------------------------------------------------------- |
| pc                        | 标签的pc数据                                                 |
| epc                       | 标签epc数据                                                  |
| rssi                      | 盘存到标签的rssi值                                              |

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
