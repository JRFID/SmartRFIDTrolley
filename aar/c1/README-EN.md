# Smart RFID Trolley JRI-ST-C1 SDK Instructions

### [📖中文文档](README.md) | 📖English Document

This document is the standard integration guide document for the Smart RFID Trolley JRI-ST-C1 (hereinafter referred to as the trolley) SDK, 
used to explain the usage of the trolley SDK. The default readers are already familiar with the basic usage of Android Studio, familiar with the basic syntax of Kotlin, and have a certain foundation in Android programming.

The trolley SDK contains two `aar` packages, namely `jri-manager-core-*.aar` located in [aar/core](/aar/core/) and `jri-manager-st-c1-*.aar` located in [aar/c1](/aar/c1/). Please import both latest versions of the `aar` package into your project.

## Update 

#### 1.0.4 📅`2024.03.04`

* Simplify init logic

[Click to view more update records](CHANGE-LOG.md)

## Import SDK

### Copy File

1. Download the latest version of `jri-manager-core-*.aar` file from the `core` folder.
2. Download the latest version of `jri-manager-st-c1-*.aar` file from the `c1` folder.
3. Copy the two files to the `libs` folder in your project.

### Add Dependency

```
dependencies {
    implementation files('libs/jri-manager-core-*-release.aar')
    implementation files('libs/jri-manager-st-c1-*-release.aar')
}
```

### Import Package

```
import com.jrfid.manager.trolley.c1.JRIDevicesManager
```

## Function Introduction

### Init Connect Device

Init connect device is a long-running task,recommend using kotlin coroutines.

```
lifecycleScope.launch(Dispatchers.IO) {
    JRIDevicesManager.instance.initDevices()
}
```

### Device Control

#### IC/ID Module

##### Listening IC/ID card data

```
JRIDevicesManager.instance.addOnReceivedICCardDataCallback(object : ReceivedICCardDataCallback {
    override fun onReceived(model: ICCardPacketData) {

            
        }

    })
```

`ICCardPacketData` class description

```
class ICCardPacketData{

    val icCardNo: ByteArray

    val icCardNoText: String

}
```

| field        | description                  |
|--------------|------------------------------|
| icCardNo     | IC/ID card number byte array |
| icCardNoText | IC/ID card number string     |

#### Drawer Control

##### Open the drawer

The trolley has a total of 6 drawers from top to bottom, corresponding to numbers 1-6.

```
JRIDevicesManager.instance.openTheDoor(drawer:Int)
```

| param      | description        |
|------------|--------------------|
| drawer:Int | drawer number(1-6) |

##### Listening drawer state

```
JRIDevicesManager.instance.addOnReceivedBasicDataCallback(object : ReceivedBasicDataCallback {

            override fun onDoorOpen(drawer:Int) {

            }


            override fun onDoorClose(drawer:Int) {

            }

        })
```

`ReceivedBasicDataCallback` interface description

| method                      | description                    |
|-----------------------------|--------------------------------|
| fun onDoorOpen(drawer:Int)  | callback when drawer is opened |
| fun onDoorClose(drawer:Int) | callback when drawer is closed |

#### UHF Control

##### Start inventory

```
JRIDevicesManager.instance.startUhfInventory(drawer:Int)
```

| param      | description        |
|------------|--------------------|
| drawer:Int | drawer number(1-6) |

##### Listening inventory result

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

`ReceivedUhfInventoryDataCallback` interface description

| method                                            | description                   |
|---------------------------------------------------|-------------------------------|
| fun onReceivedTag(tag: InventoryTag)              | callback when inventory tag   |
| fun onInventoryEnd()                              | callback when inventory end.  |
| fun onInventoryFailure(failure: InventoryFailure) | callback when inventory fail. |

`InventoryTag` class description

```
class InventoryTag {

    val pc: String

    val epc: String

    val rssi: Int
    
}
```

| field | description    |
|-------|----------------|
| pc    | tag pc data    |
| epc   | tag epc data   |
| rssi  | tag rssi value |

#### Finger Vein Control

##### Recommended usage process

###### Get finger vein

get finger vein characteristic value in three stages -> composite finger vein template -> save finger vein template

###### Check finger vein

import finger vein template into the search library -> save the returned id and bind it to the user -> get finger vein characteristic value -> check if the finger vein characteristic value exists in the search library

##### Whether to place finger

