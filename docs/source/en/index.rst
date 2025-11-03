Welcome to Maze Documentation!
================================

.. toctree::
   :hidden:
   :maxdepth: 2

   overview
   installation
   quick_start
   maclient_python_sdk
   langgraph_client
   examples
   maclient_api
   langgraph_api

Overview
--------

Maze 是一个专为构建与编排大型语言模型 (LLM) 多智能体工作流而设计的分布式框架。它帮助你在一致的抽象层上定义、调度和监控智能体任务，并提供与现有基础设施的无缝集成。

**核心优势**

- 模块化：通过解耦的任务、管道与服务模块快速构建复杂工作流。
- 可扩展：内建分布式执行与状态管理，支持生产级部署。
- 可观测：内置监控与追踪，让你随时掌握代理执行的全貌。

**核心概念**

- **Agent / Node**：执行具体推理或行动的最小单元。
- **Workflow**：由节点组成的有向图，定义任务的运行顺序与依赖关系。
- **Runtime**：管理资源调度、扩缩容以及错误恢复的执行层。

想了解更多设计理念，请阅读 :doc:`Maze 概述 <overview>`。

Install
-------

按照 :doc:`安装指南 <installation>` 配置运行 Maze 所需的依赖与环境。该章节涵盖：

- 必备系统与 Python 依赖
- 推荐的虚拟环境管理方式
- 可选的 GPU 与分布式部署配置

Quick Start
-----------

跟随 :doc:`快速上手指南 <quick_start>` 创建你的第一个 Maze 工作流。章节内容包括：

- 初始化工程与配置文件
- 使用 MaClient 定义最小可运行工作流
- 启动运行并验证结果

Client
------

Maze 提供两种客户端以满足不同的开发偏好，可根据需求选择或混合使用。

MaClient
~~~~~~~~

在 :doc:`MaClient 指南 <maclient_python_sdk>` 中了解如何：

- 使用 Python SDK 构建工作流拓扑与节点逻辑
- 配置输入输出、资源与重试策略
- 将工作流发布到 Maze Runtime

LanggraphClient
~~~~~~~~~~~~~~~

如果你更熟悉声明式图结构，可在 :doc:`LanggraphClient 指南 <langgraph_client>` 中：

- 使用图编辑器或 YAML/JSON 定义工作流
- 对接已有的 LangGraph 项目
- 在 Maze 中复用并运行图描述

Examples
--------

示例库展示了不同场景下的最佳实践。每个示例同时给出 MaClient 与 LanggraphClient 的实现版本，帮助你快速比较两种风格。请参考 :doc:`示例索引 <examples>`。

API
---

MaClient API
~~~~~~~~~~~~

完整的 MaClient Python API 参考请见 :doc:`maclient_api`。

LanggraphClient API
~~~~~~~~~~~~~~~~~~~

LanggraphClient 的详细 API 文档位于 :doc:`langgraph_api`。
