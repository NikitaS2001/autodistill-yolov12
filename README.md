<div align="center">
  <p>
    <a align="center" href="" target="_blank">
      <img
        width="850"
        src="https://media.roboflow.com/open-source/autodistill/autodistill-banner.jpg"
      >
    </a>
  </p>
</div>

# Autodistill: YOLOv12 Target Model

This repository contains the code implementing [YOLOv12](https://github.com/sunsmarterjie/yolov12) as a Target Model for use with [`autodistill`](https://github.com/autodistill/autodistill). You can also use a YOLOv12 model as a base model to auto-label data. 

YOLOv12 is a Convolutional Neural Network (CNN) that supports realtime object detection, instance segmentation, keypoint detection, and more.

Read the full [Autodistill documentation](https://autodistill.github.io/autodistill/).

## Installation

Install the library (and all required dependencies, including the patched Ultralytics fork) directly from git:

```bash
pip install "autodistill-yolov12 @ git+https://github.com/NikitaS2001/autodistill-yolov12"
```

> Internally the package depends on `ultralytics @ git+https://github.com/sunsmarterjie/yolov12`, so the compatible fork is pulled automatically and you can keep importing `YOLO` from `ultralytics` as usual.

You will also need to install a base model like Grounded SAM (`autodistill-grounded-sam`) to label data.

You can find a full list of `detection` Base Models on [the main autodistill repo](https://github.com/autodistill/autodistill).

## Quickstart (Train a YOLOv12 Model)

```python
from autodistill_grounded_sam import GroundedSAM
from autodistill.detection import CaptionOntology
from autodistill_yolov12 import YOLOv12

# define an ontology to map class names to our GroundingDINO prompt
# the ontology dictionary has the format {caption: class}
# where caption is the prompt sent to the base model, and class is the label that will
# be saved for that caption in the generated annotations
base_model = GroundedSAM(ontology=CaptionOntology({"shipping container": "container"}))

# label all images in a folder called `context_images`
base_model.label(
  input_folder="./images",
  output_folder="./dataset"
)

target_model = YOLOv12("yolo12n.pt")
target_model.train("./dataset/data.yaml", epochs=200)

# run inference on the new model
pred = target_model.predict("./dataset/valid/your-image.jpg", confidence=0.5)
print(pred)

# optional: upload your model to Roboflow for deployment
from roboflow import Roboflow

rf = Roboflow(api_key="API_KEY")
project = rf.workspace().project("PROJECT_ID")
project.version(DATASET_VERSION).deploy(model_type="yolov12", model_path=f"./runs/detect/train/")
```

## Quickstart (Use a YOLOv12 Model to Label Data)

```python
from autodistill_yolov12 import YOLOv12Base
from autodistill.detection import CaptionOntology

# define an ontology to map class names to our YOLOv12 classes
# the ontology dictionary has the format {caption: class}
# where caption is the prompt sent to the base model, and class is the label that will
# be saved for that caption in the generated annotations
# then, load the model

# replace weights_path with the path to your YOLOv12 weights file
base_model = YOLOv12Base(ontology=CaptionOntology({"car": "car"}), weights_path="yolo12n.pt")

# run inference on a single image
results = base_model.predict("container.jpeg")

base_model.label(
  input_folder="./images",
  output_folder="./dataset"
)
```

## Choosing a Task

YOLOv12 supports training both object detection and instance segmentation tasks at various sizes (larger models are slower but can be more accurate). This selection is done in the constructor.

For example:
```python
# initializes a nano-sized instance segmentation model
target_model = YOLOv12("yolov12n-seg.pt")
```

Available object detection initialization options are:

* `yolo12n.pt` - nano
* `yolo12s.pt` - small
* `yolo12m.pt` - medium
* `yolo12l.pt` - large
* `yolo12x.pt` - extra-large

## License

The code in this repository is licensed under an [AGPL 3.0 license](LICENSE).

## 🏆 Contributing

We love your input! Please see the core Autodistill [contributing guide](https://github.com/autodistill/autodistill/blob/main/CONTRIBUTING.md) to get started. Thank you 🙏 to all our contributors!
