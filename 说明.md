# ClassicPack —— 自定义物品资源包（说明文档）

> 位置：`D:\mcserver\大逃杀【29589】\源码\资源包\`
> 目标版本：服务端 Paper **MC 26.2**（`lobby\versions\26.2\paper-26.2.jar`）
> 生成日期：本次任务执行时（脚本可随时重跑，见下文）

---

## 1. 这个包做了什么

用**程序化改色原版贴图**的方式做了 5 个自定义物品图标。命名空间统一是 **`classic`**，
所以游戏里/插件里要引用的 **模型 key** 是：

| # | 模型 key（必须完全一致） | 外观 | 来源贴图 |
|---|---|---|---|
| 1 | `classic:furnace_pickaxe` | 铁镐整体染成熔炉红（铁头炉火红/橙，木柄烧成暗红木炭） | 原版 `iron_pickaxe.png` |
| 2 | `classic:emerald_sword` | 钻石剑染成绿宝石色（剑身 6 级青色阶 → 6 级绿色阶，**剑柄木料保持不变**） | 原版 `diamond_sword.png` |
| 3 | `classic:diamond_golden_apple` | 金苹果染成钻石色（青白/浅蓝，9 级金阶 → 9 级冰蓝阶） | 原版 `golden_apple.png` |
| 4 | `classic:enchanted_diamond_golden_apple` | 同源但**更亮 + 明显偏紫罗兰**（暗部 `#2B165E` 紫靛 → 亮部 `#FAF0FF` 淡紫白） | 原版 `golden_apple.png` |
| 5 | `classic:bridge_egg` | 鸡蛋上加一道**白色羊毛横带**（第 7、8 行，按蛋身 alpha 实测居中） | 原版 `egg.png` |

第 4 个和第 3 个是"两个都要、且要能一眼分开"，所以第 4 个走的是**紫罗兰**色相而
不是简单调亮，两个苹果摆一起颜色差异很明显。

### 用法（插件侧）

物品的 `item_model`（1.21.4+ 的物品模型组件）填对应的模型 key 即可，例如：

```java
meta.setItemModel(NamespacedKey.fromString("classic:emerald_sword"));
// 或 1.21.4+ 的 ItemMeta#setItemModel(NamespacedKey)
// 数据包/命令里则是： /give @s diamond_sword[minecraft:item_model="classic:emerald_sword"]
```

> 本次任务**只做资源包**，没有改任何插件/Java 代码。

### ✅ 已和现有 `ClassicItems` 插件做过 key 对上号的核对

顺手读了（**只读，没改**）`源码\物品插件\config.yml`，里面现有的
`item-model` 配置和本包的模型 key **完全一致**，不用再改插件侧配置：

| config.yml 里的键 | 配置里的 `item-model` | 本包是否提供 |
|---|---|---|
| `bridge_egg` | `classic:bridge_egg` | ✅ |
| `furnace_pickaxe` | `classic:furnace_pickaxe` | ✅ |
| `emerald_sword` | `classic:emerald_sword` | ✅ |
| `diamond_golden_apple` | `classic:diamond_golden_apple` | ✅ |
| `enchanted_diamond_golden_apple` | `classic:enchanted_diamond_golden_apple` | ✅ |
| `enchanted_fireball` | `""`（空 = 用原版火焰弹 + 光效） | 本包**故意不做**，符合配置意图 |

另外这几件在 config.yml 里都开了 `glint: true`，所以客户端会在我们的贴图上面
再叠一层原版附魔闪光——`enchanted_diamond_golden_apple` 就是"亮紫苹果 + 闪光"，
和 `diamond_golden_apple` 的区分会更明显。

---

## 2. 包的结构

```
源码\资源包\
  ClassicPack.zip              ← 交付的成品包（zip 根目录直接是 pack.mcmeta 和 assets\）
  ClassicPack.zip.sha1         ← zip 的 sha1（纯文本一行）
  pack\                        ← 源目录，以后改贴图/加物品直接改这里
    pack.mcmeta
    pack.png                   ← 包图标（128x128，5 个图标横排）
    assets\classic\
      items\<name>.json        ← 物品定义，指向 classic:item/<name>
      models\item\<name>.json  ← {"parent":"minecraft:item/handheld|generated","textures":{"layer0":"classic:item/<name>"}}
      textures\item\<name>.png ← 16x16 改色贴图
  改色.py                      ← 贴图改色脚本
  打包.py                      ← 生成 pack.png + 打 zip + sha1 + 全套自检
  _vanilla\                    ← 原版素材工作区（客户端 jar + 抽出来的原版贴图/json）
  对照图.png                    ← 左原版 / 右改色 的 5 组对照图（便于肉眼验收）
```

