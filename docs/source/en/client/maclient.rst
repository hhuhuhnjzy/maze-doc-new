MaClient User Guide
===================

The MaClient SDK provides a simple and intuitive way to build distributed workflows. This guide covers the key concepts and common usage patterns.

Core Concepts
-------------

Workflow
~~~~~~~~

A **workflow** is a Directed Acyclic Graph (DAG) composed of tasks and their dependencies. Workflows define:

- The execution order of tasks based on their dependencies
- Data flow between tasks
- Resource allocation for parallel execution

Key characteristics:

- **Parallel Execution**: Tasks without dependencies can run simultaneously
- **Automatic Scheduling**: The system determines execution order based on dependencies
- **Resource Management**: Intelligent allocation of CPU, memory, and GPU resources

Task
~~~~

A **task** is the basic execution unit in a workflow. Each task contains:

- **Input Parameters**: Data required for execution
- **Execution Code**: The processing logic (Python function)
- **Output Parameters**: Results produced after execution
- **Resource Requirements**: CPU, memory, GPU specifications

Tasks are defined using the ``@task`` decorator:

.. code-block:: python

    @task(
        inputs=["input_key"],
        outputs=["output_key"],
        resources={"cpu": 1, "cpu_mem": 128, "gpu": 0, "gpu_mem": 0}
    )
    def my_task(params):
        value = params.get("input_key")
        # Process the value
        result = value.upper()
        return {"output_key": result}

Edge
~~~~

An **edge** represents a dependency and data flow between tasks:

- **Dependency**: ``A → B`` means task B executes only after task A completes
- **Data Flow**: Task A's outputs can be used as task B's inputs

Maze automatically creates edges when you reference task outputs:

.. code-block:: python

    task1 = workflow.add_task(func1, inputs={"in": "value"})
    task2 = workflow.add_task(func2, inputs={"in": task1.outputs["out"]})
    # Edge automatically created: task1 → task2

Workflow Patterns
-----------------

Single End Task Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~

A linear workflow where all tasks eventually flow to a single end task:

.. code-block:: text

    task1 → task2 → task3 (END)

Example - Data Processing Pipeline:

.. code-block:: python

    from maze.client.maze.client import MaClient
    from maze.client.maze.decorator import task

    @task(inputs=["filename"], outputs=["data"])
    def load_data(params):
        filename = params.get("filename")
        # Simulate loading data
        data = f"Data from {filename}"
        return {"data": data}

    @task(inputs=["data"], outputs=["cleaned_data"])
    def clean_data(params):
        data = params.get("data")
        cleaned = data.strip().upper()
        return {"cleaned_data": cleaned}

    @task(inputs=["data"], outputs=["result"])
    def analyze_data(params):
        data = params.get("data")
        result = f"Analysis: {len(data)} characters"
        return {"result": result}

    # Create workflow
    client = MaClient()
    workflow = client.create_workflow()

    # Build linear pipeline
    t1 = workflow.add_task(load_data, inputs={"filename": "data.txt"})
    t2 = workflow.add_task(clean_data, inputs={"data": t1.outputs["data"]})
    t3 = workflow.add_task(analyze_data, inputs={"data": t2.outputs["cleaned_data"]})

    # Run workflow
    workflow.run()
    for msg in workflow.get_results():
        if msg.get("type") == "finish_workflow":
            break

Multiple End Tasks Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A workflow with multiple independent end tasks, useful for parallel processing:

.. code-block:: text

              ┌→ task2 (END)
    task1 ────┤
              └→ task3 (END)

Example - Parallel Data Export:

.. code-block:: python

    from maze.client.maze.client import MaClient
    from maze.client.maze.decorator import task

    @task(inputs=["source"], outputs=["data"])
    def fetch_data(params):
        """Fetch data from a source"""
        source = params.get("source")
        data = f"Data from {source}"
        return {"data": data}

    @task(inputs=["data"], outputs=["json_result"])
    def export_json(params):
        """Export data as JSON"""
        import json
        data = params.get("data")
        result = json.dumps({"data": data})
        return {"json_result": result}

    @task(inputs=["data"], outputs=["csv_result"])
    def export_csv(params):
        """Export data as CSV"""
        data = params.get("data")
        result = f"data\n{data}"
        return {"csv_result": result}

    # Create workflow
    client = MaClient()
    workflow = client.create_workflow()

    # Build branching workflow
    t1 = workflow.add_task(fetch_data, inputs={"source": "database"})
    t2 = workflow.add_task(export_json, inputs={"data": t1.outputs["data"]})
    t3 = workflow.add_task(export_csv, inputs={"data": t1.outputs["data"]})
    
    # t2 and t3 run in parallel after t1 completes
    # Both are end tasks (no tasks depend on them)

    workflow.run()
    for msg in workflow.get_results():
        if msg.get("type") == "finish_workflow":
            break

