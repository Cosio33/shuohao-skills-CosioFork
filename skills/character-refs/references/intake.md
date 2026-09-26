# 一次性输入：怎么把一段描述拆成字段

用户一段话描述角色，你拆进 `intake-template` 的各字段，缺的自己补，然后 `intake-check` 出确认表给用户看。
**不要一项一项问。**只有年龄、性别、年代推不出来时才问——这三样错了，整组图都错。

## 语言

`lang` 是确认表和报告的语言，照用户说话的语言填：`zh` / `en` / `ja` 内置。其他语言（法语、韩语……）：

```bash
node scripts/character-refs.mjs ui-template fr
```

把打印出的英文文案逐项翻译（占位符 `{n}` `{x}` 原样保留），整块放进 intake 的 `ui` 字段。缺一项 `intake-check` 就报错。

每个字段两段文字：`en` 进出图提示词，**不管 `lang` 是什么都写英文**（出图模型吃英文最稳）；`text` 给人看，用 `lang` 的语言写。
旧输入写的是 `zh`，照样认。

## 字段

| 字段 | 进哪些图 | 写什么 |
| --- | --- | --- |
| `identity` | 全部 | 年龄（整数）、性别（female / male）、族裔、年代、身份。`en` 写成一句完整的人物介绍 |
| `face` | 全部 | 脸型、眼、眉、鼻、嘴、肤色、有辨识度的面部特征 |
| `hair` | 全部 | 发色、长短、发型、发饰 |
| `build` | 全身视图 | 身形、体态。可省 |
| `skin` | 全部 | 皮肤状态。可省，省了按年龄给缺省 |
| `outfit.top` | 全部（大头照只用这一项服装） | 上装，连同领口、袖子的状态 |
| `outfit.bottom` | 全身视图 | 下装与鞋袜 |
| `outfit.details` | 第三档 | 四个槽位 `hair` / `neck` / `sleeve` / `feet`，每个只写这个部位本身 |
| `backCue` | 背面 | 只有背面看得到的东西。可省，省了按发型与服装拼 |

服装拆上下两段，是因为**大头照只能带上装**：提示词里写了裤子和鞋，改图模型就会把镜头拉远去画它们。

## 来源标记

每个字段带 `source`：

- `stated` —— 用户原话里有
- `inferred` —— 你按身份、年代推出来的（1920 年代采茶姑娘 → 黑布裤、草鞋）
- `default` —— 惯例缺省（皮肤、背面）

**如实标。**确认表把推断和默认的项标出来，用户只看这几项就够了。把推断标成原话，等于替用户做了决定却不说。

## 英文怎么写

- 写能画出来的东西：「wary eyes」可以，「a girl who has been through a lot」不行
- **不写角色名**，不写画风词（anime、photorealistic、cinematic……）——画风层统一加，写进角色里会和它打架
- 小特征写成真实状态：疤写「愈合多年的旧疤：细、平、略浅、几乎不凸」，否则会画成粉红色的新伤
- 皮肤写到什么程度是角色属性，不是画风：统一写「泛红、有瑕疵」会让十九岁女生出成病容
- 族裔推不出来默认东亚，并落到具体五官上

## 细节槽位

默认四个，都是实测能稳定出来的：

| 槽位 | 写什么 | 例 |
| --- | --- | --- |
| `hair` | 发饰、辫梢、发髻的一处 | the knot of her faded blue cotton headscarf at the nape |
| `neck` | 领口与最上面**一颗**扣子 | the small standing collar and the single cloth-knot button at her throat |
| `sleeve` | 一只袖口 | the indigo cotton sleeve rolled up above the wrist, with tea stains on the edge |
| `feet` | 鞋与脚踝 | woven straw sandals and the trouser hems tied at the ankles |

只写部位本身，**不写「特写」「close-up」之类取景词**，取景由脚本加。

**部位要小到一眼看得全。**写「斜襟和一排布扣」，斜襟从领口一路开到腋下，模型就把整个前胸都框进来（实测）；
写「小立领和喉下那一颗布扣」就对了。领口只写领子加最上面一颗扣子，袖口只写一只。

四个之外还要，使用 `slot: "custom"`，另给 `id`（小写英文）、`part`（hair / face / neck / body / feet），
英文写整句，并以 `Zoom in to an extreme close-up of only ` 开头——改图模型只有第一句要求拉近才会拉近。
实测不稳的：面部小疤（像新伤）、双排扣这种大面积的扣子（取景偏远）。

## 细节只收长在角色身上的

发饰、领口、袖口、鞋、配饰。**道具不收**——皮箱、信、刀归 novel-art；角色图要「空手」，拿着东西的手会污染参考图。
