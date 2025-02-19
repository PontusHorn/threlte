---
title: PerspectiveCamera
---

!!!module_summary title=PerspectiveCamera|sourcePath=cameras/PerspectiveCamera.svelte|name=PerspectiveCamera|from=core|type=component
A camera that uses perspective projection.

This projection mode is designed to mimic the way the human eye sees. It is the most common projection mode used for rendering a 3D scene.
!!!

### Example

```svelte
<script>
  import { PerspectiveCamera } from '@threlte/core'
</script>

<PerspectiveCamera fov={60} position={{ x: 3, y: 3, z: 3 }} lookAt={{ x: 0, y: 0, z: 0 }} />
```

### Properties

```ts
// optional
position: Position | undefined = undefined
scale: Scale | undefined = undefined
rotation: Rotation | undefined = undefined
lookAt: LookAt | undefined = undefined
viewportAware: boolean = false
castShadow: boolean | undefined = undefined
receiveShadow: boolean | undefined = undefined
frustumCulled: boolean | undefined = undefined
renderOrder: number | undefined = undefined
visible: boolean | undefined = undefined
useCamera: boolean = true
near: number = undefined
far: number = undefined
fov: number = undefined
```

### Bindings

```ts
inViewport: boolean
camera: THREE.PerspectiveCamera
```

### Events

```ts
viewportenter: undefined
viewportleave: undefined
```
