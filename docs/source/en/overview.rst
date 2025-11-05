========
Overview
========

What is Maze?
=============

Maze is a agent workflow orchestration framework designed to address critical challenges in parallel task execution and distributed computing. It provides intelligent resource management, true task parallelism, and native distributed deployment capabilities, making it ideal for building scalable and high-performance workflow applications.


Core Features
=============

Task-level
-------------------

Unlike LangGraph’s agent-level execution model—which runs the entire agent workflow sequentially in a single process—Maze employs task-level parallelism, enabling true concurrent execution of individual tasks. In compute-intensive scenarios, Maze can significantly improve end-to-end (e2e) performance. **Moreover, Maze can serve as a runtime backend for LangGraph**, allowing existing LangGraph workflows to be seamlessly migrated to Maze and automatically gain task-level parallelism without modifying original logic. `Example <https://github.com/QinbinLi/Maze/tree/develop/examples/financial_risk_workflow>`_

Resource Management
---------------------

When multiple tasks run in parallel within a single workflow—or when multiple workflows execute concurrently—resource contention can occur. Without proper coordination, this may lead to severe resource overloads, such as GPU out-of-memory (OOM) errors.

Distributed Deployment
----------------------

Maze natively supports distributed deployment, allowing you to build highly available and scalable Maze clusters to meet the demands of large-scale concurrency and high-performance computing.