# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

保温箱完成清洁接口返回成功，但再次读取仍是清洁中，最后清洁时间也未更新，因此后续退役和运输安排被错误阻塞。请修复清洁完成状态的持久化。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-18
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-18.git
- parent SHA：6c90ad4984c23ce321f72ec505c81fbc0a8db7e5

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-18.git bug-repro
cd bug-repro
git checkout --detach 6c90ad4984c23ce321f72ec505c81fbc0a8db7e5
go test ./internal/service -run "^TestContainerCleaningAndRetirementLifecycle$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestContainerCleaningAndRetirementLifecycle$" -count=1
--- FAIL: TestContainerCleaningAndRetirementLifecycle (0.52s)
    service_test.go:325: complete cleaning = {ID:box_c59080cf45b741185c36eb32 SerialNumber:BOX-1 State:cleaning CapacityMilliLit:1000 CalibrationDueAt:2026-08-20 08:00:00 +0000 UTC LastCleanedAt:2026-08-18 08:00:00 +0000 UTC ReservedShipmentID: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:2}, error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	0.528s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestContainerCleaningAndRetirementLifecycle$" -count=1
--- FAIL: TestContainerCleaningAndRetirementLifecycle (1.15s)
    service_test.go:325: complete cleaning = {ID:box_0d659a07c5ea56ad630ad444 SerialNumber:BOX-1 State:cleaning CapacityMilliLit:1000 CalibrationDueAt:2026-08-20 08:00:00 +0000 UTC LastCleanedAt:2026-08-18 08:00:00 +0000 UTC ReservedShipmentID: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:2}, error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	1.334s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令 go test ./internal/service -run ^TestContainerCleaningAndRetirementLifecycle$ -count=1 必须由修复前失败变为修复后通过；相关包与 go test ./... -count=1 全量回归通过，回退 gold 关键修改后定向命令重新失败。
