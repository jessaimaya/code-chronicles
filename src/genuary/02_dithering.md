# Day 02 - Dithering

<input type="file" id="file" onchange="process()" style="display: block; margin: 1rem auto;"/>

<label for="number" id="selector" style="display: none;margin-top:1.5rem;">
Depth: 
<input type="text" id="number" name="example_name" value="" />
</label>

<canvas style="margin: 1rem auto;display:none;" id="canvas"></canvas>
<canvas style="margin: 1rem auto; display: block;" id="show"></canvas>

<button id="button" style="display:none;" onclick="toggleColors(this)" data-color="color">Color/B&W</button>


<link rel="stylesheet" href="../scripts/rangeslider.css" />
<script src="../scripts/rangeslider.min.js"></script>

<script type="text/javascript">
    var wasm;
    var color = true;
    var bytes;
    var mode = "color";
    var factor = 1;
    var ready = false;
    
    const toggle_colors = () => {
        if (color) {
            wasm.grayscale();
        }
        wasm.color();
        color = !color;
    };

    const load = async() => {
        wasm = await import("../wasm/dithering/dithering.js");
        await wasm.default();
        ionRangeSlider('#number', {
            min: 5,
            max: 255,
            grid: true,
            grid_num: 5,
            step: 5,
        });
        const selectElement = document.getElementById('number');
        selectElement.addEventListener('change', (event) => {
            ready = false;
            factor = event.target.value;
            ui();
            sync();
        });
    };
    const get_array = async (file) => {
        const data_file = await read_file(file);
        return new Promise((resolve, reject) => {
            const img = new Image();

            img.onload = () => {
                const canvas = document.getElementById('canvas');
                const ctx = canvas.getContext('2d');
                canvas.width = img.width;
                canvas.height = img.height;

                ctx.drawImage(img, 0, 0);

                let buff = ctx.getImageData(0, 0, img.width, img.height).data;
                resolve({data: buff, width: img.width, height: img.height});
            };

            img.onerror = e => reject(e);

            img.src = data_file;
        });

    };

    const read_file = async(file) => {
        return new Promise((resolve, reject) => {
            let reader = new FileReader();
            
            reader.addEventListener("loadend", e => resolve(e.target.result));
            reader.addEventListener("error", reject);

            /*reader.readAsArrayBuffer(file);*/
            reader.readAsDataURL(file);
        });
    };

    const process = async() => {
        const input = document.getElementById("file");
        const file = input.files[0];
         if(file.size > 524288){
           alert("File is too big! (Max 500kb)");
           input.value = "";
            return;
        }
        bytes = await get_array(file);
        let wasm_bytes = await wasm.dithering(bytes.data, 5.0, bytes.width, bytes.height, false);
        drawImage(wasm_bytes, bytes.width, bytes.height);
        ready = true;
        ui();
   };
   
    const drawImage = (data, width, height)  => {
        let arr = new Uint8ClampedArray(data);
        let img = new ImageData(arr, width, height);
        let canvas = document.getElementById("show");
        let ctx = canvas.getContext("2d");
        canvas.width = width;
        canvas.height = height;
        canvas.style.width  = width +'px';
        canvas.style.height = height + 'px';
        ctx.putImageData(img, 0, 0);
    };

    const reset = () => process();

    const toggleColors = (e) => {
        if (mode === "color"){
            e.dataset.color = "black";
        } else {
            e.dataset.color = "color";
        }
        mode = e.dataset.color;
        sync();
    }; 

    const sync = async() => {
        let wasm_bytes = await wasm.dithering(bytes.data, factor, bytes.width, bytes.height, mode === "black");
        drawImage(wasm_bytes, bytes.width, bytes.height); 
        ready = true;
        ui();
    };
    
    const ui = () => {
        const selector = document.getElementById("selector");
        const button = document.getElementById("button");
        
        if (ready) {
            selector.style = "";
            button.style = "";
        } else {
            selector.style = "display:none;";
            button.style = "display:none;";
        
        }
    };

    load();
</script>