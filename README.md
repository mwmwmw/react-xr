# react-xr

[![npm version](https://badge.fury.io/js/react-xr.svg)](https://badge.fury.io/js/react-xr) ![npm](https://img.shields.io/npm/dt/react-xr.svg)

WebXR + react-three-fiber

React components and hooks for creating VR/AR/XR applications with [react-three-fiber](https://github.com/react-spring/react-three-fiber)

**Note: Extremely early in development. Contributors welcome!**

<p align="center">
  <a href="https://codesandbox.io/s/react-xr-paddle-demo-v4uet"><img width="288" src="https://i.imgur.com/K71D3Ts.gif" /></a>
  <a href="https://codesandbox.io/s/react-xr-simple-demo-8i9ro"><img width="288" src="https://i.imgur.com/5yh7LKz.gif" /></a>
</p>
<p align="middle">
  <i>These demos are real, you can click them! They contain the full code, too.</i>
</p>

## Installation

```
npm install react-xr
```

## Getting started

Add `XRCanvas` component (or replace your existing react-three-fiber `Canvas` component)

```js
import { XRCanvas } from 'react-xr'

function App() {
  return <XRCanvas>{/* All the stuff goes here */}</XRCanvas>
}
```

## Adding controllers to the scene

To get started with default controller models add `DefaultXRControllers` component. It will fetch appropriate input profile models. You can learn more [here](https://github.com/immersive-web/webxr-input-profiles/tree/master/packages/motion-controllers).

```js
import { XRCanvas, DefaultXRControllers } from 'react-xr'

function App() {
  return (
    <XRCanvas>
      <DefaultXRControllers />
    </XRCanvas>
  )
}
```

You can access controllers' state (position, orientation, etc.) by using `useXR()` hook

```js
const { controllers } = useXR()
```

## API

### XRCanvas

Extended react-three-fiber [Canvas](https://github.com/react-spring/react-three-fiber/blob/master/api.md#canvas) that includes:

- Button to start VR session
- Color management
- VR Mode
- react-xr context

```js
import { XRCanvas } from 'react-xr'

function App() {
  return <XRCanvas>{/* All the stuff goes here */}</XRCanvas>
}
```

### useXR

Hook that can only beused by components insde <XRCanvas> component.

```js
const { controllers } = useXR()
```

Controllers is an array of `XRController` objects

```js
interface XRController {
  grip: Group
  controller: Group
  inputSource?: XRInputSource
  // ...
  // more in XRController.ts
}
```

`grip` and `controller` are ThreeJS groups that have the position and orientation of xr controllers. `grip` has an orientation that should be used to render virtual objects such that they appear to be held in the user’s hand and `controller` has an orientation of the preferred pointing ray.

<img width="200" height="200" src="https://i.imgur.com/3stLjfR.jpg" />

`inputSource` is the WebXR input source [(MDN)](https://developer.mozilla.org/en-US/docs/Web/API/XRInputSource). Note that it will not be available before controller is connected.

### useXREvent

Every controller emits following events: select, selectstart, selectend, squeeze, squeezestart, squeezeend.

To listen to those events use `useXREvent` hook:

```js
const onSqueeze = useCallback(() => console.log('Squeezed'), [])

useXREvent('squeeze', onSqueeze)
```

it supports optional third parameter with options

```js
const onSqueeze = useCallback(() => console.log('Left controller squeeze'), [])

useXREvent('squeeze', onSqueeze, { handedness: 'left' })
```

### Interactions

`react-xr` comes with built-in high level interaction components.

#### `<Hover>`

`Hover` component will allow you for detecting when ray shot from the controllers is pointing at the given mesh.

```js
<Hover onChange={(value) => console.log(value ? 'hovered' : 'blurred')}>{/* your mesh here */}</Hover>
```

#### `<Select>`

`Select` can be used when you need to select some mesh. Component will trigger `onSelect` function when controller is pointing at the given mesh and `select` event was fired.

```js
<Select onSelect={() => console.log('selected')}>{/* your mesh here */}</Select>
```

## Getting the VR Camera (HMD) Location

To get the position of the VR camera, use three's WebXRManager instance. 

```
useFrame(({gl, camera})=>{
const cam = gl.xr.isPresenting ? gl.xr.getCamera(camera) : camera;
myObj.lookAt(cam.position) 
});
```

## Parent VR HMD and Controllers to another object

If you want to attach the user to an object so it can be moved around, just parent the VR camera and controllers to an object3D.
```
  const mesh = useRef()
  const { gl, camera } = useThree()
  
  useEffect(() => {
    const cam = gl.xr.isPresenting ? gl.xr.getCamera(camera) : camera
    mesh.current.add(cam)
  }, [gl.xr.isPresenting, gl.xr, camera, mesh])
  
  const { controllers } = useXR()
 // bundle add the controllers to the same object as the camera so it all stays together.
  useEffect(() => {
    if (controllers.length > 0) {
      controllers.forEach(c => mesh.current.add(c.grip))
    }
  }, [controllers, mesh])
```
