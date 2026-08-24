# Agent Guidelines — lqmath1824.github.io（Qiao Li 的个人主页）

本仓库是 Qiao Li（李乔）的个人学术主页，基于 **al-folio v1.x** 架构（Jekyll + `al_folio_core` 主题插件 + Tailwind v4），由 GitHub Actions 在推送 `master` 时自动构建并部署到 GitHub Pages。

## 两条最高优先级决策（用户明示，务必遵守）

1. **已与上游 al-folio 脱钩，不再跟随上游更新。**
   - 不要运行 `bundle exec al-folio upgrade audit` / `overrides audit` / `report` 等升级审计命令。
   - 不需要维护 `.al-folio-overrides.yml` 覆盖清单，不需要关注 gem 上游版本漂移。
   - 不需要把改动"路由"回上游 gem 仓库；本仓库只做本地覆盖（`_includes/`、`_pages/`、`_data/` 等），这是最终形态，不是临时 fork。
   - Gemfile / Gemfile.lock / `_config.yml` 中的插件版本按现状冻结使用，除非用户明确要求升级。

2. **不需要本机验证，信任智能体的改动。**
   - 不要在本机运行 `bundle exec jekyll build/serve`、集成测试、Playwright、lint 等验证命令（本机 Ruby 2.6.10 也跑不动本项目）。
   - 不要做逐文件的详细本地复查；用户信任大部分操作是正确的。
   - 改完即提交推送，由 GitHub Actions CI 负责构建验证（可关注 Actions 结果，但不必在本地复现）。
   - 改动仍需认真、自洽、符合仓库现有约定——只是不需要花时间做本地验证。

## 项目结构与常见改动位置

- `_pages/` — 页面（about / publications / teaching / cv / blog / projects / news / interests/*）
- `_bibliography/papers.bib` — 论文库（新增论文写这里，由 jekyll-scholar 渲染）
- `_news/` — 新闻条目（每条一个 md）
- `_teachings/` — 课程页（front matter 含 schedule，可挂习题课讲义 PDF）
- `_data/` — 结构化数据：`cv.yml`（CV 内容）、`socials.yml`（社交账号）、`upcoming_events.yml`（首页近期活动）、`coauthors.yml` 等
- `_includes/` — 本地覆盖/自建模板（`head.liquid`、`upcoming_events.liquid`、`cv/*.liquid` 都是自建或覆盖的）
- `assets/pdf/` — PDF 资源（CV、习题课讲义等）

## 架构要点（理解用，不影响上述决策）

- 运行时逻辑在 `al_folio_core` 等 gem 里；本仓库通过本地 `_includes/` shadow 覆盖来定制渲染，这是仓库已确立的做法。
- 功能开关在 `_config.yml`：`search_enabled`、`enable_math`、`enable_darkmode`、`enable_masonry` 等。
- 部署：push 到 `master` → `.github/workflows/deploy.yml` 构建 `_site` 并发布。更新站点 = 改文件 → commit → push。

### al-folio v1 渲染机制速查（原 CLAUDE.md 保留内容，做高级改动时有用）

- **功能是两层门控**：站点级 `_config.yml` 开关（`enable_math`、`search_enabled` 等）+ 页面级 front matter（`chart.*`、`mermaid.*`、`tikzjax`、`giscus_comments` 等）。开关/插件缺失时标签静默输出空串，不会报错。
- **`_includes/plugins/*.liquid` 是薄包装**：调用各插件 gem 的自定义 Liquid 标签，只有"插件在 plugins 列表 + 开关打开"两者都满足才渲染。常用映射：`al_search_assets`→al_search（Cmd-K 搜索）、`al_math_styles/scripts`→al_math（MathJax）、`al_icons_styles`→al_icons、`al_folio_cv_render`→al_folio_cv、`al_folio_distill_render`→al_folio_distill、`al_charts_scripts`→al_charts、`al_img_tools_styles/scripts`→al_img_tools。
- **两份清单需保持一致**：`Gemfile`（固定版本）与 `_config.yml` 的 `plugins:` 列表；增删插件要同时改两处。
- **v1 配置契约**（`al_folio.api_version: 1`、`style_engine: tailwind`、`tailwind.{version,css_entry,preflight}`、`distill.{engine,source}`）由 `al_folio_core` 构建时校验，不要删除这些键。
- **第三方库 CDN + SRI**：`_config.yml` 的 `third_party_libraries:` 块按需注入 JS/CSS（MathJax、tocbot、mermaid、highlight.js 等），有版本和完整性哈希。
- **搜索索引**：`al_search` 在构建时从内容生成索引，`search_enabled: true` 时 Cmd-K 可用。
