# 石家庄非遗文化 · AI 艺术共创平台

融合「石家庄非物质文化遗产展示」与「AI 艺术创作」的 Web 平台。
技术栈：**TypeScript + SCSS + TailwindCSS + Next.js（App Router）**，可视化使用 **ECharts** 与 **Leaflet**，AI 智能体接入 **DeepSeek**。

视觉主题为**米色暖调 · 非遗风格**（参考图色板：米白宣纸底 + 暖沙金渐变 + 朱红/珊瑚点缀），
全站带剪纸角花、回纹标题、朱红印章与动态背景（暖光漂移 / 宣纸受光 / 纸纹颗粒 / 剪纸纹样 / 金粉粒子）。

> 面向使用者的完整操作指引（启动、页面导览、AI 配置、内容维护、常见问题）见 **[使用说明.md](使用说明.md)**。

---

## 一、快速开始

```bash
# 1. 安装依赖
npm install

# 2. 配置 DeepSeek（可选，不配置会自动降级为「本地知识库模式」）
copy .env.local.example .env.local
#   然后编辑 .env.local 填入 DEEPSEEK_API_KEY

# 3. 启动开发服务器
npm run dev
# 打开 http://localhost:3000
```

生产构建：`npm run build && npm start`

> 依赖安装如果遇到 npm 缓存目录无权限，可以指定项目内缓存：
> `npm install --cache ./.npm-cache`

### 素材处理（已执行过，重复执行也没问题）

原始素材解压到 `素材源/非遗项目/` 后，执行：

```bash
npm run assets
```

脚本 `scripts/prepare-assets.ps1` 会把中文、带空格、序号混乱的原始文件名，
规范化复制到 `public/ich/<项目 slug>/`（封面 `cover`、图集 `g1…`、AI 共创图 `ai1…`、影像 `video.mp4`）。

---

## 二、目录结构

```
src/
├─ app/
│  ├─ layout.tsx                 # 全站布局：导航 + 页脚 + 登录态 Provider
│  ├─ page.tsx                   # 首页（平台简介 / 项目介绍 / 十大非遗 / 精选 / 评价 / 推荐作品）
│  ├─ create/page.tsx            # AI 创作（智能体接入位 + 共创工作台）
│  ├─ map/page.tsx               # 文化地图（离线行政区划图 + 实心小点位 + 统计图表 + 点位侧滑详情）
│  ├─ heritage/page.tsx          # 非遗列表（项目 / AI 共创作品 + 检索 + 筛选 + 只看收藏）
│  ├─ heritage/[id]/page.tsx     # 非遗详情（介绍 / 列入原因 / 意义 / 传承 / 图集 / 视频 / AI 作品）
│  ├─ news/page.tsx              # 相关非遗（全国著名非遗 + 非遗影像 + 政策新闻 + 从石家庄到全国）
│  ├─ news/[id]/page.tsx         # 内容详情
│  ├─ me/page.tsx                # 个人中心（用户信息 / 我的收藏 / 浏览历史）
│  ├─ login/page.tsx             # 登录 / 注册
│  └─ api/chat/route.ts          # DeepSeek 转发接口（服务端持有 API Key）
├─ components/
│  ├─ AIAgentSlot.tsx            # ★ 外部智能体接入位（已接入 DeepSeek）
│  ├─ HeroShowcase.tsx           # 首页首屏动态轮播（可点击跳转）
│  ├─ GeoMap.tsx                 # 离线行政区划地图（本地 GeoJSON + ECharts）
│  ├─ AnimatedArt.tsx            # 全国非遗动态视觉（canvas / CSS 动画）
│  ├─ VideoWall.tsx              # 非遗影像墙（本地视频 / 外链嵌入 / 站外检索）
│  ├─ SiteHeader.tsx / SiteFooter.tsx
│  ├─ HeritageCard.tsx           # 非遗卡片（悬停左上角渐入「详细」）
│  ├─ CollectButton.tsx          # 收藏按钮（未登录自动跳登录页）
│  ├─ EChart.tsx                 # ECharts 封装
│  ├─ Particles.tsx / Reveal.tsx / CountUp.tsx   # 粒子、滚动进场、数字滚动
│  └─ Protected.tsx / HistoryRecorder.tsx        # 登录守卫、浏览历史埋点
├─ lib/
│  ├─ ich.ts                     # 18 个非遗项目完整数据
│  ├─ mapData.ts                 # 文化地图点位（name/location/coordinates/count/description/image）
│  ├─ nationalIch.ts             # 全国著名非遗 + 中国地图点位 + 全国数据
│  ├─ videos.ts                  # 影像墙配置（src / embedUrl / searchUrl 三种接入方式）
│  ├─ newsData.ts                # 政策 / 新闻 / 倡议
│  ├─ works.ts                   # 推荐作品、精选内容、用户评价、平台统计
│  ├─ knowledge.ts               # AI 知识库摘要 + 本地检索兜底
│  ├─ auth.tsx                   # 演示用登录态（localStorage）
│  ├─ store.ts                   # 收藏 / 浏览历史
│  └─ chatClient.ts              # 前端 SSE 流式对话客户端
└─ styles/
   ├─ tailwind.css               # Tailwind v4 入口（@theme 设计令牌）
   ├─ globals.scss               # 全局样式：动效、渐变、玻璃拟态、纹样
   └─ _tokens.scss               # SCSS 变量
```

