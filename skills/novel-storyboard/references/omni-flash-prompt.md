# Google Flow / Gemini Omni Flash 1.1 视频提示词 · 写法规范（内化版）

方法论学自 Google 官方文档（ai.google.dev/gemini-api/docs/omni，模型 `gemini-omni-1.1-flash`），**内化成本 skill 自带文档，不依赖外部页面**。**正文内容怎么写见 `shot-writing.md`**（一切一个运镜、动作要做得完、台词逐字、声音分层、不写画风），本文只讲 Omni/Flow 特有的语法、符号和禁忌。

跟另外两个协议的差别是结构：H3 靠正文把参考图钉到秒数上；Seedance 用镜头顺序表达时间层、不写秒数；**Omni 是唯一两者都要的——它支持时间码 `[0-3s]`，同时用 `[# Sources]` / `[# References]` 声明附件角色**。第一张分镜图是**字面首帧**（`<FIRST_FRAME>`），其余图片只是**一致性参考**（`<IMAGE_REF_k>`），绝不能当字面画面——这是官方文档反复强调的一条。

## 语言：整条英文

官方口径：**提示词用英文**（其他语言未评测）。台词原文保留中文，写在引号里由说话人行带出。这与 Seedance（整条中文）相反，所以本管线给 Omni 单开一个字段：

| 提示词里的位置 | 字段 |
| --- | --- |
| 每切的时间行正文 | 每切的 `shotOmni`（**英文**）；没写则退回 `shot`（仅当 `shot` 本身是英文时） |
| Blocking 行 | 每段的 `blocking`（沿用，程序原样带上） |
| 运镜 / 景别 / 镜头 / 机位 / 构图 | 每切的 `camera` / `size` / `lens` / `cameraPosition` / `composition`，程序转成英文短语（`locked-off static camera`、`slow push in toward the subject`、`extreme wide shot`…） |
| Lighting / Eyeline / Focus 行 | 每切可选 `lighting` + 必填 `eyeline` / `focus` |
| 台词 `"…"` | 不用写——程序从剧本认领的节拍逐字取，按 speaker 排进行内 |
| Sound design / Music 行 | 每段的 `soundscape` / `music` |
| Constraints 编号列表 | 调用方 `--constraints <文件>` + 程序自动补「无字幕」与「单一连续段」 |

## 时间层：写时间码，但交给结构推导

Omni **支持** timecode（与 Seedance 相反），官方示例形如 `[0-3s] … [3-6s] …`。在本管线里：

- **不写** `[0-3s]`、`0:03`、`3 seconds` 这类时间——切点区间由分镜秒数**确定性推导**，程序逐切加
- **不写** `[Shot k]`、`【镜头1】`——Omni 没有镜头标记协议，按行分段即可
- **不写** `@图片N`、`<Picture k>`、`<FIRST_FRAME>`、`<IMAGE_REF_k>`——引用声明由程序按真实附件顺序生成
- **不写** 引号 `""`、尖括号 `<>`、花括号 `{}`、圆括号——台词引号和声景行由程序套（Omni **没有 negative prompt 参数**，负向指令以编号句写进 Constraints，程序负责）
- 每一切只写这几秒发生什么：**主体动作与表情 → 位置或空间变化**；运镜、景别等量化字段程序已经从枚举转好拼在同一行

第 19 道质量门 `omni-shot` 盯着这些：`shotOmni` 含中文、写了时间/编号/引用/符号都会拦；只有中文 `shot` 且没写 `shotOmni` 时，要求该切必须有分镜图提示词 `frame`（齐图路径下构图由 storyboard frame 钉死，文本不承担）。

## 附件两条路（程序自动判定）

- **每切都有分镜图** → 首张是 `<FIRST_FRAME>@ImageN`（字面起始帧），其余分镜图依次拿 `<IMAGE_REF_0>…`，逐切正文尾部带 `(match the composition of <IMAGE_REF_k>)`
- **缺分镜图** → 场景设定图 + 出场角色设定图当 `<IMAGE_REF_k>` 一致性参考，正文不带构图引用，manifest 的 `missing` 里列出缺哪些附件

## 程序拼出来的形状（示意，你不用写）

```
Generate one 12.4-second video segment as a single continuous piece. Live-action cinematic short-drama look, photorealistic, high detail.

[# Sources <FIRST_FRAME>@Image1]
[# References <IMAGE_REF_0>@Image2 <IMAGE_REF_1>@Image3]

Blocking: 张三坐于榻沿，李四立于榻前约两步，两人面对面。…

[0-3.5s] The woman steps back half a pace, her hand tightening on the sleeve. locked-off static camera, medium shot, 50mm 标准 lens, camera position: …, composition: 三分法
   Lighting: 侧逆光勾出轮廓
   Eyeline: 对方面部
   Focus: 锁定李四上半身
   Character C02: "你为什么要骗我"
[3.5-8s] … slow push in toward the subject, close-up …

Sound design: 木屐踏地声、衣料摩擦声。
Music: 弦乐慢板，克制的心跳节奏。

Constraints:
1. No subtitles, captions or on-screen text anywhere in the video.
2. One single continuous segment only, no scene changes beyond the cuts listed above.
```

结尾的程序附注会明确告诉模型哪张是字面首帧、其余只做一致性——不要手写在正文里。

## 提交参数（export --protocol omni 已内置）

`omni-request.json` 是给 `/v1beta/interactions` 的现成请求体：`model: gemini-omni-1.1-flash`，`response_format: {type: "video", aspect_ratio: "9:16", resolution: "720p"}`（短剧竖屏；官方支持 360p/720p/1080p/4k）。在 Google Flow 界面里操作时：先传首帧图，再按 manifest 顺序传参考图，然后把 `omni.md` 整段粘进提示框。

`render --html` 的提示词面板有 Omni 页签，`export --protocol omni` 出投产包（每段 `omni.md` + `omni-request.json`，根部 `omni-manifest.json`）。
