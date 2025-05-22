# E-commerce AI Agent with Self-Improving Evaluation Loop

## 1. Overview

This project implements an advanced AI-driven customer service agent for an e-commerce platform. The system is designed not only to handle user queries and perform operational tasks but also to learn and improve over time through a sophisticated human feedback mechanism that refines both the agent and its AI-driven evaluator.

The core idea is to use a primary AI agent (Worker Agent) to interact with users and e-commerce data, an AI Evaluator to assess the Worker Agent's performance, and a robust RAG (Retrieval Augmented Generation) system fueled by human feedback to guide both AIs towards better performance and understanding.

## 2. Key Features

* **Dual AI Woker Agent & Evaluator System + Query Router**:
    * **Worker Agent**: Powered by Anthropic's Claude 3.5 Sonnet (or user-defined model), responsible for understanding user queries, interacting with the e-commerce datastore, and generating user-friendly responses.
    * **Evaluator AI**: Powered by Google's Gemini 2.5 Pro (or user-defined model), responsible for critically assessing the Worker Agent's performance on each turn based on multiple criteria.
    * **Query Classifier**: Uses Google's Gemini 1.5 Flash for fast and efficient classification of user queries into "OPERATIONAL" or "METACOGNITIVE_LEARNINGS_SUMMARY" types, directing the Worker Agent to use appropriate prompts and tools.
* **Tool-Using Agent**: The Worker Agent can use a predefined set of tools (e.g., creating orders, fetching product details, updating customer information, etc.) to interact with the e-commerce system.
* **In-Memory Datastore**: Simulates an e-commerce backend with customers, products, and orders, managed by a `Storage` class.
* **Knowledge Management & RAG System**:
    * Human feedback is captured and processed into "learnings."
    * Learnings are synthesized into concise, actionable statements by an LLM (Gemini 2.5 Pro).
    * Includes an interactive conflict resolution step where if new feedback contradicts existing learnings, the user is prompted to clarify or modify.
    * Learnings are stored as versioned JSON files on Google Drive.
    * The RAG system provides relevant learnings as context to both the Worker Agent and the AI Evaluator.
* **Targeted Learnings**:
    * Feedback on the Agent's response is tagged for both "AgentAndEvaluator."
    * Feedback on the Evaluator's assessment is tagged for "EvaluatorOnly."
    * The RAG system provides learnings to the Agent and Evaluator based on these targets and query context.
* **Comprehensive Evaluation**: The AI Evaluator assesses the Worker Agent on:
    * Accuracy (including tool use verification against datastore changes).
    * Efficiency.
    * Context Awareness.
    * Helpfulness & Clarity (of the user-facing response).
    * Scores are provided on a 1-10 scale with detailed reasoning.
* **Interactive Human Feedback Loop**:
    * After the Worker Agent responds, the user can provide feedback on that response.
    * After the AI Evaluator provides its assessment, the user can provide feedback on the evaluation itself.
* **Conversational Context Management**: Maintains conversation history and relevant entity context for the Worker Agent.

## 3. System Architecture & Components

The system is modular, with distinct classes handling specific responsibilities:

* **`AgentOrchestrator`**: The central coordinator. It manages the overall flow of a user request, from query classification to agent response, evaluation, and feedback processing.
* **`Storage`**: Manages the in-memory e-commerce datastore (customers, products, orders).
* **`QueryClassifier`**: Uses an LLM to classify user queries, determining the type of task for the Worker Agent.
* **`ConversationManager`**: Maintains the history of user and assistant messages, and a summary of recently interacted-with entities.
* **`WorkerAgentHandler`**: Manages all interactions with the Worker Agent LLM (Claude). It passes prompts, handles tool calling logic (requesting tool execution and sending results back to the LLM), and returns the agent's final user-facing textual response along with a log of executed tools.
* **`ToolExecutor`**: Executes the actual Python functions corresponding to the tools requested by the Worker Agent (e.g., `create_order`, `get_product_info`).
    * **Tool Functions**: Individual Python functions (e.g., `create_customer`, `update_product_status`) that implement the e-commerce operations.
