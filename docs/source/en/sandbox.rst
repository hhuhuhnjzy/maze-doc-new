.. _sandbox:

=======
Sandbox
=======

Maze provides a secure code sandbox environment that allows you to execute code in an isolated environment. This enables running potentially dangerous code snippets without compromising the host system.

Starting the Sandbox Service
=============================

.. code-block:: python

    maze-sandbox start --host 0.0.0.0 --port 8000


Basic Usage
===========

To start using the Sandbox, you need to create a ``CodeSandboxClient`` instance and connect to the Maze Sandbox server:

.. code-block:: python

    from maze.sandbox.client import CodeSandboxClient
    import asyncio

    async def main():
        # Create a code sandbox client instance
        codesandbox = CodeSandboxClient(
            url="http://0.0.0.0:8000", 
            cpu_nums=1, 
        )
        
        # Run simple code
        result = await codesandbox.run_code("print('hello world')")
        print("Execution result:", result['stdout'])

    if __name__ == "__main__":
        asyncio.run(main())

Resource Allocation
===================

When creating a ``CodeSandboxClient`` instance, you can specify the following resource limits:

- ``cpu_nums``: Number of CPU cores allocated to the sandbox
- ``gpu_nums``: Number of GPUs allocated to the sandbox
- ``memory_mb``: Memory size allocated to the sandbox (in MB)

Example: Running Complex Code
=============================

The following example demonstrates how to run complex code that requires specific libraries in the sandbox:

.. code-block:: python

    from maze.sandbox.client import CodeSandboxClient
    import asyncio

    async def main():
        codesandbox = CodeSandboxClient(
            url="http://localhost:8000", 
            cpu_nums=1, 
            gpu_nums=1, 
            memory_mb=100
        )
        
        # Run code that requires specific libraries
        code = """
    import torch
    import time
    print(torch.__version__)
    arr = torch.rand(1000, 1000)
    time.sleep(60)
    """
        
        result = await codesandbox.run_code(code)
        print("Execution result:", result['stdout'])

    if __name__ == "__main__":
        asyncio.run(main())

Concurrent Usage
================

You can create multiple sandbox client instances to support concurrent execution:

.. code-block:: python

    from maze.sandbox.client import CodeSandboxClient
    import asyncio

    async def main():
        # Create multiple sandbox client instances
        codesandbox1 = CodeSandboxClient(
            url="http://localhost:8000", 
            cpu_nums=1, 
            gpu_nums=1, 
            memory_mb=100
        )
        codesandbox2 = CodeSandboxClient(
            url="http://localhost:8000", 
            cpu_nums=1, 
            gpu_nums=1, 
            memory_mb=100
        )
        
        # Concurrently run code
        result1 = await codesandbox1.run_code("print('hello world')")
        result2 = await codesandbox2.run_code("print('hello world')")
        
        print("First execution result:", result1['stdout'])
        print("Second execution result:", result2['stdout'])

    if __name__ == "__main__":
        asyncio.run(main())