---
title: "Hardware Camera MediaStream Bridge with Biometric Focus Overlay & Async Base64 Ingestion"
type: pattern
tags: [pattern, frontend, camera, media-stream, biometrics, hardware, react-webcam]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# Hardware Camera MediaStream Bridge with Biometric Focus Overlay & Async Base64 Ingestion

Hardware camera bridge managing device streams, biometric targeting UI, and base64 snapshot processing.

## 1. Overview & Architecture

Hardware camera media stream bridge using react-webcam with biometric guide overlays and async base64 photo capture pipeline.

## 2. Implementation & Code Structure

src/pages/PersonAccess/components/Dialog/CameraDialog.tsx -> videoConstraints, onUserMedia readiness, biometric focus mask, capturePhoto and regex-based base64 normalization

## 3. Key Implementation Points

- Configures environment/user video constraints and synchronizes readiness via onUserMedia stream event
- Renders concentric biometric radial mask and dashed target circle over the video feed for face alignment
- Strips MIME headers from canvas snapshot base64 URI using regex before payload submission
- Guards modal dismissals during photo processing to prevent aborted capture operations

## 4. Code Examples

### Webcam media stream capture component with biometric guide overlay and base64 sanitization

```typescript
import React, { useCallback, useRef, useState } from "react";
import Webcam from "react-webcam";
import { Dialog, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Camera, Loader2 } from "lucide-react";

interface ICameraDialog {
  open: boolean;
  onClose: () => void;
  onPhotoCapture?: (imageData: string) => void;
}

const videoConstraints = {
  width: 704,
  facingMode: "environment",
};

export const CameraDialog: React.FC<ICameraDialog> = ({ open, onClose, onPhotoCapture }) => {
  const webcamRef = useRef<Webcam | null>(null);
  const [url, setUrl] = useState<string | null>(null);
  const [isProcessing, setIsProcessing] = useState(false);
  const [isCameraReady, setIsCameraReady] = useState(false);

  const capturePhoto = useCallback(() => {
    if (webcamRef.current && !isProcessing) {
      const imageSrc = webcamRef.current.getScreenshot();
      if (imageSrc) {
        setIsProcessing(true);
        const base64Data = imageSrc.replace(/^data:image\/[a-z]+;base64,/, "");

        setTimeout(() => {
          setUrl(base64Data);
          setIsProcessing(false);
          if (onPhotoCapture) onPhotoCapture(base64Data);
        }, 2000);

        setTimeout(() => {
          onClose();
        }, 2500);
      }
    }
  }, [isProcessing, onPhotoCapture, onClose]);

  const onUserMedia = () => {
    setTimeout(() => {
      setIsCameraReady(true);
    }, 650);
  };

  return (
    <Dialog open={open} onOpenChange={(isOpen) => !isOpen && onClose()}>
      <DialogContent className="w-[706px] p-0 gap-0" onInteractOutside={(e) => e.preventDefault()} onEscapeKeyDown={(e) => e.preventDefault()}>
        <DialogHeader className="p-6">
          <DialogTitle className="text-lg font-semibold text-gray-900">Face Recognition Camera</DialogTitle>
        </DialogHeader>
        <div className="flex flex-col items-center relative">
          {!url ? (
            <div className="relative">
              <Webcam ref={webcamRef} audio={false} screenshotFormat="image/jpeg" videoConstraints={videoConstraints} onUserMedia={onUserMedia} className="rounded" />
              {isCameraReady && (
                <div className="absolute inset-0 flex items-center justify-center">
                  <div className="absolute inset-0 bg-black/60 mask-radial" />
                  <div className="w-52 h-52 rounded-full border-2 border-dashed border-white z-10" />
                </div>
              )}
            </div>
          ) : (
            <img src={`data:image/jpeg;base64,${url}`} alt="Captured" className="rounded" />
          )}
          <button className={`p-3.5 m-4 rounded-full text-white ${isProcessing ? "bg-gray-700 cursor-not-allowed" : "bg-red-700 hover:bg-red-800"}`} onClick={capturePhoto} disabled={isProcessing}>
            {isProcessing ? <Loader2 size={24} className="animate-spin" /> : <Camera size={24} />}
          </button>
        </div>
      </DialogContent>
    </Dialog>
  );
};
```

## 5. Considerations & Best Practices

- Handle user media permission denial (NotAllowedError/NotFoundError) with informative UI
- Ensure stream tracks are properly released when modal unmounts to free camera hardware

## 6. Related Knowledge

- `W3C Media Capture and Streams API (navigator.mediaDevices.getUserMedia)`
- `HTML5 Canvas 2D image capture & toDataURL`
- `Biometric face recognition intake workflows`

## 7. Source

- Harvest 1787132260416 42b894f9.json