* **`ResponseEvaluator`**: Manages interactions with the AI Evaluator LLM (Gemini). It constructs the detailed prompt for evaluation (including before/after datastore states, agent's response, tool logs, RAG context) and extracts scores and reasoning.
* **`KnowledgeManager`**:
    * Handles the RAG system.
    * Loads learnings from Google Drive.
    * Persists new/updated learnings to Google Drive.
    * Uses an LLM to synthesize human feedback into structured "finalized learning statements."
    * Manages the interactive conflict resolution process if new feedback clashes with existing learnings.
    * Provides relevant learnings to the Agent and Evaluator based on query type and recipient role.

## 4. Setup and Prerequisites

1.  **Environment**: Python 3 (developed and tested in Google Colab).
2.  **Python Libraries**: Install the required libraries by running the first code cell in the notebook:
    ```python
    %pip install anthropic
    %pip install -q -U google-generativeai
    %pip install fuzzywuzzy
    # google-colab is usually pre-installed in Colab
    ```
3.  **API Keys**:
    * **Anthropic API Key**: For accessing Claude models.
    * **Google API Key**: For accessing Gemini models.
    * These keys must be stored as "Secrets" in Google Colab for the notebook to access them via `userdata.get('API_KEY_NAME')`.
        * Name them `ANTHROPIC_API_KEY` and `GOOGLE_API_KEY` in Colab Secrets.
4.  **Google Drive Setup**:
    * The system uses Google Drive to store and retrieve RAG learnings (JSON files).
    * The notebook will attempt to mount your Google Drive. You'll need to authorize this.
    * A specific directory path is used. By default: `My Drive/AI/Knowledgebases`. Ensure this path is accessible or can be created by the notebook. You can change `DEFAULT_LEARNINGS_DRIVE_SUBPATH` if needed.

## 5. How it Works (Workflow per User Turn)

1.  **User Input**: The user enters a query in the `main()` loop.
2.  **Orchestration Begins**: `AgentOrchestrator.process_user_request()` is called.
3.  **Query Classification**: `QueryClassifier` determines if the query is "OPERATIONAL" or "METACOGNITIVE_LEARNINGS_SUMMARY".
4.  **Agent RAG Retrieval**: `KnowledgeManager` retrieves relevant learnings for the Worker Agent (filtering for `learning_target="AgentAndEvaluator"`).
5.  **Worker Agent Processing**:
    * `WorkerAgentHandler` interacts with the Claude LLM using the appropriate system prompt and RAG context.
    * If OPERATIONAL, the Agent may decide to use tools. Tool schemas are provided to the LLM.
    * Tool execution requests are handled by `ToolExecutor`.
    * Tool results are sent back to the Worker LLM.
    * The Worker LLM formulates a **user-friendly textual response**.
    * `WorkerAgentHandler` also returns a structured **log of executed tools** (inputs and outputs).
6.  **Display Agent Response**: `main()` prints the user-friendly response from the Worker Agent.
7.  **Feedback on Agent**: `main()` prompts the user for feedback on the Worker Agent's response.
    * If feedback is given, `AgentOrchestrator.handle_feedback_on_worker_response()` calls `KnowledgeManager` to process it.
    * The learning is tagged with `learning_target="AgentAndEvaluator"`.
    * `KnowledgeManager` synthesizes, checks for conflicts (with user interaction if needed), and persists the learning.
8.  **Evaluator RAG Retrieval**: `KnowledgeManager` retrieves relevant learnings for the AI Evaluator (filtering for `learning_target="AgentAndEvaluator"` OR `learning_target="EvaluatorOnly"`).
9.  **AI Evaluation**:
    * `ResponseEvaluator` interacts with the Gemini LLM.
    * It provides a rich context: user query, Agent's user-friendly response, the detailed log of tools executed by the Agent, datastore state before and after the Agent's actions, RAG context for the evaluator, and any clarification interactions.
    * The Gemini LLM assesses the Worker Agent's performance based on the `evaluator_system_prompt` and provides scores and detailed reasoning.
10. **Display Evaluation**: `main()` prints the AI Evaluator's assessment.
11. **Feedback on Evaluator**: `main()` prompts the user for feedback on the AI Evaluator's assessment.
    * If feedback is given, `AgentOrchestrator.handle_feedback_on_evaluation()` calls `KnowledgeManager`.
    * The learning is tagged with `learning_target="EvaluatorOnly"`.
    * `KnowledgeManager` synthesizes, handles conflicts, and persists this learning.
12. **Loop**: The system is ready for the next user query.

## 6. Running the System

1.  **Open the Notebook**: Open the `.ipynb` file in Google Colab or a compatible Jupyter environment.
2.  **Install Dependencies**: Run the first code cell to install necessary libraries.
3.  **Set API Keys**: Ensure your Anthropic and Google API keys are set as secrets in Colab.
4.  **Mount Drive**: When prompted, authorize Google Drive access.
5.  **Run All Cells**: Execute all notebook cells in order to define classes and functions.
6.  **Start Interaction**: Call the `main()` function in a code cell (the last cell usually has `main()`).
7.  **Interact**:
    * Enter your e-commerce related queries at the prompt: `Enter query (or 'quit' to exit):`
    * When prompted for feedback on the Worker AI's response, type your feedback or "skip".
    * Review the AI Evaluator's assessment.
    * When prompted for feedback on the AI Evaluator's assessment, type your feedback or "skip".
    * Type `quit` to end the session. Learnings updated during the session will be persisted to Google Drive.

## 7. Key Configuration Variables

These are defined near the beginning of the notebook and can be adjusted:

* `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY`: Fetched from Colab `userdata`.
* `ANTHROPIC_MODEL_NAME`: e.g., `"claude-3-5-sonnet-latest"`
* `EVAL_MODEL_NAME`: e.g., `"gemini-2.5-pro-preview-05-06"` (used for evaluation and learning synthesis)
* `CLASSIFIER_MODEL_NAME`: e.g., `"gemini-1.5-flash-latest"` (used for query classification)
* `DRIVE_MOUNT_PATH`: e.g., `'/content/drive'`
* `DEFAULT_LEARNINGS_DRIVE_SUBPATH`: e.g., `"My Drive/AI/Knowledgebases"` (relative to the Drive root)

## 8. Structure of RAG Learnings (JSON)

Learnings are stored in JSON files on Google Drive. Each learning is a dictionary with a structure similar to this:

```json
{
    "learning_id": "unique_uuid_string",
    "timestamp_created": "iso_format_datetime_string",
    "original_human_input": "The raw text of the human's feedback.",
    "processed_human_input": "The feedback text after potential modification during conflict resolution.",
    "final_learning_statement": "The LLM-synthesized, concise, actionable learning.",
    "status": "active" OR "active_forced_redundancy",
    "learning_target": "AgentAndEvaluator" OR "EvaluatorOnly",
    "notes": "Optional field for extra notes, e.g., if forced storage."
}
