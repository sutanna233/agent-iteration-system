# 基于 OpenClaw 的多 Agent 永动迭代系统设计方案

## 一、系统架构概述

本系统构建于 OpenClaw 框架之上，采用「一主多从」的分层架构。

| 角色 | 代号 | 职责 |
|------|------|------|
| 主控 Agent | 太子 | 统筹规划、任务分发、代码整合、论坛决策 |
| 执行 Agent | executor1~N | 需求开发、代码实现、PR 提交 |
| 审核 Agent | verifier | 代码审查、测试验证、质量把关 |

每个 Agent 配备完整的 MD 文件体系：

```
agent_workspace/
├── AGENTS.md    # 角色定义与能力边界
├── SOUL.md      # 人格设定与行为风格
├── MEMORY.md    # 长期记忆（经验沉淀）
├── TASK.md      # TODO 格式待办清单
├── GOAL.md      # 阶段目标与战略方向
└── LOG.md       # 工作日志（可选）
```

---

## 二、任务调度机制

### 2.1 任务分类

| 类型 | 周期 | 触发方式 | 示例 |
|------|------|----------|------|
| 长期任务 | 每周/每月 | 主控主动规划 | 架构重构、技术调研、版本规划 |
| 短期任务 | 每日/每小时 | Issue 驱动 | bug 修复、功能开发、文档更新 |
| 临时任务 | 随时 | 用户需求 | 紧急 hotfix、新需求接入 |

### 2.2 Cron 调度配置

**主控 Agent Cron 任务**：

```bash
# 每15分钟：扫描 GitHub Issue，更新各 Agent 的 TASK.md
*/15 * * * * openclaw cron run master-sync --interval=15m

# 每小时：汇总论坛讨论，生成 GOAL.md
0 * * * * openclaw cron run master-forum-summary --interval=1h

# 每2小时：检查未合并的 PR，执行合并或标记需人工介入
0 */2 * * * openclaw cron run master-pr-merge --interval=2h

# 每日凌晨：生成每日工作报告
0 0 * * * openclaw cron run master-daily-report --interval=1d
```

**小弟 Agent Cron 任务**：

```bash
# 每30分钟：REPL 循环 - 拉取任务 → 执行 → 反馈
*/30 * * * * openclaw cron run executor-repl --interval=30m

# 紧急任务：每5分钟检查 P0/P1 级别 Issue
*/5 * * * * openclaw cron run executor-urgent --interval=5m
```

---

## 三、MD 文件规范

### 3.1 TASK.md 格式（TODO）

```markdown
# TASK.md - 本周期待办清单

## 待处理 (TODO)

- [ ] T001: 实现用户登录功能
  - Issue: #45
  - 优先级: P1
  - 截止: 2026-03-15
  - 验收条件: 登录成功后返回 JWT，支持刷新令牌

- [ ] T002: 修复商品列表分页 bug
  - Issue: #48
  - 优先级: P2
  - 截止: 2026-03-12

## 进行中

- [ ] T003: 优化数据库查询性能
  - Issue: #42
  - 状态: 进行中 (executor1)
  - 预计完成: 2026-03-11

## 已完成

- [x] T004: 添加 API 文档注释
  - Issue: #40
  - PR: #44
```

### 3.2 GOAL.md 格式

```markdown
# GOAL.md - 阶段目标

## 本周目标 (2026-03-10 ~ 2026-03-16)

- [ ] 完成用户模块开发（登录/注册/找回密码）
- [ ] 优化核心 API 响应时间 < 200ms
- [ ] 建立 CI/CD 流水线

## 战略方向

1. **短期**：优先交付 MVP 功能集
2. **中期**：引入缓存层，提升系统吞吐量
3. **长期**：探索 AI 代码生成辅助

## 关键指标

- PR 合并率 > 80%
- 测试覆盖率 > 70%
- 平均任务完成周期 < 3 天
```

---

## 四、GitHub 自动化流程

### 4.1 主控 Agent 工作流

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  扫描 Issue  │ -> │  解析任务   │ -> │  更新 TASK  │
└─────────────┘    └─────────────┘    └─────────────┘
                                              │
                                              v
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   合并 PR    │ <- │  审查代码   │ <- │  分配任务   │
└─────────────┘    └─────────────┘    └─────────────┘
```

### 4.2 Issue 模板

```markdown
## 任务描述
[简要说明要做什么]

## 验收条件
- [ ] 条件1
- [ ] 条件2

## 实现建议
[可选，主控提供的参考方案]

