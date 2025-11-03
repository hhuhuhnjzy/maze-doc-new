LanggraphClient API 参考
========================

本文档覆盖 LanggraphClient 提供的 Python 与命令行接口，帮助你在代码与 CI/CD 管道中自动化管理工作流。

模块结构
--------

.. rubric:: 核心模块

- ``langgraph_client.session``：会话管理、认证与权限控制。
- ``langgraph_client.graph``：图构建、验证与导出。
- ``langgraph_client.deploy``：部署、版本管理与回滚。

.. rubric:: 辅助模块

- ``langgraph_client.cli``：命令行入口，提供 `init`、`validate`、`deploy` 等子命令。
- ``langgraph_client.utils``：常用的序列化、日志和异常辅助方法。

常用类与方法
------------

.. list-table::
   :header-rows: 1

   * - 类 / 函数
     - 描述
     - 关键参数
   * - ``LanggraphClient(config)``
     - 创建客户端实例并初始化与 Maze 的连接。
     - ``config``：配置文件路径或字典。
   * - ``GraphBuilder(name)``
     - 通过编程方式构建工作流图。
     - ``name``：图名称；``version``：可选版本号。
   * - ``builder.add_node(node)``
     - 向图中添加节点。
     - ``node``：节点描述或模板 ID。
   * - ``builder.link(src, dst, condition=None)``
     - 建立节点之间的数据或控制流。
     - ``condition``：可选触发条件表达式。
   * - ``client.validate(graph)``
     - 静态校验图定义，返回诊断信息。
     - ``graph``：`GraphBuilder` 生成的对象或 YAML。
   * - ``client.deploy(graph, runtime)`` 
     - 将图部署到指定运行时环境。
     - ``runtime``：目标环境标识（如 ``dev``、``prod``）。

CLI 子命令
---------

.. code-block:: console

   $ langgraph-client init        # 初始化项目结构
   $ langgraph-client validate    # 校验当前图定义
   $ langgraph-client deploy prod # 部署到生产环境
   $ langgraph-client status      # 查看部署状态
   $ langgraph-client logs <run>  # 追踪运行日志

错误处理
--------

整理常见异常（如认证失败、图校验错误、部署冲突等），并给出异常类或错误码对应的排查建议。完成表格后可链接到 FAQ。

API 生成
--------

若计划使用 Sphinx 自动生成 API，请在本文件中引入相应的 autodoc 指令，例如：

.. code-block:: rst

   .. automodule:: langgraph_client.graph
      :members:
      :show-inheritance:

在项目配置中启用 ``sphinx.ext.autodoc`` 后，确保 ``PYTHONPATH`` 包含 LanggraphClient 的源代码路径。 
