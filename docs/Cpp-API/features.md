# Features


## Profiling Support in Runtime
---------------------------------------

When running inferences on AIC100, we might want to get deeper insights into the performance of the AIC100 stack to either triage low performance or for general monitoring.

Current AIC100 stack provides mechanism to get performance metrics of key milestones through the inference cycle.

Profiling data can be broadly classified into two components:

1. Host metrics
   An inference passes through various software layers on host before it make it to the network on device. Examining the performance on host we can identify if tweaking in network pre/post processing stages or host side multi-threading knobs is required.
2. Network/device metrics
   Network metrics provide plethora of information, starting from whole network execution time to detailed operator level performance. This information can be put to use to make the most out of existing network or to optimize network itself.
   Note: Network perf collection is not baked into the network by default. It needs to be enabled during network compilation itself. The amount of network perf details present depends on the parameters passed to AIC compiler.

### Profiling Report Types
-----------------------------

Using AIC100 software stacks, profiling information can be requested in following types (layouts):

#### Latency type

Latency type is a CSV style table of key of AIC100 stack. Contains both host and device side information.


#### Trace type

Trace type is a json formatted Chrome trace data. It can be viewed on any interface that consumes Chrome traces. Contains both host and device side information.


### Profiling Report Collection methods

The above information can be requested from AIC100 stack using two mechanisms. The core content is the same in both mechanisms, just the delivery mechanism changes. The two ways are as follows:

1. Num-iter based profiling
2. Duration based profiling

#### Num-iter based profiling

Alias: Legacy profiling

When user creates a profiling handle for num-iter based profiling, they need to specify:

1. The program to profile,
2. Number of samples to collect,
3. Profiling callback and,
4. Type of profiling output type i.e. latency or trace.

The fundamental idea is that during the creation of profiling handle, the user specifies the number of inferences that needs to be sampled. After profiling is started by the user, the profiling stops and calls user provided callback when:

a. Number of samples requested by user has been collected.

b. User explicitly stops profiling
   In this case, the number of samples collected might be less than that requested during profiling handle creation.

After the profiling is stopped, the user can again call start profiling using the same handle. The behavior of the infra will be as if the handle is being triggered for the first time.

Refer to section `ProfilingHandle`_ for HPP API interface.

Note in the APIs how at the creation of profiling handle, the user needs to be aware of the program needs to be profiled. Only 1 program can be profiled by a given profiling handle. If user wants to profile multiple programs, multiple profiling handles needs to be created, one for each program.

#### Duration based profiling

Alias: Stream profiling

When user creates a profiling handle of type duration based profiling or stream profiling, the user needs to specify:

1. Reporting rate,
2. Sampling rate,
3. Callback,
4. RegEx, and
5. Type of profiling output type i.e. latency or trace.

Notice how there is no condition to specify when profiling should automatically end, hence once the user calls start profiling, the samples are collected till user explicitly calls stop profiling.
Also, we do not specify which program we want to profile. We add and remove programs at runtime (even after profiling has started) using appropriate API. Hence, allowing more than one program to be profiled by same profiling handle.

##### Reporting Rate

The profiling callbacks are called at every reporting rate boundary, i.e. suppose the reporting rate set by user is 500ms, and profiling is started at 4seconds and 400ms time-point, the first callback will be called at 4 seconds and 500ms time-point (not the callback did not get called after 500ms but at 500ms boundary).
Next callback at 5 seconds time-point and then at 5 seconds 500ms time-point and so on.
The callback contains profiling data for inferences that took place between the last report and the current report.

##### Sampling rate

User may not be interested in performance of each and every inference, they may want to just get an over view of the performance and hence can choose to record data for every - second, fourth, eighth or sixteenth inference using the sampling rate knob.

##### RegEx

User may want to profiling all the programs running under the process matching a regular expression say "Resnet50*". If the user creates a profiling handle with a specific regular expression, any new program created, whose name passes the regEx filter, will automatically start getting profiled.
Once the program is released by the user, it automatically is also removed from the profiling handle's list of programs.

Note: Addition/removal of program can lead to a report generation getting delayed or preponed.
Note: RegEx engine used is ECMAScript.

Refer to section `StreamProfilingHandle`_ for HPP API interface.

## Device Partitioning Tool
---------------------------------------
Device partitioning is a virtualization technique for Qualcomm AIC 100 card that creates a Shadow device with discrete set of resources on the native
device. The Shadow Device is tied to a "Resource Group Reservation" made by the application and is presented with Device id(QID) greater than 100 [Eg: QID 100].
The Shadow Device is allowed to use only the pre-reserved set of resources tied to its reservation and is non-persistent in nature.

Either of the below two scenarios will destroy the device and release the resources associated with it.
  a. The device would be destroyed when the Resource Group Reservation is
     released by application either explicitly or when the application dies.
  b. A hard reset is performed on the device.

Below listed are the set of resources that are available for reservation under
"Resource Group"
  - Neural Signal Processors(NSP)
  - Multicast IDs (MCID)
  - Virtual Channels for DMA data
  - Device Semaphores
  - Device Memory (DDR, L2)

qaic-dev-partition, is an application tool to create the Shadow Device(s) with the specified set of device resources.

Since the device allocation is non-persistent in nature, below use cases are UNSUPPORTED as they may invalidate the QID
allocated by an already existing instance of application.
1. Multiple instances of qaic-dev-partition tool.
2. Hotplug a QAic device when there is a active qaic-dev-partition.

### Usage
----------------------------------------------------------------------------------------------
```bash
/opt/qti-aic/tools/qaic-dev-partition -h
```

### Create and Test the Shadow Device(s)
----------------------------------------------------------------------------------------------
#### 1. Create the Shadow device
   
   Launch the qaic-dev-partition utility as a daemon, either in foreground or background to keep
   the Shadow device alive.

   To request resource reservation using the QPC file on Device-id 2
      
      sudo qaic-dev-partition -q <path to qpc>/programqpc.bin -d 2 
      
   To request resource reservation using json config file:

      sudo qaic-dev-partition -p <path_to_json_config>

   Multiple reservations can be created through json configuration. The first resource set is allotted QID 100,
   and QID is incremented by 1 for every resource set listed thereafter.

#### 2. Verify the Derived device creation:
   Run qaic-util tool to list the newly generated device. Typically the
   Shadow devices have QID >= 100. Note that PCI fields are invalid with dummy values as it is an
   emulated device, and PCI Hardware attributes are not applicable.
   Example: `sudo qaic-util -q -d 101`

#### 3. Destroy the device.
   On killing the utility the Shadow device should disappear and one may verify that the resources are added
   back to the native device's resource pool.

Example configuration files are present in `/opt/qti-aic/test-data/json/qaic-dev-partition`

### Partition Device Configuration
-----------------------------

Alias names: Partition Device/ Shadow Device/Derived Device.

Documentation uses the term "Derived Device". The starting Device ID for Derived Device(s) is QID 100.
Multiple Shadow/Derived devices can be created on the same QAIC device.
Each shadow device would have unique QID and Dev link assigned as shown below.

Example:

```
Parent device:  QID 17
                Dev Link: /dev/qaic_aic100_15

Shadow devices for QID 17
 QID 112 :
           Dev Link : /dev/qaic_aic100_31
     Parent Dev Link: /dev/qaic_aic100_15

 QID 113 :
           Dev Link : /dev/qaic_aic100_33
     Parent Dev Link: /dev/qaic_aic100_15
```