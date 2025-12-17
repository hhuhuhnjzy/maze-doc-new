LLM Instance Management
=======================

Maze provides functionality to manage LLM instances directly through the MaClient. This allows you to dynamically start, query, and stop LLM instances as needed for your workflows.

start_llm_instance
------------------

Starts a new LLM instance with the specified model.

**Function Signature:**

.. code-block:: python

    def start_llm_instance(self, model: str)

**Parameters:**

- ``model`` (str): The name of the model to deploy (e.g., "facebook/opt-125m")

**Returns:**

- ``instance_id`` (str): A unique identifier for the created LLM instance

**Example:**

.. code-block:: python

    from maze import MaClient
    
    client = MaClient()
    instance_id = client.start_llm_instance("facebook/opt-125m")
    print(f"Started LLM instance with ID: {instance_id}")

**Notes:**

- The function automatically allocates resources (5 CPUs, 1024MB memory, 1 GPU) for the instance
- The instance runs as a separate service that can be queried through the OpenAI API interface
- The instance ID is needed for subsequent operations (querying or stopping the instance)

query_llm_instance
------------------

Queries an LLM instance with a given prompt and returns the model's response.

**Function Signature:**

.. code-block:: python

    def query_llm_instance(self, query: str, instance_id: str)

**Parameters:**

- ``query`` (str): The prompt or query to send to the LLM
- ``instance_id`` (str): The unique identifier of the LLM instance to query

**Returns:**

- ``response`` (str): The text response from the LLM

**Example:**

.. code-block:: python

    from maze import MaClient
    
    client = MaClient()
    instance_id = client.start_llm_instance("facebook/opt-125m")
    
    response = client.query_llm_instance("What is the capital of France?", instance_id)
    print(f"LLM Response: {response}")
    
    # Stop the instance when done
    client.stop_llm_instance(instance_id)

**Notes:**

- Uses the OpenAI API interface internally to communicate with the LLM instance
- The function uses the model name associated with the instance ID
- The response is the raw text output from the model

stop_llm_instance
-----------------

Stops and cleans up a running LLM instance.

**Function Signature:**

.. code-block:: python

    def stop_llm_instance(self, instance_id: str)

**Parameters:**

- ``instance_id`` (str): The unique identifier of the LLM instance to stop

**Returns:**

- ``response`` (dict): Server response confirming the instance was stopped

**Example:**

.. code-block:: python

    from maze import MaClient
    
    client = MaClient()
    instance_id = client.start_llm_instance("facebook/opt-125m")
    
    # ... use the instance ...
    
    # Clean up when done
    client.stop_llm_instance(instance_id)
    print("LLM instance stopped successfully")

**Notes:**

- Frees all resources allocated to the instance
- Removes the instance from the client's internal tracking
- Should always be called when the instance is no longer needed to prevent resource leaks

Complete Example
----------------

Here's a complete example showing how to use all three functions together:

.. code-block:: python

    import pytest
    from maze import MaClient

    class TestLLMInstance:
        def test_llm_instance(self):
            client = MaClient()
            # 1. Create instance
            instance_id = client.start_llm_instance("facebook/opt-125m")
            # 2. Query instance
            response = client.query_llm_instance("What is the capital of France?", instance_id)
            print(response)
            # 3. Stop instance
            client.stop_llm_instance(instance_id)

    if __name__ == "__main__":
        pytest.main([__file__, "-v", "-s"])