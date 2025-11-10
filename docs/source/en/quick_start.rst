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

    from maze import MaClient, task

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
    run_id = workflow.run()
    print(f"Workflow started with run_id: {run_id}")
    
    # Display results with automatic progress printing
    result = workflow.show_results(run_id)
    
    if result["workflow_completed"]:
        print(f"Workflow completed successfully!")
        print(f"Task results: {result['task_results']}")

Complete Example
----------------

Here's the complete code in one place:

.. code-block:: python

    from maze import MaClient, task

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

    # Run and show results
    run_id = workflow.run()
    result = workflow.show_results(run_id)
    
    if result["workflow_completed"]:
        print(f"Workflow completed!")
        print(f"Results: {result['task_results']}")

Expected Output
~~~~~~~~~~~~~~~

When you run this example, you should see output similar to:

.. code-block:: text

    Workflow started with run_id: a1b2c3d4-e5f6-7890-abcd-ef1234567890
    🔗 Connected to server, starting workflow execution...
    ▶ Task started: 12345678
    ✓ Task completed: 12345678
      result: HELLO WORLD
    ▶ Task started: 87654321
    ✓ Task completed: 87654321
      result: Processed: HELLO WORLD
    ✅ Workflow execution completed!
    
    Workflow completed!
    Results: {
        'task_id_1': {'result': 'HELLO WORLD'},
        'task_id_2': {'result': 'Processed: HELLO WORLD'}
    }

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
- **Run ID**: Each workflow execution gets a unique ``run_id`` for result retrieval
- **Multiple Runs**: Same workflow can be run multiple times with independent results
- **Result Methods**: Use ``show_results(run_id)`` for formatted output or ``get_results(run_id)`` for raw messages
- **Client-side Caching**: Results are cached automatically for efficient repeated queries

