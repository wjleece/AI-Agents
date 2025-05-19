# E-commerce AI Agent with Advanced Evaluation & RAG

## Overview

This Google Colab notebook implements a sophisticated AI-driven customer service assistant for a simulated e-commerce system. The system features a worker AI agent (powered by Anthropic's Claude) capable of using tools to interact with a mock e-commerce datastore (managing customers, products, and orders). A key aspect of this project is its advanced evaluation framework, where another AI (Google's Gemini) assesses the worker agent's performance. The system incorporates an innovative **Adaptive Knowledge Integration (AKI) via Human Feedback** mechanism (a form of Retrieval Augmented Generation refinement) for persistent "learnings," and a gated prompting system to handle different types of user queries more effectively. This allows the agent's knowledge base to continually improve based on interactions and explicit guidance.

## Features

* **Tool-Using Worker AI:** The primary agent can understand user requests and utilize a predefined set of tools to perform actions like creating orders, updating product information, and fetching customer details.
* **LLM-Based Evaluation:** A separate LLM acts as an evaluator, scoring the worker AI's responses based on accuracy, efficiency, context awareness, and helpfulness.
* **Stateful Evaluation:** The evaluator is provided with "before" and "after" snapshots of the datastore to accurately assess the impact and correctness of the worker AI's actions.
* **Adaptive Knowledge Integration (AKI) via Human Feedback:**
    * This system allows for continuous improvement of the agent's knowledge base without direct model retraining.
    * Human feedback provided during interactions (e.g., clarifications given to the agent, explicit corrections, or new procedural knowledge offered by the user/admin) is captured.
    * An LLM (Gemini) processes this feedback to synthesize concise, actionable "learnings."
    * These learnings are persisted as JSON files on Google Drive, forming an evolving knowledge base.
    * Relevant learnings are retrieved and injected into the worker AI's context during subsequent interactions, enabling it to adapt and improve its responses and decision-making over time. This is an exciting way for the system to become more effective through ongoing use.
* **Gated Prompting System:**
    * User queries are classified into types (e.g., "OPERATIONAL" or "METACOGNITIVE_LEARNINGS_SUMMARY").
    * The worker AI uses specialized system prompts tailored to the query type, enhancing response quality and relevance.
    * The evaluator AI is made aware of the query type to apply more focused evaluation criteria.
* **Interactive Command-Loop:** Allows users to interact with the AI agent in a conversational manner.
* **Fuzzy Matching:** Used in product search to handle minor misspellings.

## Architecture

The system comprises several key components:

1.  **Worker Agent (Anthropic Claude):**
    * Responsible for understanding user queries and interacting with the e-commerce system.
    * Uses tools to modify or query the datastore.
    * Its behavior is guided by a system prompt that changes based on the classified query type and incorporates learnings from the AKI system.

2.  **Evaluator Agent (Google Gemini):**
    * Assesses the worker agent's performance on each turn.
    * Receives the user query, query type, conversation context, AKI learnings provided to the worker, the worker's final response, and snapshots of the datastore *before* and *after* the worker's actions.
    * Its evaluation criteria are also guided by the type of query the worker was handling.

3.  **Datastore (`Storage` class):**
    * An in-memory Python class that simulates an e-commerce database (customers, products, orders).
    * Provides methods for the tools to interact with the data.

4.  **Tools:**
    * A set of Python functions that the worker AI can call (e.g., `create_order`, `get_product_info`, `update_customer`).
    * These functions directly interact with the `Storage` instance.

5.  **Adaptive Knowledge Integration (AKI) System (Learnings):**
    * **`active_learnings_cache`**: An in-memory list holding synthesized learnings for the current session.
    * **Feedback Capture**: Human input during evaluator clarifications or general end-of-turn feedback is collected.
    * **Learning Synthesis (`process_and_store_new_learning`)**: The Gemini model is used to analyze human feedback against existing learnings, checking for conflicts or redundancy, and then synthesizes a "finalized learning statement."
    * **Persistence**: These structured learnings are saved to timestamped JSON files in a specified Google Drive path (`LEARNINGS_DRIVE_BASE_PATH`). The system loads from the most recent file on startup, ensuring knowledge continuity.
    * **Retrieval (`check_relevant_learnings`)**: Relevant learnings from the cache are retrieved based on the current query and query type to provide targeted knowledge to the worker AI for its current task.

6.  **Gated Prompts & Query Classification:**
    * **`_classify_query_type`**: A method in `AgentEvaluator` that categorizes user queries (e.g., as "OPERATIONAL" or "METACOGNITIVE_LEARNINGS_SUMMARY").
    * **Specialized Worker Prompts**: `worker_operational_system_prompt` and `worker_metacognitive_learnings_system_prompt` are used based on the classification.
    * **Evaluator Awareness**: The evaluator is informed of the query type to tailor its assessment.