### `items/*.json` 是照抄本版本原版格式写的（**已实开原版文件确认**）

我解开了 `_vanilla\client-26.2.jar` 里的 `assets/minecraft/items/diamond_sword.json`，
26.2 的真实格式是「`model` 对象 + `type: minecraft:model`」：

```json
{
  "model": {
    "type": "minecraft:model",
    "model": "minecraft:item/diamond_sword"
  }
}
```

`assets/minecraft/items/golden_apple.json` 结构完全一样（只有 model 值不同）。
我们的 5 个 `items/*.json` 就照这个结构写，只把值换成 `classic:item/<name>`。

`models/item/*.json` 的 parent：
- 镐、剑用 `minecraft:item/handheld` —— **和原版 `models/item/diamond_sword.json`、
  `iron_pickaxe.json` 一致**（handheld = generated + 手持时的旋转/缩放），
  这样拿在手里/挂在展示框里的姿态和原版铁镐钻石剑一样。
- 两个苹果和鸡蛋用 `minecraft:item/generated`（和原版 `models/item/golden_apple.json` 一致）。

> 任务书里给的模板写的是全部用 `generated`。镐和剑我用 `handheld`，原因是原版这两件
> 就是 `handheld`，用 `generated` 会导致手持姿态不对（平摊在手上）。这是有意的偏离。

---

## 3. 怎么改贴图 / 怎么重跑

### 原图在哪
`源码\资源包\_vanilla\` 下：

```
_vanilla\client-26.2.jar                                     ← 官方客户端 jar（39,193,383 字节）
_vanilla\assets\minecraft\textures\item\{iron_pickaxe,diamond_sword,golden_apple,egg}.png
_vanilla\assets\minecraft\items\{iron_pickaxe,diamond_sword,golden_apple,enchanted_golden_apple,egg}.json
_vanilla\version.json                                        ← 从 jar 根目录抽出来的
```

**客户端 jar 来源**（本地 `.minecraft\versions\26.2\` 里只有 `26.2.json`，没有 jar，
`lobby\cache\mojang_26.2.jar` 按任务书提示是服务端 jar 就没有去用它）：
先读本地已有的 `C:\Users\DELL\AppData\Roaming\.minecraft\versions\26.2\26.2.json`
拿到 `downloads.client.url`，再 `Invoke-WebRequest -OutFile` 下载：

```
https://piston-data.mojang.com/v1/objects/2dc72797acbc1b63fc16a11c4ac393605f453754/client.jar
```

（这条 URL 与 `version_manifest_v2.json` → `26.2` → `url` → `downloads.client.url` 是同一个东西；
本机 `Invoke-WebRequest` 抓 manifest 时会弹非交互模式报错，所以直接用了本地那份版本 json 里的地址。）

贴图要从 jar 里抽出来，命令（PowerShell，Python 单行）：

```powershell
python -c "import zipfile,os;z=zipfile.ZipFile(r'D:\mcserver\大逃杀【29589】\源码\资源包\_vanilla\client-26.2.jar');[open(os.path.join(r'D:\mcserver\大逃杀【29589】\源码\资源包\_vanilla',n.replace('/',os.sep)),'wb').write(z.read(n)) for n in ['assets/minecraft/textures/item/iron_pickaxe.png']]"
```

### 改色脚本：`改色.py`

```powershell
python "D:\mcserver\大逃杀【29589】\源码\资源包\改色.py"
```

它做的是 **调色板级精确映射**：原版这些物品贴图每张只有 6~12 种颜色（alpha 非 0 即 255），
所以脚本里每张图写了一张「源色 RGB → 目标色 RGB」的字典，逐像素替换颜色。
- **尺寸严格 16×16，不做任何缩放/旋转**
- **alpha 逐像素原样搬运**（脚本里有 assert，改不动就报错）
- **像素轮廓完全不变**（不重绘，只换颜色）
- 亮度关系靠"目标色阶自己保持单调"来保留：源色越亮 → 目标色也越亮

改色思路不是简单涂纯色，而是**分级映射**，例如钻石剑剑身的 6 级青色阶
`#082520 / #0E3F36 / #156355 / #1E8A77 / #2BC7AC / #33EBCB / #A4FDF0`
→ 6 级绿色阶
`#062E14 / #0A4E22 / #0F7432 / #17A048 / #21CD60 / #2EE87A / #A8FFC4`。

