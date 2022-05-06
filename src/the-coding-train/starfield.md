# Starfield

Here is the first challenge from this series.
You can control the speed with your cursor on X axis, just move your cursor inside canvas from left to right.

<canvas style="margin: 1rem auto;display:block;" id="canvas"  height="600" width="600"/>

 <script type="text/javascript">
    const load = async() => {
        let wasm = await import("../wasm/starfield/starfield.js");
        wasm.default();
    };
    load();
</script> 
