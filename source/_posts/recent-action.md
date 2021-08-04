---
title: 爷回来辣！
date: 2021-08-04 17:37:35
tags: 
    - CG
    - GameDev
    - Rust
---

竟然有一年没写博客了，我回头看了看之前写的东西，~~我看不懂，但我大受震撼~~，真是一言难尽啊。虽然估计没人看，还是写一下最近在干嘛吧。

寒假的时候相位猛冲写了[N2-Station](https://n2station.live)的[后端](https://github.com/M1k0t0/N2-Station)，结果[@SBCK](https://github.com/M1k0t0)不用我的后端，我实属伤透了心。在此祝福CK玩screeps必被周围玩家~~掘腚眼子~~友善对待，您的好友switefaster敬上。

可以看见我又开了个[新坑](https://github.com/switefaster/nederite)，啊？你问我[multi_pong](https://github.com/switefaster/multi_pong)怎么弃了？~~摸鱼人的事，怎么能叫弃坑呢~~。其实是觉得写2D游戏没意思，而且用别人搓好的引擎着实无趣，所以决定自己也搓一个！当然，ECS很香，我还要，而且有了[wgpu-rs](https://github.com/gfx-rs/wgpu)这个安全的图形库，搓起来那叫一个香啊……

不妨先来看看[nederite](https://github.com/switefaster/nederite)的依赖:

```toml
[dependencies]
anyhow = "1.0.42"
bytemuck = { version = "1.7.2", features = ["derive"] }
env_logger = "0.9.0"
nalgebra = { version = "0.28.0", features = ["convert-bytemuck"] }
futures = "0.3.16"
image = "0.23.14"
log = "0.4.14"
rayon = "1.5.1"
tobj = "3.1.0"
wgpu = "0.9.0"
winit = "0.25.0"
rapier3d = { version = "0.10.1", features = ["parallel", "simd-nightly"] }
tokio = { version = "1.9.0", features = ["macros", "rt", "time"] }
wgpu_glyph = "0.13.0"
glyph_brush = "0.7.2"
specs = { version = "0.17.0", features = ["storage-event-control"] }
gag = "1.0.0"
```

哇，好长！
但是里面有两个库还没有用到，具体是

- rapier3d
- tokio

后者肯定是拿来搓网络啦，那么经典，但是这次我还是自己搓一个吧，总是用pca的RUDP不太好，而且竟然也有别的库名字叫RUDP，抄袭pca的创意，罪不可恕。

前者是物理引擎，说实话我有点好奇，和[nphysics](https://github.com/dimforge/nphysics)同样都是DimForge家的，为啥要整两个……不过据说rapier3d是游戏特供，而且看着比nphysics简单，那我必用啊。

还有个比较特殊的库是gag，说实话要不是yys说要把日志写进文件，我都想不到用这个库……

Nederite 采用了我们的老朋友[SPECS](https://github.com/amethyst/specs)，说实话这个库越搓越像amtehyst...但是毕竟是自己写的，爽啊，这就够了(跑)

接下来说说这次过程里面踩的坑吧

重点关照一下SPECS，不知道哪个老哥为了Script Engine的Example给Resource加了个'static，导致Resource不能存&World，我问候他全家。

还有就是nalgebra竟然没有look_at……我把它和look_to搞混了迷惑了好久，佛了...但是bytemuck支持还是很爽的

wgpu_glyph 传变换还要经历一次放大缩小，不然那字渲染的比天还大，我傻了，以后研究研究 layout 看能不能调整掉。

还有env_logger，0.8.4加了个Pipe竟然还是坏的，等更新，差评

最近博客的位置会改一下，我决定写一篇WGPU中文教程，造福香肠:P

先扯到这了

Glorious Quit