After calling this method, it will return a result of type `Boolean`,`true` is finger placed on the module,otherwise `false`.

```
val result:Boolean = JRIDevicesManager.instance.checkFingerIn()
```

##### Whether to remove finger

After calling this method, it will return a result of type `Boolean`,`true` is finger removed from the module,otherwise `false`.

```
val result:Boolean = JRIDevicesManager.instance.checkFingerOut()
```

##### Waiting for finger placement

This operation is a blocking operation that needs to be placed in the coroutine for calling. After calling, the program will be blocked all the time. When it is detected that the finger is placed on the finger vein module, it will be released.

```
lifecycleScope.launch {
    JRIDevicesManager.instance.waitingFingerIn()
 }
```

##### Waiting for finger removed

This operation is a blocking operation that needs to be placed in the coroutine for calling. After calling, the program will be blocked all the time. When it is detected that the finger is removed from the finger vein module, it will be released.

```
lifecycleScope.launch {
    JRIDevicesManager.instance.waitingFingerOut()
}
```

##### Get finger vein characteristic value

Real time acquisition, only when the finger is correctly placed on the finger vein module and called, will immediately return the `String` type finger vein characteristic value, and in other cases, return `null`.

```
val result:String? = JRIDevicesManager.instance.getFingerVeinChara()
```

Blocking acquisition,this needs to be placed in the coroutine for calling. After calling, the program will be blocked until the finger vein characteristic value is obtained. If the finger vein characteristic value is not obtained within the set timeout, it will return `null`.
```
lifecycleScope.launch {
    val result: String? = JRIDevicesManager.instance.getFingerVeinChara(timeoutMillis = 1000)
}
```

##### Get finger vein template

Merge the finger vein characteristic value obtained in three stages into a finger vein template. Success return true, otherwise return `null`.

```
val result: String? = JRIDevicesManager.instance.createFingerVeinTemp("", "", "")
```

##### Continuously get finger vein characteristic value:boom:

By continuously obtaining the finger vein characteristic values in the form of `flow` in `kotlin`, users can take corresponding actions based on the returned `MODEL_TYPE` feedback.

```
lifecycleScope.launch {
    JRIDevicesManager.instance.getFingerVeinCharaFlow()
        .flowOn(Dispatchers.IO)
        .collect {
            if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_IN) {
                Log.d("Please place finger")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_OUT) {
                Log.d("Please remove finger")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_CHARA_DATA) {
                Log.d("finger vein characteristic value：${it.data}")
            }
        }
}
```

##### Continuously get finger vein template:boom:

By continuously obtaining the finger vein template in the form of `flow` in `kotlin`, users can take corresponding actions based on the returned `MODEL_TYPE` feedback.

```
lifecycleScope.launch {
    JRIDevicesManager.instance.getFingerVeinTempFlow()
        .flowOn(Dispatchers.IO)
        .collect {
            if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_IN) {
                 Log.d("Please place finger")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_FINGER_OUT) {
                 Log.d("Please remove finger")
            } else if (it.type == FingerVeinDataModel.MODEL_TYPE_TEMP_DATA) {
                 Log.d("finger vein template：${it.data}")
            }
         }
}
```

##### Add finger vein template into search library

```
val id:Long = JRIDevicesManager.instance.addFingerVeinTempIntoLib(temp:String)
```

| param         | description                           |
|---------------|---------------------------------------|
| temp:String   | create/get finger vein template value |

| return      | description                                                                                                                                        |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| id:Long     | id>0 added successfully,and its value is the id value of the finger vein template in the search library.id<0 added failed,its value is error code. |


##### Remove finger vein template from search library

```
JRIDevicesManager.instance.removeFingerVeinTempFromLib(id:Long)
```
| param          | description                                             |
|----------------|---------------------------------------------------------|
| id:Long        | The id value returned when adding to the search library |

##### Remove all finger vein template from search library

```
JRIDevicesManager.instance.removeAllFingerVeinTempFromLib()
```

##### Check if finger vein characteristic value exist in the search library

```
val id:Long = JRIDevicesManager.instance.checkCharaIsExist(chara:String)
```
| param         | description                           |
|---------------|---------------------------------------|
| chara:String  | got finger vein characteristic value  |

| return      | description                                                                                                               |
|-------------|---------------------------------------------------------------------------------------------------------------------------|
| id:Long     | id>0 exist, the id value is the value returned by adding the finger vein template to the search library, id<=0 otherwise. |