---

## 三、AI 智能体接入说明（DeepSeek）

智能体接入位在 [`src/components/AIAgentSlot.tsx`](src/components/AIAgentSlot.tsx)：

```tsx
const AGENT_ENDPOINT = '/api/chat';

<div id="ai-agent-container" data-agent="deepseek" data-endpoint="/api/chat">
  {/* 此处接入外部智能体，无需实现具体对话逻辑 */}
</div>
```

- 前端只负责把 `{ messages }` POST 给 `/api/chat` 并消费 SSE 流（`data: {"delta":"..."}`）；
- 服务端 [`src/app/api/chat/route.ts`](src/app/api/chat/route.ts) 持有 `DEEPSEEK_API_KEY`，
  把平台数据（10 个项目 + 地图点位 + 政策新闻）作为 system 提示词上下文一起发给 DeepSeek；
- **未配置 Key 或调用失败时**，自动降级为本地知识库检索作答，保证演示环境也能问答；
- 想换成扣子 / Dify / 百炼 / 自建服务：只改 `AGENT_ENDPOINT` 与请求体字段即可，页面其余部分不用动。

环境变量（`.env.local`）：

```
DEEPSEEK_API_KEY=sk-xxxx
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-chat
```

可用 `GET /api/chat` 检查接入状态：返回 `{ agent, configured, model, endpoint }`。

---

## 四、权限规则（已在代码中实现）

| 页面 / 功能 | 游客 | 登录用户 |
| --- | --- | --- |
| 首页、文化地图、非遗列表与详情、相关非遗 | ✅ 可浏览 | ✅ |
| AI 创作（`/create`） | ❌ 自动跳转登录页 | ✅ |
| 收藏非遗项目 | ❌ 点击跳转登录页 | ✅ |
| 个人中心（`/me`） | ❌ 自动跳转登录页 | ✅ |

登录态为**演示实现**（`localStorage` + `AuthProvider`），任何账号密码都可登录；
接入真实后端时替换 `src/lib/auth.tsx` 与 `/api` 即可。

---

## 五、数据与素材说明

- 图片 / 视频来自用户提供的《非遗项目.zip》素材包；
- 项目文字说明来自素材包内的文档（常山战鼓、无极剪纸、桃林坪花脸社火、正定高照、井陉木雕、石家庄酒酿造技艺）
  以及公开资料整理（井陉拉花、耿村民间故事、石家庄丝弦、赞皇原村土布）；
- 名录等级与批次依据素材包文件夹标注与文档内容整理，**若与官方最新公布名录不一致，以官方公布为准**；
  如需修正，直接修改 `src/lib/ich.ts` 中对应字段，全站页面会自动同步；
- 相关非遗页的政策 / 新闻 / 用户评价 / 文化地图中未建立详情页的点位为示例数据，可替换为真实接口或 CMS。

---

## 六、页面动效清单（对应需求）

- 滚动进场：`Reveal`（IntersectionObserver，错峰延迟）
- 悬浮：卡片抬升 + 图片缓动放大；非遗卡片左上角「详细」渐入
- 渐变流动：标题渐变文字、按钮渐变位移、装饰分割线
- 粒子 / 光晕：首屏 canvas 粒子（鼠标排斥 + 邻近连线）、`glow-orb` 模糊光斑、气泡脉冲
- 非遗科技风：剪纸纹样 SVG 底纹 + 云纹 + 玻璃拟态卡片 + 霓虹紫 / 鎏金配色
- 响应式：桌面 / 平板 / 移动端（导航折叠、地图侧滑面板改为底部抽屉）
