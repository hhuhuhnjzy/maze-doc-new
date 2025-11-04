Financial Risk Assessment Workflow
====================================

Overview
--------

This example demonstrates a **portfolio risk management system** that simulates the risk assessment process used by banks and asset management companies. The workflow leverages LLM capabilities combined with Monte Carlo simulations to perform comprehensive risk analysis.

Business Scenario
-----------------

Imagine you are a risk management manager at an asset management company. A client entrusts you with managing a 100 million yuan investment portfolio composed of:

* 60% stocks
* 30% bonds  
* 10% cash

Before investing, you need to assess the various risks this portfolio may face and provide the client with a professional risk assessment report.

Workflow Architecture
---------------------

The workflow follows this execution flow:

.. code-block:: text

   1. Investment Portfolio Description Input
      ↓
   2. [LLM Intelligent Analysis] Extract Key Parameters
      ↓
   3. Execute Three Risk Assessments in Parallel
      ├─ [Market Risk] Calculate Potential Market Losses
      ├─ [Credit Risk] Assess Default Risk
      └─ [Liquidity Risk] Test Liquidation Capability
      ↓
   4. [Summary Report] Generate Comprehensive Risk Assessment

Key Components
--------------

1. LLM Analysis Node
~~~~~~~~~~~~~~~~~~~~

**Purpose:** Acts as an experienced financial analyst, reading text descriptions of investment portfolios and automatically extracting key data.

**Input Example:**

.. code-block:: text

   "Mixed investment portfolio with a total scale of 100 million yuan,
   containing 60% stocks, 30% bonds, 10% cash,
   investment period of 1 year, with moderate risk preference."

**What LLM Does:**

* Understands natural language descriptions
* Extracts key parameters: total value, volatility, confidence level, time horizon
* Converts text into computable numerical parameters

**Output:**

.. code-block:: python

   {
       "portfolio_value": 10000,      # 100 million yuan
       "volatility": 0.25,            # 25% volatility
       "confidence_level": 0.95,      # 95% confidence level
       "num_simulations": 500000,     # 500,000 simulations
       "time_horizon": 252            # 1 year (252 trading days)
   }

**Why LLM?** In actual business, clients or business departments provide text descriptions, not structured data. LLM automatically understands and extracts this information, eliminating manual data entry.

2. Market Risk Assessment Tool
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Purpose:** Answers "If the market declines, how much money will I lose in the worst case?"

**Algorithm:** Monte Carlo Simulation + VaR (Value at Risk) Calculation

**Process:**

1. **Simulate 500,000 Future Price Paths**
   
   * Each path represents a possible market trend
   * Uses Geometric Brownian Motion model (financial market standard)
   * Considers volatility: 25% (daily price fluctuation)

2. **Calculate VaR Value**
   
   * At 95% confidence level, identifies the worst 5% scenarios
   * The average loss in these 5% scenarios is the VaR

**Business Interpretation:**

.. code-block:: text

   "With 95% probability, the maximum loss within 1 year 
   will not exceed 25 million yuan"

This tells the client: under normal circumstances (95% probability), the loss will not exceed this amount. However, there is still a 5% chance of extreme cases with greater losses.

3. Credit Risk Assessment Tool
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Purpose:** Answers "If bond issuers default, how much will I lose?"

**Business Context:** The investment portfolio contains 30% corporate bonds. Companies may go bankrupt and default, resulting in principal loss.

**Process:**

1. **Assume 100 Bond Issuers**
   
   * Each issuer has a 2% default probability
   * 60% of principal is lost upon default

2. **Simulate 500,000 Default Scenarios**
   
   * Each simulation randomly determines which companies default
   * Calculates total loss in each scenario

3. **Statistical Analysis**
   
   * Expected Loss: average loss across all scenarios
   * 95% CVaR: maximum loss at 95% probability
   * 99% CVaR: maximum loss at 99% probability

**Business Interpretation:**

