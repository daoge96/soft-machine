# SOFT MACHINE · SM-16

> **一块软陶，四个旋钮，十六步。**
> 一个可以真的弹响的 neumorphic 合成器，全部塞进一个 HTML 文件。

---

## 一句话

`007-neumorphic-synth.html` 是一台虚构型号 **SM-16** 的软件合成器：整台琴从同一块软陶页面（`#e4e8ee`）里雕塑出来——凸起的旋钮、凹进的屏幕、按下才亮的软垫——全页只有一种彩色，珊瑚红 `#ff6b5a`，只用在会发光的地方。打开就能弹，不联网、不加载任何外部资源。

## 它是怎么响的

声音不是采样，是**现场算出来**的。每按一个键，就会新建一条独立声部：

```
振荡器 → 低通 ×2（24 dB/oct，共振在第一级，带短促 pluck）
      → 音量包络（attack / release）
      → 声像 → 回声总线（330 ms，2.2 kHz 阻尼）
      → 压缩 → 输出
```

复音上限 16 声部；超过时最老的声部被"接管"——它的振荡器不会重建，而是直接改音高、重排包络，所以**先按的音永远不会突然变哑**。所有音符松手后都会在 release 结束后拆掉节点，不留尾巴。

## 屏幕：先预告，再直播

左上角那块 OLED 风格的凹陷屏幕是整台琴的门面，也是它最有意思的一处设计：

- **你没弹的时候**，它不是一张摆设图，而是一段**真的带限预览**——把选定波形的 48 次谐波乘上双极点低通的解析响应（幅值平方、相位加倍，和真实信号链一致），所以你拧截止频率或共振，波形会**当场改变形状**，共振会真的"起铃"。
- **只要你按下第一个键**，屏幕无缝切到 `AnalyserNode` 的**真实输出**：找上升过零点做触发对齐，所以波形稳稳地站着不会左右乱跑；珊瑚色描边分三遍叠加出辉光；右侧徽标从 `preview` 翻成 `live output`。
- 静音时它会**如实变平**（拾音点在静音增益之后，画的就是你听到的），但**继续刷新**不黑屏。

## 你能操作的东西

| 操作 | 做什么 |
| --- | --- |
| `A W S E D F T G Y H U J K`（`O L P ;` 往上延伸） | 弹奏；按 `Tab` 进键盘后可用 `←`/`→`/`Home`/`End` 逐键走音 |
| 鼠标/触摸在琴键上横向滑动 | 滑音（glissando） |
| `Z` / `X` 或 `−` / `+` | 整键盘移八度 |
| 拖动四个旋钮（`Shift` 微调、滚轮、方向键、双击复位） | 截止 120 Hz–11 kHz、共振 0.1–14、起音 1–900 ms、释音 30–2600 ms |
| 波形四分段按钮（方向键切换） | 正弦 / 三角 / 锯齿 / 方波，正在响的音会立刻换音色 |
| 空格 | 播放 / 停止音序器（60–180 BPM） |
| 点一个空软垫，再弹任意键 | 把那个音**录**进这一步；虚线环表示"在听" |
| Shuffle / Clear | 生成新的一句 A 小调五声音阶，或清空 |
| 喇叭按钮 | 淡出静音 / 恢复 |

底部还有一段小结：**音色、静音、速度、八度、Pattern 全部记在本地**（`localStorage`，写失败也不崩），下次打开还在。

## 为什么它看起来"是块东西"

- 所有控件都从**同一块陶土**里长出来：凸起用一对白/灰外阴影，按下换成一对应内阴影，两者之间是动画过渡——没有第二种底色。
- 旋钮是内联 SVG：刻槽、高光环、11 道刻度、沿 `pathLength` 归一化圆的珊瑚色取值弧；凹陷的旋钮帽绕 270° 旋转，指示线与光点一起走。
- 屏幕外的装饰全是代码画的：程序化噪点底纹、CSS 扬声器格栅、四角带旋转槽的螺丝、随输出电平呼吸的电源 LED。
- 布局从 360 px 手机一路排到宽屏：手机上屏幕叠到旋钮上方、音序器折成 8×2、键盘自动重建成一个紧凑八度——**任何宽度都不会横向滚动**。
- 跟随 `prefers-reduced-motion`：保留静止的预览波形，关掉脉冲与呼吸。

## 辅助功能与工程细节

- 语义化 HTML5、唯一 id、每个控件都有 `aria-label`/`aria-valuetext`；旋钮是 `role="slider"` 且带完整取值三元组；波形是真正的 `radiogroup`；琴键暴露 `aria-pressed`。
- 键盘是**一个** tab stop（不是 34 个），方向键在琴键间行走并顺带发声。
- `:focus-visible` 焦点环在陶土底色上清晰可见；播放头、状态、提示都有 `aria-live`。
- 标签页切走时停止音序器并释放按住的音；画布按容器 × `devicePixelRatio`（上限 2）重建；动画全部走 `requestAnimationFrame` 与 transform/opacity。
- 只有一个 `<style>` 和一个 `<script>`；**0 个外部请求**（无 CDN、无字体、无图片、无音频文件），`file://` 双击即用。

## 技术一览

| | |
| --- | --- |
| 文件 | 1 个 HTML，约 90 KB，1904 行 |
| 依赖 | 无。0 个外部请求 |
| 音频 | 纯 Web Audio：振荡器 + 双级双二阶低通 + 包络 + 回声 + 压缩 |
| 视觉 | CSS / 内联 SVG / Canvas（示波器、噪点），无位图 |
| 字体 | 仅系统字体栈，末端为通用族 |
| 状态 | `localStorage`（`try/catch` 包裹），首帧即可用 |

## 顺带一说

SM-16 是**虚构型号**，页面上的文案、参数与那句 "clay series" 都是它自己的设定——但所有数字都对得上真实存在的信号链，没有一个是为了好看编的。

---

## English blurb

**SOFT MACHINE (SM-16)** is a playable neumorphic synthesizer sculpted from a single slab of soft clay (`#e4e8ee`), with warm coral (`#ff6b5a`) as the only colour. One standalone HTML file, 0 external requests, entirely hand-written Web Audio: oscillator → two-stage 24 dB/oct low-pass with resonance and a filter pluck → attack/release envelope → darkened echo → compressor → out, 16-voice polyphony with graceful voice adoption. Its OLED-style scope shows a real band-limited preview (48 harmonics shaped by the filter's analytic response) that warps as you turn the knobs, then hands over to the live `AnalyserNode` trace the moment you play — zero-crossing triggered, with an honest flatline while muted. Four knobs, a four-way waveform selector, a 16-step sequencer in four trays, and a two-octave soft keyboard playable with A–K or by sliding for glissando. Responsive from 360 px to desktop, honours `prefers-reduced-motion`, keyboard-navigable, and it remembers your patch.
