# Sierpinski Triangle

All the process and rendering is managed by wasm.

You can control the depth from this slider. 

<label for="number">
Depth: 
<input type="text" id="number" name="example_name" value="" />
</label>

<canvas style="margin: 1rem auto;display:block;" id="canvas"  height="600" width="600"/>



<link rel="stylesheet" href="../scripts/rangeslider.css" />
<script src="../scripts/rangeslider.min.js"></script>
 <script type="text/javascript">
    const load = async() => {
        let wasm = await import("../wasm/sierpinski/index.js");
        wasm.default();
        ionRangeSlider('#number', {
            min: 1,
            max: 10,
            grid: true,
            grid_num: 5,
            step: 1,
        });
        const selectElement = document.getElementById('number');
        selectElement.addEventListener('change', (event) => {
            wasm.update(event.target.value);
        });
    };
    load();
</script> 