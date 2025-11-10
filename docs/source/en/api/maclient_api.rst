MaClient API Reference
======================

This page provides detailed API documentation for the Maze Client SDK.

MaClient
--------

.. py:class:: MaClient(server_url="http://localhost:8000")

   The main client class for connecting to the Maze server and managing workflows.

   :param str server_url: The URL of the Maze server. Defaults to ``"http://localhost:8000"``.

   **Example:**

   .. code-block:: python

       from maze import MaClient

       # Connect to local server
       client = MaClient()

       # Connect to remote server
       client = MaClient("http://192.168.1.100:8000")

   .. py:method:: create_workflow()

      Create a new workflow on the server.

      :returns: A new workflow instance
      :rtype: MaWorkflow
      :raises Exception: If the workflow creation fails

      **Example:**

      .. code-block:: python

          workflow = client.create_workflow()
          print(f"Created workflow: {workflow.workflow_id}")

MaWorkflow
----------

.. py:class:: MaWorkflow(workflow_id, server_url)

   Represents a workflow that manages tasks and their execution.

   .. note::
      Workflows are typically created using :py:meth:`MaClient.create_workflow` rather than direct instantiation.

   .. py:attribute:: workflow_id

      The unique identifier of the workflow.

      :type: str

   .. py:attribute:: server_url

      The URL of the Maze server.

      :type: str

   .. py:method:: add_task(task_func=None, inputs=None, task_type="code", task_name=None, **kwargs)

      Add a task to the workflow.

      :param callable task_func: A function decorated with ``@task``. When provided, the task is automatically configured from the decorator metadata.
      :param dict inputs: Dictionary of input parameters. Values can be:
         
         - Direct values: ``{"key": "value"}``
         - Task output references: ``{"key": other_task.outputs["output_key"]}``
      
      :param str task_type: Type of task (default: ``"code"``)
      :param str task_name: Custom name for the task. If not provided, uses the function name.
      :returns: The created task instance
      :rtype: MaTask
      :raises Exception: If task creation or configuration fails

      **Example 1 - Using decorated function (recommended):**

      .. code-block:: python

          @task(inputs=["text"], outputs=["result"])
          def process_text(params):
              text = params.get("text")
              return {"result": text.upper()}

          task1 = workflow.add_task(
              process_text,
              inputs={"text": "hello"}
          )

      **Example 2 - Referencing other task outputs:**

      .. code-block:: python

          task1 = workflow.add_task(func1, inputs={"input": "value"})
          task2 = workflow.add_task(
              func2,
              inputs={"input": task1.outputs["output"]}
          )
          # Automatically creates edge: task1 → task2

      **Example 3 - Custom task name:**

      .. code-block:: python

          task = workflow.add_task(
              my_function,
              inputs={"data": "value"},
              task_name="Custom Task Name"
          )

      .. note::
         When ``inputs`` contains references to other task outputs (``TaskOutput`` objects), Maze automatically creates dependency edges between tasks.

   .. py:method:: get_tasks()

      Retrieve all tasks in the workflow.

      :returns: List of task information dictionaries
      :rtype: List[dict]
      :raises Exception: If the request fails

      **Example:**

      .. code-block:: python

          tasks = workflow.get_tasks()
          for task in tasks:
              print(f"Task ID: {task['id']}, Name: {task['name']}")

      **Return Format:**

      .. code-block:: python

          [
              {"id": "task_id_1", "name": "task_name_1"},
              {"id": "task_id_2", "name": "task_name_2"},
              ...
          ]

   .. py:method:: add_edge(source_task, target_task)

      Manually add a dependency edge between two tasks.

      :param MaTask source_task: The source (predecessor) task
      :param MaTask target_task: The target (successor) task
      :raises Exception: If edge creation fails

      **Example:**

      .. code-block:: python

          workflow.add_edge(task1, task2)  # task1 → task2

      .. note::
         You typically don't need to call this method manually. When you reference a task's output in another task's input, the edge is created automatically.

      .. warning::
         Adding an edge that would create a cycle in the workflow DAG will cause an error.

   .. py:method:: del_edge(source_task, target_task)

      Delete a dependency edge between two tasks.

      :param MaTask source_task: The source task
      :param MaTask target_task: The target task
      :raises Exception: If edge deletion fails

      **Example:**

      .. code-block:: python

          workflow.del_edge(task1, task2)

   .. py:method:: run()

      Submit the workflow for execution.

      :returns: Unique run ID for this execution
      :rtype: str
      :raises Exception: If the submission fails

      **Example:**

      .. code-block:: python

          run_id = workflow.run()
          print(f"Workflow started with run_id: {run_id}")

      .. note::
         This method submits the workflow and returns a ``run_id``. The same workflow can be run multiple times, 
         with each execution having a unique ``run_id``. Use this ``run_id`` with :py:meth:`get_results` or 
         :py:meth:`show_results` to retrieve execution results.

   .. py:method:: get_results(run_id, verbose=True)

      Retrieve execution results for a specific workflow run. Results are cached client-side for efficient repeated queries.

      :param str run_id: The run ID returned by :py:meth:`run`
      :param bool verbose: If ``True``, print raw messages to console. Default is ``True``.
      :returns: List of all execution messages
      :rtype: List[dict]
      :raises Exception: If an error occurs during execution
      :raises ValueError: If ``run_id`` is invalid

      **Message Format:**

      Each message is a dictionary with the following structure:

      .. code-block:: python

          {
              "type": "message_type",
              "data": {
                  # Type-specific data
              }
          }

      **Message Types:**

      - ``start_task``: A task has started execution

        .. code-block:: python

            {
                "type": "start_task",
                "data": {
                    "workflow_id": "workflow_123",
                    "task_id": "task_12345",
                    "node_ip": "127.0.0.1"
                }
            }

      - ``finish_task``: A task has completed successfully

        .. code-block:: python

            {
                "type": "finish_task",
                "data": {
                    "workflow_id": "workflow_123",
                    "task_id": "task_12345",
                    "result": {"output_key": "output_value"}
                }
            }

      - ``task_exception``: A task failed with an exception

        .. code-block:: python

            {
                "type": "task_exception",
                "data": {
                    "workflow_id": "workflow_123",
                    "task_id": "task_12345",
                    "result": "Traceback (most recent call last):\\n..."
                }
            }

      - ``finish_workflow``: The workflow has completed

        .. code-block:: python

            {
                "type": "finish_workflow",
                "data": {}
            }

      **Example 1 - Basic usage (raw messages):**

      .. code-block:: python

          run_id = workflow.run()
          
          messages = workflow.get_results(run_id, verbose=True)
          # Prints raw messages like:
          # {'type': 'start_task', 'data': {...}}
          # {'type': 'finish_task', 'data': {...}}
          
          for msg in messages:
              if msg["type"] == "finish_task":
                  print(f"Task result: {msg['data']['result']}")

      **Example 2 - Silent mode (no printing):**

      .. code-block:: python

          run_id = workflow.run()
          
          messages = workflow.get_results(run_id, verbose=False)
          # No console output, just returns messages
          
          task_results = {}
          for msg in messages:
              if msg["type"] == "finish_task":
                  task_id = msg["data"]["task_id"]
                  task_results[task_id] = msg["data"]["result"]

      **Example 3 - Multiple queries (uses cache):**

      .. code-block:: python

          run_id = workflow.run()
          
          # First query - fetches from server
          messages1 = workflow.get_results(run_id, verbose=False)
          
          # Second query - returns from cache (no server connection)
          messages2 = workflow.get_results(run_id, verbose=False)
          
          assert messages1 == messages2  # Same data

      .. note::
         Results are cached client-side after the first query. Subsequent calls with the same 
         ``run_id`` return cached data without reconnecting to the server. This is a workaround 
         for the server's one-time consumption design.

      .. tip::
         For user-friendly formatted output, use :py:meth:`show_results` instead. Use ``get_results`` 
         when you need the raw message data for custom processing.

   .. py:method:: show_results(run_id, output_dir="workflow_results")

      Display workflow execution results with formatted, user-friendly output.

      This is a high-level method that fetches execution messages and presents them in a 
      readable format. It also handles file downloads for the ``client.front`` module.

      :param str run_id: The run ID returned by :py:meth:`run`
      :param str output_dir: Directory for downloading result files (default: ``"workflow_results"``, only used in ``client.front``)
      :returns: Dictionary with execution summary and task results
      :rtype: dict
      :raises Exception: If an error occurs during execution

      **Return Value:**

      For ``client.maze`` module:

      .. code-block:: python

          {
              "task_results": {
                  "task_id_1": {"output_key": "value"},
                  "task_id_2": {"output_key": "value"}
              },
              "workflow_completed": True,
              "has_exception": False,
              "exception_tasks": []
          }

      **Example 1 - Basic usage:**

      .. code-block:: python

          run_id = workflow.run()
          result = workflow.show_results(run_id)
          # Prints formatted output:
          # 🔗 Connected to server, starting workflow execution...
          # ▶ Task started: task_12345678
          # ✓ Task completed: task_12345678
          #   result: HELLO WORLD
          # ✅ Workflow execution completed!
          
          print(result["task_results"])

      **Example 2 - Complete workflow:**

      .. code-block:: python

          from maze import MaClient, task

          @task(inputs=["text"], outputs=["result"])
          def process(params):
              return {"result": params["text"].upper()}

          client = MaClient()
          workflow = client.create_workflow()
          
          task1 = workflow.add_task(process, inputs={"text": "hello"})
          
          run_id = workflow.run()
          result = workflow.show_results(run_id)
          
          if result["workflow_completed"] and not result["has_exception"]:
              print("Workflow succeeded!")
              print(f"Results: {result['task_results']}")

      **Example 3 - Error handling:**

      .. code-block:: python

          run_id = workflow.run()
          result = workflow.show_results(run_id)
          
          if result["has_exception"]:
              print("Some tasks failed:")
              for task_id in result["exception_tasks"]:
                  print(f"  - {task_id}")

      **Example 4 - Multiple runs:**

      .. code-block:: python

          # Run the same workflow multiple times
          run_id1 = workflow.run()
          result1 = workflow.show_results(run_id1)
          
          run_id2 = workflow.run()
          result2 = workflow.show_results(run_id2)
          
          # Each run has independent results

      .. note::
         ``show_results()`` is designed for human-readable output. For raw message data,
         use :py:meth:`get_results` instead.

      .. tip::
         This method automatically:
         
         - Fetches results using :py:meth:`get_results` (with caching)
         - Prints formatted progress to console
         - Simplifies exception messages for readability
         - Returns a structured summary dictionary

   .. py:method:: print_graph()

      Print an ASCII visualization of the workflow structure to console.

      **Example:**

      .. code-block:: python

          workflow.print_graph()
          # Output:
          # ╔══════════════════════════════════════════════════════════════╗
          # ║              Workflow Structure Visualization               ║
          # ╚══════════════════════════════════════════════════════════════╝
          # 
          # Nodes (Tasks):
          #   • task_12345678... [process_data]
          #   • task_87654321... [format_output]
          # 
          # Dependencies (Edges):
          #   task_12345678... → task_87654321...

   .. py:method:: draw_graph(output_path="workflow.png", method="auto", figsize=(12, 8), dpi=300)

      Export the workflow graph as an image file.

      :param str output_path: Path for the output image file. Supported formats: PNG, PDF, SVG, JPG
      :param str method: Rendering method - ``"auto"`` (default), ``"graphviz"``, or ``"matplotlib"``
      :param tuple figsize: Figure size for matplotlib method (width, height in inches)
      :param int dpi: Image resolution in dots per inch
      :returns: Path to the generated image file
      :rtype: str
      :raises ImportError: If required libraries are not installed

      **Example 1 - Basic usage:**

      .. code-block:: python

          workflow.draw_graph("my_workflow.png")
          # Creates my_workflow.png using available library

      **Example 2 - High-resolution PDF:**

      .. code-block:: python

          workflow.draw_graph(
              output_path="workflow.pdf",
              method="graphviz",
              dpi=600
          )

      **Example 3 - Custom size with matplotlib:**

      .. code-block:: python

          workflow.draw_graph(
              output_path="workflow.png",
              method="matplotlib",
              figsize=(16, 10),
              dpi=150
          )

      .. note::
         Requires either ``graphviz`` (recommended) or ``matplotlib`` + ``networkx``:
         
         .. code-block:: bash
         
             # Option 1: Graphviz (recommended)
             pip install graphviz
             
             # Option 2: Matplotlib + NetworkX
             pip install matplotlib networkx

   .. py:method:: get_graph_mermaid()

      Generate Mermaid diagram syntax for the workflow.

      :returns: Mermaid diagram code as string
      :rtype: str

      **Example:**

      .. code-block:: python

          mermaid_code = workflow.get_graph_mermaid()
          print(mermaid_code)
          # Output:
          # graph TD
          #     task_12345678["process_data"]
          #     task_87654321["format_output"]
          #     task_12345678 --> task_87654321

      .. tip::
         Copy the output to `mermaid.live <https://mermaid.live>`_ for online visualization.

   .. py:method:: get_graph_info()

      Get detailed structural information about the workflow graph.

      :returns: Dictionary with nodes, edges, and statistics
      :rtype: dict

      **Return Format:**

      .. code-block:: python

          {
              "nodes": {
                  "task_id_1": {
                      "name": "task_name",
                      "func_name": "function_name",
                      "inputs": ["input1", "input2"],
                      "outputs": ["output1", "output2"],
                      "resources": {"cpu": 1, "cpu_mem": 0, "gpu": 0, "gpu_mem": 0}
                  },
                  ...
              },
              "edges": [
                  ["task_id_1", "task_id_2"],
                  ...
              ],
              "stats": {
                  "total_nodes": 5,
                  "total_edges": 4,
                  "max_depth": 3
              }
          }

      **Example:**

      .. code-block:: python

          info = workflow.get_graph_info()
          print(f"Total tasks: {info['stats']['total_nodes']}")
          print(f"Total dependencies: {info['stats']['total_edges']}")

   .. py:method:: get_task_result(run_id, task_id)

      Query a specific task's result from cached workflow execution.

      :param str run_id: The run ID of the workflow execution
      :param str task_id: The task ID to query
      :returns: Task result dictionary, or ``None`` if not found
      :rtype: Optional[dict]
      :raises ValueError: If ``run_id`` is not in cache (must call :py:meth:`get_results` or :py:meth:`show_results` first)

      **Example 1 - Query specific task:**

      .. code-block:: python

          run_id = workflow.run()
          workflow.get_results(run_id, verbose=False)  # Cache results
          
          # Query specific task
          task_result = workflow.get_task_result(run_id, task1.task_id)
          print(f"Task1 output: {task_result}")

      **Example 2 - Check multiple tasks:**

      .. code-block:: python

          run_id = workflow.run()
          workflow.show_results(run_id)  # Cache results
          
          for task in [task1, task2, task3]:
              result = workflow.get_task_result(run_id, task.task_id)
              if result:
                  print(f"{task.task_name}: {result}")
              else:
                  print(f"{task.task_name}: No result found")

      .. note::
         This method requires that results for ``run_id`` are already cached. 
         Call :py:meth:`get_results` or :py:meth:`show_results` first to populate the cache.

   .. py:method:: list_cached_runs()

      List all run IDs that have cached results.

      :returns: List of cached run IDs
      :rtype: List[str]

      **Example:**

      .. code-block:: python

          # Run workflow multiple times
          run_id1 = workflow.run()
          workflow.get_results(run_id1, verbose=False)
          
          run_id2 = workflow.run()
          workflow.get_results(run_id2, verbose=False)
          
          # List cached runs
          cached = workflow.list_cached_runs()
          print(f"Cached runs: {cached}")
          # Output: ['run_id1', 'run_id2']

   .. py:method:: clear_cache(run_id=None)

      Clear cached workflow results.

      :param str run_id: Specific run ID to clear. If ``None``, clears all cached results.

      **Example 1 - Clear specific run:**

      .. code-block:: python

          workflow.clear_cache(run_id)
          print(f"Cleared cache for {run_id}")

      **Example 2 - Clear all cache:**

      .. code-block:: python

          workflow.clear_cache()
          print("All cache cleared")

