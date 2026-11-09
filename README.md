# QuickJob

定时任务 / 后台作业调度：cron + interval + 一次性 + 按需四类 job，重试/并发门/重叠档，`[QuickJob]` 编译期登记，OAuth 登录仪表盘。

从原 QuickNet 单仓拆分出的独立项目。MIT 协议。

## 安装

```bash
dotnet add package QuickJob --prerelease
dotnet add package QuickJob.Generators --prerelease   # [QuickJob] 特性路需要
```

## 快速上手

```csharp
using QuickJob;

// 路 1：[QuickJob] 特性 —— 生成器编译期登记（零反射、AOT 干净）
public sealed class Housekeeping
{
    [QuickJob("daily-roll", Cron = "0 2 * * *", MaxRetries = 2, OnOverlap = OverlapPolicy.Skip)]
    public static async Task Roll(CancellationToken ct) => await DayRoller.RollAsync(ct);
}

// 路 2：DSL 现场挂名 —— 两种登记产出同一定义
var scheduler = QuickJobScheduler.Create(b =>
{
    b.Interval("beat", TimeSpan.FromSeconds(30),
        ctx => { ctx.Report("flat pulse"); return ValueTask.CompletedTask; });
});

await scheduler.StartAsync();
await scheduler.StopAsync();
```

## 特性

- 四类 job 一家收：cron / interval / once / on-demand
- 重试四件套：`MaxRetries` × `Fixed/Linear/Exponential` × 基阶/上钳 × 抖动
- 并发门按 job 一颗：`MaxConcurrentPerJob` + `Skip/Queue/RunAnyway`，另有 scheduler 级 `MaxParallelism` 与三级优先
- 迟到档策略：`RunWithinGrace` / `SkipLate` / `CatchUpLatest`，夏令时不炸
- Dashboard 挂件（OAuth 登录档）；`Time`（TimeProvider）与 `Store`（IJobStore）两个接缝便于确定性测试

## 构建与测试

```bash
dotnet build QuickJob.slnx
dotnet test tests/QuickJob.Tests/QuickJob.Tests.csproj
```

## 文档

见 [docs/quickjob-usage.md](docs/quickjob-usage.md)。

## License

MIT
