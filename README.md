# 用 GitHub Pages 托管资源包 —— 操作步骤

这个目录里就是**要上传到 GitHub 的全部内容**，不要再改结构。

```
github-pages\
  ClassicPack.zip    ← 资源包本体 (7578 字节, sha1 = 256ddf3c54ab1ea7f94225d0f563cdb844485466)
  .nojekyll          ← 阻止 GitHub Pages 用 Jekyll 处理仓库 (纯静态资源不需要它)
  index.html         ← 站点根页, 免得访问根目录 404
```

## 一、为什么选 Pages（实测依据）

```
github.io IP 段 (Fastly CDN, 185.199.108~111.153)
  443 端口全部能连上
  https://pages-themes.github.io/minimal/   HTTP 200  0.9 秒
  https://github.io/                        HTTP 200  1.8 秒
```

对比 GitHub 的**另外两套**基础设施（不要混为一谈）：

| 方式 | 国内实测 |
|---|---|
| **GitHub Pages**（Fastly CDN） | ✅ 0.9 秒，DNS 未污染 |
| `github.com`（网页/仓库） | ⚠️ 走 20.205.243.166，慢 |
| `raw.githubusercontent.com` | ⚠️ 185.199.108~111.**133**，限流 |
| Releases 直链 | ⚠️ 302 跳到 objects.githubusercontent.com，常超时 |

**结论：Pages 是这三条里最稳的，用它对。**

## 二、上传步骤（不用装 git，网页拖拽即可）

1. 登录 GitHub → 右上角 **+** → **New repository**
2. 仓库名随便，例如 `classic-pack`；**必须选 Public**（私有仓库的资源客户端拿不到）
3. 不要勾 "Add a README"（避免多一次提交），直接 **Create repository**
4. 进入空仓库页 → 点 **uploading an existing file**
5. 把本目录里**三个文件全部**拖进去（`.nojekyll` 是隐藏文件，Windows 资源管理器要开"显示隐藏项目"；如果拖不进去可以先跳过，它只影响 Jekyll，不影响 zip 下载）
6. Commit changes
7. 仓库 → **Settings** → 左侧 **Pages**
   - Source: **Deploy from a branch**
   - Branch: **main** / 目录 **/ (root)**
   - Save
8. 等 **约 1 分钟**（Pages 首次发布需要构建时间）
9. 你的地址就是：

```
https://<你的用户名>.github.io/classic-pack/ClassicPack.zip
```

> 如果仓库名就叫 `<你的用户名>.github.io`，地址则变成
> `https://<你的用户名>.github.io/ClassicPack.zip`（没有中间的仓库名）

## 三、填进服务器（**这一步不能手抄**）

双击项目根目录的 **`设置资源包.bat`**，粘贴上面那个地址。

它会**先把 URL 真下一次**，校验：
- 下到的是不是 zip（看开头是不是 `PK`，能挡住"填了网页地址"）
- sha1 是否和本地包一致（能挡住"传错了包/传了旧包"）

两项都过才会写进 6 个实例的 `server.properties`，然后回读校验一遍。**任何一步不对就一个字节都不写。**

写完**重启 6 个实例**生效（资源包是连接建立时下发的）。

## 四、两件容易踩的事

1. **每次改包，sha1 必然变**
   重跑 `python 打包.py` 出新 zip → 覆盖 GitHub 上的 `ClassicPack.zip` → **再用 `设置资源包.bat` 走一遍**（sha1 它自己现算，不用你抄）。

2. **Pages 有发布延迟**
   刚 commit 完立刻下载可能拿到旧文件或 404。等 1 分钟；如果 `设置资源包.bat` 报"下到的不是 zip"，八成是拿到了 Pages 的 404 页面。

## 五、切到 Pages 之后可以退休的东西

当前用的是自建方案（frp 隧道 + 本机静态服务）：

```
resource-pack=http://42.202.37.94:36891/ClassicPack.zip
```

换成 Pages 之后，这些**都不再需要**：
- `启动资源包服务.bat`（那个常驻窗口）
- frpc 里 `[classicpack]` 那条隧道

留着也行（多一条备用线路），但没必要一直开着。
