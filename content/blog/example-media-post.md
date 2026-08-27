---
title: "Media Example: Images, GIFs & Video in Blog Posts"
date: 2026-08-28
tags: ["web", "idor", "writeup"]
---

This post shows every media type you can embed. Delete it when you no longer need it.

---

## Images

Drop your file into `static/images/` then reference it with `/images/filename.png`.

![Burp Suite capturing the vulnerable request](/images/burp-request.png)

You can add a caption underneath manually:

![Target application login panel](/images/target-login.png)

*Fig 1. The login panel exposed at `/admin/dashboard` without authentication.*

Keep images under ~1MB for fast page loads. PNG for screenshots, JPG for photos.

---

## GIFs

GIFs work exactly the same as images. Drop into `static/images/` and embed:

![Exploit running against the target — IDOR to account takeover](/images/exploit-demo.gif)

GIFs are great for showing:
- Exploit steps in sequence
- Tool output scrolling
- PoC in one loop (keep under 5MB, or use MP4 instead — much smaller file size for the same quality)

---

## Self-Hosted Video (MP4)

Drop your video into `static/videos/` and embed with a raw HTML block:

<video controls width="100%">
  <source src="/videos/demo.mp4" type="video/mp4">
</video>

`controls` adds the play/pause/seek bar. Works on all browsers. Use MP4 (H.264) for broadest compatibility. For screen recordings, [OBS](https://obsproject.com) → record → export as MP4.

---

## YouTube Embed

Use Hugo's built-in shortcode — just the video ID from the URL:

{{< youtube dQw4w9WgXcQ >}}

The ID is the part after `?v=` in the YouTube URL. No API key, no iframe code — Hugo handles it.

---

## Real-World Example Layout

Here's how a proper writeup might flow:

### Discovery

I found the endpoint while intercepting traffic with Burp Suite.

![Burp showing the `/api/v1/user?id=1337` request](/images/burp-discovery.png)

### Exploitation

Changing `id` to another user's value returned their full profile data.

![IDOR — changing id=1337 to id=1338 returning victim data](/images/idor-exploit.png)

Here's the exploit running against a test account:

![GIF of exploit script looping through user IDs](/images/idor-loop.gif)

And a full walkthrough video:

{{< youtube dQw4w9WgXcQ >}}

---

## File Structure Reference

```
static/
  images/
    burp-request.png      ← screenshots
    exploit-demo.gif      ← animated demos
    target-login.png
  videos/
    demo.mp4              ← screen recordings
  exploits/
    cve-2024-40422.py     ← PoC scripts
```

Replace `/images/burp-request.png` and `/videos/demo.mp4` with your actual files and the paths will resolve automatically.
