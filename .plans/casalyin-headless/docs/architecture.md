# casalyin-headless - 架构

> 系统架构和关键设计决策。
> 维护者：team-lead, frontend-dev（架构变更后更新）

## 系统概览

casalyin-Headless 是 C 端前台，通过 Next.js 14 App Router 向用户展示农业科技信息。
页面数据通过 Server Components 从 casalyin-java（端口 8690）获取，SEO 友好。

## 技术栈

| 技术 | 版本/说明 |
|------|---------|
| 框架 | Next.js 14 (App Router) |
| 语言 | TypeScript |
| 样式 | Tailwind CSS |
| UI | Headless UI |
| 图标 | Lucide React |
| 状态管理 | React Hooks |
| 进程管理 | PM2 (ecosystem.config.js) |
| 容器化 | Docker (docker-compose) |

## 页面模块

| 路由 | 功能 |
|------|------|
| `/` | 首页 |
| `/productos` | 产品列表/详情 |
| `/marcas` (empresas) | 品牌/企业列表 |
| `/agronomists` | 农艺师列表/详情 |
| `/cultivos` | 作物列表/详情 |
| `/plagas` | 病虫害列表/详情 |
| `/ingredientes` | 化学成分列表/详情 |
| `/unete` | 加入我们 |
| `/contacto` | 联系我们 |
| `/sobre-nosotros` | 关于我们 |
| `/privacidad` | 隐私政策 |
| `/terminos` | 条款 |

## 数据流

```
浏览器请求
  → Next.js Server Component
  → fetch → casalyin-java API (8690)
  → 返回 HTML（SSR）or 静态页面（SSG）
  → 浏览器渲染
```

## 目录结构

```
casalyin-Headless/
  app/                  ← Next.js App Router 页面
    layout.tsx          ← 根布局
    page.tsx            ← 首页
    productos/          ← 产品页面
    agronomists/        ← 农艺师页面
    cultivos/           ← 作物页面
    plagas/             ← 病虫害页面
    ingredientes/       ← 化学成分页面
    empresas/           ← 企业/品牌页面
    api/                ← Next.js API Routes（如有）
  components/           ← 共享组件
  contexts/             ← React Context
  lib/                  ← 工具函数、API 客户端
  types/                ← TypeScript 类型定义
  public/               ← 静态资源
```
