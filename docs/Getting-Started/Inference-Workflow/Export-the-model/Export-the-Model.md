# Exporting ONNX Model from Different Frameworks

We will start with the process of exporting an ONNX model from various deep learning frameworks such as PyTorch, TensorFlow, and Caffe2.

## Exporting from PyTorch
PyTorch is a popular deep learning framework that provides dynamic computation graphs. To export a PyTorch model to ONNX format, you need to follow these steps:

```python
import torch
import torchvision.models as models

# Load a pre-trained PyTorch model
model = models.resnet18(pretrained=True)

# Set the model to evaluation mode
model.cpu().eval()

# Define example input tensor
input_tensor = torch.randn(1, 3, 224, 224).cpu()  # Replace with your own input shape

# Export the model to ONNX format
torch.onnx.export(model, input_tensor, "model.onnx", verbose=True, opset=13)
```

In the above code, we import the necessary libraries and load a pre-trained ResNet-18 model. We then set the model to evaluation mode and define an example input tensor. Finally, we use the `torch.onnx.export` function to export the model to the ONNX format. You can replace `"model.onnx"` with the desired output file path.

---
## Exporting from TensorFlow
TensorFlow is another widely used deep learning framework that provides both static and dynamic computation graphs. Here's how you can export a TensorFlow model to ONNX format:

```python
import tensorflow as tf
from tensorflow.keras.applications import ResNet50
import tf2onnx

# Load a pre-trained TensorFlow model
model = ResNet50(weights='imagenet')

# onnx_model = tf2onnx.convert.from_keras(model)
input_tensor_spec = (tf.TensorSpec((None, 224, 224, 3), tf.float32, name="input"),)

# Convert the TensorFlow model to ONNX format
# Save the ONNX model to a file
model_proto, _ = tf2onnx.convert.from_keras(model, input_signature=input_tensor_spec, opset=13, output_path="model.onnx")

```

In the above code, we import the necessary libraries and load a pre-trained ResNet-50 model using Keras. We then convert the TensorFlow model to the ONNX format using the `tf2onnx.convert.from_keras` function. Finally, we save the ONNX model to a file using the `tf2onnx.save_model` function.

In case you have save tensorflow model or saved checkpoint path of tensorflow model, you can load the model in tensorflow/keras or you can directly use the following commands from `tf2onnx` libraries.

```bash
# You have path to your saved TensorFlow model
python -m tf2onnx.convert --saved-model tensorflow-model-path --opset 11 --output model.onnx

# For checkpoint format:
python -m tf2onnx.convert --checkpoint  tensorflow-model-meta-file-path --output model.onnx --inputs input0:0,input1:0 --outputs output0:0

# For graphdef format:
python -m tf2onnx.convert --graphdef  tensorflow-model-graphdef-file --output model.onnx --inputs input:0,input:0 --outputs output0:0
```

You can refer this [notebook from tf2onnx](https://github.com/onnx/tensorflow-onnx/blob/main/tutorials/ConvertingSSDMobilenetToONNX.ipynb)


## Exporting from Caffe2
Caffe2 is a deep learning framework primarily developed by Facebook. It focuses on performance and is widely used for production deployments. Here's an example of exporting a Caffe2 model to ONNX format:

```python
from caffe2.python.onnx import backend
from caffe2.python.predictor import predictor_exporter

# Load the Caffe2 model from .pb and .init files
init_net = "model.init"  # Replace with the path to the .init file
predict_net = "model.pb"  # Replace with the path to the .pb file

# Export the Caffe2 model to ONNX format
onnx_model = backend.prepare(model, device='CPU')
onnx_model.export_onnx("model.onnx")
```

In the above code, we pre-trained caffe2 model from .init and .pb file. Finally, we use the `caffe2.python.onnx.backend.prepare` function to export the model to the ONNX format. You can replace `"model.onnx"` with the desired output file path. Please refer [caffe2-installation](https://caffe2.ai/docs/getting-started.html?platform=ubuntu&configuration=prebuilt) to setup the environment


## Loading ONNX Model

After exporting the model from your preferred framework to ONNX, you can load it for inference using the `onnx` package. Here's an example:

```python
import onnx
import onnx.checker

# Load the ONNX model
model = onnx.load("path/to/exported_model.onnx")

try:
    onnx.checker.check_model(model)
except:
    onnx.checker.check_model_path("path/to/exported_model.onnx")

```




