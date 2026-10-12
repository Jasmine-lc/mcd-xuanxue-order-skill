# 🔮 麦麦赛博算命 Skill（mcd-xuanxue-order-skill）

> 生辰八字 × 今日五行，掐指一算，为你钦点今日的麦当劳套餐。
> 一个把中国传统八字五行玄学与麦当劳**真实菜单**融合的趣味点餐 Skill。
>
> 麦当劳程序员创意开发大赛参赛作品 · 基于麦当劳 MCP 能力开发。

---

## ✨ 这是什么

「麦麦赛博算命」是一个趣味点餐 Skill：

1. 根据你的**出生日期**推算八字日柱，得到你的**本命五行**；
2. 用麦当劳 MCP 的 `now-time-info` 取**当天日期**，推算**今日当令五行**；
3. 结合两者的生克关系，算出「**今日宜补的五行**」；
4. 从麦当劳 MCP 拉取**真实菜单与营养成分**，把每道餐品归入五行，
   挑出最契合今日五行的餐品，生成一张「**今日玄学套餐卡**」。

两种使用情况：

- **情况 1 · 有生日**：按「本命五行 × 今日五行」的生克关系给出宜补五行。
- **情况 2 · 无生日**：直接按「今日五行」为你奉上今日能量餐。

> 🔖 本项目为趣味娱乐应用，输出仅供娱乐与参考，不构成医疗、营养或任何专业建议；
> 餐品、价格与供应状态一律以麦当劳官方实时结果为准。

---

## 🎯 目标用户

- 有点「选择困难」、每天纠结吃啥的麦门信徒；
- 喜欢星座/MBTI/八字这类轻玄学、爱分享运势卡片的年轻人；
- 想看看「AI + MCP + 传统文化」能怎么混搭的开发者。

---

## 🧩 项目结构

```
mcd-xuanxue-order-skill/
├── README.md                   # 本文件
├── CONTEST_DECLARATION.md      # 参赛声明（官方原文，未改动）
├── MCP_INTEGRATION.md          # 麦当劳 MCP 集成说明（工具/流程/价值）
├── SKILL.md                    # ⭐ Skill 主指令（核心，载入 AI 客户端即用）
├── mcp-config.example.json     # 脱敏 MCP 配置示例（仅环境变量占位符）
├── .env.example                # 环境变量示例（Token 占位符）
├── requirements.txt            # Python 依赖
├── src/                        # 源代码
│   ├── bazi.py                 # 八字/干支/五行推算（确定性历法算法）
│   ├── wuxing_mapping.py       # 五行↔餐品 营养+象征映射与打分
│   ├── proxy_server.py         # 本地 FastAPI MCP 代理 / 五行推算 API（可选）
│   └── tests/                  # 单元测试（pytest）
└── web/
    └── preview.html            # 纯静态效果预览（双击即开，无需配置）
```

---

## 🚀 快速开始

### 零、申请麦当劳 MCP Token（真实数据的前提）

1. 打开麦当劳 MCP 开放平台官网：<https://open.mcd.cn/mcp>
2. 用手机号登录 → 进入「控制台」→ 点「激活」申请 **MCP Token**。
3. 复制 Token 备用（**切勿把 Token 写进任何会提交到 Git 的文件**）。

> 详见官方指南：<https://github.com/M-China/mcd-mcp-server>

### 方式 A：作为「纯 Skill」在 AI 客户端中使用（推荐）

本项目的主体是 [`SKILL.md`](./SKILL.md)——一套可直接驱动的 Skill 指令。

1. 在支持 MCP 的客户端（Cherry Studio / Cursor / Trae / Kiro / VSCode / WorkBuddy 等）
   中，按 [`mcp-config.example.json`](./mcp-config.example.json) 配置 `mcd-mcp` 连接器，
   把 `${MCD_MCP_TOKEN}` 替换为你的真实 Token（填在客户端配置里，不要提交到仓库）。
2. 把 [`SKILL.md`](./SKILL.md) 的内容作为**系统提示 / 自定义指令**载入该客户端。
3. 直接对话，例如：
   - 「我是 1995 年 8 月 15 日生的，今天中午吃啥？我在上海南京东路。」
   - 「不想说生日，就想知道今天适合吃麦当劳的什么。」

AI 会按 SKILL.md 的流程：取今日 → 推五行 → 定位门店 → 拉真实菜单与营养 →
五行归类打分 → 查套餐组成 → 产出「今日玄学套餐卡」。

### 方式 B：零配置看效果（纯静态预览）

直接**双击打开** [`web/preview.html`](./web/preview.html)。
无需联网、无需 Token，输入生日即可看到「今日玄学套餐卡」的效果。
此页所有推算（八字五行 + 餐品打分）在浏览器本地完成，餐品为演示数据；
真实菜单请走方式 A（Skill + MCP）。

### 方式 C（可选，面向开发者）：本地 MCP 代理 / 五行推算 API

项目附带一个本地 FastAPI 服务 [`src/proxy_server.py`](./src/proxy_server.py)，
可作为**安全连接 MCP 的本地代理**（从环境变量读 Token、注入 `Authorization`、
规避 CORS，Token 不入前端/仓库），同时直接暴露复用的五行推算与推荐 API，
便于二次开发或自建前端。

