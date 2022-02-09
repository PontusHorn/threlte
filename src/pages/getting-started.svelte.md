# Getting Started

## Installation

Install threlte and three.js:

```bash
npm install threlte three
```

For TypeScript users, install three.js types:

```bash
npm install -D @types/three
```

## Configuration

If you are using threlte with SvelteKit, you need to adapt your `svelte.config.js` to prevent three.js from being externalized for SSR by [vites externalization step](https://vitejs.dev/guide/ssr.html#ssr-externals):

```js
const config = {
  // …
  kit: {
    // …
    vite: {
      ssr: {
        noExternal: ['three']
      }
    }
  }
}

export default config
```

> :zap: [See this issue](https://github.com/grischaerbe/threlte/issues/8#issuecomment-1024085864) for tips on how to reduce bundle size when working with vite and three.js.

## Your first scene

[Open in a Svelte REPL](https://svelte.dev/repl/14f38c03710945b797d0c421f55e4373?version=3.46.2)

```svelte
<script>
  import { CircleBufferGeometry, MeshStandardMaterial, BoxBufferGeometry, DoubleSide } from 'three'
  import {
    Canvas,
    DirectionalLight,
    HemisphereLight,
    Mesh,
    OrbitControls,
    PerspectiveCamera
  } from 'threlte'
</script>

<div>
  <Canvas>
    <PerspectiveCamera position={{ x: 10, y: 10, z: 10 }}>
      <OrbitControls autoRotate />
    </PerspectiveCamera>

    <DirectionalLight shadow color={'white'} position={{ x: -15, y: 45, z: 20 }} />
    <HemisphereLight skyColor={'white'} groundColor={'#ac844c'} intensity={0.4} />

    <Mesh
      castShadow
      geometry={new BoxBufferGeometry(1, 1, 1)}
      material={new MeshStandardMaterial({ color: '#ff3e00' })}
    />

    <Mesh
      receiveShadow
      position={{ y: -0.5 }}
      rotation={{ x: -90 * (Math.PI / 180) }}
      geometry={new CircleBufferGeometry(3, 72)}
      material={new MeshStandardMaterial({ side: DoubleSide, color: 'white' })}
    />
  </Canvas>
</div>

<style>
  div {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
</style>
```

<br />

It should look something like this:

![getting-started-screenshot](https://raw.githubusercontent.com/grischaerbe/threlte/main/static/new-getting-started-screenshot.png)

Congratulations :tada:  
Orbit around the cube and have fun using threlte.

Have a look at the slightly more elaborate example including interactivity in a [Svelte REPL](https://svelte.dev/repl/bcb9474112ca440cb3c1f67e74250bcf?version=3.46.2).
