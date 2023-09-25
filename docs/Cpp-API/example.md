# InferenceSet IO Example
------ 

The following document describes `AIC100` Example named `InferenceSetIOBuffersExample.cpp`.

This example contains a single C++ file and a `CMakeLists.txt` that can be used for compiling as part of `Qualcomm Cloud AI 100` distributed Platform SDK.


??? example "InferenceSetIOBuffersExample.cpp"
    ``` cpp linenums="1" title="InferenceSetIOBuffersExample.cpp"
    //-----------------------------------------------------------------------------
    // Copyright (c) 2023 Qualcomm Innovation Center, Inc. All rights reserved.
    // SPDX-License-Identifier: BSD-3-Clause-Clear 
    //-----------------------------------------------------------------------------

    #include <string>
    #include <vector>
    #include <iostream>
    #include <random>
    #include "QAicApi.hpp"

    namespace {

    /**
    * used to generate radnom data into input buffers
    * Input buffers are uint8_t arrays
    */
    struct RandomGen final {
        static constexpr const int from = std::numeric_limits<uint8_t>::min();
        static constexpr int to = std::numeric_limits<uint8_t>::max();
        std::random_device randdev;
        std::mt19937 gen;
        std::uniform_int_distribution<uint8_t> distr;
        explicit RandomGen() : gen(randdev()), distr(from, to) {}
        [[nodiscard]] auto next() { return distr(gen); }
    };

    /**
    * Simple helper to return true if the buffer mapping instance is an input one
    * @param bufmap buffer mapping instance
    * @return true if the instance is an input buffer one.
    */
    [[nodiscard]] bool isInputBuffer(const qaic::rt::BufferMapping &bufmap) {
        return bufmap.ioType == BUFFER_IO_TYPE_INPUT;
    }

    /**
    * Helper function to print input/output buffer counts so far, zero based
    * @param bufmap buffer map instance
    * @param inputCount input count to use if the instance is input
    * @param outputCount output count to use if the instance is output
    * @return string formatted using the above info.
    */
    [[nodiscard]] std::string getPrintName(const qaic::rt::BufferMapping &bufmap,
                                        const std::size_t inputCount,
                                        const std::size_t outputCount) {
        using namespace std::string_literals;
        return isInputBuffer(bufmap) ? ("Input "s + std::to_string(inputCount))
                                    : ("Output "s + std::to_string(outputCount));
    }

    /**
    * Populate input, output vectors with QBuffer information
    * @param bufmap Buffer mapping instance
    * @param buf Actual QBufffer that was generated at callsite/caller.
    * @param inputBuffers Vector to use in case this is input instance
    * @param outputBuffers Vector to use in case this is an output instance
    */
    void populateVector(const qaic::rt::BufferMapping &bufmap, const QBuffer &buf,
                        std::vector<QBuffer> &inputBuffers,
                        std::vector<QBuffer> &outputBuffers) {
        if (isInputBuffer(bufmap)) {
            inputBuffers.push_back(buf);
        } else {
            outputBuffers.push_back(buf);
        }
    }

    /**
    * Given a buffer and size, populate it with random [0..128] random data
    * @param buf buffer to populate
    * @param sz size of this buffer
    */
    void populateBufWithRandom(uint8_t *buf, const std::size_t sz) {
    RandomGen gen;
        for (auto iter = buf; iter < buf + sz; ++iter) {
            *iter = gen.next();
        }
    }

    /**
    * Prepare buffers, vectors given a single buffer mapping. Depending on the
    * input/output instance of the buffer mapping, handle logic accordingly.
    * Only inputbuffers needs to be populated with random data.
    * @param bufmap buffer mapping passed as const
    * @param inputCount input buffers counter
    * @param outputCount output buffers counter
    * @param inputBuffers input vector of QBuffer to append to new QBuffer
    * @param outputBuffers output vector of QBuffer to append to new QBuffer
    */
    void prepareBuffers(const qaic::rt::BufferMapping &bufmap,
                        std::size_t &inputCount, std::size_t &outputCount,
                        std::vector<QBuffer> &inputBuffers,
                        std::vector<QBuffer> &outputBuffers) {
        std::cout << getPrintName(bufmap, inputCount, outputCount) << '\n';
        std::cout << "\tname = " << bufmap.bufferName << '\n';
        std::cout << "\tsize = " << bufmap.size << '\n';
        QBuffer buf{bufmap.size, new uint8_t[bufmap.size]}; // Need to dealloc
        populateVector(bufmap, buf, inputBuffers, outputBuffers);
        //
        // Provide the input to the inference in "inputBuffers". Here random data
        // is used. For providing input as file, use a different api. This example
        // is for input in memory.
        //
        if (isInputBuffer(bufmap)) {
            populateBufWithRandom(buf.buf, buf.size);
            ++inputCount;
        } else {
            ++outputCount;
        }
    }

    /**
    * Given input and output buffers, release all heap allocated
    * @param inputBuffers vector of QBuffers - inputs
    * @param outputBuffers vector of Qbuffers - outputs
    */
    void releaseBuffers(std::vector<QBuffer> &inputBuffers,
                        std::vector<QBuffer> &outputBuffers) {
        const auto release([](const QBuffer &qbuf) { delete[] qbuf.buf; });
        std::for_each(inputBuffers.begin(), inputBuffers.end(), release);
        std::for_each(outputBuffers.begin(), outputBuffers.end(), release);
    }

    /**
    * Given buffer mapping instance, return true if this instance does not
    * contain input or output buffers (e.g. it contains uninitialized or invalid)
    * @param bufmap buffer mapping instance
    * @return true if the buffer mapping instance does not container a valid buffer
    */
    [[nodiscard]] bool notInputOrOutput(const qaic::rt::BufferMapping &bufmap) {
        const std::initializer_list<QAicBufferIoTypeEnum> bufTypes{
            BUFFER_IO_TYPE_INPUT, BUFFER_IO_TYPE_OUTPUT};
        const auto func([type = bufmap.ioType](const auto v) { return v == type; });
        return std::none_of(bufTypes.begin(), bufTypes.end(), func);
    }

    } // namespace

    int main([[maybe_unused]] int argc, [[maybe_unused]] char *argv[]) {
        QID qid = 0;
        std::vector<QID> qidList{qid};

        // *** QPC ***
        constexpr const char *qpcPath =
            "/opt/qti-aic/test-data/aic100/v2/2nsp/2nsp-conv-hmx"; // (1)
        auto qpc = qaic::rt::Qpc::Factory(qpcPath);

        // *** CONTEXT ***
        constexpr QAicContextProperties_t *NullProp = nullptr;
        auto context = qaic::rt::Context::Factory(NullProp, qidList);

        // *** INFERENCE SET ***
        constexpr uint32_t setSize = 10;
        constexpr uint32_t numActivations = 1;
        auto inferenceSet = qaic::rt::InferenceSet::Factory(
            context, qpc, qidList.at(0), setSize, numActivations);

        // *** SETUP IO BUFFERS ***
        qaic::rt::shInferenceHandle submitHandle;
        auto status = inferenceSet->getAvailable(submitHandle);
        if (status != QS_SUCCESS) {
            std::cerr << "Error obtaining Inference Handle\n";
            return -1;
        }
        std::size_t numInputBuffers = 0;
        std::size_t numOutputBuffers = 0;
        std::vector<QBuffer> inputBuffers, outputBuffers;
        const auto &bufferMappings = qpc->getBufferMappings();
        for (const auto &bufmap : bufferMappings) {
            if (notInputOrOutput(bufmap)) {
                continue;
            }
            prepareBuffers(bufmap, numInputBuffers, numOutputBuffers, inputBuffers,
                        outputBuffers);
        }
        submitHandle->setInputBuffers(inputBuffers);
        submitHandle->setOutputBuffers(outputBuffers);

        // *** SUBMIT ***
        constexpr uint32_t inferenceId = 0; // also named as request ID
        status = inferenceSet->submit(submitHandle, inferenceId);
        std::cout << status << '\n';

        // *** COMPLETION ***
        qaic::rt::shInferenceHandle completedHandle;
        status = inferenceSet->getCompletedId(completedHandle, inferenceId);
        std::cout << status << '\n';
        status = inferenceSet->putCompleted(std::move(completedHandle));
        std::cout << status << '\n';

        // *** GET OUTPUT ***
        //
        // At this point, the output is available in "outputBuffers" and can be
        // consumed.
        //

        // *** Release user allocated buffers ***
        releaseBuffers(inputBuffers, outputBuffers);
    }

    ```

    1.  :warning: Replace the path with model QPC file of interest.



