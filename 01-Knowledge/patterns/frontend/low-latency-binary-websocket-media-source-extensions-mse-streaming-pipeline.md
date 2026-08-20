---
title: "Low-Latency Binary WebSocket Media Source Extensions (MSE) Streaming Pipeline"
type: pattern
tags: [pattern, frontend, video-streaming, mse, websocket, cctv, media-source]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# Low-Latency Binary WebSocket Media Source Extensions (MSE) Streaming Pipeline

MSE and binary WebSocket streaming pipeline for ultra-low latency real-time CCTV surveillance feeds.

## 1. Overview & Architecture

Low-latency real-time video streaming architecture utilizing W3C Media Source Extensions and binary WebSockets for CCTV camera feeds.

## 2. Implementation & Code Structure

src/components/common/Video/videoMSE.tsx -> MediaSource creation, WebSocket arraybuffer binding, header codec decoding (byte 9), FIFO queue drain on updateend, drift compensation

## 3. Key Implementation Points

- Initializes MediaSource object URL and establishes raw binary WebSocket connection
- Decodes packet type 9 as UTF-8 MIME codec descriptor to initialize SourceBuffer dynamically
- Buffers binary video chunks in a FIFO queue and appends sequentially on updateend events
- Corrects playback timestamp drift when tab is hidden to maintain real-time low-latency CCTV playback

## 4. Code Examples

### Media Source Extensions (MSE) pipeline receiving binary WebSocket CCTV stream with packet queue and codec negotiation

```typescript
import React, { useEffect, useRef, useState } from "react";

interface IPropsVideoMSE {
  localCamera?: { ws?: string; cctvName?: string };
}

export const VideoMSE: React.FC<IPropsVideoMSE> = ({ localCamera }) => {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    if (!localCamera?.ws) return;

    let mseQueue: Uint8Array[] = [];
    let mseSourceBuffer: SourceBuffer | null = null;
    let mseStreamingStarted = false;
    let ws: WebSocket | null = null;

    const pushPacket = () => {
      const videoEl = videoRef.current;
      if (!videoEl || !mseSourceBuffer) return;

      if (!mseSourceBuffer.updating) {
        if (mseQueue.length > 0) {
          const packet = mseQueue.shift();
          if (packet) mseSourceBuffer.appendBuffer(packet);
        } else {
          mseStreamingStarted = false;
        }
      }

      if (videoEl.buffered.length > 0 && typeof document.hidden !== "undefined" && document.hidden) {
        videoEl.currentTime = videoEl.buffered.end(videoEl.buffered.length - 1) - 0.5;
      }
    };

    const readPacket = (packet: Uint8Array) => {
      if (!mseStreamingStarted && mseSourceBuffer) {
        mseSourceBuffer.appendBuffer(packet);
        mseStreamingStarted = true;
        return;
      }
      mseQueue.push(packet);
      if (mseSourceBuffer && !mseSourceBuffer.updating) {
        pushPacket();
      }
    };

    const startPlay = (videoEl: HTMLVideoElement, url: string) => {
      const mse = new MediaSource();
      videoEl.src = window.URL.createObjectURL(mse);

      mse.addEventListener("sourceopen", () => {
        ws = new WebSocket(url);
        ws.binaryType = "arraybuffer";
        ws.onopen = () => setLoading(true);

        ws.onmessage = (event) => {
          setLoading(false);
          const data = new Uint8Array(event.data);

          if (data[0] === 9) {
            const decodedArr = data.slice(1);
            const mimeCodec = window.TextDecoder ? new TextDecoder("utf-8").decode(decodedArr) : "avc1.640029";
            mseSourceBuffer = mse.addSourceBuffer(`video/mp4; codecs="${mimeCodec}"`);
            mseSourceBuffer.mode = "segments";
            mseSourceBuffer.addEventListener("updateend", pushPacket);
          } else {
            readPacket(data);
          }
        };
      }, false);
    };

    const videoEl = videoRef.current;
    if (videoEl) startPlay(videoEl, localCamera.ws);

    return () => {
      if (ws) ws.close();
    };
  }, [localCamera?.ws]);

  return (
    <div className="relative w-full h-full bg-black rounded-lg" style={{ aspectRatio: "16/9" }}>
      <video ref={videoRef} autoPlay muted playsInline className="w-full h-full object-contain" />
    </div>
  );
};
```

## 5. Considerations & Best Practices

- Manage buffer memory limits with periodic SourceBuffer.remove for long-running continuous streams
- Handle browser codec support variability between H.264 (AVC) and H.265 (HEVC)

## 6. Related Knowledge

- `W3C Media Source Extensions (MSE) Specification`
- `WebSocket Binary Protocols (ArrayBuffer / Uint8Array)`
- `Fragmented MP4 (fMP4) and ISOBMFF streaming`

## 7. Source

- Harvest 1787132260416 42b894f9.json
