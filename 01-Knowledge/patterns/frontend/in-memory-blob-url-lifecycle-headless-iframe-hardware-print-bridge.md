---
title: "In-Memory Blob URL Lifecycle & Headless IFrame Hardware Print Bridge"
type: pattern
tags: [pattern, frontend, blob-url, printing, pdf, hardware-printer, memory-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# In-Memory Blob URL Lifecycle & Headless IFrame Hardware Print Bridge

In-memory Blob URL lifecycle management and isolated iframe hardware printer bridge.

## 1. Overview & Architecture

In-memory Blob URL lifecycle and hardware print controller pattern using an isolated iframe bridge.

## 2. Implementation & Code Structure

src/pages/Inventory/components/BarcodeModal.tsx -> useBarcodeQuery binary fetch, Blob URL generation & revocation cleanup, iframe embedding, iframe.contentWindow.print triggering

## 3. Key Implementation Points

- Transforms raw ArrayBuffer/binary server responses into MIME typed application/pdf Blobs
- Allocates in-memory DOM URLs via URL.createObjectURL and deterministically revokes via URL.revokeObjectURL
- Isolates document printing into an iframe and invokes contentWindow.print without affecting main window
- Protects modal dismissal during active hardware print spooling via state guards

## 4. Code Examples

### PDF preview modal converting binary data to transient Blob URL and invoking OS print subsystem via isolated iframe

```typescript
import React, { useEffect, useState, useRef } from "react";
import { Printer, X } from "lucide-react";
import { Button } from "@/components/ui/button";
import { useBarcodeQuery } from "../hooks/useBarcodeQuery";

interface BarcodeModalProps {
  isOpen: boolean;
  onClose: () => void;
  assetId: string;
  assetName: string;
}

export const BarcodeModal: React.FC<BarcodeModalProps> = ({ isOpen, onClose, assetId, assetName }) => {
  const [barcodeUrl, setBarcodeUrl] = useState<string | null>(null);
  const [isPrinting, setIsPrinting] = useState(false);
  const iframeRef = useRef<HTMLIFrameElement>(null);

  const { data: barcodeData, isLoading } = useBarcodeQuery(assetId, isOpen);

  useEffect(() => {
    if (barcodeData) {
      const blob = new Blob([barcodeData], { type: "application/pdf" });
      const url = URL.createObjectURL(blob);
      setBarcodeUrl(url);

      return () => {
        if (url) {
          URL.revokeObjectURL(url);
        }
      };
    }
  }, [barcodeData]);

  useEffect(() => {
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === "Escape" && isOpen && !isPrinting) {
        onClose();
      }
    };

    if (isOpen) {
      document.addEventListener("keydown", handleEscape);
      document.body.style.overflow = "hidden";
    }

    return () => {
      document.removeEventListener("keydown", handleEscape);
      document.body.style.overflow = "unset";
    };
  }, [isOpen, onClose, isPrinting]);

  const handlePrint = () => {
    if (iframeRef.current) {
      setIsPrinting(true);
      try {
        const iframe = iframeRef.current;
        iframe.contentWindow?.focus();
        iframe.contentWindow?.print();
      } catch (error) {
        console.error("Hardware print trigger failed:", error);
      } finally {
        setTimeout(() => setIsPrinting(false), 500);
      }
    }
  };

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4">
      <div className="bg-white rounded-lg p-6 max-w-lg w-full">
        <h3 className="text-lg font-bold mb-4">Barcode Preview - {assetName}</h3>
        {barcodeUrl && (
          <iframe ref={iframeRef} src={barcodeUrl} title="Barcode PDF" className="w-full h-80 border rounded mb-4" />
        )}
        <div className="flex justify-end gap-2">
          <Button variant="outline" onClick={onClose} disabled={isPrinting}>Close</Button>
          <Button onClick={handlePrint} disabled={isLoading || isPrinting || !barcodeUrl}>
            <Printer className="mr-2 h-4 w-4" /> Print Barcode
          </Button>
        </div>
      </div>
    </div>
  );
};
```

## 5. Considerations & Best Practices

- Ensure URL.revokeObjectURL is always executed in cleanup to prevent C++ runtime memory leaks
- Be aware of cross-origin constraints when attempting to access iframe.contentWindow across origins

## 6. Related Knowledge

- `W3C File API & URL Object Lifecycles`
- `Browser Hardware Print Subsystem (window.print / iframe.print)`
- `Binary Stream & ArrayBuffer handling in JavaScript`

## 7. Source

- Harvest 1787132260416 42b894f9.json