```powershell
# 1) 安装依赖
pip install -r requirements.txt

# 2) 配置 Token（可选，仅在需要真实连 MCP 时）
$env:MCD_MCP_TOKEN = "你的真实MCPToken"   # 或复制 .env.example 为 .env 填入

# 3) 启动服务（从项目根目录运行）
uvicorn src.proxy_server:app --port 8000
```

主要端点：

- `GET  /api/health` — 查看代理状态与 Token 是否已配置（不回显 Token）。
- `GET  /api/fortune?birthday=YYYY-MM-DD` — 五行命理推算（有/无生日两种情况）。
- `GET  /api/real-recommend?birthday=YYYY-MM-DD` — **一站式真实推荐**：自动拉取 MCP
  `list-nutrition-foods` 真实营养表 → 解析 → 五行推算 → 打分排序，直接返回今日套餐（需有效 Token）。
- `POST /api/recommend` — 传入餐品列表，按目标五行打分排序。
- `POST /api/mcp/tool/{tool_name}` — 注入鉴权后转发到麦当劳 MCP 工具。
（此页餐品为**演示数据**；真实菜单请用方式 B。）

---

## 🀄 五行推算怎么算的

- **日柱干支**：用儒略日数（JDN）确定性推算，锚点为 **2000-01-01＝戊午日**
  （已对权威干支万年历逐日核验），`日柱序号 = (JDN + 49) mod 60`，甲子=0。
- **本命五行**：取日柱天干的五行（甲乙→木，丙丁→火，戊己→土，庚辛→金，壬癸→水）。
- **今日宜补**：按「今日五行」对「本命五行」的生克关系判定——
  - 今日克你 → 补「生你」之五行化解；
  - 你生今日（被泄）→ 补同类五行扶助；
  - 其余顺遂 → 顺势补今日五行添彩。

> 说明：严谨排盘还涉及出生时辰、真太阳时与子时换日等；本项目面向趣味场景，
> 按公历日界处理日柱，已足够好玩且结果稳定可复现。

## 🍔 五行怎么对应到餐品（营养为主、象征为辅）

| 五行 | 营养画像（主） | 象征（辅） | 示例餐品 |
|---|---|---|---|
| 火 | 高热量 / 高脂 / 高钠 | 辣、烤、红、热 | 麦辣鸡翅、辣堡 |
| 水 | 饮品（液体属性） | 可乐 / 咖啡 / 茶 / 冰 | 可乐、拿铁、麦旋风 |
| 土 | 高碳水主食 | 薯 / 派 / 米 / 堡 | 薯条、苹果派 |
| 金 | 高蛋白 + 中脂、成形酥脆 | 鸡块 / 炸鸡 / 脆 | 麦乐鸡、脆鸡 |
| 木 | 清淡低热、果蔬纤维 | 沙拉 / 蔬菜 / 玉米 / 果 | 田园沙拉、玉米杯 |

**主食/汉堡按「荤料构成」定五行（优先）**：汉堡等主食不一律归土，而是按里面的主料肉分五行——
鸡/禽 → 金（鸡腿堡、麦辣鸡腿汉堡），鱼/虾 → 水（鳕鱼堡、麦香鱼），牛（含安格斯、巨无霸）→ 土，
猪/培根 → 水（猪柳麦满分）。主料优先于配料（培根牛堡仍归土）；无明确荤料的汉堡默认牛肉堡 → 土。

其余餐品（薯条、鸡块、饮品、沙拉、甜品）计算五行得分向量（营养 0.6 + 象征 0.4），取最高项为其五行；
最后按与「今日目标五行」的契合度排序（同五行最优，能「生」目标者次之）。

---

## 🧪 开发与测试

```powershell
pip install -r requirements.txt
python -m pytest            # 运行全部单元测试
python -m src.bazi 1995-08-15   # 命令行小 demo：输出日主与今日五行
```

- `src/tests/test_bazi.py`：日柱推算（含锚点、闰年、世纪边界、60 甲子连续性）。
- `src/tests/test_wuxing_mapping.py`：五行归类与推荐排序。
- `src/tests/test_proxy_server.py`：代理的 Token 注入、401/429 透传、SSE 解析等（httpx 全程 mock，不触网）。

> Web B（preview.html）内嵌的 JS 算法已与 Python 版交叉校验，日柱与推荐排序完全一致。

---

## 🔐 安全与合规

- 仓库内**不含任何真实 Token / 密钥 / 他人个人信息**；
  配置文件 [`mcp-config.example.json`](./mcp-config.example.json) 仅使用 `${MCD_MCP_TOKEN}` 占位符。
- `.env` 已被 [`.gitignore`](./.gitignore) 忽略，Token 只存在于你本机环境变量。
- 真实 MCP 请求由本地代理在你的机器上发起，前端不接触 Token。
- 本项目为参赛者独立开发的趣味作品，**非麦当劳官方产品**；详见 [`CONTEST_DECLARATION.md`](./CONTEST_DECLARATION.md)。

---

## 📎 相关链接

- 麦当劳 MCP 开放平台：<https://open.mcd.cn/mcp>
- 麦当劳 MCP Server 使用指南：<https://github.com/M-China/mcd-mcp-server>
- MCP 集成细节见 [`MCP_INTEGRATION.md`](./MCP_INTEGRATION.md)
