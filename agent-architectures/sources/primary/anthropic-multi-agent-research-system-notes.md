# Anthropic Engineering — How we built our multi-agent research system

- URL: https://www.anthropic.com/engineering/multi-agent-research-system
- 类型: 官方工程实践文章
- 记录日期: 2026-04-30

## 这份资料提供了什么

这篇文章不是泛泛谈 multi-agent，而是展示一个生产级 orchestrator-worker 系统为什么成立、在哪些任务上显著优于单 agent。

## 核心架构

### Lead agent
- 负责理解任务
- 制定研究策略
- 拆分子任务
- 汇总子 agent 结果

### Subagents
- 并行执行不同研究子任务
- 各自搜索、分析、补缺
- 再把结果回传给 lead

### Citation agent
- 对最终结果做引用定位和归因

## 架构优势

- 宽任务可并行展开
- 单个 agent 的上下文压力下降
- 信息搜集速度快很多
- 对“同时沿多个方向探索”的任务特别有效

## 代价与失败模式

- token 成本显著上升
- 拆分不清会造成重复劳动
- 同步等待 subagent 会造成系统阻塞
- 过多 agent 会让协调开销反噬收益
- trace/debug 比单 agent 难很多

## 适用范围判断

### 适合
- 研究型任务
- 宽搜索问题
- 比较分析
- 多源信息收集与综合

### 不适合
- 强共享上下文任务
- 高耦合编码任务
- 很短很直接的问题

## 对本次研究的贡献

这份资料支持了一个重要判断：

> multi-agent 最擅长的不是“更聪明”，而是“更并行”。

因此它适用于可拆分、可并行、信息面很宽的问题，而不是所有问题。