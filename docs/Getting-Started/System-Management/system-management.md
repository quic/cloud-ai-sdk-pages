# System Management
`qaic-util` command line utility enables developers to query 

- card and SoC(s) health
- firmware version
- compute/memory/IO resources available vs in-use
- card power and temperature 
- status of certain device capabilities like ECC etc

Cloud AI Platform SDK installation is required for `qaic-util` usage. 

`QID` a.k.a `deviceID` are indentifiers (integers) assigned to each AI 100 SoC present in the system. Note that certain SKUs may contain more than one AI 100 SoC per Card. 

`qaic-util` displays information in 2 formats:

- vertical format where cards/SoCs are queried once and the parameters are listed one per line. 
```
sudo /opt/qti-aic/tools/qaic-util -q 
sudo /opt/qti-aic/tools/qaic-util -q  -d <QID#> #To display information for a specific `QID`
```
- tabular format where certain parameters (compute, IO, power, temperature etc) are listed in a tabular format, refreshed every 'n' seconds (user input)
```
sudo /opt/qti-aic/tools/qaic-util -q -t 1 
```

`-d` flag can be used to display information for a specific `QID`

`qaic-util`, provides --filter(-f) option along with -q and -t options, which can filter by certain device properties. Also to dump output to the .json file by using -j option.

Examples: <br>
To display information for a specific `Card`.
```
sudo /opt/qti-aic/tools/qaic-util -q -f "Board serial==<BOARD_SERIAL_OF_CARD>"
```
To display information for a specific `Card` in tabular format.
```
sudo /opt/qti-aic/tools/qaic-util -t 1 -f "Board serial==<BOARD_SERIAL_OF_CARD>"
```
To dump output from the qaic-util, option -j can be used,
```
sudo /opt/qti-aic/tools/qaic-util -j <output-file-name>.json -f "Board serial==<BOARD_SERIAL_OF_CARD>"
```

Developers can `grep` for keywords like `Status`, `Capabilities`, `Nsp`, `temperature`, `power` to get specific information from the cards/SoCs.

## Health
`Status` field indicates the health of the card. 

- `Ready` indicates card is in good health.
- `Error` indicates card is in error condition or user lacks permissions (use `sudo`). 

```
sudo /opt/qti-aic/tools/qaic-util -q | grep -e Status -e QID
QID 0
        Status:Ready
QID 1
        Status:Ready
QID 2
        Status:Ready
QID 3
        Status:Ready
```

[**Verify the function**](../Installation/Checklist/checklist.md#verify-card-healthfunction) steps can be used to run a sample workload on `QIDs` to ensure HW/SW is funtioning correctly. 


## SoC Reset 
Developers can reset the `QIDs` using `soc_reset` sysfs node to recover the SoCs if they are in `Error` condition. These are the steps to issue a `soc_reset`. 

1. Identify the `MHI ID` associated with the `QID`

    ```
    sudo /opt/qti-aic/tools/qaic-util -q | grep -e MHI -e QID
    ```
    In the sample output below, `MHI ID:0` is associated with `QID 0` and so on. 

    ???+ note
        MHI and QID do **not** always map to the same integer. It is imperative for developers to identify the mapping first before issuing the `soc_reset`
    ```
    sudo /opt/qti-aic/tools/qaic-util -q | grep -e MHI -e QID
    QID 0
            MHI ID:0
    QID 1
            MHI ID:1
    QID 2
            MHI ID:2
    QID 3
            MHI ID:3

    ```

2. Issue `soc_reset` to the `MHI ID` identified in step 1. 

    ```
    sudo su 
    echo 1 > /sys/bus/mhi/devices/mhi<MHI ID>/soc_reset  #MHI ID is 0,1,2...  
    ```

    [Verify the health/function](../Installation/Checklist/checklist.md#verify-card-healthfunction) of the SoCs/Cards after a `soc_reset`. 



## Advanced System Management 
For advanced system management details, refer to [Cloud AI Card Management](https://docs.qualcomm.com/bundle/resource/topics/80-PT790-995B) 

This document is shared with System Integrators and covers the following topics. 

- Boot and firmware management
- Security - Secure boot enablement and attestation 
- BMC integration
- Platform validation tools 
- Platform error management 

## Python APIs
Python APIs also provide the abilty to monitor the health and resources of the cards/SoCs. Refer to [Util class](../../Python-API/qaicrt/class_util.md)
