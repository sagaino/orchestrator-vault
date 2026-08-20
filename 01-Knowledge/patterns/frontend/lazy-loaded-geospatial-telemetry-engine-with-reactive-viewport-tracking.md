---
title: "lazy-loaded-geospatial-telemetry-engine-with-reactive-viewport-tracking"
type: pattern
tags: [pattern, frontend, geospatial, leaflet, telemetry, device-tracking, react-leaflet, maps]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# lazy-loaded-geospatial-telemetry-engine-with-reactive-viewport-tracking

Lazy-loaded geospatial telemetry engine with dynamic React DivIcon badges and reactive viewport tracking.

## 1. Overview & Architecture

A reactive, lazy-loaded geospatial map visualization engine built on Leaflet and React-Leaflet that converts live GPS sensor telemetry from field identity devices into clustered, interactive, theme-aware map layers with smooth viewport transitions.

## 2. Implementation & Code Structure

- src/renderer/src/components/ui/map.tsx (SSR/bundler safe lazy Leaflet wrapper, DivIcon renderer, theme-aware tiles, clustering)
- src/renderer/src/pages/Dashboard/components/MapSection.tsx (Device GPS telemetry integration and reactive camera flyTo controller)

## 3. Key Implementation Points

- Lazy dynamic component loading factory ensuring Leaflet DOM instances only instantiate after component mount.
- React DOM SSR server rendering (renderToString) to generate dynamic Tailwind-styled HTML markers inside Leaflet DivIcon.
- Theme-reactive tile layer switcher (CARTO Light/Dark) integrated with next-themes.
- Imperative camera trajectory controller (FocusMapMarker) triggering animated flyTo upon coordinate updates.

## 4. Code Examples

### SSR-safe dynamic Leaflet component factory and DOM-rendered HTML marker icon bridge

```tsx
// src/renderer/src/components/ui/map.tsx
function createLazyComponent<T extends ComponentType<any>>(
    factory: () => Promise<{ default: T }>
) {
    const LazyComponent = lazy(factory)

    return (props: React.ComponentProps<T>) => {
        const [isMounted, setIsMounted] = useState(false)

        useEffect(() => {
            setIsMounted(true)
        }, [])

        if (!isMounted) {
            return null
        }

        return (
            <Suspense>
                <LazyComponent {...props} />
            </Suspense>
        )
    }
}

const LeafletMapContainer = createLazyComponent(() =>
    import("react-leaflet").then((mod) => ({
        default: mod.MapContainer,
    }))
)

function MapMarker({
    icon = <MapPinIcon className="size-6" />,
    iconAnchor = [12, 12],
    bgPos,
    popupAnchor,
    tooltipAnchor,
    ...props
}: Omit<MarkerProps, "icon"> &
    Pick<DivIconOptions, "iconAnchor" | "bgPos" | "popupAnchor" | "tooltipAnchor"> & {
        icon?: ReactNode
        ref?: Ref<Marker>
    }) {
    const { L } = useLeaflet()
    if (!L) return null

    return (
        <LeafletMarker
            icon={L.divIcon({
                html: renderToString(icon),
                iconAnchor,
                ...(bgPos ? { bgPos } : {}),
                ...(popupAnchor ? { popupAnchor } : {}),
                ...(tooltipAnchor ? { tooltipAnchor } : {}),
            })}
            riseOnHover
            {...props}
        />
    )
}
```

### Reactive geospatial tracking view with animated camera focus and custom telemetry status badges

```tsx
// src/renderer/src/pages/Dashboard/components/MapSection.tsx
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

const MapSection: React.FC<IMapSectionProps> = ({ locations, selectedPosition }) => {
  const mappedLocations = locations.filter(
    (device): device is IDeviceLocation & { position: [number, number] } => device.position !== null
  )

  return (
    <div className="w-full rounded-[12px] bg-accent p-0">
      <div className="overflow-hidden rounded-[12px] border border-[#334155] h-full">
        <Map center={[0.7893, 113.9213]} zoom={5} minZoom={4} className="h-full min-h-0 w-full rounded-none">
          <MapLayers defaultTileLayer="Street">
            <FocusMapMarker position={selectedPosition} />
            <MapTileLayer name="Street" url="https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}.png" />
            <MapTileLayer name="Dark" url="https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}.png" />
            {mappedLocations.map((device) => (
              <MapMarker
                key={device.rowKey}
                position={device.position}
                iconAnchor={[44, 12]}
                icon={
                  <span className="inline-flex h-6 min-w-22 items-center justify-center whitespace-nowrap rounded-full bg-primary px-3 py-0.5 text-xs font-semibold text-white shadow-md">
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

- Leaflet requires window/document access; lazy mounting with createLazyComponent avoids bundler errors and ensures clean mount cycles.
- Using renderToString with L.divIcon allows rich React/Tailwind component rendering inside native Leaflet canvas markers.
- Smooth camera navigation (map.flyTo) enhances UX when inspecting specific field devices from list tables.

## 6. Related Knowledge

- `Leaflet & React-Leaflet Component Architecture`
- `Dynamic React-to-DOM Marker Icon Generation (L.divIcon + renderToString)`
- `Geospatial Telemetry Visualizations (GPS/LongLat coordinates)`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
