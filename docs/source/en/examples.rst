========
Examples
========

This section provides practical examples demonstrating Maze's capabilities in real-world scenarios.

Financial Risk Assessment Workflow
===================================

This example demonstrates a comprehensive portfolio risk management system that simulates the risk assessment process used by banks and asset management companies.

**Key Features:**

* LLM-powered natural language understanding for parameter extraction
* Parallel execution of multiple risk assessment tools
* Monte Carlo simulation for financial risk modeling
* Multi-dimensional risk analysis (market, credit, and liquidity risks)

**What You'll Learn:**

* How to integrate LLM capabilities with financial computations
* Building parallel workflows for efficiency
* Implementing industry-standard risk assessment methodologies
* Generating comprehensive risk reports

**Source Code:**

For complete implementation details and usage instructions, please refer to the `Financial Risk Workflow README <https://github.com/QinbinLi/Maze/blob/develop/examples/financial_risk_workflow/README.md>`_.

**Quick Start:**

.. code-block:: bash

   # Install dependencies
   pip install -r requirements.txt
   
   # Configure API key
   export DASHSCOPE_API_KEY="your_api_key_here"
   
   # Start Maze
   mazea start --head
   
   # Run the example
   python main.py