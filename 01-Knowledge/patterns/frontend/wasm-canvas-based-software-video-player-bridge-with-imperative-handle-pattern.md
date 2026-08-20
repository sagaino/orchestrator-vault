---
title: "WASM/Canvas-Based Software Video Player Bridge with Imperative Handle Pattern"
type: pattern
tags: [pattern, frontend, jsmpeg, wasm, canvas, video-player, cctv]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# WASM/Canvas-Based Software Video Player Bridge with Imperative Handle Pattern

WASM and canvas-based software video player bridge with imperative lifecycle control for real-time camera streams.

## 1. Overview & Architecture

Software-decoded video player bridge converting WebSocket MPEG-1 streams onto HTML5 canvas with an imperative ref handle pattern.

## 2. Implementation & Code Structure

src/components/common/Video/jsmpegPlayer.tsx & jsmpeg-player.d.ts -> Ambient type declaration, JSMpeg.VideoElement wrapper, onVideoDecode state hook, onRef lifecycle bridge, canvas layout styling

## 3. Key Implementation Points

- Defines ambient TypeScript declarations for the untyped @cycjimmy/jsmpeg-player module
- Encapsulates imperative canvas video decoder instance inside a reactive React lifecycle container
- Exposes imperative player methods (play, pause, stop, destroy) via an onRef callback interface
- Tracks decode frames via onVideoDecode to control loading spinners and fullscreen expand actions

## 4. Code Examples

### React wrapper component for JSMpeg software video player rendering on HTML5 canvas with imperative ref bridge

```typescript
import React, { useEffect, useRef, useState } from "react";
import JSMpeg from "@cycjimmy/jsmpeg-player";
import { Loader2, Video, Expand } from "lucide-react";

export interface JsmpegPlayerRef {
  play: () => void;
  pause: () => void;
  stop: () => void;
  destroy: () => void;
}

interface JsmpegPlayerProps {
  videoUrl: string;
  options?: object;
  overlayOptions?: object;
  onRef?: (ref: JsmpegPlayerRef) => void;
  cameraName?: string;
  onExpand?: () => void;
}

export const JsmpegPlayer: React.FC<JsmpegPlayerProps> = ({ videoUrl, options, overlayOptions, onRef, cameraName, onExpand }) => {
  const videoWrapperRef = useRef<HTMLDivElement | null>(null);
  const [playing, setPlaying] = useState<boolean>(false);

  useEffect(() => {
    const wrapper = videoWrapperRef.current;
    if (!wrapper) return;

    wrapper.innerHTML = "";
    const newVideo = new JSMpeg.VideoElement(
      wrapper,
      videoUrl,
      { ...options, control: false },
      {
        ...overlayOptions,
        onVideoDecode: (decoded: boolean) => setPlaying(Boolean(decoded)),
      }
    );

    if (onRef) {
      onRef({
        play: () => newVideo.play(),
        pause: () => newVideo.pause(),
        stop: () => newVideo.stop(),
        destroy: () => newVideo.destroy(),
      });
    }

    const canvas = wrapper.querySelector("canvas");
    if (canvas) {
      canvas.style.width = "100%";
      canvas.style.height = "100%";
      canvas.style.display = "block";
      canvas.style.objectFit = "contain";
    }

    return () => {
      try {
        setPlaying(false);
        newVideo.destroy();
      } catch (error) {
        console.error("Error destroying JSMpeg video:", error);
      }
    };
  }, [videoUrl, options, overlayOptions, onRef]);

  return (
    <div className="relative w-full">
      <div ref={videoWrapperRef} className="w-full h-full" />
      {!playing && (
        <div className="absolute inset-0 flex justify-center items-center bg-black/20">
          <Loader2 className="animate-spin text-white" />
        </div>
      )}
      {cameraName && (
        <div className="absolute bottom-1 left-1 flex items-center gap-1 text-white ml-3">
          <Video size={16} />
          <p className="text-sm">{cameraName}</p>
        </div>
      )}
      {playing && onExpand && (
        <button className="cursor-pointer absolute bottom-1 right-1 bg-white/90 rounded p-1 mr-3" onClick={onExpand}>
          <Expand size={16} />
        </button>
      )}
    </div>
  );
};
```

## 5. Considerations & Best Practices

- Software decoding on CPU/WASM consumes higher resources than native GPU hardware decoding
- Ensure canvas context and memory are destroyed when video stream changes or component unmounts

## 6. Related Knowledge

- `JSMpeg WebAssembly & WebGL/Canvas Video Decoder`
- `React Imperative Handle & Ref Bridge Pattern`
- `HTML5 Canvas DOM Manipulation & Memory Management`

## 7. Source

- Harvest 1787132260416 42b894f9.json
