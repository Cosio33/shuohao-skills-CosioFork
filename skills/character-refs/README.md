[![中文](https://img.shields.io/badge/%E4%B8%AD%E6%96%87-8b1a1a?style=for-the-badge)](README.md)
[![English](https://img.shields.io/badge/English-f2e3e3?style=for-the-badge&labelColor=f2e3e3&color=b07070)](README.en.md)
[![关注作者 X](https://img.shields.io/badge/%E5%85%B3%E6%B3%A8%E4%BD%9C%E8%80%85-%40eternityspring-b07070?style=for-the-badge&labelColor=8b1a1a&logo=x&logoColor=f2e3e3)](https://x.com/eternityspring)

# character-refs

给任何故事里的角色**真出**参考图——小说改编、自己原创的故事、单独设计一个角色都行，不需要小说原文。一段话描述角色，出一组能直接挂进视频模型（H3 / Wan / Seedance）的参考图。

**一组图只有一个根**：正面全身锚点。只有它是文生图，大头照、侧面、背面、细节图都只参考这张锚点，
图与图之间不再互相参考——任何一张出坏了，单独重出这一张，不牵连别的。

## 分档，按需升档

| 档位 | 内容 | 什么时候出 |
| --- | --- | --- |
| 1（默认） | 正面全身（锚点） | 总是 |
| 2 | 正脸大头照、90° 侧面、背面 | 有特写 / 近景、侧身、背影 |
| 3 | 4 张细节：头发 / 发饰、领口、袖口、鞋 | 有配饰或服装局部的特写 |
| 4（默认不出） | 45° 大头照 | 明确要 |

依据是一次 H3 实测：只挂正面全身，服装和背影都对，但脸会走样；加一张大头照，脸最接近。所以大多数角色一档够用，升档时大头照最优先。

## 一次性输入

不用一项一项填。一段话：

> 阿禾，十六岁，1920 年代江南茶山上的采茶姑娘。个子小，常年在山上晒，脸是麦色的，眉眼弯弯，笑起来左边有个酒窝……

agent 拆进身份、脸、头发、身形、皮肤、上装、下装、细节各字段，缺的自己补，每项标来源（原话 / 推断 / 默认），
打印确认表给你看。只有年龄、性别、年代推不出来才会问。

## 模型

四种，第一次使用时选，之后一直沿用：

| | 说明 |
| --- | --- |
| `qwen` | Qwen Image 2.1，走自己的 ComfyUI。每张 25–50 秒，尺寸精确，不吃订阅额度 |
| `codex` | 本机 codex 内置出图（GPT）。不要 API key，但吃 ChatGPT 订阅额度（Plus 约 25 张触顶） |
| `openai` | OpenAI Images API，默认 gpt-image-2，按量计费 |
| `custom:<名字>` | 自己的出图命令，模板里填 `{prompt_file}` `{refs}` `{out}` `{width}` `{height}` |

**主图和派生图可以使用不同模型，重出单张也能换。**一致性来自同一张锚点，不来自同一个模型——
Qwen 锚点 + GPT 派生、GPT 锚点 + Qwen 派生、同一组里混用，实测都衔接得上。每张图记录自己的模型，混用只提醒。

## 标识与重出

每张图都有标识 `角色/造型/视图/版本`（如 `阿禾/default/face-front/v2`），写在文件名、`asset.json` 和 PNG 元数据三处。

```bash
node scripts/character-refs.mjs gen 阿禾/asset.json detail-neck --model codex   # 单独重出一张，存成新版本
```

重出不覆盖旧版。冻结的只有三样：锚点文件、画风快照、文字描述。任何一样变了，依赖它的图标成过期：
重出锚点 → 其余全部过期；重出大头照 → 挂了它的头发 / 领口细节过期；改了描述 → 全部过期。只标记，不删除。

## 质量门

代码检查，`check` 不过就 exit 1：

| 门 | 规则 |
| --- | --- |
| 尺寸 | 单边 300–5760 像素 |
| 宽高比 | 0.4–2.5 |
| 比例 | 跟视图要求的一致（2:3 / 4:5 / 1:1） |
| 大小 | ≤ 20MB |
| 背景 | 全身图查四边够白；大头照只查上边和两侧上半段；细节图跳过并明说 |
| 过期 | 见上 |

前四道是 H3 / Wan / Seedance 参考图要求的交集。**门查不了长相、角度和取景**，那几项要自己看图。

## 报告

`render` 出一张双击就能开的 HTML：一档只有一张卡片；二档起是设定图版面（左大头照，右上正面 / 侧面 / 背面，右下细节条），
默认不显示细节图，页面上一键切换。界面内置中文、英文、日文（`render --lang en`），其他语言由 agent 现场翻一份文案；
角色描述保持原文，提示词永远英文。版面由代码排，不生成拼接大图——拼接图当参考图，模型会把人画小。

## 命令行直接使用

```bash
node scripts/character-refs.mjs config --model qwen --confirm-anchor yes --qwen-env-file ~/comfy.env
node scripts/character-refs.mjs intake-check examples/阿禾-intake.json
node scripts/character-refs.mjs new examples/阿禾-intake.json --out out/
node scripts/character-refs.mjs gen out/阿禾/asset.json --tier 1
node scripts/character-refs.mjs confirm out/阿禾/asset.json
node scripts/character-refs.mjs gen out/阿禾/asset.json --tier 2 --reason "E03 有面部特写"
node scripts/character-refs.mjs check out/阿禾/asset.json
node scripts/character-refs.mjs render out/阿禾/asset.json --out out/character-refs.html
```

## 文件

```
SKILL.md                    给 agent 读的工作流
scripts/
  character-refs.mjs  config / intake-check / new / gen / confirm / check / render
  core.mjs                  视图、提示词、输入校验、版本与过期、检查门
  models.mjs                四个出图适配器
  png.mjs                   PNG 读写与元数据（零依赖）
  selftest.mjs              自测，不调模型
references/
  intake.md                 一段描述怎么拆成字段
  prompting.md              实测出来的提示词写法
  models.md                 四个模型的接法与实测
  schema.md                 asset.json 结构
examples/
  阿禾-描述.txt              一句话描述
  阿禾-intake.json           拆好的输入
```

## 自测

```bash
node scripts/selftest.mjs
```

171 项断言，不调模型、不花额度，Qwen 和 GPT Image 使用本地假服务器校验请求格式，完整流程使用自定义命令造的白底图跑通。

## 已知短板

- 画风只有写实照片一种
- 小疤痕容易画成新伤；双排扣这类大面积细节取景偏远
- Qwen 的领口细节会把粗布画得像丝绒，这一张换 codex 重出
- `openai` 只验过请求格式，没有使用真 key 跑过
- **只在 macOS + Node 24 上实测过**
