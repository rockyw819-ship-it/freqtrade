# Freqtrade Core Fork 协作规范

## 仓库定位

本仓库负责跟踪官方 `freqtrade/freqtrade`，并只保存外围服务无法实现的最小 Core Patch。私有策略、生产配置、平台 Dashboard、AI业务、多 Bot 管理和任何密钥均不属于本仓库。

配套策略唯一发布源为同级 `freqtrade-strategies/` 私有 Fork（源自官方 `freqtrade/freqtrade-strategies`），策略保留在其 `user_data/strategies/` 下；管理平面为同级 `quant-platform/`。执行容器以只读方式挂载固定策略 SHA 的目录。策略上游同步和二开在策略 Fork 完成，不向 Core 搬入策略文件。具体步骤见 `../quant-platform/docs/deployment-testing.md`。

优先级：Adapter/Service > Plugin/现有扩展点 > Core Patch。只有 Pre-Execution Risk Gate、低层 Audit Hook 或 Reconciliation Hook 确实无法在外围可靠实现时，才考虑修改 Core。

## 上游与分支

- `upstream` 指向官方仓库。
- `origin` 指向 Rocky Fork。
- 推荐晋升链：`upstream/develop` → `integration` → `rocky/core` → `staging` → `production`。
- 禁止 production 直接追踪上游。
- 每项 Core Patch 独立提交，避免把风险、审计、对账混在一个大提交中。

## Docker 镜像规则

服务器现用 `rocky-freqtrade` 保存 UID/GID 适配。不得为了固定版本而直接改用官方镜像，从而绕过适配层。

正确固定方式：

1. 自定义 Dockerfile 的 `FROM` 使用官方不可变摘要，例如
   `freqtradeorg/freqtrade@sha256:50720a4af314a812be2cfbf5cc6331c63e9332b06f3f4372241f54bc61a35486`。
2. 构建并测试 `rocky-freqtrade`。
3. 推送自定义镜像到受控 Registry。
4. Compose 使用自定义镜像自己的 `repo@sha256:<digest>`。
5. 发布记录同时保存 Freqtrade 版本、官方基础摘要、自定义镜像摘要和 Core Git SHA。

`rocky-freqtrade:stable` 可以作为开发别名，但不能作为唯一生产身份。

## 验证和文档

- 修改 Core 前先运行目标测试，修改后运行相关单元测试、静态检查和必要的集成测试。
- Core Patch 必须有中文设计说明：问题、为何外围方案不足、接口、失败模式、回滚和上游合并影响。
- 不得在测试输出、示例配置或文档中提交真实密钥。
- 多策略 Dry-run 的隔离由部署层完成，不通过修改 Core 让一个 Bot 同时执行多个策略。
- 服务器现行操作见工作区 [运维指南](../quant-platform/运维指南.md)。引擎镜像、UID/GID、部署/网络、备份恢复或升级方式变化时，同一次交付同步该指南；指南须在 quant-platform 同批提交，工作区根 AGENTS 仍需单独备份。
