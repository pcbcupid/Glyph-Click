# Glyph Click: laptop-only version (no ESP32)

Prints photos on a **Cashino CSN-A1** 58 mm thermal printer straight from a laptop.
The laptop does all the image processing and sends the result to the printer's own USB port.
No microcontroller is needed.

## What it does

1. **Finds faces** with OpenCV's YuNet model (`models/face_detection_yunet_2023mar.onnx`).
2. **Frames the photo**: crops to the people (or keeps the whole photo with `--full`), and turns it sideways when that prints bigger.
3. **Evens out the brightness**: contrast stretch, CLAHE local contrast, then a gamma that puts every face at the same tone, so dark and washed-out photos print alike.
4. **Dithers** with Atkinson error diffusion. This was chosen by an objective comparison (`evaluate.py`, a paper model scored with SSIM) as the most accurate of the styles tried.
5. **Prints** with the Cashino `DC2 *` bitmap command over the printer's USB. Unlike `GS v 0`, it doesn't stop every 24 rows, so there are **no white lines**.

## Setup

```bash
python3 -m venv .venv
.venv/bin/pip install -r glyph_cam_direct/requirements.txt
```

Plug in the printer's USB cable. It appears as `/dev/ttyUSB0` (a PL2303 chip; the baud setting is ignored). Power the printer with 5–9 V and at least 2 A.

## Print

Put photos in `images/` at the project root, then:

```bash
.venv/bin/python glyph_cam_direct/print_image.py photo.jpg --auto           # crop to faces
.venv/bin/python glyph_cam_direct/print_image.py photo.jpg --auto --full    # whole photo
.venv/bin/python glyph_cam_direct/print_image.py photo.jpg --auto --preview out.png   # no paper
.venv/bin/python glyph_cam_direct/print_image.py                            # list images/
```

| Option | Effect |
|---|---|
| `--face-tone 0.45` / `0.60` | darker / lighter faces (default 0.52) |
| `--max-height N` | print height in dots (8 dots = 1 mm) |
| `--printer /dev/ttyUSB1` | a different printer port |
| `--style halftone\|dither` | other styles (Atkinson is the default) |

## Files

| File | Purpose |
|---|---|
| `print_image.py` | load, frame, tone, dither, send to the printer |
| `auto_enhance.py` | face detection, crop/rotate, even brightness (`--auto`) |
| `evaluate.py` | compares print styles with a paper model and SSIM |
| `models/face_detection_yunet_2023mar.onnx` | YuNet face detector from the [OpenCV Zoo](https://github.com/opencv/opencv_zoo) (MIT license) |

## Printer notes (CSN-A1, firmware A1X V7.08)

- 384 dots per line. Self-test page: hold FEED while switching on.
- `GS v 0` raster prints white lines every 24 rows. Use `DC2 *` (≤255 rows per command) or `DC2 V`.
- `GS *` / `GS /` (downloaded bitmaps) are **not** supported: the printer prints the bytes as text.
- Don't send `ESC 7` heat settings; the factory values print fine.

## Sample prints

Photos of real CSN-A1 printouts made with `print_image.py --auto` (Atkinson dither, even face tone, no white lines):

| | | | | |
|---|---|---|---|---|
| ![](sample%20images/image1.jpeg) | ![](sample%20images/image2.jpeg) | ![](sample%20images/image3.jpeg) | ![](sample%20images/image4.jpeg) | ![](sample%20images/image5.jpeg) |
