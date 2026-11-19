# ADR-064: Calls ring on the desktop helper; WhatsApp calling is out of mobile release 1

Status: proposed (G1, module `apps`).

## Context
- Calls use WebRTC in the page (`frontend/src/stores/calling.ts`, `internal/calling`).
- The server routes a call only to users online on the hub (`IsUserOnline`).
- iOS needs CallKit and PushKit VoIP push for background calls. Each VoIP push must report a call to CallKit. WebView audio stops in the background.
- Android needs a foreground service and a full-screen intent permission for ringing.

## Decision
- Desktop: a "Ready for calls" toggle keeps the WebView loaded while hidden, so the user stays online and the call rings. The helper plays the user's `call_ringtone`, shows a native notification and brings the window up. Answering happens in the page.
- With the toggle off, the helper unloads the WebView to save memory. The user then gets message notifications only.
- Linux: the helper rings, then opens the call in the default browser. WebKitGTK WebRTC is not reliable.
- Mobile release 1: no calling. Phones get a missed-call push.

## Alternatives
- Native WebRTC with CallKit and ConnectionService in release 1. Rejected: large native code, heavier apps, long store review, and IVR and transfer flows to port.
- Answer calls in the phone WebView while open. Rejected: drops when the screen locks; poor call quality signals for store review.

## Consequences
- Agents who take calls keep the desktop helper or a browser tab open.
- Mobile calling becomes a release 2 candidate with its own pack.
