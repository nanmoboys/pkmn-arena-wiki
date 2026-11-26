# 宝可梦竞技场 百科（对战向维基）

站点版本 **v3.70.5（M19）** ｜ 对应 ROM `宝可梦竞技场_v14_m19.gba`，md5 `593b76a0733b16dfc2daaf964d85158d`
收录范围 **全国图鉴 #0001—#0649**，**只讲对战**（不含野生分布／地图／剧情流程）。
游戏数值一律不手写：全部由源码自动抽取，所以维基不会和 ROM 脱节。

## 地址

| 场景 | 地址 | 说明 |
|---|---|---|
| 本地带编辑 | `http://127.0.0.1:8000` | 需先运行 `python wiki/edit_server.py`（可传端口参数）；每页右下角「编辑本页」 |
| 纯本地离线 | 双击 `wiki/site/index.html` | 但**搜索不可用**：`file://` 下浏览器取不到 `search/search_index.json`，要搜索请走上面的 http |
| 局域网 | `http://<本机IP>:8000` | 需把 edit_server 的绑定从 `127.0.0.1` 改成 `0.0.0.0` 并放行防火墙 |
| 公网 | 尚无 | 见下面「部署（免费）」，部署后地址形如 `https://<你的用户名>.github.io/<仓库名>/` |

## 内容规模（本轮实测）

- 站点 1949 个 HTML、site 目录 2645 个文件；`search_index.json` 5.4 MB
- 招式页 954 ｜ 宝可梦页 649（#0001—#0649 无缺号）｜ 特性页 339 ｜ 道具 763（= 状态编辑器「一键添加道具」可得的那批）
- 属性相性按本作构型求值后并入每只宝可梦的详情页；「界面与快捷操作」按要求不收录

## 目录

```
wiki/
  mkdocs.yml            站点配置（MkDocs + Material，中文搜索分词）
  docs/                 生成的 Markdown 页面
    index.md            首页（入口表）
    rules.md            对战条款（顶部 <style> 里就是可编辑的空隙数值）
    pokemon/            649 只详情页 + index.md 列表
    moves/              954 条招式详情页 + index.md 总表
    abilities/          339 条特性页 + index.md 总表
    items/index.md      道具总表
    img/pokemon/        图鉴像素图（游戏内 front sprite）
    stylesheets/        由生成器写出的 extra.css
  site/                 mkdocs build 产物，可直接托管
  overrides/            用户接管层（见下）
  data/wiki.json        抽取器输出（唯一数据源）
  edit_server.py        本地预览 + 实时编辑 + 自动重建
```

## 三步重新生成（改了游戏源码之后）

```powershell
# 1) 抽数据（必须在 WSL，源码树在那儿）
wsl -d Ubuntu -e python3 /mnt/c/Users/86193/AccioWork/2026-09-17-19-41-44-786-ccb82055/v24_extract.py

# 2) 生成页面（Windows python，需要 openpyxl 读桌面那份 xlsx 的人工层）
python v30_pages.py

# 3) 构建
python -m mkdocs build -f wiki/mkdocs.yml
```

跑完必看的三条自检（打印在生成器末尾）：

1. 抽取器：`覆盖图鉴号: 649 ｜ 缺号: 0`、`未解析表达式` 为空或可解释
2. 生成器：`合法招式池为 0 的图鉴号: 0 个`
3. 生成器：`页面原始常量残留: 0 处`（页面里不许出现 `TARGET_*` / `MOVE_EFFECT_*` 这类原始常量）

## 实时编辑（edit_server.py）

- 数据页（pokemon / moves / abilities / items）：正文来自源码，改数值要改游戏源码；
  你保存的文字进 `overrides/<同路径>.md`，重新生成后**拼在该页末尾**（当批注用）。
- 对战条款页 `rules.md`：**整页接管**——一旦保存过就以 `overrides/rules.md` 为准，
  重跑生成器不会覆盖；要恢复自动生成，删掉 `wiki/overrides/rules.md` 即可。
  页面顶部的 `<style>` 数字就是小节/条款空隙，直接改数值最省事。
- 首次编辑生成页时，编辑框会自动装入当前生成正文（不是空盒子）。

## 部署（免费）

**方式 A：GitHub Pages + Actions（推荐）**
1. 新建仓库，把 `wiki/mkdocs.yml` 与 `wiki/docs/` 放到仓库根。
2. Settings → Pages → Source 选 **GitHub Actions**。
3. 加 `.github/workflows/deploy.yml`，构建 `pip install mkdocs-material && mkdocs build`，
   再用 `actions/upload-pages-artifact` + `actions/deploy-pages` 发布 `site/`。

**方式 B：`mkdocs gh-deploy`**（把 `site/` 推到 `gh-pages` 分支）
```powershell
python -m mkdocs build -f wiki/mkdocs.yml
python -m mkdocs gh-deploy -f wiki/mkdocs.yml
```

**方式 C：Cloudflare Pages / Netlify**：直接把 `wiki/site/` 整个目录拖上去即可（纯静态）。

## 已知限制

- 招式／特性描述取树内原文，本作没填的显示「本作未填描述」。
- 招式池两处来源合并：`showdown_learnsets.c`（按物种枚举下标）为主，
  `all_learnables.json` 兜底宏生成条目（盖诺赛克特、未知图腾、阿尔宙斯等）。
- 形态：同名形态合并到该图鉴号；超级进化／极巨化／自定义 M 形态一律不收录。
- 单属性宝可梦在源码里是同一属性写两遍，数据层已去重，相性不会被平方。
- 列表页尚无分页；伤害计算器／速度线未做。
