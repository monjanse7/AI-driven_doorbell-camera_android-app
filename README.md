# AIDoorMonitor v0.5.13 — Live event + camera-phone temperature overlay

## New in v0.5.13

- Monitor live view now shows a translucent status strip inside the video at the bottom.
- Camera-phone temperature is reported from Android's battery temperature sensor and refreshed through `/health`.
- Android thermal severity is also shown when the device supports `PowerManager.currentThermalStatus` (Android 10+).
- While an AI event is being retained, the overlay shows `EVENT REC mm:ss` plus the current AI label, e.g. `Dog`, `Cat`, `Person`, `Dog + Cat`, or `Dog whining / whimpering`.
- Event labels update during the recording as object/sound evidence arrives.
- When no event is active, the overlay reports whether the rolling DVR/pre-record buffer is active.
- The same status strip remains visible in fullscreen and automatically sits above the fullscreen Record / Audio / Exit controls.

# AI Door Monitor v0.5.12.1 – Dog sound AI + fullscreen crop zoom (build fix)

## Dog vocalization AI events

The Camera2 DVR now expands the existing on-device MediaPipe/YamNet sound AI to recognize and record more dog vocalizations:

- Bark / bow-wow
- Growling
- Whining / whimpering (`Whimper (dog)` in YamNet)
- Yip / short yelp-like vocalizations
- Howling

These detections use the existing shared DVR microphone pipeline, so there is still only one `AudioRecord`. A supported dog-sound hit can start/extend the DVR event and uses the same rolling pre-record audio/video buffer as the existing bark/growl trigger. The existing **Dog sounds start DVR event** preference now covers all of these dog vocalization events.

New built-in audio rules are merged into existing installations without overwriting user-created or previously edited rules.

## Fullscreen local crop zoom and pan

Monitor Mode fullscreen now changes the live gesture surface to **local viewer crop mode**:

- Pinch with two fingers: 1.0×–8.0× local crop magnification.
- Drag with one finger while magnified: pan freely around the received frame.
- The crop is applied only to the monitor phone's `MjpegView` matrix. It does **not** send a camera zoom/pan command.
- Any remote camera zoom already selected remains underneath the local viewer crop, so camera zoom and local crop can be combined.
- Leaving fullscreen resets only the local crop and restores the existing remote-camera gesture mode.
- Entering/leaving fullscreen keeps the same live MJPEG connection and live-audio session.
- Uniform matrix scaling is retained, so portrait video is not stretched.

The fullscreen overlay still includes **RECORD / STOP RECORDING**, **AUDIO ON/OFF**, and **EXIT**, plus a live crop-zoom status indicator.

---

# AI Door Monitor v0.5.11 – Fullscreen monitor live view

In **Monitor Mode**, tap **FULLSCREEN** on the live preview to expand the same active live viewer to the display. The fullscreen overlay provides:

- **RECORD / STOP RECORDING** — sends the existing remote MP4 start/stop commands to the camera phone.
- **AUDIO: ON / OFF** — toggles monitor-side live camera audio and stays synchronized with the normal `Live camera audio` checkbox.
- **EXIT** — returns to the normal Monitor Mode layout without intentionally restarting the MJPEG stream.
- Android **Back** exits fullscreen first.

The fullscreen viewer uses the same aspect-ratio-preserving `MjpegView` matrix as the normal preview, so portrait frames remain portrait instead of being stretched to fill the screen. System bars are hidden in fullscreen and can still be revealed transiently with the normal Android swipe gesture.

v0.5.11 includes the v0.5.10 **RESTART CAMERA APP / DVR** feature and all earlier DVR/AI/remote-control features.

---

# AI Door Monitor v0.3.3 DVR

Use **NEW DVR Camera2 • Motion + Pre-record** on the camera phone.

## Why v0.3.3 changes the camera pipeline

The Motorola Edge 30 Ultra log showed that camera 0 refuses every tested combination of 3840×2160 MediaRecorder plus a CPU-readable YUV ImageReader. v0.3.3 stops asking the camera for that unsupported combination.

The new DVR uses two PRIVATE-style camera outputs instead:

1. **4K MediaRecorder surface** for the rolling MP4 pre-record buffer.
2. **SurfaceTexture preview surface** for the viewer and motion detector.

The preview SurfaceTexture is consumed by an off-screen OpenGL worker. It downsamples the preview on the GPU, reads motion frames at a target of **10 fps**, and generates LAN-viewer JPEG frames at roughly **5 fps**.

## Suggested Motorola Edge 30 Ultra settings