??? example "CMakeLists.txt"
    ```cmake linenums="1" title="CMakeLists.txt"
    # ==============================================================================
    # Copyright (c) 2023 Qualcomm Innovation Center, Inc. All rights reserved.
    # SPDX-License-Identifier: BSD-3-Clause-Clear 
    # ==============================================================================

    project(inference-set-io-buffers-example)
    cmake_minimum_required (VERSION 3.15)
    set(CMAKE_CXX_STANDARD 17)

    include_directories("/opt/qti-aic/dev/inc")

    add_executable(inference-set-io-buffers-example InferenceSetIOBuffersExample.cpp)
    set_target_properties(
        inference-set-io-buffers-example
        PROPERTIES
        LINK_FLAGS "-Wl,--no-as-needed"
    )
    target_compile_options(inference-set-io-buffers-example PRIVATE
                        -fstack-protector-all
                        -Werror
                        -Wall
                        -Wextra
                        -Wunused-variable
                        -Wunused-parameter
                        -Wnon-virtual-dtor
                        -Wno-missing-field-initializers)
    target_link_libraries(inference-set-io-buffers-example PRIVATE
                        pthread
                        dl)

    ```

## Main Flow
------ 

Main function has 8 parts. The example using few helper functions defined in the top anonymous namespace.

