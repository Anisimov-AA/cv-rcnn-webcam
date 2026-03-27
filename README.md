# rcnn-webcam

Real-time object detection from a webcam with a pretrained Faster R-CNN. It draws a box and a label around each object it recognizes (person, cup, phone, and the other 80 COCO classes), shows the FPS, and saves the video to `output.avi`.

The goal was to make R-CNN fast enough for live video on a CPU, and to measure what each setting costs.

## How it works

For every webcam frame:

1. **Resize** the frame so its longer side is 480 px. Smaller input means faster detection.
2. **Detect**, but only on every 2nd frame. The other frames reuse the last boxes.
3. **Filter**: drop boxes with a score below 0.6, then remove overlapping boxes (NMS, IoU 0.4).
4. **Draw** the boxes, labels and FPS, show the frame, and write it to `output.avi`.

Press `q` to stop. At the end it prints the average FPS.

## Results (CPU)

**Backbone**

| Model | FPS |
|---|---|
| MobileNet v3 | 7.8 |
| ResNet50 (`--model vgg16`) | 0.9, about 8.7× slower |

**Settings** (with MobileNet, the other settings at their defaults)

| Setting | Value | FPS | Effect |
|---|---|---|---|
| Score threshold | 0.3 | 8.3 | more detections, more false positives |
| | **0.6** | 7.8 | best balance |
| | 0.7 | 8.0 | fewer detections |
| NMS IoU | 0.3 | 7.8 | fewer overlapping boxes |
| | **0.4** | 7.8 | best balance |
| | 0.6 | 8.0 | more overlapping boxes |
| Proposals | 50 | 8.8 | slightly less accurate |
| | **100** | 7.8 | best balance |
| | 300 | 6.1 | slower |

Best config: score 0.6, IoU 0.4, 100 proposals. These are the defaults.

## Requirements

- **Python 3** and a webcam
- Packages from `requirements.txt` (PyTorch, torchvision, OpenCV). The model weights download automatically on the first run.

## Run

In PowerShell, from the project folder:

```powershell
python -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt
```

```powershell
python webcam_rcnn.py
```

Try other settings:

```powershell
python webcam_rcnn.py --model vgg16
python webcam_rcnn.py --score_thresh 0.3
python webcam_rcnn.py --iou_thresh 0.3
python webcam_rcnn.py --proposals 50
```

| Option | Default | Meaning |
|---|---|---|
| `--model` | `mobilenet` | `mobilenet` or `vgg16` (actually loads ResNet50) |
| `--score_thresh` | `0.6` | minimum confidence to draw a box |
| `--iou_thresh` | `0.4` | how much two boxes can overlap before one is removed |
| `--proposals` | `100` | how many candidate regions the model checks (MobileNet only) |
| `--skip_frames` | `2` | run detection on every Nth frame |
| `--resize` | `480` | longer side of the frame before detection |
