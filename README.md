# 国内开发者友好全栈部署平台 · 严选调研

> 目的：指引国内开发者直接部署全栈项目（带域名、可国内访问、免费、文档友好，支持 DB / KV / Cron）。  
> 调研日期：2026-07-23。结论会随各平台定价与网络环境变化，使用前请复核官网。

## 严选标准

| 维度 | 要求 |
| --- | --- |
| 域名 | 提供平台子域名，并支持自定义域名 |
| 国内可访问 | 默认域名或绑定自定义域名后，大陆网络可稳定打开（不依赖特殊 DNS/代理） |
| 免费 & 新手友好 | 有可用的长期免费档；注册/部署门槛低 |
| 文档 | 官方文档清晰，最好有中文或社区教程 |
| 全栈能力 | 同一平台内尽量覆盖：**DB**、**KV**、**Cron** |

---

## 总评（先看这个）

**原清单四个平台里，没有一家同时「满分」满足全部严选标准。**

| 平台 | 域名 | 国内访问 | 免费 | 文档 | DB | KV | Cron | 建议 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [Deno Deploy](#1-deno-deploy) | ✅ | ⚠️ 一般 | ✅ | ✅（有中文站） | ⚠️ 仅 KV，无托管 SQL | ✅ | ✅ | **可作首选之一**（KV 场景） |
| [Railway](#2-railway) | ✅ | ❌ 差 | ⚠️ 试用后 $1/月 | ✅ | ✅ 可挂 Postgres 等 | ❌ 无原生 KV | ❌ Free 长期无 Cron | **不建议作为免费严选** |
| [Cloudflare](#3-cloudflare-workers--pages) | ✅ | ❌ `workers.dev` 常不可用；自定义域名也常慢/不稳 | ✅ | ✅ 极好 | ✅ D1 | ✅ | ✅ Cron Triggers | **功能最强，国内访问硬伤** |
| [void.cloud](#4-voidcloud) | ✅ | ❌ 跑在 Cloudflare 上，同 CF | ✅ 有 free 档 | ✅ | ✅ D1 / PG | ✅ | ✅ | **能力对齐严选，但国内访问与产品归属有风险** |
| [EdgeOne](#补充-edgeone腾讯云建议纳入makers--加速套餐要分开看)（补充） | ✅ | ✅ 国内优势明显 | ⚠️ **两层计费**：Makers 部署有免费档；国内站 EO 加速套餐从个人版起付费 | ✅ 中英文 | ⚠️ 无原生 SQL（接 Supabase 等） | ✅（Makers） | ❌ 未见原生 Cron | **国内访问首选；别把 EO 套餐当成「全免费」** |

**实操建议（按场景）：**

1. **要国内稳定打开 + 尽量免费起步** → 优先 **EdgeOne Makers 免费档** 先用平台域名验证；正式绑备案域名并开大陆加速时，通常还要买 **EO 预付费套餐**（个人版起）。Cron / SQL 需外接。
2. **要同平台自带 KV + Cron、可接受海外边缘** → 优先 **Deno Deploy**。
3. **要最完整的免费边缘全家桶（D1/KV/Cron）且主要服务海外用户** → **Cloudflare**；国内用户务必绑自定义域名并实测。
4. **Railway / 纯 void.cloud** → 暂不作为「国内免费严选」主推。

---

## 原清单逐项调研

### 1. Deno Deploy

- 官网 / 定价：https://deno.com/deploy/pricing  
- 文档：https://docs.deno.com/deploy/  
- 中文镜像：https://deno.org.cn/

| 项 | 结论 |
| --- | --- |
| 域名 | 免费送 `*.deno.dev`；Free 档最多约 50 个自定义域名 / org |
| 国内访问 | 默认可开的情况多于 CF 默认域，但节点主要在 **US / EU**，延迟与稳定性一般，需实测 |
| 免费 | Free $0：约 100 万请求/月、20GB 出站、KV 1GiB 等；超额硬限制 |
| 文档 | 官方文档完善；有中文站，新手友好 |
| DB | **无托管 Postgres/MySQL**；内置 **Deno KV** |
| KV | ✅ 免费档含 KV |
| Cron | ✅ `Deno.cron()`，平台自动发现并调度 |

注意：

- Deploy Classic 已迁移/关闭，请用新 Deno Deploy。
- 新平台区域较少（文档提及 `us` / `eu`），对国内延迟不友好。
- Queues（`Deno.Kv.enqueue`）在新 Deploy 上不可用。

**适合：** 轻量 API、计数器/会话、带定时任务的小项目。  
**不适合：** 强依赖 SQL、强依赖国内低延迟。

### 2. Railway

- 定价：https://railway.com/pricing  
- Cron 文档：https://docs.railway.com/reference/cron-jobs

| 项 | 结论 |
| --- | --- |
| 域名 | 提供平台域名 + 自定义域名 |
| 国内访问 | `*.railway.app` 及解析到其 IP 的域名，社区大量反馈 **大陆访问困难/不可达** |
| 免费 | 30 天试用送 $5；之后 Free 为 **$1/月**（含极少额度），不是长期 $0 |
| 文档 | 英文文档清晰，生态模板多 |
| DB | ✅ 可一键部署 Postgres/MySQL/Redis 等 |
| KV | ❌ 无平台级 KV 产品（可用 Redis 服务替代，但占额度） |
| Cron | 定价表写明：**Cron 仅 Free Trial**；正式 Free 长期档不带 Cron（需 Hobby+） |

**结论：不满足「免费 + Cron + 国内可访问」组合，移出严选主推。**

### 3. Cloudflare Workers / Pages

- Workers 定价：https://developers.cloudflare.com/workers/platform/pricing/  
- KV / D1 / Cron：均有官方 Free 档支持

| 项 | 结论 |
| --- | --- |
| 域名 | `*.workers.dev` + 自定义域名 |
| 国内访问 | `workers.dev` **DNS 污染/无法访问** 是常见问题；绑自定义域名后仍常出现慢、超时、不稳定 |
| 免费 | Workers Free：约 10 万请求/天；KV/D1/Cron Triggers 均有免费额度 |
| 文档 | 业界顶尖，示例与框架指南极多 |
| DB | ✅ **D1**（SQLite） |
| KV | ✅ Workers KV |
| Cron | ✅ Cron Triggers |

**结论：全栈能力最贴近「DB+KV+Cron」，但「国内可访问」一项经常不合格。**  
仅推荐：受众以海外为主，或已有备案域名且实测大陆可达。

### 4. void.cloud

- 官网：https://void.cloud  
- 文档：https://void.cloud/guide/  
- 归属动态：VoidZero 已宣布加入 Cloudflare（2026-06）

| 项 | 结论 |
| --- | --- |
| 域名 | 支持 `void domain add` 自定义域名 |
| 国内访问 | **构建在 Cloudflare Workers 之上**，国内可达性与 CF 同类问题 |
| 免费 | CLI/文档出现 `free / solo / pro` 档；**无独立公开定价页**，额度需以控制台为准 |
| 文档 | Vite 向、CLI 友好（`void deploy`），上手快 |
| DB | ✅ D1（默认）/ PostgreSQL（自备 + Hyperdrive） |
| KV | ✅（Workers KV 封装） |
| Cron | ✅ `crons/` 目录 + `defineScheduled` |

注意：

- Evan You 文中写明：做 Void 是为了给 Vite 生态卖「服务」，随后团队并入 Cloudflare，目标是强化 **Cloudflare 上部署 Vite 应用** 的体验。
- `void.cloud` 仍可试用，但产品长期形态可能并入 CF 体系，**不宜当作独立国内友好平台押宝**。

---

## 补充：EdgeOne（腾讯云，建议纳入；Makers 与加速套餐要分开看）

原 README 未列出。按严选标准，**国内访问维度明显优于上述四家**，但计费容易混淆：EdgeOne 家族至少要拆成两层。

### 层 A：EdgeOne Makers（原 Pages）——应用部署平台

- 产品站：https://pages.edgeone.ai/  
- Makers 定价：https://pages.edgeone.ai/pricing · https://pages.edgeone.ai/zh/pricing  
- 配额：https://pages.edgeone.ai/document/limits-and-quotas  
- KV：https://pages.edgeone.ai/document/kv-storage

| 项 | 结论 |
| --- | --- |
| 域名 | 平台默认域名 + 自定义域名（免费 SSL） |
| 国内访问 | 腾讯边缘网络，大陆访问优势强；社区反馈普遍优于 Vercel/CF |
| 免费 | Makers 明确宣传 **永久免费档 $0**（Git 部署、函数、Blob/KV 等）；当前为商业化前「限时宽松」，配额可能收紧 |
| 文档 | 中英文 + CLI / MCP / 模板，新手友好 |
| DB | ❌ 无自研托管 SQL；接 Supabase 等 |
| KV | ✅ 边缘 KV（免费约 1GB） |
| Cron | ❌ 未见原生 Cron |

Makers 免费档量级（Limits 文档，可能变更）：项目 40、构建 500/月、边缘函数约 300 万次/月、云函数约 100 万次/月、KV/Blob 各约 1GB。

### 层 B：EdgeOne 边缘安全加速（EO）——域名 CDN / 防护套餐（国内站）

这是中国站「套餐选型」里的预付费产品，**不是 Makers 免费档本身**。给自有域名做安全加速时，通常按这套买：

来源：[EdgeOne 产品定价](https://cloud.tencent.com/product/teo/pricing) / 套餐选型对比（价格以控制台为准，活动价会变）。

| 套餐类型 | 价格 | 计费方式 | 计费周期 | 套餐内含流量 | 套餐内含请求数 |
| --- | --- | --- | --- | --- | --- |
| 个人版 | **29.9 元/套/月**（页面或有折扣价，如 9.9） | 预付费 | 月 | 50 GB | 300 万次 |
| 基础版 | **399 元/套/月** | 预付费 | 月 | 500 GB | 2000 万次 |
| 标准版 | **3,800 元/套/月** | 预付费 | 月 | 3 TB | 5000 万次 |
| 企业版 | 定制报价 | 可定制 | 月 | 可定制 | 可定制 |

能力概览（定价页摘要）：

- **个人版**：绑 1 站；CDN / 智能加速、免费 HTTPS、平台级 DDoS、基础 CC、Web 基础访问管控等。
- **基础版**：在个人版上增加精准匹配、精准 CC、OWASP 托管规则等。
- **标准版**：再加 Bot 管理、智能 CC、AI 引擎防护等。
- **企业版**：中国大陆网络优化、四层加速、独立 DDoS、策略模板等可定制能力。

**套餐内含流量的大区抵扣比例**（实际消耗 1 GB 时，抵扣套餐额度如下；加量包同逻辑）：

| 中国大陆 CN | 北美 NA | 欧洲 EU | 亚太1 AP1 | 亚太2 AP2 | 亚太3 AP3 | 中东 ME | 非洲 AA | 南美 SA |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 GB | 1.71 GB | 1.71 GB | 2.49 GB | 2.68 GB | 2.78 GB | 2.91 GB | 2.91 GB | 2.91 GB |

含义：同样跑 1 GB，**中国大陆按 1:1 扣额度最「划算」**；海外大区会按更高比例扣套餐流量。超套餐可用流量包 / 请求包等加量包抵扣。

国际站 `edgeone.ai/pricing` 另有 Free Plan / Personal（约 $1.4 起）等标价，与中国站人民币套餐体系不完全同一页面，选型时按账号所在站点核对。

### 和严选标准怎么对齐？

| 问题 | 答案 |
| --- | --- |
| 能不能「$0 部署一个能打开的项目」？ | **能**，走 **Makers 免费档 + 平台域名**。 |
| 能不能「$0 + 自有备案域名 + 稳定大陆加速」？ | **通常不能默认成立**；自有域名开大陆加速一般要买 **EO 预付费套餐**（个人版起约 29.9 元/月）并完成 ICP 等合规要求。 |
| 是否还值得纳入严选？ | **值得**——国内可达性仍是原清单里最强的；但 README 必须写清「Makers 免费 ≠ EO 套餐免费」。 |

**缺口补法：** Cron → 外部定时调 API；SQL → Supabase / 自建库。

---

## 其他看过但不主推

| 平台 | 原因 |
| --- | --- |
| Zeabur | Free 更偏「管理自有服务器」；真正托管算力要付费档 |
| Vercel / Netlify / Fly / Render | 国内默认访问或免费额度/全栈组合不达严选 |
| Sealos 等国产 PaaS | 可关注，但需单独按「免费额度 + DB/KV/Cron」再验一轮 |

---

## 怎么用？（最小路径）

### A. 国内优先：EdgeOne（先 Makers，再按需买 EO 套餐）

1. 打开 https://pages.edgeone.ai/ 注册，用 Git / 模板部署（Next.js、Vite 等）。
2. 先用 **平台默认域名** 验证；需要 KV → 控制台开通并绑定。
3. SQL → Supabase（或自有库）；定时任务 → 外部 Cron HTTP 回调。
4. 要正式域名 + 大陆加速：域名先 **ICP 备案** → 在中国站购买 **EO 预付费套餐**（个人版起）→ 按控制台添加域名 / CNAME。
5. 有海外流量时留意套餐流量的 **大区抵扣比例**（海外 1 GB 可能扣掉 1.7～2.9 GB 额度）。

### B. KV + Cron 一体：Deno Deploy

1. 安装 Deno，按 https://docs.deno.com/deploy/ 创建组织/应用。
2. 用 Deno KV 存数据；用 `Deno.cron()` 写定时任务。
3. 部署后用 `*.deno.dev` 先测；国内用户建议再绑自定义域名并做多运营商测速。
4. 需要 SQL 时外接 Neon/Supabase/自建库。

### C. 海外用户 / 功能最全：Cloudflare

1. 注册 Cloudflare，用 Wrangler 部署 Worker / 全栈框架。
2. 绑定 **D1 + KV + Cron Triggers**。
3. **不要依赖 `workers.dev` 给国内用户**；必须自定义域名并实测。
4. 国内不可达时，改走 EdgeOne，或 CF 仅作海外入口。

---

## 准备度结论

| 问题 | 答案 |
| --- | --- |
| README 原先是否「准备好」可直接指引部署？ | **否**，仅有候选名单，缺对比与用法 |
| 原四家是否都达标？ | **否**。Railway 基本出局；CF / void 卡在国内访问；Deno 卡在 SQL 与延迟 |
| 还缺什么？ | 建议纳入 **EdgeOne**，但写清 **Makers 免费档 vs EO 预付费套餐**；补 Cron/SQL 外接；上线前做多运营商可达性实测 |

---

## 参考链接（调研时核对）

- Deno Pricing：https://deno.com/deploy/pricing  
- Railway Pricing：https://railway.com/pricing  
- Cloudflare Workers Pricing：https://developers.cloudflare.com/workers/platform/pricing/  
- Void：https://void.cloud/guide/  
- VoidZero 加入 Cloudflare：https://voidzero.dev/posts/voidzero-cloudflare  
- EdgeOne Makers Pricing / Limits：https://pages.edgeone.ai/pricing · https://pages.edgeone.ai/document/limits-and-quotas  
- EdgeOne 中国站套餐定价：https://cloud.tencent.com/product/teo/pricing  