Diamond Pattern Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~

A more complex pattern where parallel tasks merge back to a final task:

.. code-block:: text

              ┌→ task2 ─┐
    task1 ────┤         ├→ task4 (END)
              └→ task3 ─┘

Example - Parallel Processing with Merge:

.. code-block:: python

    from maze.client.maze.client import MaClient
    from maze.client.maze.decorator import task

    @task(inputs=["input"], outputs=["data"])
    def prepare_data(params):
        """Prepare data for processing"""
        input_data = params.get("input")
        return {"data": f"Prepared: {input_data}"}

    @task(inputs=["data"], outputs=["result_a"])
    def process_method_a(params):
        """Process using method A"""
        data = params.get("data")
        return {"result_a": f"{data} -> Method A"}

    @task(inputs=["data"], outputs=["result_b"])
    def process_method_b(params):
        """Process using method B"""
        data = params.get("data")
        return {"result_b": f"{data} -> Method B"}

    @task(inputs=["result_a", "result_b"], outputs=["final_result"])
    def merge_results(params):
        """Merge results from both methods"""
        result_a = params.get("result_a")
        result_b = params.get("result_b")
        return {"final_result": f"Combined: [{result_a}] + [{result_b}]"}

    # Create workflow
    client = MaClient()
    workflow = client.create_workflow()

    # Build diamond pattern
    t1 = workflow.add_task(prepare_data, inputs={"input": "raw_data"})
    t2 = workflow.add_task(process_method_a, inputs={"data": t1.outputs["data"]})
    t3 = workflow.add_task(process_method_b, inputs={"data": t1.outputs["data"]})
    t4 = workflow.add_task(
        merge_results,
        inputs={
            "result_a": t2.outputs["result_a"],
            "result_b": t3.outputs["result_b"]
        }
    )
    
    # Execution flow:
    # 1. t1 runs first
    # 2. t2 and t3 run in parallel after t1
    # 3. t4 runs after both t2 and t3 complete

    workflow.run()
    for msg in workflow.get_results():
        if msg.get("type") == "finish_workflow":
            break

Complex Workflows
~~~~~~~~~~~~~~~~~

You can combine these patterns to create more complex workflows:

.. code-block:: text

                  ┌→ task2 ─┐
    task1 ────────┤         ├→ task5 (END)
                  └→ task3 ─┘
                     │
                     └────────→ task4 (END)

Example - Multi-stage Processing:

.. code-block:: python

    # task1: Initial processing
    t1 = workflow.add_task(initial_process, inputs={"data": "input"})
    
    # Parallel branch 1
    t2 = workflow.add_task(process_a, inputs={"in": t1.outputs["out"]})
    
    # Parallel branch 2
    t3 = workflow.add_task(process_b, inputs={"in": t1.outputs["out"]})
    
    # Independent end task from branch 2
    t4 = workflow.add_task(export_data, inputs={"in": t3.outputs["out"]})
    
    # Merge branch 1 and 2
    t5 = workflow.add_task(
        final_process,
        inputs={
            "a": t2.outputs["out"],
            "b": t3.outputs["out"]
        }
    )
    
    # This creates:
    # - t2 and t3 run in parallel after t1
    # - t4 depends only on t3 (runs after t3)
    # - t5 depends on both t2 and t3 (runs after both complete)
    # - t4 and t5 are both end tasks

Working with Task Outputs
--------------------------

Referencing Outputs
~~~~~~~~~~~~~~~~~~~

Use the ``.outputs`` attribute to reference task outputs:

.. code-block:: python

    task1 = workflow.add_task(func1, inputs={"input": "value"})
    
    # Reference a single output
    task2 = workflow.add_task(
        func2,
        inputs={"input": task1.outputs["output_key"]}
    )

Multiple Outputs
~~~~~~~~~~~~~~~~

Tasks can have multiple outputs, and you can reference any of them:

.. code-block:: python

    @task(inputs=["a", "b"], outputs=["sum", "product", "difference"])
    def calculate(params):
        a = params.get("a")
        b = params.get("b")
        return {
            "sum": a + b,
            "product": a * b,
            "difference": a - b
        }

    t1 = workflow.add_task(calculate, inputs={"a": 10, "b": 5})
    
    # Different tasks can use different outputs
    t2 = workflow.add_task(use_sum, inputs={"value": t1.outputs["sum"]})
    t3 = workflow.add_task(use_product, inputs={"value": t1.outputs["product"]})

