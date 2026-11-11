# 🔋🌙🛑 Background Battery Fix

A plugin for Nagram XF. It stops the app asking for a new screen frame on every display refresh when nothing is moving, including while it sits in the background. That background CPU use is what makes Android close Nagram XF for "excessive CPU".

## Why I made it

Android kept closing Nagram XF on my Pixel 8 Pro. The app exit log said excessive CPU: 43.6 seconds of CPU in 5 minutes while in the background. Profiling the main thread showed most of it was a frame loop that never stops.

## The cause

`Choreographer60FpsContent` (Nagram XF's 60 fps animation helper) posts a new frame callback on every vsync for the life of the app, even when no animation is registered and nothing needs a redraw, and even with the app in the background. On a 120 Hz screen that wakes the main thread 120 times a second, all day.

## What the plugin does

After each frame it checks whether the helper has any callback or anything to redraw. If not, it stops asking for frames. The moment any animation or redraw is added, it asks for the next frame again, timed so that frame runs at once. Animations look and run exactly as before.

## Performance

The check runs in Java (an embedded dex hooked with the app's own Xposed bridge), not Python, so the frame path stays fast. No timers, no polling, no logging on the frame path. Logcat tag `XFIdle` for install and errors.

## Measured

Pixel 6 Pro, Nagram XF 12.10.1 plugin build, my real account, app in the background with the screen on, main thread CPU over 60 seconds:

* Before: 320 ticks (3.2 seconds of CPU a minute)
* After: 8 ticks (0.08 seconds a minute)

After coming back to the app, chats, typing indicators and animations ran normally.

## Install

1. Use the plugin build of Nagram XF and turn on the plugin engine (Settings, N Settings, Plugins).
2. Download `idle_frames.plugin` from the latest release.
3. Send it to your Saved Messages, tap it and choose install.

Switching the plugin off removes the hooks and restarts the frame loop as stock.

## License

MIT

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=firestarter1996/xf-background-battery-fix&type=Date)](https://star-history.com/#firestarter1996/xf-background-battery-fix&Date)
