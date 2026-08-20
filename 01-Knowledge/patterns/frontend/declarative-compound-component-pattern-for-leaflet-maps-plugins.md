---
title: "Declarative Compound Component Pattern for Leaflet Maps & Plugins"
type: pattern
tags: [pattern, frontend, react, leaflet, maps, geospatial, compound-components, ui]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111316881-96cf68d4
sources: ["Harvest 1787111316881 96cf68d4.json"]
---

# Declarative Compound Component Pattern for Leaflet Maps & Plugins

Declarative compound component architecture for React Leaflet map composition, plugin integration, and camera transitions.

## 1. Overview & Architecture

A declarative React compound component pattern wrapping Leaflet and its ecosystem plugins (MarkerCluster, Draw, Tile Switcher), enabling rich geospatial tracking and animated camera transitions in a clean JSX hierarchy.

## 2. Implementation & Code Structure

src/renderer/src/
├── components/ui/
│   └── map.tsx             # Compound Leaflet container encapsulating tiles, draw tools, marker clusters, fullscreen controls
└── pages/Dashboard/
    └── components/
        └── MapSection.tsx  # Declarative usage composing Map, MapLayers, MapTileLayer, FocusMapMarker, and MapMarker

## 3. Key Implementation Points

- Compound component design pattern (<Map>, <MapLayers>, <MapTileLayer>, <MapMarker>) for clean DX and declarative syntax.
- Custom child controller component (FocusMapMarker) utilizing useMap() hook to execute imperative flyTo animations smoothly.
- Integration of Leaflet plugins (MarkerCluster, Draw, Fullscreen, TileLayers) behind modular UI primitives.
- Dynamic custom HTML/Tailwind-styled markers using DivIcon projections.

## 4. Code Examples

### Declarative compound map component composition with dynamic markers, tile switching, and camera fly-to hook

```typescript
// src/renderer/src/components/ui/map.tsx & MapSection.tsx
import React from 'react'
import { useMap } from 'react-leaflet'
import { Map, MapLayers, MapMarker, MapTileLayer } from '@/components/ui/map'
import type { IDeviceLocation } from '../data'

interface IMapSectionProps {
  locations: IDeviceLocation[]
  selectedPosition: [number, number] | null
}

const FocusMapMarker: React.FC<{ position: [number, number] | null }> = ({ position }) => {
  const map = useMap()

  React.useEffect(() => {
    if (!position) return

    map.flyTo(position, Math.max(map.getZoom(), 12), {
      animate: true,
      duration: 0.8,
    })
  }, [map, position])

  return null
}

export const MapSection: React.FC<IMapSectionProps> = ({ locations, selectedPosition }) => {
  const mappedLocations = locations.filter(
    (device): device is IDeviceLocation & { position: [number, number] } => device.position !== null
  )

  return (
    <div className="w-full rounded-[12px] bg-accent p-0">
      <div className="overflow-hidden rounded-[12px] border border-[#334155] h-full">
        <Map center={[0.7893, 113.9213]} zoom={5} minZoom={4} className="h-full min-h-0 w-full rounded-none">
          <MapLayers defaultTileLayer="Street">
            <FocusMapMarker position={selectedPosition} />
            <MapTileLayer
              name="Street"
              url="https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png"
              attribution="&copy; OpenStreetMap contributors, &copy; CARTO"
            />
            <MapTileLayer
              name="Dark"
              url="https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png"
              attribution="&copy; OpenStreetMap contributors, &copy; CARTO"
            />
            {mappedLocations.map((device) => (
              <MapMarker
                key={device.rowKey}
                position={device.position}
                iconAnchor={[44, 12]}
                icon={
                  <span className="inline-flex h-6 min-w-22 items-center justify-center rounded-full bg-primary px-3 text-xs font-semibold text-white shadow-md">
                    {device.id}
                  </span>
                }
              />
            ))}
          </MapLayers>
        </Map>
      </div>
    </div>
  )
}
```

## 5. Considerations & Best Practices

- Dynamic Leaflet tiles and markers must properly handle component unmounting and container resize (invalidateSize).
- Custom marker icons rendered via React elements need to be projected cleanly into Leaflet DivIcons.
- Imperative animations like map.flyTo should be isolated in small child components that consume useMap().

## 6. Related Knowledge

- `React Compound Components`
- `React Leaflet & Leaflet Plugins`
- `Leaflet DivIcon & Camera Animations`

## 7. Source

- Harvest 1787111316881 96cf68d4.json
