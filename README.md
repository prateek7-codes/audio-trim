# MediaShear — Browser-Based Media Editor

A professional, privacy-first audio/video editor that runs entirely in the browser.  
**No server. No uploads. Zero data leaves your device.**

---

## ⚡ Quick Start (Standalone)

Open `mediashear.html` in any modern browser (Chrome 90+, Firefox 88+, Safari 15+).  
That's it — no installation needed.

---

## 🏗 React Project Structure

```
mediashear/
├── public/
│   ├── index.html              Main HTML shell
│   └── mediashear.html         ← Standalone version (ships in this output)
├── src/
│   ├── App.js                  Root component, state management
│   ├── index.js                React entry point
│   ├── components/
│   │   ├── Upload/
│   │   │   ├── UploadZone.js   Drag-and-drop file ingestion
│   │   │   └── ClipList.js     Left sidebar clip panel
│   │   ├── Timeline/
│   │   │   ├── TimelineEditor.js   Core timeline with trim handles, ruler, zoom
│   │   │   └── WaveformTrack.js    Canvas waveform renderer per clip
│   │   ├── Player/
│   │   │   └── PreviewPlayer.js    HTML5 playback with seekbar
│   │   ├── Effects/
│   │   │   └── EffectsPanel.js     Per-clip trim/fade controls
│   │   ├── Export/
│   │   │   └── ExportPanel.js      Format/bitrate/resolution + FFmpeg export
│   │   ├── Toolbar.js
│   │   └── StatusBar.js
│   ├── hooks/
│   │   ├── useFFmpeg.js        Lazy FFmpeg loader + processClips()
│   │   └── useHistory.js       Undo/redo stack
│   └── utils/
│       └── time.js             fmtDur, toFFmpegTime, clamp, snapToGrid
└── package.json
```

## 📦 Installation (React version)

```bash
cd mediashear
npm install --legacy-peer-deps
npm start            # Dev server at http://localhost:3000
npm run build        # Production build in /build
```

---

## 🎛 Core Features

| Feature | Detail |
|---|---|
| **Formats** | MP3, WAV, M4A, AAC, FLAC, MP4, MOV, WEBM, MKV |
| **Timeline** | Zoomable (0.3×–20×), snapping, ms-precision handles |
| **Trim** | Drag handles on timeline OR numeric inputs (0.001s precision) |
| **Fades** | Fade-in / fade-out with live SVG waveform preview |
| **Merge** | Drop multiple files, reorder by drag-and-drop |
| **Undo/Redo** | 50-step history (Ctrl+Z / Ctrl+Y) |
| **Export** | FFmpeg.wasm — MP3/WAV/AAC/M4A/MP4/WEBM |
| **Privacy** | 100% local, zero network requests for media |

---

## ⌨️ Keyboard Shortcuts

| Key | Action |
|---|---|
| `Space` | Play / Pause |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` or `Ctrl+Shift+Z` | Redo |
| `+` / `-` | Zoom timeline in/out |
| `Delete` / `Backspace` | Remove selected clip |

---

## 🔧 FFmpeg Command Examples

### Trim audio (single clip)
```
ffmpeg -i input.mp3 -af "atrim=start=5.0:end=30.0,asetpts=PTS-STARTPTS" -codec:a libmp3lame -b:a 192k output.mp3
```

### Fade in + out
```
ffmpeg -i input.mp3 -af "atrim=start=0:end=60,asetpts=PTS-STARTPTS,afade=t=in:st=0:d=2,afade=t=out:st=58:d=2" output.mp3
```

### Merge multiple clips (concat demuxer)
```
# concat.txt:
# file 'clip1.wav'
# file 'clip2.wav'

ffmpeg -f concat -safe 0 -i concat.txt -codec:a libmp3lame -b:a 192k merged.mp3
```

### Trim video
```
ffmpeg -i input.mp4 -ss 00:00:05.000 -to 00:00:30.000 -c:v libx264 -preset fast -crf 23 -c:a aac -b:a 192k output.mp4
```

### Format conversion
```
ffmpeg -i input.wav -codec:a libmp3lame -b:a 256k output.mp3
ffmpeg -i input.mp3 -codec:a pcm_s16le output.wav
ffmpeg -i input.mp4 -c:v libx264 -c:a aac output.mp4
```

### Video resolution
```
ffmpeg -i input.mp4 -vf scale=1280:720 -c:v libx264 output.mp4
```

---

## 🏗 Architecture Notes

- **FFmpeg.wasm** is loaded lazily (only on first export click or manual preload)
- **Waveform rendering** uses the Web Audio API to decode audio, then paints to a `<canvas>` — results are cached per clip+trim
- **State** is a single flat JS object; all mutations go through `updateClip()` which triggers a full re-render
- **Undo/Redo** serialises trim/fade/name fields (not File objects) into a 50-entry ring buffer
- **Timeline drag** uses raw DOM `mousemove`/`mouseup` listeners for maximum performance (no React re-renders during drag)