## 关联信息
- 分支: feature/xxx
- 负责人: @executor1
- 优先级: P1/P2/P3
```

### 4.3 小弟提交 PR 规范

- 必须关联对应 Issue（使用 `Closes #xxx`）
- 必须通过 CI 测试
- 必须包含更新后的 TASK.md（标记完成）

---

## 五、论坛自主迭代机制

### 5.1 论坛定位

论坛作为 Agent 群体的「协作大脑」，核心目的是：

1. **沉淀经验**：每个 Agent 的工作日志可供其他 Agent 参考
2. **集体决策**：重大方向通过讨论形成共识
3. **问题求助**：遇到阻塞时可以发帖请求支援
4. **创意孵化**：新功能想法先在论坛验证可行性

### 5.2 帖子类型

| 类型 | 发布者 | 内容要求 |
|------|--------|----------|
| 工作日志 | 小弟 | 今日完成、遇到的问题、解决方案 |
| 技术讨论 | 任意 | 技术方案、选型对比、改进建议 |
| 需求评估 | 主控 | 新需求可行性分析、优先级建议 |
| 决策投票 | 主控 | 关键方向选择，附各方观点 |

### 5.3 永动循环流程

```
用户需求 / 战略规划
       │
       v
   [需求池] -> 主控评估可行性
       │
       v
   论坛讨论 -> 形成共识
       │
       v
   生成 GOAL -> 拆解为 Issue
       │
       v
   小弟执行 -> 提交 PR
       │
       v
   主控合并 <- 代码审查
       │
       v
   论坛复盘 -> 更新记忆
       │
       v
   循环迭代
```

---

## 六、REPL 循环实现

### 6.1 主控 REPL（Master Loop）

```python
def master_repl():
    while True:
        # Read: 拉取最新 Issue 和论坛动态
        issues = github.fetch_issues(labels=['task', 'bug'])
        posts = forum.fetch_recent_posts()
        
        # Evaluate: 评估任务优先级，识别新需求
        new_tasks = evaluate_issues(issues)
        insights = analyze_forum(posts)
        
        # Plan: 更新 GOAL，分配任务
        if new_tasks:
            update_goals(new_tasks)
            distribute_tasks(new_tasks)
        
        # Act: 执行定时任务（PR 合并、状态同步）
        merge_ready_prs()
        sync_agent_status()
        
        sleep(CRON_INTERVAL)
```

### 6.2 小弟 REPL（Executor Loop）

```python
def executor_repl():
    while True:
        # Read: 读取自己的 TASK.md 和分配的 Issue
        tasks = read_task_md()
        my_issues = github.get_assigned_issues()
        
        # Execute: 执行当前任务
        for issue in my_issues:
            if can_start(issue):
                result = implement(issue)
                if result.success:
                    submit_pr(issue, result.branch)
                    post_to_forum(f"完成 #{issue.id}: {issue.title}")
        
        # Plan: 规划下一步
        update_task_status()
        
        # Feedback: 记录到 MEMORY.md
        record_to_memory()
        
        sleep(CRON_INTERVAL)
```

---

## 七、配置示例

### 7.1 主控 Agent 配置 (master.json)

```json
{
  "id": "taizi",
  "role": "master",
  "github": {
    "repo": "owner/repo",
    "token": "ghp_xxx"
  },
  "cron": {
    "sync": "0 * * * *",
    "forum": "0 */6 * * *",
    "merge": "0 0 * * *"
  },
  "forum": {
    "url": "http://forum.internal",
    "category": "agent-collab"
  }
}
```

### 7.2 小弟 Agent 配置 (executor.json)

```json
{
  "id": "executor1",
  "role": "executor",
  "github": {
    "repo": "owner/repo",
    "token": "ghp_xxx"
  },
  "cron": {
    "repl": "0 * * * *"
  },
  "capabilities": ["backend", "api"]
}
```

---

## 八、总结

本方案以 OpenClaw 为底层框架，通过以下核心机制构建永动迭代系统：

1. **分层架构**：主控统筹 + 小弟执行 + 论坛协作
2. **MD 驱动**：TASK.md / GOAL.md 作为任务流转中枢
3. **Cron 驱动**：定时触发 REPL 循环，保持系统活力
4. **GitHub 自动化**：Issue → PR → 合并全链路自动
5. **论坛永动**：讨论 → 决策 → 执行 → 复盘 → 进化

系统运行一段时间后，Agent 群体的 MEMORY.md 将积累大量经验值，呈现出「智能涌现」的特质，真正成为一支不知疲倦的虚拟开发团队。

---

*文档版本: v1.0*
*更新日期: 2026-03-10*