MaTask
------

.. py:class:: MaTask(task_id, workflow_id, server_url, task_name=None, output_keys=None)

   Represents a task in a workflow.

   .. note::
      Tasks are typically created using :py:meth:`MaWorkflow.add_task` rather than direct instantiation.

   .. py:attribute:: task_id

      The unique identifier of the task.

      :type: str

   .. py:attribute:: workflow_id

      The ID of the workflow this task belongs to.

      :type: str

   .. py:attribute:: task_name

      The name of the task.

      :type: str

   .. py:attribute:: outputs

      Collection of task outputs that can be referenced by other tasks.

      :type: TaskOutputs

      **Example:**

      .. code-block:: python

          task1 = workflow.add_task(func1, inputs={"in": "value"})
          
          # Reference task1's output
          output_ref = task1.outputs["output_key"]
          
          # Use in another task
          task2 = workflow.add_task(
              func2,
              inputs={"in": task1.outputs["output_key"]}
          )
   .. py:method:: delete()

      Delete the task from the workflow.

      :raises Exception: If deletion fails

      **Example:**

      .. code-block:: python

          task.delete()

TaskOutput
----------

.. py:class:: TaskOutput(task_id, output_key)

   Represents a reference to a task's output parameter.

   .. note::
      ``TaskOutput`` objects are created automatically when accessing ``task.outputs["key"]``.

   .. py:attribute:: task_id

      The ID of the task that produces this output.

      :type: str

   .. py:attribute:: output_key

      The key of the output parameter.

      :type: str

   .. py:method:: to_reference_string()

      Convert to the server's reference string format.

      :returns: Reference string in format ``"{task_id}.output.{output_key}"``
      :rtype: str

      **Example:**

      .. code-block:: python

          output = task1.outputs["result"]
          ref_str = output.to_reference_string()
          # Returns: "task_id_123.output.result"

