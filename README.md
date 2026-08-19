# v0.5.15.1 - Camera battery status fix

- Fixes `Battery: --% • Unavailable` on camera phones where the sticky `ACTION_BATTERY_CHANGED` query returns null or fails.
- Uses `BatteryManager.BATTERY_PROPERTY_CAPACITY` as a percentage fallback.
- Uses `BatteryManager.BATTERY_PROPERTY_STATUS` / `isCharging` as charge-status fallbacks.
- Uses the modern exported receiver flag on Android 13+ for the sticky system battery query.
- Keeps USB / AC / Wireless / Dock source reporting when the sticky intent provides it.

# v0.5.15 – Camera battery status overlay

- Monitor live view now shows the camera phone battery percentage.
- Shows Charging, Full, Discharging, Not charging, or Unknown state.
- Shows AC, USB, Wireless, or External power when Android reports a power source.
- Battery data is added centrally to `/health`, so it remains available across camera endpoint hand-offs.
- Battery information is visible in both normal live view and fullscreen together with temperature and AI event recording status.

# v0.5.13

- Added live-view camera-phone temperature overlay using Android battery temperature.
- Added thermal severity status from PowerManager on Android 10+.
- Added active event recording indicator with elapsed `mm:ss` timer.
- Added live AI event label display (`Dog`, `Cat`, `Person`, multi-object combinations, dog sound labels, etc.).
- Status overlay is visible in both normal and fullscreen live view and stays above fullscreen controls.
- `/health` now exposes `cameraPhoneTemperatureC`, `cameraPhoneThermalStatus`, `eventRecordingActive`, `eventRecordingLabel`, and `eventRecordingElapsedMs`.

# AIDoorMonitor v0.5.12.1 – Kotlin Build Fix

- Fixed the compiler-breaking `Map.mapValues` lambdas in `DvrEventLabeler.kt`.
- Replaced invalid two-parameter forms such as `mapValues { _, items -> ... }` with valid `Map.Entry` destructuring: `mapValues { (_, items) -> ... }`.
- Fixed all three occurrences, including evidence aggregation and `currentBestLabel()`.
- This removes the cascade of `Argument type mismatch`, `Cannot infer type`, `Unresolved reference it`, and `Unresolved reference size` errors reported by Android Studio.
- Keeps all v0.5.12 dog-sound AI and fullscreen crop-zoom features.

# AIDoorMonitor v0.5.12 – Dog Sound AI + Fullscreen Crop Zoom

- Expanded the existing MediaPipe/YamNet dog-sound detector with `dog_whining`, `dog_yelping`, and `dog_howling` events.
- Recognizes YamNet-style Bark/Bow-wow, Growling, Whimper (dog), Yip, and Howl outputs, with common label aliases.
- Expanded the per-window classifier scan from the top 16 to the top 24 categories so quieter/subtler dog vocalizations are less likely to be discarded before thresholding.
- New dog vocalization events can start or extend the same DVR event/pre-record pipeline as bark/growl.
- Added built-in AI rules for Dog whining/whimpering, Dog yelping/yipping, and Dog howling. `RuleStore` automatically merges them on upgrade without replacing existing user settings.
- Added dog-audio evidence floors so a valid low-confidence sound trigger is not automatically renamed to generic `Motion` at event-save time.
- Updated the DVR dog-sound trigger UI text to include bark, growl, whine, whimper, yelp/yip, and howl.
- Fullscreen live viewer now switches from remote-camera gestures to local crop gestures.
- Two-finger pinch applies 1×–8× local viewer zoom to the received MJPEG frame.
- One-finger drag pans the locally magnified frame without sending any camera pan command.
- Local crop resets when fullscreen closes; the pre-existing remote camera zoom/pan state is not altered.
- Added a fullscreen `VIEW CROP n.n×` gesture hint/status overlay.
- Existing fullscreen Record, Audio and Exit controls and v0.5.10 remote-camera restart remain included.

