# class Util

Help on class Util in module `qaicrt`:

Utility class to get information on the AIC device. Supports basic operations on device such as getting device ID, device information, resource information, and so on.

## **Util()**

`Util() -> util`

Creates and returns a new util object.

Methods defined here:

### __init__

```python
__init__(self: qaicrt.Util) -> None
```
### checkLibraryVersion

```python
checkLibraryVersion(self: qaicrt.Util) -> qaicrt.QStatus
```

**Description**: 

Checks Library Version with AicApi header.

- Major library version should be equal to *LRT_LIB_MAJOR_VERSION*.
- Minor library version should be less than *LRT_LIB_MINOR_VERSION*.

**Returns**:

- ``qaicrt.QStatus.QS_SUCCESS`` Successful completion
- `qaicrt.QStatus.QS_ERROR` Driver init failed

### getAicVersion

```python
getAicVersion(self: qaicrt.Util) -> Tuple[qaicrt.QStatus, int, int, str, str]
```

**Description**: 

Gets Aic version of the library.

**Returns**: 

Tuple of operational status, major, minor, patch, variant

- Operational status:
    - ``qaicrt.QStatus.QS_SUCCESS`` Successful completion
    - `qaicrt.QStatus.QS_INVAL` Internal error in getting aic version
- major: Major version of the library
- minor: Minor version of the library
- patch: Patch being used by the library
- variant: Variant of the library being used

###  getDeviceIds

```python
getDeviceIds(self: qaicrt.Util, deviceType: qaicrt.QAicDeviceType = <QAicDeviceType.QAIC_DEVICE_TYPE_DEFAULT: 1>) -> Tuple[qaicrt.QStatus, qaicrt.QIDList]
```

**Description**

Get the list of AIC devices. Optional argument deviceType specifies the type of the device.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `deviceType`       | [Optional] Type of device |

**Returns**

Tuple of operational status and the list of AIC devices.

- Operational Status: 
    - ``qaicrt.QStatus.QS_SUCCESS`` Successful completion
    - `qaicrt.QStatus.QS_INVAL` Internal error in getting device list
    - `qaicrt.QStatus.QS_NODEV` No valid device
    - `qaicrt.QStatus.QS_ERROR` Driver init failed or Bad device AddrInfo

### getDeviceInfo
```python
getDeviceInfo(self: qaicrt.Util, devID: int) -> Tuple[qaicrt.QStatus, qaicrt.QDevInfo]
```
**Description**

Get device information for the device specified. 

**Parameters**  

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `devID`       | A valid device ID returned from getDeviceIds() |


**Returns**

Tuple of operational status and Device info.

- Operational Status: 
    - ``qaicrt.QStatus.QS_SUCCESS`` Successful completion
    - `qaicrt.QStatus.QS_INVAL` Internal error in getting device info
    - `qaicrt.QStatus.QS_NODEV` No valid device
    - `qaicrt.QStatus.QS_ERROR` Driver init failed
    - `qaicrt.QStatus.QS_AGAIN` Error on reload Device Id

### getPerformanceInfo
```python
getPerformanceInfo(self: qaicrt.Util, devID: int) -> Tuple[qaicrt.QStatus, qaicrt.QPerformanceInfo]
```
**Description**

Get device performance information, this is a simple query that returns performance info. Note the same information is availablethrough GetDeviceInfo, however this is a more lightweight ap
allowing to retrieve only the performance info.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `devID`       | A valid device ID returned from getDeviceIds() |

**Returns**

Tuple of operational status and Performance info.

- Operational Status: 
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_INVAL` Internal error in getting performance info
    - `qaicrt.QStatus.QS_NODEV` Device not found, the device ID provided is not available
    - `qaicrt.QStatus.QS_ERROR` Driver init failed
    - `qaicrt.QStatus.QS_DEV_ERROR` Device validity error

### getResourceInfo
```python
getResourceInfo(self: qaicrt.Util, devID: int) -> Tuple[qaicrt.QStatus, qaicrt.QResourceInfo]
```

**Description**

Get device performance information. This is a simple query that returns status of
dynamic resources such as free memory, etc..Note that the same information is available
through GetDeviceInfo,however this is a more lightweight api allowing to retrieve only the
resourceinfo.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `devID`       | A valid device ID returned from getDeviceIds() |

**Returns**

Tuple of operational status  and Resource info.

- Operational Status: 
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_INVAL` Internal error in getting resource info
    - `qaicrt.QStatus.QS_NODEV` Device not found, the device ID provided is not available
    - `qaicrt.QStatus.QS_ERROR` Driver init failed
    - `qaicrt.QStatus.QS_DEV_ERROR` Device validity error

### lockDevice
```python
lockDevice(self: qaicrt.Util, devID: int, lock: bool, block: bool) -> qaicrt.QStatus
```

**Description**

Lock a device for exclusive access. This is an advisory lock, so if an uncooperative
app accesses a locked device the operations are not blocked. This functionality is
enabled by setting env var __QAIC_SERIALIZE_DEVICE__ to 1.If the env var is unset,
the call returns QS_UNSUPPORTED.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `devID`       | A valid device ID returned from getDeviceIds() |
| `lock`       | locks or unlocks the device |
| `block`       | Blocking or non-blocking. No-op if lock is false. |

**Returns**

Operational Status.

- Operational Status
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_NODEV` Device not found
    - `qaicrt.QStatus.QS_ERROR` Driver init failed
    - `qaicrt.QStatus.QS_UNSUPPORTED` Env var for advisory lock not set
    - `qaicrt.QStatus.QS_BUSY` Device locked