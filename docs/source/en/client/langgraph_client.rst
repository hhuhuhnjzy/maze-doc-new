LanggraphClient Usage Guide
===========================

This document describes how to use the `LanggraphClient` class to define and execute distributed tasks within a LangGraph workflow.

Initialization
------------

To use `LanggraphClient`, first create an instance by specifying the address of the Maze server:

.. code-block:: python

   from maze import LanggraphClient

   MAZE_SERVER_ADDR = "MAZE_HEAD_IP:MAZE_HEAD_PORT"
   client = LanggraphClient(addr=MAZE_SERVER_ADDR)

Defining Tasks
--------------

Use the `@client.task` decorator to register functions as executable nodes in the graph. These functions will be executed remotely via the Maze server.

.. code-block:: python

   @client.task
   def task1(state: GraphState) -> GraphState:
       result = "task1"
       return {"result1": result}

   @client.task
   def task2(state: GraphState) -> GraphState:
       result = "task2"
       return {"result2": result}

   @client.task
   def task3(state: GraphState) -> GraphState:
       result = "task3"
       return {"result3": result}

Building the Graph
------------------

Construct your workflow using `StateGraph` from LangGraph. Add nodes and edges to define the execution flow.

.. code-block:: python

   from langgraph.graph import StateGraph, START, END

   class GraphState(TypedDict):
       result1: str
       result2: str
       result3: str

   builder = StateGraph(GraphState)
   builder.add_node("task1", task1)
   builder.add_node("task2", task2)
   builder.add_node("task3", task3)

   builder.add_edge(START, "task1")
   builder.add_edge(START, "task2")
   builder.add_edge(START, "task3")
  
   builder.add_edge("task1", END)
   builder.add_edge("task2", END)
   builder.add_edge("task3", END)

   graph = builder.compile()

Executing the Workflow
----------------------

Invoke the compiled graph with an initial state to start execution.

.. code-block:: python

   initial_state = {"results": []}
   result = graph.invoke(initial_state)



