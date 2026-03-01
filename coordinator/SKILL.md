---
name: ficc-coordinator
description: FICC经营分析报告主协调器，负责工作流程编排、阶段路由与状态管理，协调各子技能完成报告生成全流程
dependency: []
---

# FICC经营分析报告主协调器

## 概述

本协调器作为FICC经营分析报告生成的主控模块，负责：
- 工作流编排：定义和管理6个Phase的执行顺序
- 阶段路由：根据用户输入和数据状态决定执行路径
- 状态管理：跟踪报告生成过程中的状态和数据
- 子技能协调：调用数据收集、指标计算、报告生成等子技能

## 工作流程

```
用户请求
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 0: 需求解析与初始化                                    │
│  - 识别业务团队和业务线                                       │
│  - 确定分析周期                                              │
│  - 初始化状态管理器                                           │
│  - 选择数据收集模式（A:引导式/B:数据库）                      │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: 数据收集（调用data-collection子技能）              │
│  模式A: 引导式数据收集                                        │
│    - 交互式对话收集必需字段                                    │
│    - 按优先级引导用户输入                                      │
│  模式B: 数据库采集                                            │
│    - 连接企业数据库                                           │
│    - 执行预定义SQL查询                                        │
│    - 数据格式转换                                             │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: 数据验证（调用validation子技能）                    │
│  - 完整性检查：必需字段是否齐全                                │
│  - 一致性检查：数值逻辑是否合理                               │
│  - 业务规则验证：收入分解是否等于总收入                         │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 3: 指标计算（调用metrics-calculation子技能）          │
│  - RAROC计算（风险调整资本回报率）                             │
│  - EVA计算（经济增加值）                                      │
│  - 损益归因计算（债券5维模型）                                 │
│  - 对冲有效性计算                                             │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 4: 报告生成（调用report-generation子技能）            │
│  - 根据业务线选择对应模板                                      │
│  - 填充数据与撰写内容（LLM驱动）                               │
│  - 生成Markdown格式报告                                        │
│  - 交互式确认：展示报告并询问用户满意度                         │
│    ├─ 满意 → 进入Phase 5                                      │
│    └─ 需调整 → 循环修改直到满意                                │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 5: H5可视化（调用h5-visualization子技能）             │
│  - 基于确认的Markdown报告生成H5页面                            │
│  - 使用招商银行品牌设计系统                                    │
│  - 响应式设计，支持手机和PC访问                                 │
│  - 纯CSS/SVG图表，无外部依赖                                   │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
输出：完整的Markdown报告 + H5可视化页面
```

## 状态管理

### 状态定义

```python
class WorkflowState:
    """工作流状态"""
    
    # 基础信息
    business_line: str           # 业务线（如：贵金属）
    team: str                    # 团队（如：商品交易团队）
    period: str                  # 分析周期（如：2025年Q1）
    
    # 数据收集模式
    collection_mode: str         # A: 引导式, B: 数据库
    
    # 数据状态
    raw_data: dict               # 原始收集数据
    validated_data: dict          # 验证后数据
    calculated_metrics: dict     # 计算后指标
    
    # 报告状态
    markdown_report: str          # Markdown报告内容
    user_confirmed: bool         # 用户是否确认
    h5_page: str                 # H5页面HTML
    
    # 阶段状态
    current_phase: int           # 当前阶段 (0-5)
    phase_status: dict            # 各阶段状态
```

### 状态转换图

```
┌─────────┐    用户请求    ┌─────────┐
│  IDLE   │ ────────────→ │ PHASE_0 │
└─────────┘               └────┬────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
         ▼                     ▼                     ▼
    ┌─────────┐          ┌─────────┐          ┌─────────┐
    │ PHASE_1 │ ────────→│ PHASE_2 │ ────────→│ PHASE_3 │
    │数据收集 │          │数据验证 │          │指标计算 │
    └────┬────┘          └────┬────┘          └────┬────┘
         │                     │                     │
         └─────────────────────┼─────────────────────┘
                               │
                               ▼
                         ┌─────────┐
                         │ PHASE_4 │
                         │报告生成 │
                         └────┬────┘
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
            ▼                 │                 ▼
      ┌─────────┐            │           ┌─────────┐
      │用户不满意 │ ←──────────┘           │ 用户确认 │
      │ 需调整  │                        └────┬────┘
      └─────────┘                               │
                                                ▼
                                          ┌─────────┐
                                          │ PHASE_5 │
                                          │H5可视化 │
                                          └────┬────┘
                                               │
                                               ▼
                                          ┌─────────┐
                                          │ COMPLETE│
                                          └─────────┘
```

## 子技能调用接口

### 数据收集子技能 (data-collection)

```python
# 模式A: 引导式数据收集
result = invoke_skill(
    skill="data-collection/guided_mode",
    params={
        "business_line": "贵金属",
        "period": "2025年Q1",
        "required_fields": [
            "total_revenue",
            "yoy_growth", 
            "budget_completion_rate",
            "revenue_breakdown",
            "trading_strategies",
            "risk_metrics"
        ]
    }
)

# 模式B: 数据库采集
result = invoke_skill(
    skill="data-collection/database_mode",
    params={
        "db_type": "oracle",
        "business_line": "贵金属",
        "period": "2025年Q1",
        "connection_config": {
            "host": "10.10.10.100",
            "port": 1521,
            "service_name": "FICCDB"
        }
    }
)
```

### 指标计算子技能 (metrics-calculation)

```python
result = invoke_skill(
    skill="metrics-calculation",
    params={
        "calculations": [
            {"type": "raroc", "params": {...}},
            {"type": "eva", "params": {...}},
            {"type": "pnl_attribution", "params": {...}},
            {"type": "hedge_effectiveness", "params": {...}}
        ],
        "input_data": validated_data
    }
)
```

### 报告生成子技能 (report-generation)

```python
result = invoke_skill(
    skill="report-generation",
    params={
        "business_line": "贵金属",
        "template_type": "commodities/precious_metal",
        "data": calculated_data,
        "output_format": "markdown"
    }
)
```

### H5可视化子技能 (h5-visualization)

```python
result = invoke_skill(
    skill="h5-visualization",
    params={
        "markdown_report": markdown_content,
        "theme": "cmb",  # 招商银行主题
        "responsive": True,
        "charts": [
            {"type": "bar", "data": ...},
            {"type": "line", "data": ...},
            {"type": "pie", "data": ...}
        ]
    }
)
```

## 错误处理与恢复

### 错误类型

```python
class WorkflowError(Exception):
    """工作流错误基类"""
    pass

class PhaseError(WorkflowError):
    """阶段执行错误"""
    def __init__(self, phase: int, message: str):
        self.phase = phase
        self.message = message

class DataValidationError(WorkflowError):
    """数据验证错误"""
    pass

class UserInterruptError(WorkflowError):
    """用户中断错误"""
    pass
```

### 恢复策略

```python
def handle_error(error: WorkflowError, state: WorkflowState):
    """错误处理与恢复"""
    
    if isinstance(error, PhaseError):
        # 阶段错误：重试或跳过
        if can_retry(state):
            return retry_phase(state)
        else:
            return skip_to_next_phase(state)
    
    elif isinstance(error, DataValidationError):
        # 数据验证错误：返回数据收集阶段
        return rollback_to_phase(state, phase=1)
    
    elif isinstance(error, UserInterruptError):
        # 用户中断：保存状态，等待恢复
        save_checkpoint(state)
        return pause_workflow(state)
```

## 文档版本

- **Version**: 1.0
- **Last Updated**: 2026-03-01
- **Author**: OpenClaw
