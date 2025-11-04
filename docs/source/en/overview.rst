========
Overview
========

What is Maze?
=============

Maze is a workflow orchestration framework designed to address critical challenges in parallel task execution and distributed computing. It provides intelligent resource management, true task parallelism, and native distributed deployment capabilities, making it ideal for building scalable and high-performance workflow applications.

Unlike traditional Python-based workflow frameworks that are constrained by the Global Interpreter Lock (GIL), Maze enables genuine parallel execution of ready-to-run tasks while preventing resource contention issues such as GPU out-of-memory errors.

Core Features
=============

Resource Management
-------------------

Maze intelligently coordinates resource usage across concurrent workflows and parallel tasks. When multiple tasks run simultaneously—whether within a single workflow or across multiple workflows—Maze's resource management system prevents severe overload issues like GPU OOM errors through sophisticated coordination mechanisms.

True Task Parallelism
---------------------

Maze breaks free from Python's GIL limitations to deliver genuine parallel execution of ready-to-run tasks. This represents a significant advantage over frameworks like LangGraph, which cannot achieve true parallelism due to GIL constraints. Maze supports seamless migration from LangGraph-based workflows while unlocking real parallel execution capabilities.

Distributed Deployment
----------------------

With native distributed deployment support, Maze enables you to build highly available and scalable clusters. This architecture meets the demands of large-scale concurrency and high-performance computing scenarios, allowing workflows to scale horizontally across multiple machines.

Visual Workflow Designer
------------------------

Maze provides an intuitive web-based interface with drag-and-drop functionality for designing and building workflows visually. Users can create complex workflows without writing code, monitor execution in real-time, and inspect results through an easy-to-use dashboard.
