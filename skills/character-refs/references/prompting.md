# 提示词：哪些写法是实测出来的

提示词由 `buildPrompt`（`scripts/core.mjs`）按视图拼，**模型不写提示词，只填角色字段**。
改模板之前先读这里——每一条都是某一轮实测失败换来的。

实测环境：Qwen Image 2.1（ComfyUI，`TextEncodeQwenImageEditPlus` 挂参考图）和 codex 内置出图（GPT），
角色覆盖 19 岁民国女学生、70 岁老船夫、28 岁军人、16 岁采茶姑娘，多个种子。

## 总结构

| 视图 | 第一句 | 参考图 | 比例 |
| --- | --- | --- | --- |
| 正面全身（锚点） | Full-body front view: she stands naturally facing the camera… | 无，文生图 | 2:3 |
| 正脸大头照 | Zoom in to an extreme close-up head-and-shoulders portrait, passport-photo framing… | 锚点 | 4:5 |
| 90° 侧面 | Full-body side profile… both face the right edge of the image | 锚点 | 2:3 |
| 背面 | Rotate the camera 180 degrees around her… | 锚点 | 2:3 |
| 细节 | Zoom in to an extreme close-up of only … | 锚点（头发 / 脸 / 领口另挂大头照） | 1:1 |
| 45° 大头照 | 同正脸大头照，再加 45° 的几何描述 | 锚点 | 4:5 |

锚点把全部角色字段写成完整描述；派生图只写「做什么改动」+ 一句短的一致性清单 + 画风层。

## 改图模型只听第一句

参考图的构图力量很强，模型默认照着参考图重画一张差不多的。**只有第一句明确要求改构图，它才改。**

- 大头照：第一句必须是 `Zoom in to an extreme close-up head-and-shoulders portrait, passport-photo framing`，
  再写可量化的取景：头顶贴近上边、下巴在画面高度正中、下边切在锁骨。只写 close-up 会出成半身
- 背面：`Rotate the camera 180 degrees around her`，再写**只有背面看得到的东西**（辫子垂在背后、衣服背面、脚跟），
  反向词禁 face / buttons / pockets。不写这些 Qwen 会出成正面
- 侧面：写全身和头都朝画面右侧、只看得到右半边脸、不看镜头。不写「不看镜头」会回头看
- 45°：使用几何线索——鼻尖指向画面左侧、左耳可见、右耳隐藏。Qwen 上偏小（约 30°），GPT 更准
- 细节：第一句 `Zoom in to an extreme close-up of only <部位>`，再写画面边界（铺满画面、看不到脸 / 只看到脚和脚踝）

## 一致性清单要短，按视图取层

- **不要把锚点的长描述整段重复进派生图。**一句「Keep her exactly the same person as in the reference image: 脸、发型、上装、下装」就够
- **大头照只写脸、头发、上装。**写了下装和鞋，模型就把镜头拉远去画它们（实测：带上全部服装和四个细节，大头照出成了半身）
- **细节图不列整套服装**，只写一句 `Same person, same clothing and materials as in the reference image`。
  第一轮细节图 8/8 失败，就是因为一致性那段列了整套衣服，模型照着画了整个人
- 细节图反向词禁 face / head / portrait / full body / standing figure / wide shot / medium shot；
  挨着脸的细节（头发、脸上的东西）只禁 full body / whole face

## 写实（画风层「方案四」）

用户拍板的基线（2026-09-26）。整段在 `DEFAULT_LOOK`，`look-template` 打印出来：

- 不写 photorealistic、超高清、浅景深——前者会往 CG 渲染带。直接写 `A real photograph, not a render` + 全画幅、85mm、f/5.6、真实白平衡、不修图不美颜、轻微颗粒
- 光写成有方向的主光 + 弱补光（脸和衣服有体积），背景仍是打到纯白的无缝背景纸
- 反向词压 AI 感：CGI、塑料皮肤、蜡质皮肤、娃娃脸、过度对称、HDR、过饱和、平光
- **皮肤粗糙程度不放画风层**，放角色的 `skin`：统一写「泛红、小瑕疵」会让年轻角色出成病容
- **写光的效果，不写光源**：写 window 会把窗户画进画面

## 细节图露出的皮肤

Qwen 画领口、袖口、脚踝时，露出来的皮肤容易画老（没有脸做参照，年龄感丢了）。
领口、袖口、鞋三类细节加了一句 `Any visible skin is the skin of a 16-year-old.`（年龄取角色字段）。
16 岁样例上袖口和脚踝的皮肤都对了。

## 已知短板

- 小疤痕：写成「愈合多年的旧疤：细、平、略浅、几乎不凸」有改善，仍偶尔像缝过线的新伤
- 双排扣这种大面积细节：两个种子取景都偏远
- 细节部位写太大（「斜襟和一排扣子」）会把整个前胸框进来——写小（「小立领和喉下一颗扣」）
- Qwen 的领口细节会把粗布画出褶皱反光、像丝绒（两个角色都出现过）；GPT 出同一张材质是对的，领口不满意就 `--model codex` 重出这一张
- 同一组派生图里混用 Qwen 与 GPT：衔接得上，细节图之间有轻微亮度差

## 参考图怎么挂

- **一张图只参考锚点**，头发 / 脸 / 领口细节再加确认过的大头照。图与图之间不链式参考——链式会累积漂移，也会让单张重出牵连一串
- 大头照缺失或过期时，这几张退回只挂锚点，不阻塞，记录里留注
- 不用拼好的多视图大图当参考：Qwen 能认出是同一人，但会把景别往全身带、脸被摊小；分开挂最像
