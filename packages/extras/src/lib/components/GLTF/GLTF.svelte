<script lang="ts">
  import {
    InteractiveObject,
    LayerableObject,
    Object3DInstance,
    useLoader,
    useThrelte
  } from '@threlte/core'
  import { createEventDispatcher } from 'svelte'
  import { Mesh, Texture, type Material, type Object3D, type SkinnedMesh } from 'three'
  import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader'
  import type { GLTF as ThreeGLTF } from 'three/examples/jsm/loaders/GLTFLoader'
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
  import { KTX2Loader } from 'three/examples/jsm/loaders/KTX2Loader'
  import { buildSceneGraph } from '../../lib/buildSceneGraph'
  import type { GLTFProperties } from '../../types/components'
  import type { ThrelteGltf } from '../../types/types'

  export let position: GLTFProperties['position'] = undefined
  export let scale: GLTFProperties['scale'] = undefined
  export let rotation: GLTFProperties['rotation'] = undefined
  export let viewportAware: GLTFProperties['viewportAware'] = false
  export let inViewport: GLTFProperties['inViewport'] = false
  export let castShadow: GLTFProperties['castShadow'] = undefined
  export let receiveShadow: GLTFProperties['receiveShadow'] = undefined
  export let frustumCulled: GLTFProperties['frustumCulled'] = undefined
  export let renderOrder: GLTFProperties['renderOrder'] = undefined
  export let visible: GLTFProperties['visible'] = undefined
  export let lookAt: GLTFProperties['lookAt'] = undefined

  export let url: GLTFProperties['url']
  export let dracoDecoderPath: GLTFProperties['dracoDecoderPath'] = undefined
  export let ktxTranscoderPath: GLTFProperties['ktxTranscoderPath'] = undefined

  // <InteractiveObject> properties
  export let ignorePointer: GLTFProperties['ignorePointer'] = false
  export let interactive: GLTFProperties['interactive'] = false

  const { invalidate } = useThrelte()

  const dispatch = createEventDispatcher<{
    load: ThreeGLTF
    unload: undefined
    error: string
  }>()

  let interactiveMeshes: (Mesh | SkinnedMesh)[] = []
  let layerableObjects: Object3D[] = []

  export let gltf: ThreeGLTF | undefined = undefined
  export let scene: ThreeGLTF['scene'] | undefined = undefined
  export let animations: ThreeGLTF['animations'] | undefined = undefined
  export let asset: ThreeGLTF['asset'] | undefined = undefined
  export let cameras: ThreeGLTF['cameras'] | undefined = undefined
  export let scenes: ThreeGLTF['scenes'] | undefined = undefined
  export let userData: ThreeGLTF['userData'] | undefined = undefined
  export let parser: ThreeGLTF['parser'] | undefined = undefined
  export let materials: ThrelteGltf['materials'] | undefined = undefined
  export let nodes: ThrelteGltf['nodes'] | undefined = undefined

  const loader = useLoader(GLTFLoader, () => new GLTFLoader())

  if (dracoDecoderPath) {
    const dracoLoader = useLoader(DRACOLoader, () =>
      new DRACOLoader().setDecoderPath(dracoDecoderPath as string)
    )
    loader.setDRACOLoader(dracoLoader)
  }

  const { renderer } = useThrelte()
  if (renderer && ktxTranscoderPath) {
    const ktx2Loader = useLoader(KTX2Loader, () =>
      new KTX2Loader().setTranscoderPath(ktxTranscoderPath as string).detectSupport(renderer)
    )
    loader.setKTX2Loader(ktx2Loader)
  }

  const disposeGltf = () => {
    if (gltf) {
      gltf.scene.traverse((object) => {
        if (object.type !== 'Mesh') return
        const m = object as Mesh

        m.geometry.dispose()

        if (Array.isArray(m.material)) {
          m.material.forEach((mm) => {
            if (mm.isMaterial) {
              disposeMaterial(mm)
            }
          })
        } else if (m.material.isMaterial) {
          disposeMaterial(m.material)
        }
      })
      gltf = undefined
      scene = undefined
      animations = undefined
      asset = undefined
      cameras = undefined
      scenes = undefined
      userData = undefined
      parser = undefined
      nodes = undefined
      materials = undefined

      interactiveMeshes.splice(0, interactiveMeshes.length)
      interactiveMeshes = interactiveMeshes
      layerableObjects.splice(0, layerableObjects.length)
      layerableObjects = layerableObjects

      invalidate('GLTF: model disposed')
      dispatch('unload')
    }
  }

  const disposeMaterial = (material: Material) => {
    material.dispose()
    Object.values(material).forEach((value) => {
      try {
        if (value instanceof Texture) {
          value.dispose()
        }
      } catch (error) {
        console.warn('<GLTF>: Unable to dispose texture.')
      }
    })
  }

  const onLoad = (data: ThreeGLTF) => {
    disposeGltf()
    gltf = data
    scene = gltf.scene
    animations = gltf.animations
    asset = gltf.asset
    cameras = gltf.cameras
    scenes = gltf.scenes
    userData = gltf.userData
    parser = gltf.parser

    const { materials: m, nodes: n } = buildSceneGraph(data.scene)
    materials = m
    nodes = n

    scene.traverse((object) => {
      layerableObjects.push(object)
      if (object.type === 'Mesh' || object.type === 'SkinnedMesh') {
        const mesh = object as Mesh
        interactiveMeshes.push(mesh)
      }
    })
    interactiveMeshes = interactiveMeshes
    layerableObjects = layerableObjects

    invalidate('GLTF: model loaded')
    dispatch('load', gltf)
  }

  const onError = (e: ErrorEvent) => {
    console.error(`Error loading GLTF: ${e.message}`)
    disposeGltf()
    dispatch('error', e.message)
  }

  $: loader.load(url, onLoad, undefined, onError)

  $: {
    if (scene) {
      scene.traverse((obj) => {
        if (castShadow !== undefined) obj.castShadow = castShadow
        if (receiveShadow !== undefined) obj.receiveShadow = receiveShadow
        if (frustumCulled !== undefined) obj.frustumCulled = frustumCulled
        if (renderOrder !== undefined) obj.renderOrder = renderOrder
      })
    }
  }
</script>

{#if scene}
  <Object3DInstance
    object={scene}
    {position}
    {scale}
    {rotation}
    {lookAt}
    {frustumCulled}
    {renderOrder}
    {visible}
    {castShadow}
    {receiveShadow}
    {viewportAware}
    bind:inViewport
    on:viewportenter
    on:viewportleave
  >
    <slot />
  </Object3DInstance>

  {#each interactiveMeshes as mesh}
    {#key mesh.uuid}
      <InteractiveObject
        object={mesh}
        {interactive}
        {ignorePointer}
        on:click
        on:contextmenu
        on:pointerup
        on:pointerdown
        on:pointerenter
        on:pointerleave
        on:pointermove
      />
    {/key}
  {/each}

  {#each layerableObjects as object}
    {#key object.uuid}
      <LayerableObject {object} />
    {/key}
  {/each}
{/if}
