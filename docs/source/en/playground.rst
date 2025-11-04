Maze Playground
===============

Maze Playground is a visual workflow orchestration interface that allows you to create, execute, and monitor workflows through an intuitive drag-and-drop interface. It provides a browser-based graphical environment for building complex task pipelines without writing code.

Overview
--------

Key Features
~~~~~~~~~~~~

- **Visual Workflow Builder**: Drag-and-drop interface for creating task graphs
- **Built-in Task Library**: Pre-defined tasks ready to use
- **Real-time Execution**: Live monitoring of workflow execution status
- **Result Visualization**: View task outputs and workflow results
- **One-command Startup**: Launch server and UI with a single command

Architecture
~~~~~~~~~~~~

The Playground consists of three components:

- **Frontend** (React + Vite): Visual interface running on port 5173
- **Backend** (Node.js + Express): API server and WebSocket bridge on port 3001
- **Maze Server** (FastAPI + Ray): Task execution engine on port 8000

.. code-block:: text

    Browser (http://localhost:5173)
         ↓ HTTP/WebSocket
    Node.js Backend (http://localhost:3001)
         ↓ Python Bridge (maze_bridge.py)
    Maze Client (client/front)
         ↓ HTTP API
    Maze Server (http://localhost:8000)
         ↓ Ray Cluster
    Distributed Task Execution

Getting Started
---------------

Starting Playground
~~~~~~~~~~~~~~~~~~~

Launch Maze server with Playground in a single command:

.. code-block:: bash

    maze start --head --port 8000 --playground

This command will automatically start:

- Maze server on ``http://localhost:8000``
- Playground backend on ``http://localhost:3001``
- Playground frontend on ``http://localhost:5173``

Wait for the startup messages:

.. code-block:: text

    ============================================================
    🎮 正在启动 Maze Playground...
    ============================================================
    🔧 启动 Playground 后端 (http://localhost:3001)...
    ✅ Playground 后端已启动
    🎨 启动 Playground 前端 (http://localhost:5173)...
    ✅ Playground 前端已启动
    
    ============================================================
    🎉 Playground 已成功启动！
    ============================================================
    📱 前端地址: http://localhost:5173
    🔌 后端地址: http://localhost:3001
    🎮 打开浏览器访问 http://localhost:5173 开始使用
    ============================================================

Once started, open your browser and navigate to ``http://localhost:5173``.

.. note::
   Ensure Node.js (>= 16) and npm are installed on your system. The Playground requires these dependencies to run.

Starting Without Playground
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you only need the Maze server without the visual interface:

.. code-block:: bash

    maze start --head --port 8000

Stopping Services
~~~~~~~~~~~~~~~~~

Press ``Ctrl+C`` in the terminal to stop all services. The Playground will automatically shut down both frontend and backend processes.

Using the Playground
--------------------

Interface Overview
~~~~~~~~~~~~~~~~~~

The Playground interface consists of three main areas:

**Left Panel - Task Library**
    - Built-in tasks that can be dragged onto the canvas
    - Custom task editor (development in progress)

**Center Canvas**
    - Workflow visualization area
    - Drag tasks from the left panel
    - Connect tasks by dragging from output to input nodes
    - Move and arrange tasks freely

**Right Panel - Configuration**
    - Task input parameters
    - Task metadata (inputs, outputs, resources)
    - Execution results

**Top Toolbar**
    - Run button: Execute the workflow
    - Clear button: Reset the canvas
    - Save/Load buttons (development in progress)

Creating Your First Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Step 1: Add Task Nodes
^^^^^^^^^^^^^^^^^^^^^^^

1. From the **Built-in Tasks** list on the left, drag ``task1`` onto the canvas
2. Drag ``task2`` onto the canvas
3. Position the tasks as desired

Step 2: Connect Tasks
^^^^^^^^^^^^^^^^^^^^^^

1. Click and drag from ``task1``'s output node (right side)
2. Drop on ``task2``'s input node (left side)
3. A connecting line will appear showing the dependency

The workflow structure should look like:

.. code-block:: text

    task1 → task2

Step 3: Configure Task Inputs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Click on the ``task1`` node to select it
2. In the right panel, locate the ``task1_input`` parameter
3. Enter a value, for example: ``"hello"``

Step 4: Execute Workflow
^^^^^^^^^^^^^^^^^^^^^^^^^

1. Click the **Run** button in the top toolbar
2. Watch the real-time execution status:
   
   - Tasks turn orange when executing
   - Tasks turn green when completed
   - Tasks turn red if failed

3. View results in the right panel once execution completes

Expected Results
^^^^^^^^^^^^^^^^

For the example workflow ``task1 → task2`` with input ``"hello"``:

.. code-block:: json

    {
      "task2_output": "hello2025-11-04 22:10:482025-11-04 22:10:48===="
    }

This demonstrates:

- ``task1`` receives ``"hello"``, appends a timestamp
- ``task2`` receives ``task1``'s output, appends another timestamp and ``"====``"

Built-in Tasks
~~~~~~~~~~~~~~

task1
^^^^^

Processes input text by appending a timestamp.

**Inputs:**
    - ``task1_input`` (string): Input text

**Outputs:**
    - ``task1_output`` (string): Input text + timestamp

**Resources:**
    - CPU: 1 core
    - CPU Memory: 123 MB
    - GPU: 1 unit
    - GPU Memory: 123 MB

**Example:**

Input: ``"hello"``

Output: ``"hello2025-11-04 22:10:48"``

task2
^^^^^

Processes input text by appending timestamp and suffix.

**Inputs:**
    - ``task2_input`` (string): Input text

**Outputs:**
    - ``task2_output`` (string): Input text + timestamp + "===="

**Resources:**
    - CPU: 10 cores
    - CPU Memory: 123 MB
    - GPU: 0.8 units
    - GPU Memory: 324 MB

**Example:**

Input: ``"hello2025-11-04 22:10:48"``

Output: ``"hello2025-11-04 22:10:482025-11-04 22:10:48===="``

Advanced Workflow Patterns
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sequential Workflow
^^^^^^^^^^^^^^^^^^^

Linear task chain:

.. code-block:: text

    task1 → task2 → task3

Each task processes the output from the previous task.

Parallel Workflow
^^^^^^^^^^^^^^^^^

Multiple tasks execute simultaneously:

.. code-block:: text

           ┌─ task2 ─┐
    task1 ─┤         ├─ task4
           └─ task3 ─┘

``task2`` and ``task3`` run in parallel after ``task1`` completes, then ``task4`` merges their results.

Complex Workflow
^^^^^^^^^^^^^^^^

Combination of sequential and parallel patterns:

.. code-block:: text

           ┌─ task2 ─┐
           │          ├─ task5 ─┐
    task1 ─┤          │         ├─ task7
           │          └─────────┘
           └─ task3 ─ task6 ────┘

Backend Implementation
----------------------

Bridge Architecture
~~~~~~~~~~~~~~~~~~~

The Playground backend uses a Python bridge (``maze_bridge.py``) to interface between the Node.js server and the Maze Python Client:

.. code-block:: python

    # Simplified bridge structure
    def build_and_run_workflow(workflow_id, nodes, edges):
        # Create Maze client
        client = MaClient(server_url="http://localhost:8000")
        workflow = client.create_workflow()
        
        # Add tasks from node definitions
        task_map = {}
        for node in nodes:
            if node["category"] == "builtin":
                task_func = get_builtin_task(node["taskRef"])
                ma_task = workflow.add_task(task_func, inputs=node["inputs"])
                task_map[node["id"]] = ma_task
        
        # Add edges from edge definitions
        for edge in edges:
            source_task = task_map[edge["source"]]
            target_task = task_map[edge["target"]]
            workflow.add_edge(source_task, target_task)
        
        # Execute and return results
        workflow.run()
        results = workflow.get_results(verbose=True)
        return {"success": True, "results": results}

Key Components
~~~~~~~~~~~~~~

Client Module
^^^^^^^^^^^^^

The Playground uses the ``maze.client.front`` module, which provides:

- **MaClient**: Workflow lifecycle management
- **MaWorkflow**: Task orchestration and execution
- **@task decorator**: Task definition with metadata
- **Cloudpickle serialization**: Function serialization for distributed execution

Task Metadata
^^^^^^^^^^^^^

Each task is decorated with metadata:

.. code-block:: python

    from maze.client.front.decorator import task
    
    @task(
        inputs=["task1_input"],
        outputs=["task1_output"],
        resources={"cpu": 1, "cpu_mem": 123, "gpu": 1, "gpu_mem": 123}
    )
    def task1(params):
        task_input = params.get("task1_input")
        from datetime import datetime
        time_str = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        result = task_input + time_str
        return {"task1_output": result}

The decorator automatically:

1. Extracts function source code (``code_str``)
2. Serializes the function with cloudpickle (``code_ser``)
3. Defines input/output parameters
4. Specifies resource requirements

WebSocket Communication
^^^^^^^^^^^^^^^^^^^^^^^

Real-time workflow updates are streamed via WebSocket:

.. code-block:: javascript

    // Frontend subscribes to workflow events
    const ws = new WebSocket(`ws://localhost:8000/get_workflow_res/${workflowId}`);
    
    ws.onmessage = (event) => {
        const message = JSON.parse(event.data);
        
        if (message.type === 'start_task') {
            // Update UI: task started
        } else if (message.type === 'finish_task') {
            // Update UI: task completed
            console.log('Result:', message.data.result);
        } else if (message.type === 'finish_workflow') {
            // Update UI: workflow completed
        }
    };

Message format:

.. code-block:: json

    {
      "type": "start_task",
      "data": {
        "workflow_id": "abc123",
        "task_id": "task-456",
        "node_ip": "127.0.0.1"
      }
    }

.. code-block:: json

    {
      "type": "finish_task",
      "data": {
        "workflow_id": "abc123",
        "task_id": "task-456",
        "result": {
          "task1_output": "hello2025-11-04 22:10:48"
        }
      }
    }

Process Management
^^^^^^^^^^^^^^^^^^

The CLI automatically manages subprocess lifecycle:

.. code-block:: python

    def start_playground():
        # Start backend (Node.js)
        backend_process = subprocess.Popen(
            ["node", "src/server.js"],
            cwd=str(backend_dir),
            creationflags=subprocess.CREATE_NEW_PROCESS_GROUP  # Windows
        )
        
        # Start frontend (Vite)
        frontend_process = subprocess.Popen(
            [npm_cmd, "run", "dev"],
            cwd=str(frontend_dir),
            creationflags=subprocess.CREATE_NEW_PROCESS_GROUP
        )
        
        return [backend_process, frontend_process]
    
    def stop_playground(processes):
        # Clean shutdown on Ctrl+C
        for process in processes:
            if sys.platform == 'win32':
                subprocess.run(['taskkill', '/F', '/T', '/PID', str(process.pid)])
            else:
                os.killpg(os.getpgid(process.pid), signal.SIGTERM)

Design Decisions
~~~~~~~~~~~~~~~~

**Logging Separation**
    All verbose logging outputs to ``stderr``, while ``stdout`` is reserved for JSON results. This ensures the Node.js backend can reliably parse Python script output.

**Cloudpickle Serialization**
    Functions are serialized with cloudpickle to support:
    
    - Closures and nested functions
    - External module dependencies
    - Complex Python objects

**Module Isolation**
    The ``client/front`` module is separate from ``client/maze`` to:
    
    - Avoid import conflicts
    - Support different API patterns
    - Enable independent evolution

**Graceful Degradation**
    Some features are intentionally disabled in the current version:
    
    - File system operations (use absolute paths instead)
    - Workflow cleanup endpoint (skipped)
    - Task/tool distinction (unified handling)

Troubleshooting
---------------

Common Issues
~~~~~~~~~~~~~

**Port Already in Use**
    If ports 5173 or 3001 are occupied, stop the conflicting services or modify the port configuration in the respective ``package.json`` files.

**Node.js Not Found**
    Ensure Node.js >= 16 is installed:
    
    .. code-block:: bash
    
        node --version
        npm --version

**Python Dependencies Missing**
    Install Maze in development mode:
    
    .. code-block:: bash
    
        pip install -e .

**Workflow Execution Fails**
    Check the Maze server logs in the terminal for error messages. Common causes:
    
    - Ray cluster not initialized
    - Insufficient resources
    - Task input parameter errors

**Frontend Not Loading**
    1. Check if the backend is running (``http://localhost:3001/health``)
    2. Verify Node.js dependencies are installed (``npm install`` in frontend directory)
    3. Clear browser cache and reload

Next Steps
----------

Now that you understand Maze Playground, you can:

- Explore the :doc:`maclient` guide for programmatic workflow creation
- Review the :doc:`maclient_api` for detailed Python SDK documentation
- Create custom tasks for your specific use cases
- Build complex workflows with parallel execution patterns

Additional Resources
--------------------

- **Integration Documentation**: ``web/maze_playground/INTEGRATION.md``
- **Backend Technical Details**: ``web/maze_playground/backend/README.md``
- **Source Code**: ``web/maze_playground/frontend/`` and ``web/maze_playground/backend/``

.. note::
   The Playground is under active development. Custom task creation, workflow templates, and additional features are coming in future releases.

