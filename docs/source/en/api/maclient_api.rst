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

      :raises Exception: If the submission fails

      **Example:**

      .. code-block:: python

          workflow.run()

      .. note::
         This method only submits the workflow. Use :py:meth:`get_results` to monitor execution and retrieve results.

   .. py:method:: get_results(verbose=True)

      Stream real-time execution results via WebSocket.

      :param bool verbose: If ``True``, print messages to console. Default is ``True``.
      :returns: Generator yielding execution messages
      :rtype: Iterator[dict]
      :raises Exception: If an error occurs during execution

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
                    "task_id": "task_12345"
                }
            }

      - ``finish_task``: A task has completed

        .. code-block:: python

            {
                "type": "finish_task",
                "data": {
                    "task_id": "task_12345",
                    "result": {"output_key": "output_value"}
                }
            }

      - ``finish_workflow``: The workflow has completed

        .. code-block:: python

            {
                "type": "finish_workflow",
                "data": {}
            }

      **Example 1 - Basic usage:**

      .. code-block:: python

          workflow.run()
          
          for message in workflow.get_results():
              msg_type = message.get("type")
              
              if msg_type == "finish_task":
                  task_id = message["data"]["task_id"]
                  result = message["data"]["result"]
                  print(f"Task {task_id} completed: {result}")
                  
              elif msg_type == "finish_workflow":
                  print("Workflow completed!")
                  break

      **Example 2 - Silent mode:**

      .. code-block:: python

          workflow.run()
          
          results = []
          for message in workflow.get_results(verbose=False):
              if message.get("type") == "finish_task":
                  results.append(message["data"]["result"])
              elif message.get("type") == "finish_workflow":
                  break

      **Example 3 - Error handling:**

      .. code-block:: python

          workflow.run()
          
          try:
              for message in workflow.get_results():
                  if message.get("type") == "error":
                      print(f"Error: {message['data']}")
                  elif message.get("type") == "finish_workflow":
                      break
          except Exception as e:
              print(f"Workflow execution failed: {e}")

   .. py:method:: show_results(output_dir="workflow_results")

      Simple interface to run workflow and display results with automatic progress printing.

      This is a high-level wrapper around :py:meth:`get_results` that automatically enables
      verbose output and handles file downloads. Perfect for quick testing and demos.

      :param str output_dir: Directory for downloading result files (default: ``"workflow_results"``)
      :returns: Final task output with file paths replaced by local paths
      :rtype: dict
      :raises Exception: If an error occurs during execution

      **Example 1 - Basic usage:**

      .. code-block:: python

          workflow.run()
          result = workflow.show_results()
          # Automatically prints execution progress
          # Returns final task outputs

      **Example 2 - Custom output directory:**

      .. code-block:: python

          workflow.run()
          result = workflow.show_results(output_dir="my_results")
          print(f"Final result: {result}")

      **Example 3 - Complete workflow:**

      .. code-block:: python

          from maze import MaClient, task

          @task(inputs=["text"], outputs=["result"])
          def process(params):
              return {"result": params["text"].upper()}

          client = MaClient()
          workflow = client.create_workflow()
          
          task1 = workflow.add_task(process, inputs={"text": "hello"})
          
          workflow.run()
          result = workflow.show_results()
          # Output:
          # ▶ Task started: 12345678...
          # ✓ Task completed: 12345678...
          # 
          # Downloading result files...
          # ✓ Workflow completed
          
          print(result)  # {'result': 'HELLO'}

      .. note::
         ``show_results()`` is designed for simplicity and convenience. For advanced use cases
         requiring custom result processing, use :py:meth:`get_results` instead.

      .. tip::
         This method automatically:
         
         - Prints execution progress to console
         - Downloads files from file-type outputs
         - Returns the final task's output dictionary
         - Handles file path replacement with local paths

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

   - **resources** (Dict[str, Any], optional): Resource requirements for the task:

     - ``cpu`` (int/float): Number of CPU cores (default: 1)
     - ``cpu_mem`` (int): CPU memory in MB (default: 128)
     - ``gpu`` (int/float): Number of GPUs (default: 0)
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
           resources={"cpu": 1, "cpu_mem": 128, "gpu": 0, "gpu_mem": 0}
       )
       def uppercase_text(params):
           text = params.get("text")
           return {"result": text.upper()}

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

   **Example 5 - Custom data types:**

   .. code-block:: python

       @task(
           inputs=["count", "rate"],
           outputs=["total"],
           resources={"cpu": 1, "cpu_mem": 128, "gpu": 0, "gpu_mem": 0},
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

