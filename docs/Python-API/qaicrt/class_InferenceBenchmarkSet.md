# class InferenceBenchmarkSet
------ 

Help on class InferenceBenchmarkSet in module `qaicrt`:

## **InferenceBenchmarkSet(Logger)**
------ 

```python
InferenceBenchmarkSet(context, qpc, devID, setSize, numActivations, inferenceDataVector, properties) -> infrenceBenchmarkSet
```
Creates and returns a new object for running benchmarks.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `context`         | A previously created context.|
| `qpc`             | A previously created Qpc Object.|
| `devID`           | [==optional==] Device on which benchmark is to be run. By default device will be auto-picked|
| `setSize`         | Number of ExecObj per program.|
| `numActivations`  | Number of separate activations. Set this value to 1 to have a single instance of the program on device Set this value to a specific number of activations.|
|`inferenceDataVector`| A vector of QBuffers for initial input and output data. This is the first set of data to be use for the first submission during benchmark. If new data is needed for each inference, the user can register a callback which will return the ExecObj pointer, which can be used to  setData for the next inference. Alternatively, the same data can be used repeatedly|
|`properties`         | [==optional==] Properties for the program and queues to created by the benchmark set. One of the key properties is number of threads per queue which can be changed in the queue properties|

Once the Benchmark object is created, the specified program will be initialized. The
inferenceDataVector must be compatible with the program. Depending on the `setSize` and `numActivations`
configured, `ExecObj(s)` :warning: will be created per program. `numInferencesToRun` will be completed each time
the start signal is given through `run()` method. 

Methods defined here:

### __init__
------ 

```python
__init__(self: qaicrt.InferenceBenchmarkSet, context: qaicrt.Context, qpc: qaicrt.Qpc, dev: Optional[int] = None, setSize: int, numActivations: int, inferenceDataVector: List[qaicrt.QBuffer], properties: qaicrt.BenchmarkProperties = None) -> None
```

### disable
------ 

```python
disable(self: qaicrt.InferenceBenchmarkSet) -> qaicrt.QStatus
```    
**Description**

Disable InferenceBenchmarkSet. 
This will deactivate and release all resources associated with this benchmark.

**Returns**

- Status:
    - qaicrt.QStatus.QS_SUCCESS Successful completion
    - qaicrt.QStatus.QS_ERROR Failed to release benchmark object programs and resources

### disableExecObjCompletionCallback
------ 

```python
disableExecObjCompletionCallback(self: qaicrt.InferenceBenchmarkSet) -> None
```

**Description**: 

Disable ExecObj Completion Callback

### disableOpStatsCallback
------ 

```python
disableOpStatsCallback(self: qaicrt.InferenceBenchmarkSet) -> qaicrt.QStatus
```

**Returns**:

- `qaicrt.QStatus.QS_SUCCESS` Successful completion

### enable

```python
enable(self: qaicrt.InferenceBenchmarkSet) -> qaicrt.QStatus
```

**Description**: 

Enable InferenceBenchmarkSet. This will trigger the load and activation of all the program instances on device.  This may fail if resources are not available.

**Returns**: 

Operational status

- Operational status:
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_ERROR` Failed to setup benchmark object programs and resources

###  enableExecObjCompletionCallback
------ 

```python
enableExecObjCompletionCallback(self: qaicrt.InferenceBenchmarkSet, callback: Callable[[qaicrt.ExecObj, qaicrt.ExecObjInfo], None]) -> None
```

**Description**

Enable ExecObj Completion Callback. When enabled, each time an ExecObj completes, this callback will be called with the ExecObj that completed. This can be used to perform data validation or to extract the inference results.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `callback`       | Callback function |

**Returns**

Operational status.

- Operational Status: 
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion

### enableOpStatsCallback
------ 

```python
enableOpStatsCallback(self: qaicrt.InferenceBenchmarkSet, callback: Callable[[qaicrt.vector_char, int, qaicrt.QAicOpStatsFormatEnum, int, qaicrt.ExecObjInfo], None], initialSampleIndex: int, numSamplesPerProgramInstance: int, format: qaicrt.QAicOpStatsFormatEnum) -> qaicrt.QStatus
```

**Description**

Enable Operational Statistics Callback. OpStats are data collected by the AIC neural network core during execution.

**Parameters**  

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `callback`       | Callback function |
| `initialSampleIndex`       | The index from 0 to the first ExecObj completion the callback should be called. For example, if set to 3, starting at the 3rd ExecObj completion the callback will be called function |
| `format`       | The format of the data to be collected. This may be ASCII or JSON |

**Returns**

Operational status.

- Operational Status: 
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_INVAL` Invalid parameter format