Resource Configuration
----------------------

Configuring Task Resources
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Specify resource requirements for each task:

.. code-block:: python

    @task(
        inputs=["data"],
        outputs=["result"],
        resources={
            "cpu": 4,          # 4 CPU cores
            "cpu_mem": 2048,   # 2GB CPU memory
            "gpu": 1,          # 1 GPU
            "gpu_mem": 4096    # 4GB GPU memory
        }
    )
    def heavy_processing(params):
        # Compute-intensive task
        pass

Resource Configuration Guidelines:

- **CPU Cores** (``cpu``): Number of CPU cores needed
- **CPU Memory** (``cpu_mem``): Memory in MB
- **GPU** (``gpu``): Number of GPUs (can be fractional, e.g., 0.5)
- **GPU Memory** (``gpu_mem``): GPU memory in MB

Best Practices:

- Configure resources based on actual requirements to optimize cluster utilization
- Start with minimal resources and increase if needed
- Use GPU resources only for tasks that benefit from GPU acceleration
- Monitor task execution to adjust resource allocations

Handling Workflow Results
--------------------------

Real-time Result Streaming
~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``get_results()`` method returns a generator that streams execution events:

.. code-block:: python

    workflow.run()
    
    for message in workflow.get_results():
        msg_type = message.get("type")
        msg_data = message.get("data", {})
        
        if msg_type == "start_task":
            task_id = msg_data.get("task_id")
            print(f"Task {task_id} started")
            
        elif msg_type == "finish_task":
            task_id = msg_data.get("task_id")
            result = msg_data.get("result")
            print(f"Task {task_id} completed: {result}")
            
        elif msg_type == "finish_workflow":
            print("Workflow completed!")
            break

Message Types:

- ``start_task``: A task has started execution
- ``finish_task``: A task has completed (includes results)
- ``finish_workflow``: The entire workflow has completed
- ``error``: An error occurred during execution

Collecting Results
~~~~~~~~~~~~~~~~~~

You can collect all results for later processing:

.. code-block:: python

    workflow.run()
    
    task_results = {}
    for message in workflow.get_results():
        if message.get("type") == "finish_task":
            task_id = message["data"]["task_id"]
            result = message["data"]["result"]
            task_results[task_id] = result
        elif message.get("type") == "finish_workflow":
            break
    
    # Process collected results
    for task_id, result in task_results.items():
        print(f"Task {task_id}: {result}")

Silent Mode
~~~~~~~~~~~

Disable console output by setting ``verbose=False``:

.. code-block:: python

    for message in workflow.get_results(verbose=False):
        # Process messages without console output
        pass

Best Practices
--------------

Task Design
~~~~~~~~~~~

1. **Single Responsibility**: Each task should perform one clear function
2. **Clear Naming**: Use descriptive names for tasks, inputs, and outputs
3. **Error Handling**: Include try-except blocks in task functions
4. **Documentation**: Add docstrings to task functions

Workflow Design
~~~~~~~~~~~~~~~

1. **Minimize Dependencies**: Only create edges where data flow is necessary
2. **Maximize Parallelism**: Allow independent tasks to run in parallel
3. **Resource Balance**: Distribute resource requirements across tasks
4. **Reusable Tasks**: Design tasks that can be reused in multiple workflows

Performance Optimization
~~~~~~~~~~~~~~~~~~~~~~~~

1. **Right-size Resources**: Don't over-allocate resources to tasks
2. **Batch Processing**: Combine small operations into larger tasks when appropriate
3. **Data Locality**: Keep related processing steps close together
4. **Monitor Execution**: Use result messages to identify bottlenecks

Debugging Tips
~~~~~~~~~~~~~~

1. **Test Tasks Individually**: Test task functions before adding to workflows
2. **Use Print Statements**: Add logging to task functions (visible in worker logs)
3. **Check Dependencies**: Verify that task dependencies are correct
4. **Validate Outputs**: Ensure task outputs match the declared output parameters

Example Debugging:

.. code-block:: python

    @task(inputs=["value"], outputs=["result"])
    def debug_task(params):
        value = params.get("value")
        print(f"DEBUG: Input value = {value}")  # Visible in worker logs
        
        result = value * 2
        print(f"DEBUG: Output value = {result}")
        
        return {"result": result}

Next Steps
----------

- Explore the :doc:`maclient_api` for detailed API documentation
- Check example workflows in the repository
- Learn about advanced features like custom data types
- Experiment with different workflow patterns for your use case

