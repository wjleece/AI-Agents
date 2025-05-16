## AI Agent Dual Evaluation Framework

This project provides a framework for evaluating the performance of two different AI language models (Anthropic's Claude and OpenAI's GPT) acting as customer service assistants for a simulated e-commerce system. A third AI model (Google's Gemini) is used as an impartial evaluator to score the responses of the two worker agents.

### Description

The primary goal of this framework is to enable comparative analysis of different AI models in a tool-using, conversational context. It simulates an e-commerce environment where AI agents can manage customers, products, and orders using a defined set of tools. User queries are processed by both agents, and their responses, along with the conversation context, are then passed to an evaluator AI which assesses them based on predefined criteria. The system also incorporates a mechanism for human feedback to refine the evaluation process.

### Features

  * **Dual Agent Interaction:** Simultaneously processes user requests with two distinct AI models (Anthropic Claude and OpenAI GPT).
  * **Tool Integration:** Supports a range of tools for e-commerce operations, including:
      * Customer management (create, retrieve information).
      * Product management (create, update, retrieve information, list all).
      * Order management (create, retrieve details, update status).
  * **Fuzzy Matching:** Implements fuzzy string matching for product searches to handle potential misspellings.
  * **Contextual Conversations:** Maintains conversation history and relevant context (viewed customers, products, orders, last action) to enable more coherent interactions.
  * **AI-Powered Evaluation:** Utilizes a separate AI model (Google Gemini) to evaluate the worker agents' responses based on:
      * Accuracy
      * Efficiency
      * Context Awareness
      * Helpfulness
  * **Human-in-the-Loop:** Allows for human clarification during the evaluation phase if the evaluator AI deems it necessary, and stores these clarifications as "learnings."
  * **Data Simulation:** Operates on in-memory data stores for customers, products, and orders for simulation purposes.
  * **Modular Design:** Code is structured into classes for storage, conversation context, and the dual agent evaluator.

### Requirements

The script is designed to be run in a Jupyter Notebook environment (e.g., Google Colab) and requires the following Python libraries:

  * `anthropic`
  * `openai`
  * `google-generativeai`
  * `fuzzywuzzy`

These can be installed using pip:

```bash
%pip install anthropic openai google-generativeai fuzzywuzzy
```

### Configuration

**API Keys:**
Before running the notebook, you need to set up API keys for the AI services used:

  * **Anthropic API Key:** For Claude models.
  * **OpenAI API Key:** For GPT models.
  * **Google API Key:** For Gemini models.

These keys are expected to be accessible via `userdata.get()` in a Google Colab environment. You will need to configure these in your Colab secrets or modify the script to load them from your preferred source (e.g., environment variables).

**Model Names:**
The script defines specific model names to be used:

  * Anthropic Worker: `claude-3-5-sonnet-latest`
  * OpenAI Worker: `gpt-4.1`
  * * Evaluator Model: `gemini-2.5-pro-preview-05-06` (Note: At the time of writing, there was no `gemini-2.5-pro-latest` but hopefully there will be something like this in the future to make this code more easily maintainable)

These can be modified in the initial code cells if needed.

### How to Run

1.  **Set up API Keys:** Ensure your API keys are correctly configured as described above.
2.  **Install Libraries:** Run the first code cell to install the necessary Python packages.
3.  **Execute Cells:** Run the notebook cells sequentially. The `main()` function, when called in the final cell, will execute a series of predefined test queries.
4.  **Provide Human Input (If Prompted):** During the evaluation of some queries, the Gemini evaluator model might determine that human clarification is needed. If so, you will see an `input()` prompt in the output of the cell running `main()`. Provide the requested clarification or type 'skip' to continue without it.
5.  **View Results:** The `main()` function will print:
      * Detailed logs of user messages, AI responses, and tool calls for each query.
      * The raw evaluation text from the Gemini model.
      * A summary of scores for each query and overall performance.
      * Any "learnings" captured from human feedback.

### Code Structure

The Jupyter Notebook `AI_Agents_w_Evals.ipynb` is organized into several key sections:

1.  **Setup and Imports:**
      * Installs required libraries.
      * Imports necessary modules and sets up API keys and model names.
2.  **System Prompts:**
      * Defines the system prompts for the worker AI (customer service assistant) and the evaluator AI.
3.  **Evaluator Model Initialization:**
      * Creates an instance of the Google Generative AI model for evaluation, configured with its system prompt.
4.  **Global Data Stores:**
      * Initializes in-memory dictionaries for `initial_customers`, `initial_products`, and `initial_orders`.
      * Initializes `human_feedback_learnings` and `tools_schemas_list` (though `tools_schemas_list` is more concretely defined later).
5.  **Standalone Model Test Functions:**
      * Simple functions (`get_completion_anthropic_standalone`, `get_completion_openai_standalone`, `get_completion_eval_standalone`) and example calls to test basic connectivity and responses from each model.
6.  **Storage Class:**
      * Defines the `Storage` class to manage and provide access to the e-commerce data (customers, products, orders). A global instance `storage` is created.
7.  **Tool Schemas:**
      * Defines `tools_schemas_list`, which contains the JSON schema definitions for all tools available to the AI agents. These schemas describe the tool's name, description, and input parameters.
8.  **Tool Implementation Functions:**
      * Provides Python functions that implement the logic for each tool defined in `tools_schemas_list` (e.g., `create_customer`, `get_product_info`, `create_order`). These functions interact with the global `storage` instance.
      * Includes helper functions like `find_product_by_name` for fuzzy matching and `get_product_id`.
9.  **ConversationContext Class:**
      * Defines the `ConversationContext` class to store and manage the conversation history (user and assistant messages) and other contextual data (recently accessed entities, last action taken).
10. **DualAgentEvaluator Class:**
      * This is the core class orchestrating the evaluation.
      * **`__init__`**: Initializes API clients, conversation context, tool schemas (formatted for both Anthropic and OpenAI), and available tool functions.
      * **`_update_context_from_tool_results`**: Updates the conversation context based on the outcomes of tool executions.
      * **`process_tool_call`**: Dispatches calls to the appropriate tool implementation function.
      * **`get_anthropic_response` / `get_openai_response`**: Manages the multi-turn interaction with each worker AI, including handling tool calls and responses, and sending results back to the AI.
      * **`process_user_request`**: Takes a user message, gets responses from both worker AIs, and triggers the evaluation.
      * **`evaluate_responses`**: Constructs a prompt for the Gemini evaluator model (including the user query, context, both AI responses, and any relevant past learnings) and processes its evaluation. It handles potential requests for human clarification.
      * **`extract_clarification_needed`, `store_learning`, `check_relevant_learnings`, `extract_keywords`, `extract_score`**: Helper methods for the evaluation and learning process.
11. **Main Execution Logic:**
      * Defines the `main()` function which instantiates `DualAgentEvaluator` and runs a set of predefined test queries.
      * Summarizes the evaluation scores at the end.
      * The final cell calls `main()` to start the process.

### E-commerce Simulation & Tooling

The system simulates a basic e-commerce backend through the `Storage` class and a set of defined tools. AI agents can:

  * **Manage Customers:** Add new customers and retrieve existing customer details.
  * **Manage Products:** Add new products, update existing ones, retrieve product information by ID or name (with fuzzy matching for names), and list all available products.
  * **Manage Orders:** Create new orders (checking inventory and using current product prices), retrieve order details, and update order statuses (which can affect inventory).

These tools are defined with JSON schemas and implemented as Python functions. The AI agents decide when and how to use these tools based on the user's query and the conversation context.

### Evaluation Mechanism

The `DualAgentEvaluator` class uses a Google Gemini model to assess the responses from Anthropic Claude and OpenAI GPT. The evaluator is provided with:

  * The original user query.
  * The conversation context shared with the worker AIs.
  * The final text response from Anthropic Claude.
  * The final text response from OpenAI GPT.
  * Relevant past learnings from human feedback.

The evaluator model then generates a textual evaluation and attempts to provide scores (1-10) for each worker AI based on accuracy, efficiency, context awareness, and helpfulness, along with an overall score. If the evaluator identifies ambiguity that it believes requires human input, it will flag this. The system then prompts the human user for clarification, which can be used to re-evaluate and store as a "learning" for future similar situations.

### Future Improvements

Potential areas for enhancement could include:

  * **Better Evaluator System Prompt:** Sometimes the Evaluator asks for clarifications on things that really shouldn't require clarification, thus making it seem like its a dumbass instead of a smart evaluator. This should be fixed iteratively.
  * **More Sophisticated State Management:** Persisting data beyond in-memory storage (e.g., to a database or files).
  * **Expanded Toolset:** Adding more complex e-commerce tools (e.g., payment processing, shipping integration, advanced search/filtering).
  * **Refined Evaluation Criteria:** Developing more granular or objective metrics for evaluation.
  * **Automated Test Case Generation:** Creating a wider and more diverse set of test queries.
  * **Error Handling and Resilience:** More robust error handling in API calls and tool executions.
  * **Asynchronous Operations:** For improved performance, especially when dealing with multiple API calls.
  * **User Interface:** A simple UI for interacting with the system instead of just running notebook cells.
  * **Cost Tracking:** Implementing mechanisms to monitor API usage costs.
  * **Benchmarking:** Systematically comparing different model versions or configurations.
  * **Advanced Context Management:** More sophisticated techniques for injecting and retrieving relevant context for the LLMs.
  * **Structured Evaluation Output:** Ensuring the evaluator model always returns scores in a parseable format (e.g., JSON) rather than relying on regex extraction from text.