### getInfCompleted
------ 

```python
getInfCompleted(self: qaicrt.InferenceBenchmarkSet) -> List[int]
```

**Description**

Get the number of inferences completed.

**Returns**

The number of executions completed, not accounting for the batchSize, and for all activations.

### getLastRunStats
------ 

```python
getLastRunStats(self: qaicrt.InferenceBenchmarkSet) -> Tuple[int, float, int, int]
```

**Description**

Get stats on the last run of the InferenceBenchmarkSet

**Returns**

- Tuple of `infCompleted`, `infRate`, `runtimeUs` and `batchSize`.
    - `infCompleted`   : Number of inferences completed for all activations.
    - `infRate`        : Inference Rate in inferences per second for all activations.
    - `runtimeUs`      : Duration in microseconds of the last run.
    - `batchSize`      : Number of individual inferences completed in one execution of ExecObj.
                       When the program is compiled it may be configured to have a batchsize
                       of 1 or larger. The effective inference rate is the infRate multiplied
                       by the batchSize.

### run
------ 

```python
run(self: qaicrt.InferenceBenchmarkSet, numInferencesToRun: int) -> qaicrt.QStatus
```

**Description**

Run the benchmark. This call will block until the numInferencesToRun are completed.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `numInferencesToRun`       | Number of inferences to run |

**Returns**

- Operational Status.
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_INVAL` Invalid param `numInferencesToRun`
    - `qaicrt.QStatus.QS_ERROR` Inference run failed


### runForDuration
------ 

```python
runForDuration(self: qaicrt.InferenceBenchmarkSet, duration: int) -> qaicrt.QStatus
```

**Description**

Run the benchmark. This call will block until all the submitted inferences are completed. 

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `duration`       | Duration for which to keep submitting new inference |

**Returns**

- Operational Status.
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_INVAL` Invalid param duration
    - `qaicrt.QStatus.QS_ERROR` Inference run failed

### setLiveProfilingLevel
------ 

```python
setLiveProfilingLevel(self: qaicrt.InferenceBenchmarkSet, level: qaicrt.LiveReportingLevelEnum) -> qaicrt.QStatus
```

**Description**

Set the reporting level once liveReporting is enabled. Will retain the same period of reporting, but will use the new reporting level specified.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `level`       | New reporting level. :warning: levels not defined |

**Returns**

- Operational Status.
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_ERROR` Live reporting is not enabled, level change fails


### setLiveReporting
------ 

```python
setLiveReporting(self: qaicrt.InferenceBenchmarkSet, level: qaicrt.LiveReportingLevelEnum = <LiveReportingLevelEnum.LIVE_REPORTING_SUMMARY: 1>, reportingPeriodMs: int = 1000) -> qaicrt.QStatus
```

**Description**

Enable live reporting. Live reporting will print to sys.stdout a summary of the  inferences completed at the given period. The data printed may be detailed or summarized, depending on the level selected.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| `level`       | Level of detail requested for periodic live reporting. :warning: levels not defined |
| `reportingPeriodMs`       | The period at which the report will be generated, in milliseconds. |

**Returns**

- Operational Status.
    - `qaicrt.QStatus.QS_SUCCESS` Successful completion
    - `qaicrt.QStatus.QS_ERROR` Invalid param level


## Static methods 
------ 

```python
initDefaultProperties from builtins.PyCapsule
initDefaultProperties(properties: qaicrt.BenchmarkProperties) -> None
```

Initialize properties for InferenceBenchmarkSet to default.

**Parameters**

| Parameter      | Description                          |
| ----------- | ------------------------------------ |
| properties  | Properties to initialize.|