.. code-block:: text

   "Expected Loss: 1.2 million yuan
   Maximum Loss at 95% Confidence: 3.5 million yuan
   Maximum Loss at 99% Confidence: 5.2 million yuan"

This helps clients understand: on average, the loss will be 1.2 million, but in extreme cases, it could exceed 5 million.

4. Liquidity Risk Assessment Tool
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Purpose:** Answers "If liquidation is suddenly needed, how much will be lost?"

**Business Context:** Clients may suddenly need to redeem funds (e.g., urgent need for money). Assets must be sold quickly, but the market may not have enough buyers, forcing discounted sales.

**Process:**

1. **Simulate 1,000 Stress Scenarios**
   
   * Different redemption pressures (how much money clients want back)
   * Different market liquidity conditions (availability of buyers)

2. **Simulate 500 Times Per Scenario** (500,000 total simulations)
   
   * Redemption rate: what percentage of money clients want back
   * Liquidation discount: forced discount during urgent sales (5%-30%)
   * Cash reserves: available cash to give clients directly

3. **Calculate Liquidity Loss**
   
   * If cash is insufficient, assets must be sold
   * Calculates discount loss when selling assets

**Business Interpretation:**

.. code-block:: text

   "Average Liquidity Loss: 1.8 million yuan
   Maximum Liquidity Loss: 8.5 million yuan
   95th Percentile: 4.2 million yuan"

This tells clients: if urgent redemption is needed, the average loss is 1.8 million, with a worst-case scenario of 8.5 million.

5. Report Generation Node
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Purpose:** Integrates the three-dimensional risk assessment results into a complete report.

**What It Does:**

* Collects evaluation results from the three risk assessment tools
* Formats results according to professional specifications
* Outputs a comprehensive risk assessment report

**Output Example:**

.. code-block:: text

   ====================================
   Investment Portfolio Comprehensive Risk Assessment Report
   ====================================

   【Risk Dimension 1】Market Risk
   - VaR Value: 25 million yuan
   - Maximum Potential Loss Ratio: 25%

   【Risk Dimension 2】Credit Risk
   - Expected Loss: 1.2 million yuan
   - 95% CVaR: 3.5 million yuan

   【Risk Dimension 3】Liquidity Risk
   - Average Liquidity Loss: 1.8 million yuan
   - Maximum Liquidity Loss: 8.5 million yuan
   ====================================

Workflow Summary
----------------

This workflow simulates a **complete investment portfolio risk management process**:

1. **Natural Language Understanding** (LLM) → Extract Parameters
2. **Parallel Risk Calculation** (3 Tools) → Multi-dimensional Assessment  
3. **Report Generation** → Professional Report for Clients

In real business scenarios, such a system can:

* Automatically process client investment intentions
* Quickly assess various risks across multiple dimensions
* Generate reports compliant with regulatory requirements
* Help investment managers make informed decisions

Running the Example
--------------------

1. Install Dependencies
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   pip install -r requirements.txt

2. Configure API Key
~~~~~~~~~~~~~~~~~~~~

Set your Tongyi Qianwen API Key:

.. code-block:: bash

   export DASHSCOPE_API_KEY="your_api_key_here"

3. Start Maze
~~~~~~~~~~~~~

Start Maze Head (if you have multiple machines, you can start Maze workers):

.. code-block:: bash

   mazea start --head

4. Run the Example
~~~~~~~~~~~~~~~~~~

Execute the main script:

.. code-block:: bash

   python main.py

Technical Highlights
--------------------

* **LLM Integration**: Demonstrates how to use LLM for natural language understanding and parameter extraction
* **Parallel Execution**: Three risk assessment tools run simultaneously for efficiency
* **Monte Carlo Simulation**: Industry-standard approach for financial risk modeling
* **Multi-dimensional Risk Analysis**: Comprehensive coverage of market, credit, and liquidity risks
* **Real-world Application**: Simulates actual financial risk management workflows used in the industry