- Show over other apps: **Allowed**
- Camera: rear/main camera
- Recording: **4K UHD 3840×2160**
- Motion AI: **On**
- Sensitivity: **65%**
- Pre-record: **10 seconds**
- Post-record: **20 seconds**

Wait for a status beginning with **DVR armed** before turning the screen off.

## Pre-record

The recorder continuously writes short temporary MP4 segments. Old non-event segments are deleted. When motion triggers, the already-completed pre-record segments are retained, post-record segments are added, and the event segments are merged into one MP4 without re-encoding.

## Audio

The new Camera2 DVR remains video-only for now. This deliberately avoids the microphone ownership conflicts that destabilized the older camera mode.


## AI event filenames (v0.3.5)

The DVR keeps motion/pre-record as the trigger mechanism, then classifies frames
during the event and uses the strongest enabled AI rule as the saved filename.

Open **AI EVENTS AND CUSTOM RULES** from DVR Camera mode to rename or add rules.
For example, a custom OBJECT rule named `My dog at door` with signal `dog` will
override the generic `Dog` filename when the dog detector passes that rule's
confidence threshold.

If the object model is missing, use **PREPARE / DOWNLOAD AI MODELS** and restart
DVR Camera mode after the download finishes.


## Remote recording library (v0.3.6)

In **Monitor Mode**, connect to the camera phone and use **Camera phone
recordings**. The list shows the AI-named recordings stored on the camera
phone. Select an item and choose:

- **Play selected recording from camera phone** to stream it over local Wi-Fi.
- **Save selected recording on monitor phone** to copy it into
  `Movies/AI Door Monitor/Camera Phone`.

Remote playback is token-protected and supports HTTP byte-range seeking.


## Recording thumbnails and v0.3.7 player

Monitor Mode now displays preview thumbnails generated from the actual camera-phone MP4 files. Tap a thumbnail to open playback.

Remote playback uses Media3 ExoPlayer. If the LAN MP4 timeline starts but no video frame renders, the player automatically caches a temporary local copy on the monitor phone and retries from the same playback position.


## Sound and vehicle event triggers (v0.3.9)

Enable **Sound AI trigger** to let YamNet start the same pre-record event used by
motion. Dog barking and dog growling are enabled by default.

**Car** and **Bicycle** can also start events from the normal object detector.

For **Stroller / barnevogn**, import a MediaPipe-compatible custom Object
Detector `.tflite` model with a label such as `stroller`, `pram`, `pushchair`,
or `baby_carriage`, then restart DVR Camera.

The microphone is used by Sound AI in this build. Saved DVR MP4 files are still
video-only; audio muxing is a separate feature.


## Audio in saved event videos (v0.4.0)

Enable **Record microphone audio in saved videos** in DVR Camera mode.

The DVR uses a single shared 48 kHz mono microphone capture. That PCM stream is
used both for event audio and for YamNet Sound AI. Saved event MP4s receive an
AAC-LC audio track after the video segments are merged.

This avoids running a separate microphone recorder for Sound AI while another
component tries to record MP4 audio.


## Moving-car-only detection (v0.4.3)

The `Car` event trigger now requires temporal movement. A parked car may still
be present in the detector output, but it is filtered out before DVR triggering
and event naming.

The app follows each car bounding box over multiple AI frames and confirms
movement from persistent center displacement or scale/area change. Remote zoom
or pan resets the tracker so a camera crop change cannot be mistaken for a
moving car.


## v0.5.0 highlights

- Remote Camera2 zoom uses `CONTROL_ZOOM_RATIO` on Android 11+ and reports the
  hardware-applied zoom ratio.
- 8K 7680×4320 at 24 fps is available when the selected camera exposes it.
- Monitor Mode can delete selected/checked originals from the camera phone.
- Remote recordings can be sorted by time, filename/AI label, duration or size.
- Audio synchronization uses the actual AudioRecord sample rate and can apply
  up to 5 seconds of manual Audio-later correction.


## Local video playback zoom/pan (v0.5.2)

In the remote recording player, enable **VIDEO ZOOM / PAN**.

- Pinch: 1x to 5x playback zoom.
- Drag: pan while zoomed.
- Double-tap: reset.
- Disable Zoom/Pan mode to use the normal Media3 playback controls.

This only changes the monitor-side view and does not modify the saved MP4.


## Deterministic camera rotation (v0.5.3)

For a phone mounted normally upright with the **top upward and USB/charger
toward the floor**, select:

**0° • UPRIGHT • top up / USB down**

Manual 0/90/180/270 values are direct video/preview corrections and no longer
depend on the phone's Camera2 `SENSOR_ORIENTATION`. Use Auto only when you want
Android sensor/display orientation to decide the output rotation.
