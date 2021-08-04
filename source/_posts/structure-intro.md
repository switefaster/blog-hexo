---
title: 项目结构介绍
date: 2020-06-24 21:45:04
tags:
    - CG
    - GameDev
    - Rust
---

这算是一篇阶段性总结，这段时间找了不少库，也差不多该把项目的组织方式确定下来了，所以在这里做个记录，以后可能会改吧。顺便，最近感觉好像 `gfx-hal` 又有点起色，如果有机会就拿 hal 也写一个。~~咕咕咕~~

---

首先讲两句题外话，我把 `wavefront_obj` 从依赖里面删除了。主要是觉得为了加载模型需要写两个处理 parser 数据的代码巨弱智，所以干脆改成只支持 collada 了，毕竟人家还支持骨骼动画呢，是不:)，然后加了个 `try_guard`。其次有打算使用[这个库](https://crates.io/crates/meshopt)，能优化模型，感觉蛮顶的，不过代码还没来得及写。

好，进入正题。先陈列一下现在确定的项目结构：

* src
  * main.rs
  * client
    * mod.rs
    * model.rs
    * render.rs
  * common
    * mod.rs
    * TBD
  * server
    * mod.rs
    * TBD

草，好像还屁都没写嘛——确实。现在也顶多只是有思路而已，服务端和 `common` 内容的组织还是蛮复杂的，在确定具体元素之前还是只能搓搓渲染的东西。所以这篇文章大概重点是放在渲染代码上:P。

可能有读者已经注意到了，这个~~比亲妈还~~熟悉的命名。这算是游戏模块的经典组织方式了，做过 Minecraft® Mod 的读者肯定是和这种结构打过交道的。简单而言，`client` 模块主要放置只需要在客户端完成的东西，例如渲染、数据包上传等；`server` 模块同上；而 `common` 模块是两者皆有之的部分，比如游戏数据存储、游戏逻辑之类。暂时不确定是直接把服务端和客户端直接写分开还是用 `#[cfg]` 来控制编译出单独服务端和包含服务端的客户端，后者显然是更好的，可惜我不太会写，到时候再说吧(跑。

重中之重显然是 `render.rs`，里面我现在放了一个结构 `Renderer`，现在长这样：

```rust
struct Renderer<'a> {
    window: winit::window::Window,
    event_loop: winit::event_loop::EventLoop<()>,

    instance: Arc<Instance>,
    physical: PhysicalDevice<'a>,
    surface: Arc<Surface<winit::window::Window>>,
    queue_family: QueueFamily<'a>,
    device: Arc<Device>,
    graphics_queue: Arc<Queue>,
    transfer_queue: Arc<Queue>,
    swapchain: Arc<Swapchain<winit::window::Window>>,
    images: Vec<Arc<SwapchainImage<winit::window::Window>>>,
    //TODO: render_pass
}
```

可以发现这个结构只是渲染需要的底层资源的集合而已，事实也确实如此。可以发现里面并没有 `pipeline`，主要是因为 pipeline 需要指定着色器，不同的渲染方式可能天差地别，所以留给用户设定。这个结构主要是负责构造 `CommandBuffer` 的，会提供方法来帮忙写渲染指令，然后统一调用。这样也方便在 ECS 的 local_thread 里面跑东西。哦对了，到时候开个新 post 来讲 ECS 吧，嗯。反正大致构想就是可以用 `Renderer::xxx(...)` 完成绝大多数的渲染操作，比如 `Renderer::draw_model_with_pipeline_indexed(...)` 之类的。

然后是 `model.rs`，这里面放了 collada 的模型加载代码，现在应该算是完成了，不过没写测试。主要是从[这里](https://github.com/PistonDevelopers/skeletal_animation/blob/master/src/skinned_renderer.rs)抄来的。读取出来的 Vertex 的结构是：

```rust
#[derive(Debug, Clone)]
struct Vertex {
    position: [f32; 3],
    normal: [f32; 3],
    tex_coord: [f32; 2],
    joint_indices: [i32; 4],
    joint_weights: [f32; 4],
}

vulkano::impl_vertex!(Vertex, position, normal, tex_coord);
```

反正现在直接加上了关节的参数，如果要改以后再说吧……代码会在近期 push，到时候就能看到了。可以预见的是 `meshopt` 的内容会放在这个文件里面。

憋不出话了，先这样，摸了。

~~有没有带哥教我怎么写服务器啊~~