TaskOutputs
-----------

.. py:class:: TaskOutputs(task_id, output_keys)

   A collection of task outputs supporting dictionary-style access.

   .. py:method:: __getitem__(key)

      Get an output reference by key.

      :param str key: The output parameter name
      :returns: Reference to the output
      :rtype: TaskOutput
      :raises KeyError: If the output key doesn't exist

      **Example:**

      .. code-block:: python

          output = task.outputs["result"]

   .. py:method:: keys()

      Get all output keys.

      :returns: List of output parameter names
      :rtype: list

      **Example:**

      .. code-block:: python

          for key in task.outputs.keys():
              print(f"Output: {key}")

Decorators
----------

@task
~~~~~

.. py:decorator:: task(inputs, outputs, resources=None, data_types=None)

   Decorator for defining task functions with metadata.

   :param list inputs: List of input parameter names
   :param list outputs: List of output parameter names
   :param dict resources: Resource requirements (optional)
   :param dict data_types: Data type mapping for parameters (optional)
   :returns: Decorated function with task metadata
   :rtype: callable

   **Parameters:**

   - **inputs** (List[str]): Names of input parameters that the task expects. These correspond to keys in the ``params`` dictionary.

   - **outputs** (List[str]): Names of output parameters that the task returns. These must match the keys in the returned dictionary.

   - **resources** (Dict[str, Any], optional): Resource requirements for the task. If not specified or partially specified, missing values are filled with defaults:

     - ``cpu`` (int/float): Number of CPU cores (default: 1, minimum: 1)
     - ``cpu_mem`` (int): CPU memory in MB (default: 0)
     - ``gpu`` (int/float): Number of GPUs (default: 0, auto-set to 1 if ``gpu_mem > 0``)
     - ``gpu_mem`` (int): GPU memory in MB (default: 0)

   - **data_types** (Dict[str, str], optional): Data types for parameters. Defaults to ``"str"`` for all parameters.

     - Supported types: ``"str"``, ``"int"``, ``"float"``, ``"bool"``, ``"list"``, ``"dict"``

   **Function Requirements:**

   The decorated function must:

   1. Accept a single ``params`` dictionary argument
   2. Return a dictionary containing all declared output parameters

   **Example 1 - Basic task:**

   .. code-block:: python

       from maze import task

       @task(
           inputs=["text"],
           outputs=["result"],
           resources={"cpu": 1, "cpu_mem": 128}
       )
       def uppercase_text(params):
           text = params.get("text")
           return {"result": text.upper()}
       
       # Equivalent to (auto-filled):
       # resources={"cpu": 1, "cpu_mem": 128, "gpu": 0, "gpu_mem": 0}

   **Example 2 - Multiple inputs and outputs:**

   .. code-block:: python

       @task(
           inputs=["a", "b"],
           outputs=["sum", "product", "difference"],
           resources={"cpu": 2, "cpu_mem": 256, "gpu": 0, "gpu_mem": 0}
       )
       def calculate(params):
           a = params.get("a")
           b = params.get("b")
           return {
               "sum": a + b,
               "product": a * b,
               "difference": a - b
           }

   **Example 3 - Using external libraries:**

   .. code-block:: python

       @task(
           inputs=["numbers"],
           outputs=["mean", "std"],
           resources={"cpu": 2, "cpu_mem": 512, "gpu": 0, "gpu_mem": 0}
       )
       def statistics(params):
           import numpy as np
           
           numbers = params.get("numbers")
           arr = np.array(numbers)
           
           return {
               "mean": float(np.mean(arr)),
               "std": float(np.std(arr))
           }

   **Example 4 - GPU task:**

   .. code-block:: python

       @task(
           inputs=["data"],
           outputs=["result"],
           resources={"cpu": 4, "cpu_mem": 2048, "gpu": 1, "gpu_mem": 4096}
       )
       def gpu_processing(params):
           import torch
           
           data = params.get("data")
           # GPU processing logic
           result = process_with_gpu(data)
           
           return {"result": result}

   **Example 5 - Auto-filled resources:**

   .. code-block:: python

       # Only specify gpu_mem - gpu auto-set to 1, cpu auto-set to 1
       @task(
           inputs=["data"],
           outputs=["result"],
           resources={"gpu_mem": 4096}
       )
       def auto_gpu_task(params):
           return {"result": "processed"}
       
       # Effective resources: {"cpu": 1, "cpu_mem": 0, "gpu": 1, "gpu_mem": 4096}
       
       # No resources specified - uses all defaults
       @task(inputs=["x"], outputs=["y"])
       def minimal_task(params):
           return {"y": params["x"] * 2}
       
       # Effective resources: {"cpu": 1, "cpu_mem": 0, "gpu": 0, "gpu_mem": 0}

   **Example 6 - Custom data types:**

   .. code-block:: python

       @task(
           inputs=["count", "rate"],
           outputs=["total"],
           resources={"cpu": 1, "cpu_mem": 128},
           data_types={"count": "int", "rate": "float", "total": "float"}
       )
       def calculate_total(params):
           count = params.get("count")
           rate = params.get("rate")
           return {"total": count * rate}

   .. note::
      The ``@task`` decorator uses ``cloudpickle`` to serialize the function, including any imports and closures. This allows the function to be executed on remote workers with full access to its dependencies.

   .. warning::
      Ensure that any external libraries used in the task function are installed on the worker nodes.

Data Types
----------

Input Schema Types
~~~~~~~~~~~~~~~~~~

When configuring task inputs, two schema types are supported:

- **from_user**: Direct user input

  .. code-block:: python

      {"key": "direct_value"}

- **from_task**: Reference to another task's output

  .. code-block:: python

      {"key": other_task.outputs["output_key"]}

Resource Configuration
~~~~~~~~~~~~~~~~~~~~~

Resource requirements are specified as dictionaries:

.. code-block:: python

    resources = {
        "cpu": 4,          # CPU cores (int or float)
        "cpu_mem": 2048,   # CPU memory in MB (int)
        "gpu": 1,          # GPU count (int or float)
        "gpu_mem": 4096    # GPU memory in MB (int)
    }

**Resource Units:**

- CPU cores: Number of processor cores (can be fractional, e.g., 0.5)
- CPU memory: Megabytes (MB)
- GPU: Number of GPUs (can be fractional for GPU sharing)
- GPU memory: Megabytes (MB)
See Also
--------

- :doc:`quick_start` - Get started quickly with a simple example
- :doc:`maclient` - Learn about workflow patterns and best practices
- GitHub Repository - Browse example code and contribute

