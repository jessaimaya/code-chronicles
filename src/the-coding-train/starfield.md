# Starfield

Here is the first challenge from this series.
You can control the speed with your cursor on X axis, just move your cursor inside canvas from left to right.

<canvas style="margin: 1rem auto;display:block;" id="canvas"  height="600" width="600"></canvas>

 <script type="text/javascript">
    const load = async() => {
        let wasm = await import("../wasm/starfield/starfield.js");
        wasm.default();
    };
    load();
</script> 

## Code highlights

You can check all the code on this [repo](https://github.com/jessaimaya/rust_wasm).

But I really enjoy doing this challenge was this extract from the render loop: 

```rust
    pub fn start(game: impl Sketch + 'static) -> Result<()> {
    let mut mouse_receiver = mouse()?;
    let mut game = game.initialize()?;
    let mut game_loop = RenderLoop {
        last_frame: now()?,
        accumulated_delta: 0.0,
    };

    let renderer = Renderer {
        context: context()?,
    };

    let f: SharedLoopClosure = Rc::new(RefCell::new(None));
    let g = f.clone();

    *g.borrow_mut() = Some(create_raf_closure(move |perf: f64| {
        let frame_time = perf - game_loop.last_frame;
        game_loop.accumulated_delta += frame_time as f32;
        while game_loop.accumulated_delta > FRAME_SIZE {
            game.update(&mut mouse_receiver);
            game_loop.accumulated_delta -= FRAME_SIZE;
        }
        game_loop.last_frame = perf;
        game.draw(&renderer);

        request_animation_frame(f.borrow().as_ref().unwrap()).unwrap();
    }));

    request_animation_frame(
        g.borrow()
            .as_ref()
            .ok_or_else(|| anyhow!("GameLoop: Loop is None"))?,
    )?;
    Ok(())
}
```

I consider this part the core of this animated challenge, here is the `tick` method which renders and updates every entity that implements the `Sketch` trait.
It creates a `Closure` and send it to `request_animation_frame` in order to get the right timing for each execution.