7.  **`AgentEvaluator` Class:**
    * The central orchestrator of the system.
    * Manages the conversation context, AKI learnings, tool dispatch, interaction with LLMs (worker and evaluator), and the overall processing loop.

## Setup Instructions

1.  **Environment:** This notebook is designed for Google Colab.
2.  **Dependencies:** Execute the first code cell to install necessary Python packages:
    ```python
    %pip install anthropic google-generativeai fuzzywuzzy
    ```
3.  **API Keys:**
    * You will need API keys for Anthropic (Claude) and Google (Gemini).
    * Store these as secrets in your Colab environment:
        * `ANTHROPIC_API_KEY`
        * `GOOGLE_API_KEY`
4.  **Google Drive Mounting:**
    * The notebook will attempt to mount your Google Drive to persist and load AKI learnings. You'll be prompted to authorize this when the relevant code runs.
5.  **AKI Learnings Path:**
    * A default path for storing learnings on Google Drive is set: `My Drive/AI/Knowledgebases`.
    * `LEARNINGS_DRIVE_BASE_PATH = os.path.join(DRIVE_MOUNT_PATH, DEFAULT_LEARNINGS_DRIVE_SUBPATH)`
    * This directory will be created if it doesn't exist.

## How to Run

1.  **Execute Cells in Order:** Run the notebook cells sequentially from top to bottom. This ensures all classes, functions, prompts, and global variables are initialized correctly.
    * Pay attention to the output of the setup cells to confirm API clients and Drive mounting are successful.
2.  **Start the Main Loop:** The final cell typically contains a call to `main()`. Executing this cell will start the interactive loop.
3.  **Interact with the Agent:**
    * You'll be prompted to "Enter query (or 'quit'):".
    * Type your requests for the e-commerce assistant.
    * After each response from the worker AI, the evaluator AI will provide its assessment.
    * You may be prompted for human feedback (e.g., during evaluator clarification or at the end of a turn) which contributes to the AKI system.
4.  **Sample Queries:**
    * `Show me all the products available`
    * `I'd like to order 25 Perplexinators, please`
    * `Please ship my most recent order.` (Assuming an order was just placed and is 'Processing')
    * `How many Perplexinators are now left in stock?`
    * `Add a new customer: Bill Leece, bill.leece@mail.com, +1.222.333.4444`
    * `Add new product: Gizmo X, description: A fancy gizmo, price: 29.99, inventory: 50`
    * `Update Gizzmo's price to 99.99` (Tests fuzzy matching for product name)
    * `Summarize your learnings from our recent interactions.` (Tests the metacognitive path and AKI retrieval)
    * `Who won the last Super Bowl?` (Tests out-of-scope handling)
5.  **Exiting:** Type `quit` to end the session. Any new learnings acquired and synthesized during the session will be persisted to Google Drive.

## Key Code Components

* **`AgentEvaluator` class (`gated_prompts_agent_evaluator` Canvas artifact):** This is the core class orchestrating the agent's behavior, evaluation, and learning.
    * `_classify_query_type()`: Implements the gating logic.
    * `process_user_request()`: Handles a single turn of user interaction.
    * `get_anthropic_response()`: Interacts with the worker LLM.
    * `evaluate_responses()`: Interacts with the evaluator LLM.
    * AKI methods: `_load_initial_learnings_from_drive`, `_persist_active_learnings_to_drive`, `check_relevant_learnings`, `process_and_store_new_learning`.
* **System Prompts:**
    * `worker_operational_system_prompt`: For standard tasks.
    * `worker_metacognitive_learnings_system_prompt`: For summarizing AKI learnings.
    * `evaluator_system_prompt`: Base instructions for the Gemini evaluator.
* **Tool Functions:** Defined globally, these functions (e.g., `create_order`, `get_product_info`) perform actions on the `Storage` object.
* **`Storage` Class:** Simulates the e-commerce datastore.
* **`ConversationContext` Class:** Manages the history of the conversation.

## Future Enhancements & Considerations

* **More Sophisticated Query Classification:** The current keyword-based classifier could be enhanced using LLM-based classification or embeddings for more accuracy and flexibility.
* **Expanded Metacognitive Capabilities:** Add more metacognitive query types (e.g., "Why did you choose that tool?", "Explain your reasoning for the last answer.").
* **Evaluator Tool Usage:** For very complex scenarios (e.g., verifying claims from external web searches if the agent had that capability), the evaluator might also need tools, though this adds significant complexity.
* **Error Handling & Resilience:** Improve robustness in tool execution and LLM interactions.
* **User Interface:** Develop a simple web UI (e.g., using Streamlit or Flask) for a more user-friendly interaction experience.
* **Automated Test Suites:** Create automated tests to evaluate agent performance on a predefined set of scenarios.
* **Advanced Learning Synthesis:** The `process_and_store_new_learning` method could employ more sophisticated techniques for conflict resolution, generalization, or merging of new feedback with existing learnings.

