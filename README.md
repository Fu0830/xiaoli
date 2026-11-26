# 小票机 · 结单小票生成器

网页版结单小票机：左边填单，点「打印小票」，小票从打印机纸槽里滚出来。票面是原创的「游戏厅出票」结构——顶端三色出票条、店名印章式标识、手绘蜡笔手柄贴纸、金额主视觉＋SETTLED 方章、实付/余额双色卡、条码带与票尾彩条。单文件 HTML，无需安装、无需联网（除导出 PNG 时加载 html2canvas）。

## 功能

- **点单信息**：俱乐部三选一（誓言俱乐部 / VOW CLUB · 下次见俱乐部 / SEE CLUB · 星明男团 / XM CLUB）、下单时间、币种（£ / ¥ / $ / € / 无前缀）、小票编号；盖章署名随俱乐部自动切换（小梨 / 小银）
- **点单项目**：点单类型 + 副标拼音、陪陪名称、数量、金额；可「再加一项」，逐条编号 `#01 #02 …`，单条时自动隐藏删除键
- **结算**：合计（留空自动按明细汇总，手填则显示「已手动填写合计」并出现恢复自动）、实付金额（留空 = 合计）、卡内余额
- **祝福**：票面寄语、祝福人名（写进页脚「祝 ___ 在某店玩得开心玩得愉快」）、自定义页脚、结单章开关
- **配色**：6 个预设色相，外加色相滑杆（0–360 全色域），夜间模式只反 UI 与纸面外区域、小票始终白纸
- **出票**：打印动画 + 机身抖动、`prefers-reduced-motion` 下直接落位、按钮文案 打印小票 → 出票中… → 再打一张
- **导出**：html2canvas 双倍分辨率存 PNG（`小票-<单号>.png`）；`Ctrl/Cmd + P` 只打印票据，纸宽 80mm
- 所有输入自动存 `localStorage`，刷新不丢

## 本地打开

直接双击 `index.html` 即可（也可用 `python3 -m http.server` 起个本地服务）。

## 发布到 GitHub Pages（3 步）

前提：装好 `git`，有一个 GitHub 账号。

```bash
cd xiaoli
git init -b main
git add index.html README.md
git commit -m "feat: 结单小票机"
git remote add origin https://github.com/<你的用户名>/xiaoli.git
git push -u origin main
```

推送前先在 GitHub 网页端新建同名空仓库（不要勾选自动生成 README，否则要 merge）。

然后进仓库 **Settings → Pages → Source** 选 `Deploy from a branch`，分支 `main`、目录 `/ (root)`，保存。等 1–2 分钟，访问：

```
https://<你的用户名>.github.io/xiaoli/
```

## 改成自己的店

`index.html` 里搜 `DEFAULTS`，改掉默认值（店名、点单类型、陪陪名、寄语、人名、卡内余额等）即可；换插画就替换 `assets/pad.webp` 并重新内联成 base64（或直接把 `STICKER` 换成相对路径引用）。

## 角色主题

左侧第一块是六个三丽鸥角色的主题切换：凯蒂猫 / 酷洛米 / 美乐蒂 / 大耳狗 / 酷企鹅 / 布丁狗。点角色会同时套用该角色的一套配色（配色由 `--hue` 派生，所以顶部色相滑杆仍可继续微调，不会锁死）。票面里角色出现在格纹头图卡上，格纹与整体色板同源。

- 角色图：`assets/ch_kitty.png`、`ch_kuromi.png`、`ch_melody.png`、`ch_cinnamoroll.png`、`ch_badtz.png`、`ch_purin.png`（从 `assets/p1.png` 六宫格裁切、按四角底色去背得到），同目录另有 `ch_*.webp`（高 300px，体积更小）。
- 换角色图：替换对应 png/webp 后重新内联到 `index.html` 的 `var CHARS = {...}` 里，每项字段为 `{ name, hue, h, img }`（`hue` 是该角色的默认色相，`h` 是票面头图里显示的像素高度）。

## 另外六个 IP 主题

除三丽鸥六位，还接入了六个独立 IP：开开、小尾、噜噜、棉球猫、吉伊卡哇、月薪喵。这六个的原始素材是带底色的整幅插画，先用肉眼可见的底色裁切，再经 `mediakit-cli image remove-image-background` 抠成透明贴纸，和三丽鸥一致地贴在格纹底上。

- 素材：`assets/ip_kaikai.png`、`ip_xiaowei.png`、`ip_lulu.png`、`ip_mianqiu.png`、`ip_chii.png`、`ip_yuexin.png`（由 `assets/ip_*_src.png` 原图裁切），同目录另有同名 `webp`。
- 内联方式：`var CHARS` 里每项 `{ name, hue, h, img }`，`h` 是票面内的显示高度，`img` 为内联 webp。
- 吉伊卡哇原图是整版贴纸，抠完按连通域面积过滤掉零碎小贴纸，只留雪人三人组。

## 出票与导出

- 点「打印小票」：先跑 `feed` 打印动画（1.6s），随后加 `is-torn` 进入撕纸阶段——小票抬升、轻微倾斜，槽口留一截纸头，`tearOff` 关键帧 0.95s。
- 三档导出：`存图片`（html2canvas → PNG）、`存视频`（画布重放打印+撕纸，`captureStream` + `MediaRecorder` → WebM）、`存动图`（同一条画布动画，内联 gifenc 编码 → 循环 GIF）。
- 俱乐部三选一，盖章署名随所选俱乐部自动切换：誓言俱乐部 / VOW CLUB（小梨）、下次见俱乐部 / SEE CLUB（小银）、星明男团 / XM CLUB（小梨）。