# AIDoorMonitor v0.5.11 – Fullscreen Monitor Controls

- Added a **FULLSCREEN** button directly on the Monitor Mode live preview.
- Fullscreen uses the existing `MjpegView`; it does not intentionally stop/recreate the live stream when entering or leaving fullscreen.
- Added fullscreen **RECORD / STOP RECORDING** control for the camera phone's remote MP4 recording command.
- Added fullscreen **AUDIO: ON / OFF** control, synchronized with the existing Live camera audio preference/player.
- Added fullscreen **EXIT** control.
- Android Back exits fullscreen first instead of leaving Monitor Mode or disconnecting the live session.
- Fullscreen hides status/navigation bars and the rest of the monitor settings, then restores their previous visibility/padding on exit.
- The live image keeps the existing uniform FIT_CENTER matrix so portrait video is not stretched horizontally.
- All v0.5.10 remote camera restart behavior remains included.

# AIDoorMonitor v0.5.10 – Remote Camera Restart

- Added a **RESTART CAMERA APP / DVR** control to Monitor Mode.
- Added authenticated LAN endpoint `/app/restart`.
- DvrCamera2Service now rebuilds the camera/audio pipeline after a remote restart request while keeping the LAN endpoint alive.
- Added best-effort request to reopen the camera dashboard; Android may block background UI launches on newer versions.
- Camera foreground services explicitly use `stopWithTask=false`; DVR remains `START_STICKY` so swiping away the UI does not intentionally stop monitoring.
- Added task-removal recovery that reasserts the LAN endpoint and camera session when the service is still alive.
- Android **Force stop** remains intentionally non-recoverable remotely because the OS blocks all app components until the user opens the app again.

# AIDoorMonitor v0.5.9 – Multi-Object Event AI

## Fixed: owner detected, dogs missing from the event name

The previous pipeline had two independent filters that could remove a dog from
an otherwise correct event:

1. EfficientDet detections below 0.42 were discarded globally.
2. The built-in Dog rule required 65% confidence, and when another candidate
   such as Person was stronger, a second label was only retained when its score
   was within 0.12 of the strongest candidate.

This meant a large/close owner at e.g. 85% could produce `Person`, while a
smaller dog at 35–60% was either discarded or omitted from the final name.

## Pet-sensitive object thresholds

Object detection now keeps credible small-object detections at:

- Dog: 0.28
- Cat: 0.30
- Bird: 0.30
- Person: 0.36
- Bicycle: 0.34
- Other COCO classes: 0.42

These lower values are not sufficient by themselves to dominate an event name.
The event labeler applies temporal evidence afterwards.

EfficientDet now retains up to 12 objects per analysis frame instead of 6.

## Temporal / repeated evidence

For each signal the event labeler now records:

- maximum confidence,
- total hit count,
- number of distinct analysis times.

A built-in object category can qualify below its old rule threshold when it is
seen repeatedly across the event. This specifically allows smaller dogs beside
a person to remain part of the event classification.

Persistence adds a limited score bonus, capped at 0.18.

## Multi-object event names

The old "second label must be within 0.12 of the winner" rule has been removed.

Up to four independently supported event categories can now be retained.

Examples:

- `Person + Dog`
- `Person + Dog + Bicycle`
- `Person + Dog + Something thrown at the door`
- `Cat + Person`

A strong Person detection no longer suppresses Dog merely because the dog has a
lower confidence.

## Event log evidence

Saved-event log entries now include an evidence summary, for example:

`evidence=person=0.88×9/7; dog=0.46×6/5`

The first number is maximum confidence, followed by total hits and distinct
analysis times. This makes missed or false classifications much easier to tune.

All v0.5.8 features remain:
- persistent monitor connection when returning from video playback,
- live camera audio,
- native portrait GPU preview,
- remote resolution/bitrate/audio settings,
- strict saved-MP4 audio verification,
- event folders,
- moving-car-only filtering,
- remote zoom/focus/delete,
- player zoom/pan/rotation.
