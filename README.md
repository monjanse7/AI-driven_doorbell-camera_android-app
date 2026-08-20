AI Door Monitor — turn an old Android phone into a smart AI doorbell instead of buying another subscription-based camera.


AI Door Monitor is a self-hosted Android smart doorbell and security camera system that turns two Android phones into an AI-powered camera and remote monitor — with local recording, object and sound detection, live view, remote camera control, doorbell alerts, talkback, and no subscription required.

AI Door Monitor turns spare Android phones into a powerful, self-hosted AI doorbell and security camera system.

One Android phone acts as the Camera Phone, while another acts as the Monitor Phone. The camera phone continuously watches the entrance, performs local AI detection, records events, and streams live video and audio. The monitor phone provides remote live viewing, camera controls, event playback, doorbell notifications, and two-way communication.

The project is designed as an alternative to cloud-dependent smart doorbells. Recordings and AI processing can remain under your own control, with no mandatory cloud service or subscription.

Main features
📹 Android phone as a security/doorbell camera
📱 Separate Camera and Monitor modes
🤖 Local AI object detection
👤 Person detection and person counting
🐕 Dog detection, including additional small/distant-dog detection
🐈 Cat detection
🐦 Bird detection
🚗 Moving-car detection with filtering for stationary vehicles
🏍️ Motorcycle/moped detection
🚲 Bicycle and other supported object detection
🔇 Filtering of unwanted AI classes such as bench
🎙️ Sound AI and audio-triggered events
🚫 Visual-motion verification to reduce false Unidentified movement events
🎥 Automatic AI event recording with pre-record buffer
🗂️ Event folders organized by detected class
🖼️ Recording thumbnails and remote playback
💾 Save recordings from the Camera Phone to the Monitor Phone
🗑️ Remote recording management
🔴 Continuous/24-hour recording support
🔊 Live camera audio
🗣️ Two-way talk / talkback
🔔 Built-in doorbell mode
👆 Touch-screen doorbell button
👂 Experimental tap/knock sound doorbell detection
🔔 Doorbell notifications on the Monitor Phone
📺 Doorbell alerts can open directly into live view
🌙 Black low-brightness doorbell screen for unattended use
🔍 Remote digital zoom and pan
🎯 Remote/manual camera focus where supported
📷 Remote camera/lens selection
🔄 Front and rear camera support
📐 Portrait-oriented live viewing
🪞 Front-camera mirror correction
🌡️ Camera-module and phone thermal monitoring
❄️ Automatic thermal/CPU protection
🧠 Full AI detection remains active during Critical thermal mode
🔋 Battery, charging source, current and power monitoring
🌐 Local network and Tailscale remote-access support
📷 QR pairing between Camera and Monitor phones
🎞️ High-resolution recording, depending on device Camera2 capabilities


#How it works

The system uses two Android devices:

Camera Phone
Placed at the entrance and responsible for Camera2 capture, recording, AI inference, motion analysis, sound analysis, doorbell detection and streaming.

Monitor Phone
Used elsewhere to watch the live camera, listen to audio, receive AI/doorbell events, browse recordings, remotely control the camera and communicate with visitors.

The system is intended to keep the expensive camera and AI workload on the dedicated Camera Phone while providing a lightweight remote interface on the Monitor Phone.

Privacy-first design

AI Door Monitor is designed around local processing and user-controlled storage rather than requiring recordings to be uploaded to a commercial doorbell provider.

This makes it useful for people who want to reuse existing Android hardware and maintain more control over their camera recordings.

Project status

AI Door Monitor is an experimental project under active development. Camera capabilities vary considerably between Android devices, manufacturers and individual lenses. Features such as manual focus, ultra-wide camera access, high-resolution recording and thermal telemetry therefore depend on what the device exposes through Android/Camera2.

Current development has primarily targeted Motorola Android hardware.
