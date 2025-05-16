This project provides a framework for evaluating the performance of two different AI language models (Anthropic's Claude and OpenAI's GPT) acting as customer service assistants for a simulated e-commerce system. A third AI model (Google's Gemini) is used as an impartial evaluator to score the responses of the two worker agents.

## Description

The primary goal of this framework is to enable comparative analysis of different AI models in a tool-using, conversational context. It simulates an e-commerce environment where AI agents can manage customers, products, and orders using a defined set of tools. Each worker AI (Anthropic and OpenAI) operates on its own independent data store to ensure fair evaluation of actions. User queries are processed by both agents, and their responses, along with the conversation context and the final states of their respective data stores, are then passed to an evaluator AI (Gemini) which assesses them based on predefined criteria. The system also incorporates a mechanism for human feedback to refine the evaluation process and capture learnings for all AI models involved.
## Features

* **Dual Agent Interaction**: Simultaneously processes user requests with two distinct AI models (Anthropic Claude and OpenAI GPT).
* **Tool Integration**: Supports a range of tools for e-commerce operations, including:
    * Customer management (create, retrieve information).
    * Product management (create, update, retrieve information, list all).
    * Order management (create, retrieve details, update status).
* **Fuzzy Matching**: Implements fuzzy string matching for product searches to handle potential misspellings.
* **Contextual Conversations**: Maintains conversation history and relevant context (viewed customers, products, orders, last action) to enable more coherent interactions.
* **AI-Powered Evaluation**: Utilizes a separate AI model (Google Gemini) to evaluate the worker agents' responses based on:
    * Accuracy
    * Efficiency
    * Context Awareness
    * Helpfulness
    * **Data Store Comparison**: As a final step, the evaluator compares the end-state of each agent's data store for consistency and considers this in its final assessment.
* **Human-in-the-Loop**:
    * Allows for human clarification during the evaluation phase if the evaluator AI deems it necessary.
    * **Explicit Learning Solicitation**: After each evaluation, the system explicitly prompts the human user to contribute "learnings."
    * **Shared Learnings**: Captured clarifications and user-provided learnings are stored and made available to *all three AI models* (Anthropic, OpenAI, and Gemini) for subsequent interactions to improve future performance and evaluations.
* **Data Simulation**:
    * Operates on **separate in-memory data stores** (implemented as dictionaries) for customers, products, and orders for each worker agent, ensuring isolated environments for their actions. These are initialized from a common starting state.
* **Modular Design**: Code is structured into classes for storage, conversation context, and the dual agent evaluator.

## Requirements

The script is designed to be run in a Jupyter Notebook environment (e.g., Google Colab) and requires the following Python libraries:
* `anthropic`
* `openai`
* `google-generativeai`
* `fuzzywuzzy`

These can be installed using `pip`:
```python
%pip install anthropic openai google-generativeai fuzzywuzzy