## QID
------ 

The first part of the `main()` example will pick `QID 0`. This is usually the first Enumerated device ID.

Though the API is capable of accepting a list of *QID*'s, in this example we only pass a single one in the `vector<> int` container.

## QPC
------ 

[QPC](runtime.md#qpc) is a container file that includes various parts of the compiled network.

The path is hardcoded in this example and could be changed or passed to the program via other means like environment variable or command line arguments.

`qaic::rt::Qpc::Factory` API will accept a path to the *QPC* and returns pack a *QPC* object to use in the next steps.

## Context
------ 

`QAIC` Runtime requires a [*Context*](runtime.md#context) object to be created and passed around to various APIs.

In this phase we use `qaic::rt::Context::Factory` API to obtain a new instance of *Context*.

We pass `NullProp` for no special `QAicContextProperties_t` attributes and the *QID* vector that was instantiated before.

## Inference Set
------ 

Creating an instance of [*InferenceSet*](runtime.md#inferenceset) is the next step.

*InferenceSet* is considered as a top-level entity when it comes to running Inferences on Hardware.

In this example, we set the Size of the Software/Hardware backlog as 10 possible pending buffers.

We have a single activation requested. This means that the provided program (encapsulated in *QPC*), will be activated as a single instance on the Hardware. We use the single *QID* provided when creating the InferenceSet instance.

## IO Buffers
------ 

Next step is required for setting up Input as well as output buffers.

In both Input and Output buffers, the user application will be allocating buffers and also will need to deallocate the buffers before application tear-down.

This part have few subsections:

1. Obtain [*InferenceHandle*](runtime.md#inferencehandle) to submit the I/O buffers once these created.
2. Allocate the buffers using *BufferMappings* container. We iterate over each *BufferMapping* instance to obtain information that helps us to allocate new buffers.
3. Once we have a vectors of allocated Input and Output buffers, we will use the *InferenceHandle* to submit it. It will be used during inference.

In this example, the helper functions will populate Input buffers with random data just to demonstrate the capabilities of the system.

## Submission
------ 

This is the part where the actual submission request is happening.

The *inferenceSet* is used to submit the request, passing *submitHandle* and user defined *inferenceId* (which was picked as ID 0)

## Completion
------ 

This is a blocking call to wait on inference completion and device output's buffers received in the Application.

We use *inferenceSet* to obtain the *completedHandle* passing our *inferenceId* as mentioned above.

User is responsible to return the *completedHandle* back to the Runtime pool and doing so by calling *putCompleted* using *inferenceSet*.

## Obtaining output
------ 

In this example we do not do anything with the obtained Output buffers and real-life Application will consume such output data.

## Cleanup
------ 

Since the buffers are user-allocated buffers using the System's Heap, the users is also in charge of properly releasing these buffers as demonstrated in the last phase of this example.

## Helper Functions
------ 

Throughout this example, the following helper functions and constructs are used:

* `RandomGen` : Random data generator - to generate random input buffer data.
* `isInputBuffer` : Query if a specific *BufferMapping* instance is an input one.
* `getPrintName` : Return Input or Output strings for standard output printing.
* `populateVector` : Populate vector - to populate inputs or output *vector<>* containers.
* `populateBufWithRandom` : Populate buffer with random data - using the above mentioned Random data generator, given a buffer, populate it with random values.
* `prepareBuffers` : Prepare buffers - iterates over the *BufferMappings* container and for each *BufferMapping* instance, populate input/output *vector<>* as well as invoke helper function to populate inputs buffers with random data.
* `releaseBuffers` : Release buffers - iterates over allocated inputs/outputs and release/delete buffers (return memory back to the system's Heap).
* `notInputOrOutput` : Not input our Output boolean function - Given a *BufferMapping* instance  return true if this instance is not Input, nor Output buffer instance. (For example could be invalid, or uninitialized). We skip these kind of instances.

## Compile and Run Commands
------ 

Copy the `InferenceSetIOBuffersExample.cpp` and `CMakeLists.txt` in a folder

Then compile the example with following commands
```bash
mkdir build
cd build
cmake ..
make -j 8
```
Finally, run the executable `./inference-set-io-buffers-example`, accordingly change the `qpcPath`.