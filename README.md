Wagner
======

Effects composer for three.js

This is a WIP version of a new proposal for an effect composer for three.js

Please use it only for review and test purposes. Don't hesitate to add issues or open a conversation about design decisions.

Basic usage
----------

```js
/*
Include the libs
<script src="ShaderLoader.js"></script>
<script src="Wagner.js"></script>
*/

var shaderLoader = new ShaderLoader();
shaderLoader.add( 'orto-vs', 'orto-vs.glsl' );
shaderLoader.add( 'copy-fs', 'copy-fs.glsl' );

var copyShader = new THREE.ShaderMaterial( {
    uniforms: {
		tDiffuse: { type: 't', value: new THREE.Texture() },
		resolution: { type: 'v2', value: new THREE.Vector2( 1, 1 ) }
	},
	vertexShader: shaderLoader.get( 'orto-vs' ),
	fragmentShader: shaderLoader.get( 'copy-fs' ),
	shading: THREE.FlatShading,
	depthWrite: false,
	depthTest: false,
	transparent: true
} );

var composer = new WAGNER.Composer( renderer, shaderLoader, copyShader );

var zoomBlurPass = new WAGNER.ZoomBlurPass();
var multiPassBloomPass = new WAGNER.MultiPassBloomPass();

renderer.autoClearColor = true;
composer.reset();
composer.render( scene, camera );
composer.pass( multiPassBloomPass );
composer.pass( zoomBlurPass );
composer.toScreen();
```

What works
----------

- Passes are by default RGBA
- Ping-pong buffers when chaining passes
- ShaderLoader is being replaced, but you can use it to load .glsl files. It hides all the XHR stuff
- Composing with Wagner for effects that run chained with the same resolution (e.g. full-screen effects)
- Basic effects implemented in the new WAGNER.Pass class:
    - Blend pass: all single pass. Current blend modes implemented: normal, darken, multiply, lighten, screen, overlay, soft light, hard light
    - Invert: single pass, inverts colours
    - Box Blur: single pass, blur in one direction specified in vec2 delta uniform
    - Full Box Blur: multipass, 2 box blur in two directions
    - Zoom Blur: single pass
    - Multi Pass Bloom: multipass, applies blur and blends with Screen mode
- uniform reflection from GLSL source is working enough to be usable for most cases

What still doesn't work / needs work
------------------------------------

- ShaderLoader will probably be removed, or be transparent to the user
- Passing parameters to WAGNER.ShaderPass from main code
- Correct use of textures of different dimensions along the chain
- Multiple Composers working at the same time
- Shaders that are not ported to WAGNER.Pass: pixelate, rgb split, different single-pass bloom
- Shaders that haven't even been ported to WAGNER: SSAO, DOF, camera motion blur, directional blur, gamma, levels, edge detection
- Alias definition of passes (previously loadPass()) legacy
- uniform reflection from GLSL source doesn't support structures (I don't even know if WebGL supports structures)