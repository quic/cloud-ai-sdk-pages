# Quick Start Guide
This section illustrates the ease of running inference on Cloud AI platforms using a vision transformers model for classifying images. 

## Device Status

Follow the [checklist](../Installation/Checklist/checklist.md) to make sure device is ready for use.


## Steps to run a sample model on Qualcomm Cloud AI Platforms 

### 1. Import libraries

```py
import os, sys, requests, torch, numpy, PIL
from transformers import ViTForImageClassification, ViTImageProcessor

sys.path.append('/opt/qti-aic/examples/apps/qaic-python-sdk/qaic')
import qaic

```

### 2. Pick a model from HF

```py title="Choose the Vision Transformers model for classifying images and its image input preprocessor"
model = ViTForImageClassification.from_pretrained('google/vit-base-patch16-224')
processor = ViTImageProcessor.from_pretrained('google/vit-base-patch16-224')

```

### 3. Convert to ONNX

```py
dummy_input = torch.randn(1, 3, 224, 224)       # Batch, channels, height, width

torch.onnx.export(model,                        # PyTorch model
                     dummy_input,               # Input tensor
                     'model.onnx',              # Output file
                     export_params = True,      # Export the model parameters
                     input_names   = ['input'], # Input tensor names
                     output_names  = ['output'] # Output tensor names
)


```

### 4. Compile the model 

Compile the model with the `qaic-exec` CLI tool. You can find more details about its usage [here](../Inference-Workflow/model-compilation/Compile%20the%20Model.md).
This quickstart issues the command from Python.

```py
aic_binary_dir = 'aic-binary-dir'
cmd = '/opt/qti-aic/exec/qaic-exec -aic-hw -aic-hw-version=2.0 -compile-only -convert-to-fp16 \
-aic-num-cores=4 -m=model.onnx -onnx-define-symbol=batch_size,4 -aic-binary-dir=' + aic_binary_dir

os.system(cmd)
```

### 5. Get example input

```py
url = 'http://images.cocodataset.org/val2017/000000039769.jpg'
image = PIL.Image.open(requests.get(url, stream=True).raw)
```

### 6. Run the model

```py title="Create the AIC100 session and prepare inputs and outputs"
vit_sess = qaic.Session(model_path= aic_binary_dir+'/programqpc.bin',\
   num_activations=3) # (1)

inputs = processor(images=image, return_tensors='pt')
input_shape, input_type = vit_sess.model_input_shape_dict['input']
input_data = inputs['pixel_values'].numpy().astype(input_type) 
input_dict = {'input': input_data}

output_shape, output_type = vit_sess.model_output_shape_dict['output']
```

 1. Make sure qpcPath matches where the output is generated in `qaic-exec` above. We are activating 3 instances on network. [More options](../../Python-API/qaic/qaic.md#parameters).


```py title="Run model on AIC100"
vit_sess.setup() # Load the model to the device.
output = vit_sess.run(input_dict) # Execute on AIC100 now.
```

```py title="Obtain the prediction by finding the highest probability among all classes"
logits = numpy.frombuffer(output['output'], dtype=output_type).reshape(output_shape)
predicted_class_idx = logits.argmax(-1).item()
print('Predicted class:', model.config.id2label[predicted_class_idx]) 
```


## Next Steps

We showed the ease of onboarding and running inference on Cloud AI platforms in this section. Refer to the [User Guide](../index.md) for details on SDK installation, inference workflow, system management etc. 


## Appendix

[Input image link](http://images.cocodataset.org/val2017/000000039769.jpg)

### Full quickstart code
??? example "quickstart.py"
    ```py
    # Copyright (c) 2023 Qualcomm Innovation Center, Inc. All rights reserved.
    # SPDX-License-Identifier: BSD-3-Clause-Clear
    # Import relevant libraries
    import os, sys, requests, torch, numpy, PIL
    from transformers import ViTForImageClassification, ViTImageProcessor
    
    sys.path.append('/opt/qti-aic/examples/apps/qaic-python-sdk/qaic')
    import qaic
    
    # Choose the Vision Transformers model for classifying images and its image input preprocessor
    model = ViTForImageClassification.from_pretrained('google/vit-base-patch16-224')
    processor = ViTImageProcessor.from_pretrained('google/vit-base-patch16-224')
    
    # Convert to ONNX
    dummy_input = torch.randn(1, 3, 224, 224)     # Batch, channels, height, width
    
    torch.onnx.export(model,                      # PyTorch model
                        dummy_input,              # Input tensor
                        'model.onnx',             # Output file
                        export_params=True,       # Export the model parameters
                        input_names  =['input'],  # Input tensor names
                        output_names =['output']  # Output tensor names
    )
    
    # Compile the model 
    aic_binary_dir = 'aic-binary-dir'
    cmd = '/opt/qti-aic/exec/qaic-exec -aic-hw -aic-hw-version=2.0 -compile-only -convert-to-fp16 \
    -aic-num-cores=4 -m=model.onnx -onnx-define-symbol=batch_size,4 -aic-binary-dir=' + aic_binary_dir
    
    os.system(cmd)
    
    # Get example Egyptian cat image for classification
    url = 'http://images.cocodataset.org/val2017/000000039769.jpg'
    image = PIL.Image.open(requests.get(url, stream=True).raw)
    
    # Run the model
    
    ## Create the AIC100 session and prepare inputs and outputs
    vit_sess = qaic.Session(model_path= aic_binary_dir+'/programqpc.bin',\
       num_activations=3)
    
    inputs = processor(images=image, return_tensors='pt')
    input_shape, input_type = vit_sess.model_input_shape_dict['input']
    input_data = inputs['pixel_values'].numpy().astype(input_type) 
    input_dict = {'input': input_data}
    
    output_shape, output_type = vit_sess.model_output_shape_dict['output']
    
    ## Access the hardware
    vit_sess.setup() # Load the model to the device.
    output = vit_sess.run(input_dict) # Execute on AIC100 now.
    
    ## Obtain the prediction by finding the highest probability among all classes.
    logits = numpy.frombuffer(output['output'], dtype=output_type).reshape(output_shape)
    predicted_class_idx = logits.argmax(-1).item()
    print('Predicted class:', model.config.id2label[predicted_class_idx]) 
    ```
### Output
??? example "Output"
    ```
    sudo python quickstart.py                  
    /usr/local/lib/python3.8/dist-packages/transformers/models/vit/modeling_vit.py:170: TracerWarning: Converting a  tensor to a Python boolean might cause the trace to be incorrect. We can't record the data flow of Python  values, so this value will be treated as a constant in the future. This means that the trace might not  generalize to other inputs!
    if num_channels != self.num_channels:
    /usr/local/lib/python3.8/dist-packages/transformers/models/vit/modeling_vit.py:176: TracerWarning: Converting a  tensor to a Python boolean might cause the trace to be incorrect. We can't record the data flow of Python  values, so this value will be treated as a constant in the future. This means that the trace might not  generalize to other inputs!
    if height != self.image_size[0] or width != self.image_size[1]:
    ============= Diagnostic Run torch.onnx.export version 2.0.1+cu117 =============
    verbose: False, log level: Level.ERROR
    ======================= 0 NONE 0 NOTE 0 WARNING 0 ERROR ========================
 
    Reading ONNX Model from model.onnx
    Compile started ............... 
    Compiling model with FP16 precision.
    Generated binary is present at aic-binary-dir
    Predicted class: Egyptian cat
    ```