# Ground Scan, phone-only guide

No computer needed. You will host the app free on GitHub Pages and install it on your phone from Chrome.

## Part 1: Put the app online (about 10 minutes)

1. On your phone, open Chrome and go to github.com. Create a free account.
2. Tap the + menu > New repository. Name it `ground-scan`, set it to Public, tick "Add a README", tap Create.
3. Unzip `ground-scan-web.zip` using your Files app (long press the zip > Extract).
4. In your new repository tap Add file > Upload files. Select every file from the unzipped folder: `index.html`, `sw.js`, `manifest.webmanifest`, `labels.txt`, `icon-192.png`, `icon-512.png`. Tap Commit changes.
   If the page is cramped, tap the Chrome menu and turn on Desktop site.
5. Go to Settings > Pages. Under Branch choose `main` and `/ (root)`, then Save.
6. Wait 1 to 2 minutes. Your app is at `https://YOURUSERNAME.github.io/ground-scan/`.
7. Open that address in Chrome on your phone. Tap the Chrome menu > Add to Home screen (or Install app). It now has an icon like any other app.

The first launch asks for camera permission. Tap Allow.

## Part 2: Get a detection model

You have two options. Start with A so you see something working today.

### A. Cloud mode (works right away, needs signal)

1. Make a free account at roboflow.com and copy your API key (Settings > API Keys).
2. In the app tap Settings, paste the key, and leave the model ID as `arrowhead-6nfwq/2`. Set the source at the top to Cloud.
3. This sends a small photo of what the camera sees to Roboflow about once a second. It will not work without signal, and hosted calls may count against your free monthly credits, so check your Roboflow dashboard.
4. That public model was trained on mostly clean arrowhead photos, so expect misses in dirt and gravel.

### B. On-device mode (works offline, best long term)

You train your own model in Google Colab, which also works from a phone browser if you turn on Desktop site.

1. Go to colab.research.google.com, sign in, tap New notebook. Menu > Runtime > Change runtime type > T4 GPU.
2. Run these cells one at a time. Replace the API key.

Cell 1:
```
!pip install -q ultralytics roboflow
```

Cell 2:
```
from roboflow import Roboflow
rf = Roboflow(api_key="YOUR_ROBOFLOW_API_KEY")
project = rf.workspace("samoededu").project("arrowhead-6nfwq")
dataset = project.version(2).download("yolov8")
```

Cell 3:
```
from ultralytics import YOLO
model = YOLO("yolov8n.pt")
model.train(data=f"{dataset.location}/data.yaml", epochs=60, imgsz=640, batch=16)
```

Cell 4:
```
best = YOLO("runs/detect/train/weights/best.pt")
best.export(format="onnx", imgsz=640, opset=12)
with open("labels.txt", "w") as f:
    for i in range(len(best.names)):
        f.write(best.names[i] + "\n")
```

3. Open the folder icon on the left in Colab. Open runs/detect/train/weights/, tap the three dots next to `best.onnx`, Download. Download `labels.txt` the same way.
4. Rename `best.onnx` to `model.onnx`.
5. In your GitHub repository tap Add file > Upload files and upload `model.onnx` and `labels.txt` (replace the old labels). The file is about 12 MB, which GitHub allows.
6. Wait a minute, then fully close and reopen the app. Set the source to On-device. After the first load it works with no signal.

If it is too slow on your phone, export with `imgsz=320` in Cell 4, and change `var SIZE = 640;` near the top of the script in `index.html` to 320 (edit the file in GitHub with the pencil icon).

## Making it actually good

Detection quality depends on the photos you train on. Public datasets show clean points on plain backgrounds, while real finds are dirt-colored and surrounded by look-alike rocks. Take many photos where you hunt (arrowheads and similar-looking rocks, dry and wet, different light), label them free in Roboflow, add them to the dataset, and retrain with Cell 3 and Cell 4. Add classes for quartz, agate, jasper, chert, and fossils the same way. Check each dataset's license, since many require attribution.

## Using it

- Hold the phone 2 to 3 feet above the ground and sweep slowly.
- Orange boxes are arrowheads and worked stone. Blue boxes are rocks and minerals.
- Use the slider to hide weak detections and the top menu to filter by type.
- Save find stores a photo, the label, the time, and your GPS position on the phone. Finds exports as a CSV.

## Limits to know about

- This runs in the browser, so it is slower than a native app, roughly a few frames per second.
- Keep the screen on while scanning. Browsers can pause the camera if the screen locks.
- Saved finds live in the browser's storage. Clearing Chrome's site data deletes them, so export now and then.

## Legal

Digging and collecting rules vary. Artifacts on federal, state, and tribal land are generally protected, and private land needs the owner's permission.
