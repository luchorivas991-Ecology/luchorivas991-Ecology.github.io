# Camera-trap species detection using YOLO

## Overview

This project presents a preliminary computer vision workflow for detecting mammal species in camera-trap imagery using a YOLO object detection model.

The project used open-access images from the Missouri Camera Traps dataset, focusing on three mammal species: paca, ocelot, and red fox. The images were downloaded using AWS CLI, organized through PowerShell, annotated in MakeSense.ai, and used to train a custom YOLO model in Python.

**Study system:** Camera-trap imagery  
**Dataset:** Missouri Camera Traps dataset, LILA BC / Microsoft AI for Earth wildlife datasets  
**Target species:** Paca, ocelot, red fox  
**Role:** Solo pilot project  
**Status:** Preliminary prototype  

---

## Methods & Tools

**Data sources**

- Missouri Camera Traps dataset
- LILA BC / Microsoft AI for Earth wildlife datasets
- Test video from YouTube footage of an ocelot

**Processing steps**

1. Download open-access camera-trap images using AWS CLI.
2. Organize images by species using PowerShell.
3. Annotate images manually in MakeSense.ai.
4. Export annotations in YOLO format.
5. Prepare the YOLO dataset structure with Python.
6. Train a YOLO model using Ultralytics.
7. Test the trained model using video-based inference.
8. Evaluate the workflow limitations and future improvements.

**Tools used**

| Tool | Purpose |
|---|---|
| AWS CLI | Download open-access camera-trap images |
| PowerShell | Organize image folders and files |
| MakeSense.ai | Manual image annotation |
| Python | Dataset preparation and model workflow |
| Ultralytics YOLO | Object detection model training and inference |
| Jupyter Notebook | Reproducible workflow documentation |

---

## Setup

The first step was to import the main Python libraries used to prepare the YOLO dataset.

```python
from pathlib import Path
import random
import shutil
```

The project root was defined to organize input images, annotation files, and the final YOLO dataset.

```python
ROOT = Path(r"C:\Users\Lucho\OneDrive\Desktop\QGIS Mastery\YOLO Project")
```

---

## Data

Images were organized by species after being downloaded from the Missouri Camera Traps dataset. The corresponding annotation files were exported from MakeSense.ai in YOLO format.

```python
IMAGE_DIRS = {
    "paca": ROOT / "data" / "annotation_batch" / "paca",
    "ocelot": ROOT / "data" / "annotation_batch" / "ocelot",
    "red_fox": ROOT / "data" / "annotation_batch" / "red_fox",
}

LABEL_DIRS = {
    "paca": ROOT / "data" / "annotations_fixed" / "paca",
    "ocelot": ROOT / "data" / "annotations_fixed" / "ocelot",
    "red_fox": ROOT / "data" / "annotations_fixed" / "red_fox",
}
```

The output folder was defined using the standard YOLO dataset structure.

```python
OUT = ROOT / "data" / "yolo_dataset"

SPLITS = {
    "train": 0.70,
    "val": 0.20,
    "test": 0.10,
}

random.seed(42)
```

```python
for split in SPLITS:
    (OUT / "images" / split).mkdir(parents=True, exist_ok=True)
    (OUT / "labels" / split).mkdir(parents=True, exist_ok=True)
```

---

## Analysis

### Dataset preparation

Each image was matched with its corresponding YOLO label file. Images without annotation files were excluded from the final dataset.

```python
all_items = []

for species, img_dir in IMAGE_DIRS.items():
    label_dir = LABEL_DIRS[species]

    images = sorted([
        p for p in img_dir.rglob("*")
        if p.suffix.lower() in [".jpg", ".jpeg", ".png"]
    ])

    print(f"{species}: found {len(images)} images")

    for img_path in images:
        label_path = label_dir / f"{img_path.stem}.txt"

        if not label_path.exists():
            print(f"WARNING: missing label for {img_path.name}")
            continue

        all_items.append((species, img_path, label_path))

print(f"\nTotal image-label pairs: {len(all_items)}")
```

### Train, validation, and test split

The image-label pairs were randomly split into training, validation, and testing subsets. Species names were added as filename prefixes to avoid filename collisions.

```python
random.shuffle(all_items)

n = len(all_items)
n_train = int(n * SPLITS["train"])
n_val = int(n * SPLITS["val"])

split_items = {
    "train": all_items[:n_train],
    "val": all_items[n_train:n_train + n_val],
    "test": all_items[n_train + n_val:],
}
```

```python
for split, items in split_items.items():
    print(f"{split}: {len(items)}")

    for species, img_path, label_path in items:
        new_img_name = f"{species}_{img_path.name}"
        new_label_name = f"{species}_{label_path.name}"

        shutil.copy2(img_path, OUT / "images" / split / new_img_name)
        shutil.copy2(label_path, OUT / "labels" / split / new_label_name)
```

### YOLO configuration file

The `data.yaml` file defines the dataset path, training/validation/testing folders, and class names used by YOLO.

```python
yaml_text = f"""path: {OUT.as_posix()}
train: images/train
val: images/val
test: images/test

names:
  0: paca
  1: ocelot
  2: red_fox
"""

with open(OUT / "data.yaml", "w", encoding="utf-8") as f:
    f.write(yaml_text)

print("\nYOLO dataset created successfully.")
print(f"Dataset path: {OUT}")
print(f"YAML file: {OUT / 'data.yaml'}")
```

### Model training

A YOLO model was trained using Python and the Ultralytics framework. Since the dataset was relatively small, the model should be interpreted as a proof of concept rather than a robust wildlife detector.

```python
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

model.train(
    data=str(OUT / "data.yaml"),
    epochs=20,
    imgsz=640,
    batch=8,
    name="camera_trap_yolo_pilot"
)
```

### Video-based inference

After training, the model was tested on a short ocelot video. This step was used to evaluate whether the trained detector could be applied to moving footage.

```python
model = YOLO("runs/detect/camera_trap_yolo_pilot/weights/best.pt")

results = model.predict(
    source="path/to/ocelot_video.mp4",
    save=True,
    conf=0.25
)
```

<video controls width="100%">
  <source src="/assets/images/Video%20Project%202a.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
---

## Key Findings

- The full workflow from open-access image download to annotation, dataset preparation, YOLO training, and video inference was successfully tested.
- The model was trained with a limited number of annotated images, approximately 150 per species, so the detections are not yet robust enough for ecological interpretation.
- The workflow showed the potential of combining computer vision with wildlife monitoring, especially for reducing manual camera-trap image review.
- Future improvements should include more training images, better class balance, independent validation, and performance evaluation using precision, recall, mAP, and confusion matrices.

---

## Limitations

This project should be interpreted as a preliminary prototype.

The main limitations were:

- Limited number of annotated images per species
- Small number of target classes
- Possible class imbalance
- Limited variation in image conditions
- No independent validation dataset
- Need for formal model evaluation before ecological interpretation

---

## Links

[View Data Source](https://lnkd.in/eS8CwQr6)

[Back to top](#camera-trap-species-detection-using-yolo)