想换颜色：直接改 `改色.py` 顶部的映射表（`FURNACE_PICKAXE` / `EMERALD_SWORD` /
`DIAMOND_APPLE` / `ENCHANTED_DIAMOND_APPLE`，羊毛带颜色是 `WOOL_BRIGHT/SHADE/EDGE`），
重跑脚本即可。`bridge_egg` 的横带行是**算出来的**（取蛋身不透明行的竖直中心 2 行，实测是第 7、8 行），
改 `pick_band_rows(probe, thickness=2)` 的 thickness 就能加宽/变窄。

### 打包 + 自检：`打包.py`

```powershell
python "D:\mcserver\大逃杀【29589】\源码\资源包\打包.py"
```

它会：生成 `pack\pack.png` → 把 `pack\` 打成 `ClassicPack.zip`
（**zip 根目录直接是 `pack.mcmeta` 和 `assets\`，没有多套一层文件夹**）→ 算 sha1
并写入 `ClassicPack.zip.sha1` → 打印全套自检（贴图尺寸/alpha、zip 条目、`json.load` 解析、
`items` 与 `models` 的命名空间一致性）。

> ⚠️ **改过贴图之后 sha1 一定会变**，记得重新跑 `打包.py`，
> 并把新的 sha1 填到下面第 4 节和 `server.properties` 里。

---

## 4. `pack_format` 是怎么定的（**这个是硬证据，不是猜的**）

原版客户端 jar **根目录**有一个 `version.json`，我把它抽出来看了，里面直接写了：

```json
{
    "id": "26.2",
    "name": "26.2",
    "world_version": 4903,
    "series_id": "main",
    "protocol_version": 776,
    "pack_version": {
        "resource_major": 88,
        "resource_minor": 0,
        "data_major": 107,
        "data_minor": 1
    },
    "java_component": "java-runtime-epsilon",
    "java_version": 25,
    "stable": true,
    "use_editor": false
}
```

所以 **26.2 的资源包格式号 = `pack_version.resource_major` = `88`**（`resource_minor` = 0）。
`pack.mcmeta` 里就写的这个：

```json
{
  "pack": {
    "pack_format": 88,
    "supported_formats": { "min_inclusive": 1, "max_inclusive": 999 },
    "description": "§bClassic 自定义物品包 §7| §f5 个改色物品图标"
  }
}
```

**把握程度：高。** 依据是官方客户端 jar 自带的 `version.json`（同一份文件里的
`protocol_version`=776、`world_version`=4903 与 26.2 对得上，说明确实是这个版本的元数据）。
唯一保留意见：`pack_version` 是 Mojang 给外部工具（服务端/启动器）看的元数据文件，
我没有 26.2 客户端源码去交叉验证它和运行时硬编码的 `SharedConstants` 是不是同一个数。
所以按任务书建议**额外写了很宽的 `supported_formats`（1~999）**：
即使 88 有偏差，客户端也只会提示"为其他版本制作"，而不会拒绝加载。

---

## 5. 怎么托管给客户端下载（重要）

资源包必须让**客户端**能直接下到，所以要么自己开 HTTP，要么放能直链的地方。

### 5.1 `server.properties`（每个子服都要填）

在需要生效的服务器实例的 `server.properties` 里加/改这两行：

```properties
resource-pack=https://你的域名或IP/ClassicPack.zip
resource-pack-sha1=256ddf3c54ab1ea7f94225d0f563cdb844485466
resource-pack-prompt={"text":"建议启用 Classic 自定义物品包"}
```

- `resource-pack-sha1` 必须和 zip 的 sha1 **完全一致**（小写十六进制，一行）。
  填错的后果：客户端下载完校验失败，会当成损坏包丢掉，报"下载失败"。
- 改完要**重启该子服**才生效（`server.properties` 是启动时读的）。
  本任务没有替你重启任何进程，这一步需要你自己做。

### 5.2 URL 必须满足的条件

| 要求 | 说明 |
|---|---|
| **直链** | `https://host/ClassicPack.zip` 这种，点开/请求就直接返回 zip 字节流 |
| ❌ 网盘分享页 **不行** | 百度网盘/夸克/蓝奏云/Google Drive 的分享页面返回的是 HTML，客户端解析不了，会直接判定下载失败 |
| ❌ 需要登录/Cookie 的 **不行** | 客户端不会带你的登录态 |
| ✅ 可以用 | 自建 Nginx/Apache/Caddy 静态目录、对象存储（OSS/COS/S3/R2）的**公共读**外链、GitHub Release 直链、Cloudflare R2 等 |
| 注意 Content-Type | 有的平台会返回 `application/octet-stream` 或 `text/html`。Minecraft 客户端看的是内容不是头，但**别让它返回 HTML** |
| 注意端口/防火墙 | URL 里的端口要对客户端可达；Velocity 群组下每个子服各自下发，建议用同一个公网可达的地址 |

