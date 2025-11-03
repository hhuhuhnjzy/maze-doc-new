.. _quick_start:

Quick Start
===========

This guide will help you get started with Maze Client SDK quickly. We assume you have already installed Maze following the installation guide.

Your First Workflow
-------------------

Let's create a simple workflow with two tasks that process text data.

Step 1: Define Your Tasks
~~~~~~~~~~~~~~~~~~~~~~~~~~

First, define your task functions using the ``@task`` decorator:

.. code-block:: python

    from maze.client.maze.client import MaClient
    from maze.client.maze.decorator import task

    @task(
        inputs=["text"],
        outputs=["result"],
        resources={"cpu": 1, "cpu_mem": 128, "gpu": 0, "gpu_mem": 0}
    )
    def uppercase_text(params):
        """Convert text to uppercase"""
        text = params.get("text")
        return {"result": text.upper()}

    @task(
        inputs=["text"],
        outputs=["result"],
        resources={"cpu": 1, "cpu_mem": 128, "gpu": 0, "gpu_mem": 0}
    )
    def add_prefix(params):
        """Add a prefix to the text"""
        text = params.get("text")
        return {"result": f"Processed: {text}"}

Step 2: Create a Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~

Create a client and workflow instance:

.. code-block:: python

    # Connect to the Maze server
    client = MaClient("http://localhost:8000")
    
    # Create a new workflow
    workflow = client.create_workflow()

Step 3: Add Tasks to the Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add tasks and define their dependencies by referencing task outputs:

.. code-block:: python

    # Add the first task
    task1 = workflow.add_task(
        uppercase_text,
        inputs={"text": "hello world"}
    )
    
    # Add the second task that depends on the first
    task2 = workflow.add_task(
        add_prefix,
        inputs={"text": task1.outputs["result"]}
    )

.. note::
   When you reference ``task1.outputs["result"]`` as input to ``task2``, Maze automatically creates a dependency edge between them. This means ``task2`` will only run after ``task1`` completes.

Step 4: Run the Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~

Execute the workflow and get results:

.. code-block:: python

    # Submit the workflow for execution
    workflow.run()
    
    # Get real-time results
    for message in workflow.get_results():
        msg_type = message.get("type")
        msg_data = message.get("data", {})
        
        if msg_type == "start_task":
            print(f"▶ Task started: {msg_data.get('task_id')}")
            
        elif msg_type == "finish_task":
            print(f"✓ Task completed: {msg_data.get('task_id')}")
            print(f"  Result: {msg_data.get('result')}\n")
            
        elif msg_type == "finish_workflow":
            print("🎉 Workflow completed!")
            break

Complete Example
----------------

Here's the complete code in one place:

.. code-block:: python

    from maze.client.maze.client import MaClient
    from maze.client.maze.decorator import task

    # Define tasks
    @task(
        inputs=["text"],
        outputs=["result"],
        resources={"cpu": 1, "cpu_mem": 128, "gpu": 0, "gpu_mem": 0}
    )
    def uppercase_text(params):
        text = params.get("text")
        return {"result": text.upper()}

    @task(
        inputs=["text"],
        outputs=["result"],
        resources={"cpu": 1, "cpu_mem": 128, "gpu": 0, "gpu_mem": 0}
    )
    def add_prefix(params):
        text = params.get("text")
        return {"result": f"Processed: {text}"}

    # Create client and workflow
    client = MaClient("http://localhost:8000")
    workflow = client.create_workflow()

    # Add tasks
    task1 = workflow.add_task(uppercase_text, inputs={"text": "hello world"})
    task2 = workflow.add_task(add_prefix, inputs={"text": task1.outputs["result"]})

    # Run and get results
    workflow.run()
    for message in workflow.get_results():
        if message.get("type") == "finish_workflow":
            print("Workflow completed!")
            break

Expected Output
~~~~~~~~~~~~~~~

When you run this example, you should see output similar to:

.. code-block:: text

    收到消息: {'type': 'start_task', 'data': {'task_id': '...'}}
    ▶ Task started: ...
    收到消息: {'type': 'finish_task', 'data': {'task_id': '...', 'result': {'result': 'HELLO WORLD'}}}
    ✓ Task completed: ...
      Result: {'result': 'HELLO WORLD'}

    收到消息: {'type': 'start_task', 'data': {'task_id': '...'}}
    ▶ Task started: ...
    收到消息: {'type': 'finish_task', 'data': {'task_id': '...', 'result': {'result': 'Processed: HELLO WORLD'}}}
    ✓ Task completed: ...
      Result: {'result': 'Processed: HELLO WORLD'}

    收到消息: {'type': 'finish_workflow', 'data': {}}
    🎉 Workflow completed!

Next Steps
----------

Now that you've created your first workflow, you can:

- Learn more about :doc:`maclient` to understand different workflow patterns
- Explore the :doc:`maclient_api` for detailed API documentation
- Try creating more complex workflows with parallel execution
- Experiment with different resource configurations for your tasks

Key Concepts to Remember
------------------------

- **Tasks**: Define your processing logic using the ``@task`` decorator
- **Workflows**: Container for tasks and their dependencies
- **Automatic Dependencies**: Referencing task outputs automatically creates edges
- **Real-time Results**: Use ``get_results()`` to stream execution status and outputs

