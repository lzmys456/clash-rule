# clash-rule 相似仓库调研与分析

生成日期：2026-05-08
目标仓库：`lzmys456/clash-rule`

## 处理原则

本次没有把第三方仓库源码直接复制进 `clash-rule`，原因：

1. 第三方仓库许可证、代码质量和安全性未完全审计，直接导入存在合规和安全风险。
2. `clash-rule` 当前仓库体量较小，更适合先沉淀调研结论，再决定是否 fork、引入脚本、迁移配置或重构。
3. 本文件用于把相似项目“拉到仓库里分析”，即沉淀候选项目、能力对比和可借鉴方向。

## 当前仓库初步判断

`lzmys456/clash-rule` 是一个较小的公开仓库，默认分支为 `main`。尝试读取 `README.md` 未找到，因此暂时按仓库名和搜索上下文判断，它更可能偏向 Clash / Mihomo 规则配置、订阅规则或代理规则维护。

## 相似仓库候选

### 1. jielosc/clash-subscription-tool

地址：https://github.com/jielosc/clash-subscription-tool

相关性：高。

该项目定位为从已有 Clash / Mihomo 订阅 URL 或一个或多个 `vless://` 链接生成 Clash / Mihomo YAML。它支持合并额外 `proxies`，替换顶层 `rule-providers` 和 `rules`，并输出历史文件与稳定的 `latest.yaml`。

可借鉴点：

- 将订阅输入、规则偏好和输出文件拆成清晰配置。
- 使用 `settings.yaml` 管理输入源、输出目录、超时时间等运行参数。
- 使用 `preferences.yaml` 维护 `rule-providers` 和 `rules`。
- 保留下载订阅中的其他顶层字段，只替换关键规则字段。

适合引入的方向：

- 如果 `clash-rule` 目标是维护一套规则并生成最终订阅配置，可以参考它的配置拆分方式。
- 可以增加 `preferences.yaml` / `settings.yaml.example`，让规则维护从手工编辑变成可重复生成。

风险与注意：

- 该项目是脚本型工具，若引入代码，需要先确认许可证和依赖安全性。
- 如果只是维护静态规则，完整引入脚本可能偏重。

---

### 2. 233mawile/subtrans

地址：https://github.com/233mawile/subtrans

相关性：高。

该项目用于通过自定义 `processor` 处理远程代理订阅并输出新的 YAML，部署形态偏 Cloudflare Worker。它适合对机场托管订阅中的规则、代理组、节点筛选或命名方式不满意，希望用统一逻辑处理不同客户端订阅的场景。

可借鉴点：

- 用 `processor.js` 把订阅处理逻辑插件化。
- 支持过滤节点、批量重命名节点、调整或重建 `proxy-groups`、修改 `rules`。
- 保留原始订阅内容，只覆盖自己关心的字段。
- 通过 Worker 把最终订阅变成远程 URL，客户端只需订阅一个地址。

适合引入的方向：

- 如果 `clash-rule` 后续想做“远程订阅加工服务”，这个架构值得参考。
- 可以先不引入 Worker，只借鉴 `processor` 思路，把规则变更逻辑封装成独立脚本。

风险与注意：

- Cloudflare Worker 部署会引入运行环境、请求校验和上游订阅安全问题。
- 如果订阅 URL 或 processor URL 来自外部输入，需要严格限制协议、域名和错误处理。

---

### 3. ififi2017/mihomo-subconverter

地址：https://github.com/ififi2017/mihomo-subconverter

相关性：中高。

该项目是一个轻量 Mihomo / Clash 订阅转换器，支持粘贴代理节点链接、选择规则模板，生成可用配置文件，并支持 Vercel 一键部署。README 显示其支持 Hysteria2、AnyTLS、VLESS、Trojan、VMess、Shadowsocks、TUIC 等协议，也支持订阅 URL 导入。

可借鉴点：

- Web UI 化订阅转换流程。
- 支持 ACL4SSR / sub-web INI 规则模板格式。
- 支持自定义规则优先插入。
- 支持生成长期可用的远程订阅 URL。
- 提供 API 形式：`GET /api/clash?...`。

适合引入的方向：

- 如果 `clash-rule` 未来想从规则仓库升级成可视化配置生成器，可参考该项目。
- 如果只维护规则文件，不建议直接引入完整 Next.js / Vercel 项目，成本较高。

风险与注意：

- 项目体量明显更大，引入会显著增加维护成本。
- Web 服务涉及用户提交节点链接，需考虑隐私和日志处理。

---

### 4. 569258yin/clashx-addtion-rule-workers

地址：https://github.com/569258yin/clashx-addtion-rule-workers

相关性：中。

该仓库名称显示与 ClashX 额外规则和 Worker 有关，搜索结果命中度较高。但本次未能读取到 `README.md`，因此只能作为待进一步人工核验候选。

可借鉴点：

- 可能与通过 Worker 分发 ClashX 附加规则有关。
- 仓库体量约 1.2 MB，可能包含较完整规则或 Worker 逻辑。

风险与注意：

- 未读取到 README，暂不建议直接引入。
- 需要进一步检查目录结构、许可证、入口文件和规则来源。

## 对 `clash-rule` 的建议路线

### 路线 A：保持轻量规则仓库

适合目标：只想维护规则、分流列表、rule-providers。

建议：

- 增加 `README.md`，说明仓库用途和客户端使用方式。
- 统一目录：`rules/`、`providers/`、`examples/`。
- 增加示例 Clash / Mihomo 配置。
- 增加规则更新说明和来源说明。

### 路线 B：增加订阅生成脚本

适合目标：希望把规则、节点、代理组组合成最终 YAML。

可参考：`jielosc/clash-subscription-tool`。

建议：

- 新增 `settings.yaml.example`。
- 新增 `preferences.yaml.example`。
- 新增生成脚本，例如 `scripts/build_config.py`。
- 输出 `output/latest.yaml` 和历史文件。

### 路线 C：做远程订阅加工服务

适合目标：希望客户端订阅一个 URL，服务端动态拉取、加工、返回配置。

可参考：`233mawile/subtrans`。

建议：

- 先做本地 processor，再考虑 Worker 部署。
- 对外部 URL 做白名单或严格校验。
- 明确日志策略，避免泄露订阅地址和节点信息。

### 路线 D：做 Web UI 转换器

适合目标：希望做给多人使用的订阅转换工具。

可参考：`ififi2017/mihomo-subconverter`。

建议：

- 只有在确实需要 UI、API、多协议解析时再走这条路。
- 不建议在当前小仓库阶段直接上 Next.js / Vercel 全套。

## 推荐下一步

优先走路线 A + 路线 B：

1. 补 `README.md`，明确 `clash-rule` 的定位。
2. 设计目录结构：
   - `rules/`
   - `providers/`
   - `examples/`
   - `scripts/`
3. 借鉴 `clash-subscription-tool` 的配置拆分方式，但不要直接复制代码。
4. 先实现一个最小可用生成脚本，把规则配置输出为 `latest.yaml`。
5. 后续如果需要远程订阅，再考虑借鉴 `subtrans` 的 Worker / processor 架构。

## 本次未完成项

- 未直接复制第三方源码到仓库。
- 未完整审计候选仓库许可证。
- 未读取 `clash-rule` 的完整文件树；当前 GitHub 工具未提供稳定的目录枚举能力。
- 未对候选仓库做安全审计，只做功能方向分析。
