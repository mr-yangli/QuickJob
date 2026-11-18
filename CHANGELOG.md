# 更新日志

QuickJob 各版本的变更，倒序排列（最新在上）。

> **口径**
> - 版本号跟 .NET 主版本走（.NET 10 → 10.x）。破坏性改动升主版本号，新增升次版本号（10.0.0 → 10.1.0），缺陷修复升修订号。
> - 「未发布」收 main 已合入、尚未定版的改动；定版时改节名为版本号并标「待发布」（`Directory.Build.props` 的 `<Version>` 同步就位），上包那天把「待发布」换成日期。
> - 分档为 新增 / 变更 / 修复 / 移除；标 **破坏性** 的改动，消费方升级时要动代码。
> - 版本日期取该版本在 NuGet 上可还原之日，不是提交日。

## 未发布

暂无。

## 10.1.0 — 2026-11-18

### 新增

- **多目标 `net10.0;net11.0`**：`QuickJob` 与 `QuickJob.Dashboard` 同包双 TFM，消费方按宿主框架自动还原对应资产；生成器仍 `netstandard2.0`，基准层自钉单档不变。包描述与 tags 同步标 .NET 10/11。
- **「不限」并发的明确落点**：`MaxConcurrentPerJob` 以 `int.MaxValue` 表达事实上的不限（0 会被钳成 1 串行）——需要「不设上限」语义的调用方（如后台作业桥把 0=不限 映射过来）有了官方口径，零代码改动。

## 10.0.0 — 2026-11-09

首个公开版本：QuickJob 从内部单仓拆为独立仓库（GitHub `mr-yangli/QuickJob`、Gitee `mr_yangli/quick-job`），本族 3 颗包同批上 NuGet：`QuickJob`、`QuickJob.Dashboard`、`QuickJob.Generators`。MIT。下面列的是发布内容面，不是相对上一版的 diff——10.0.0 之前没有公开版本。

### 新增

- **四类 job 一家收**：`Cron`（5–7 字段，可选秒/年、`@daily` 别名、命名月周日，按 `TimeZoneId` 壁钟游走）、`Interval`（fixed-delay，积压不走圈）、`Once`（指定时刻一打）、`OnDemand`（只接待送）。
- **两条登记路**：`[QuickJob]` 特性由生成器编译期登记（零反射、AOT 干净）；`QuickJobScheduler.Create(b => …)` DSL 现场挂名。两种登记产出同一定义。handler 统一为 `ValueTask JobHandler(JobRunContext ctx)`，常见形状由 `JobHandlerBinder` 重载解析接走。
- **按需入口**：`TriggerNowAsync`（payload 即时序列化为 UTF-8 JSON）、`EnqueueAsync(name, payload?, at?)`、`CancelAsync(runId)`。
- **重试四件套**：`MaxRetries` × `RetryBackoff`（`Fixed` / `Linear` / `Exponential`）× `RetryBaseDelay` / `RetryMaxDelay` 上钳 × `RetryJitter` 相对抖动；同一 `ExecutionId`、`Attempt` 递增。
- **并发门与重叠**：按 job 一颗门 `MaxConcurrentPerJob`（默认 1 = 串行）+ `OverlapPolicy`（`Skip` 默认 / `Queue` 配 `OverlapQueueLimit` / `RunAnyway`）；scheduler 级 `MaxParallelism`（默认 min(32, CPU)，钳 4–32）与 `JobPriority.Low/Normal/High` 认领次序；`Pause` / `Resume` 出暂停位。
- **超时**：`o.Timeout = "3m"` 起合作预算，只发 `CancellationToken` 不槌杀线程；超时按失败收场并走重试盘。
- **时钟、时区与迟到档**：`TimeProvider` 接缝 + 迟到策略 `RunWithinGrace` / `SkipLate` / `CatchUpLatest`，夏令时不炸。
- **事件流**：`scheduler.Events.Observe`，种类含 `RunStarted` / `RunCompleted` / `RunFailed` / `RunRetrying` / `RunSkippedByGate` / `SchedulerAnomaly`；两条强保证——扇出同一总序、sink 回调不交叠。
- **库存接缝**：`Store`（`IJobStore`）可换自带实现；`PeekDueRunsAsync` 供面板读未到站队列。
- **Dashboard**（`QuickJob.Dashboard`）：`AddQuickJobDashboard()` + `MapQuickJobDashboard()`（默认 `/quickjob`，`PathPrefix` 可换根）；首页看 `/stats`、`/jobs`、`/runs/pending`、`/runs/running`、`/executions`、`/events`、`/me`；鉴权 `DashboardAuthMode.Anonymous` / `OAuth2`（ClientId/Secret/三腔 endpoint + `UserIsAllowed` + 8h 会话），`ReadOnly` 关写入端；中英随手切与浅深色主题记在浏览器本地。
- **基准与门禁**：`benchmarks/QuickJob.Benchmarks`（TFM net11.0，对照组 TickerQ 11.5.x）；一条命令出红绿，可作 CI 阻断项。
- **测试面**：`tests/QuickJob.Tests` 走 `Time` / `Store` 两接缝做确定性测试。