自建静态服务最小做法（**不要用来替换现有服，只是举例**）：

```powershell
# 在放 zip 的目录里起个临时静态服务（仅示例，本任务没有执行）
python -m http.server 8080
# 然后 resource-pack=http://你的公网IP:8080/ClassicPack.zip
```

> 用 `http://`（非 https）时，部分客户端版本会二次弹窗确认，属于正常现象。

### 5.3 本包当前的 sha1

```
文件: 源码\资源包\ClassicPack.zip
大小: 7578 字节
sha1: 256ddf3c54ab1ea7f94225d0f563cdb844485466
```

（同一份值也写在本目录的 `ClassicPack.zip.sha1` 里，直接用记事本打开复制即可。）

**改过 `pack\` 里任何文件后，这个 sha1 就不对了**——
重新跑 `python 打包.py`，用新输出的 sha1 覆盖 `server.properties` 和本文档。

---

## 6. 验收自检结果（都是真跑出来的）

`打包.py` 和独立自检的输出（存档在 `自检输出.txt`）：

- **A) 5 个 PNG**：全部 `size=(16, 16)`，mode `RGBA`，alpha 取值 `[0, 255]`（有 alpha 通道）✅
- **B) zip 结构**：`pack.mcmeta` 在条目 #1，`pack.png` #2，之后是 `assets/...`；
  共 17 条目；**没有任何条目以 `pack/` 开头**（即没有多套一层文件夹）✅
  `zipfile.testzip()` CRC 自检通过 ✅
- **C) JSON**：`pack.mcmeta` + 5 个 `items/*.json` + 5 个 `models/item/*.json`，
  全部用 `json.load()` **真解析**通过（并且是从 zip 里读出来再解析一遍）✅
- **D) zip 内 PNG 与磁盘 PNG 逐字节一致** ✅
- **E) alpha 与原版逐像素相同，差异像素 0**；不透明像素数与原版完全一致
  （68 / 84 / 114 / 114 / 90）——证明像素轮廓一个没动 ✅

---

## 7. 已知限制 / 没做的事

- **没有重启任何进程，没有改任何 `.java`/`.ps1`/`.bat`/插件/`plugins` 目录/`.jar`**，
  所有产出都在 `源码\资源包\` 里。
- 没有真机启动客户端做视觉验收（无法在服务器上开游戏）。
  已用 `对照图.png`（左原版/右改色）和浅色背景渲染做过肉眼比对，形状和明暗层次都保住了。
- `enchanted_diamond_golden_apple` **没有**加附魔光效（`minecraft:enchantment_glint_override`
  之类），只是把颜色做亮做紫。如果需要"自带附魔闪光"，那要在**物品组件**侧加
  `enchantment_glint_override=true`，属于插件/数据包的活，不是资源包能单独决定的。
- `pack.mcmeta` 的 description 里用了 `§b` 颜色代码。若你的客户端对资源包描述里的
  `§` 处理异常（极少见），把它删掉即可，不影响加载。
