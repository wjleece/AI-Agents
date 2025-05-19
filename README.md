# E-Commerce Customer Service AI Agent with Evaluation Framework

## Overview

This project implements an AI-powered customer service system for e-commerce applications featuring:

1. **Customer Service Agent**: A Claude-powered AI assistant that can handle product inquiries, order management, and customer operations
2. **Automated Evaluation**: A Gemini-powered evaluation system that assesses agent responses for accuracy, efficiency, and helpfulness
3. **Learning System**: A feedback mechanism that captures human feedback to improve agent performance over time

The system is designed to demonstrate how large language models can be used for practical e-commerce applications while maintaining quality through rigorous evaluation.

## Core Components

### AI Models

- **Worker Agent**: Anthropic's Claude 3.5 Sonnet serves as the primary customer service AI
- **Evaluator**: Google's Gemini 2.5 Pro assesses Claude's responses and provides performance metrics
- **Gemini Actor**: A parallel Gemini-based agent (optional feature)

### Tool Suite

The agent has access to a comprehensive set of e-commerce tools:

- **Customer Management**: Create customers, retrieve customer information
- **Product Management**: Create, update, retrieve products and inventory information
- **Order Processing**: Create orders, update order status, track shipments
- **Inventory Management**: Automatic inventory adjustment based on order status changes

### Evaluation Framework

Each agent interaction is evaluated based on:

- **Accuracy**: Factual correctness compared against the ground truth data
- **Efficiency**: Reaching the correct solution with minimal steps
- **Context Awareness**: Proper use of conversation history and context
- **Helpfulness**: Overall usefulness to the customer

### Learning System

The project includes a RAG (Retrieval-Augmented Generation) component that:

- Stores human feedback as "learnings"
- Persists learnings to Google Drive as JSON files
- Retrieves relevant past learnings during new conversations
- Synthesizes and resolves conflicts between learnings
- Provides a feedback loop for continuous improvement

## Setup Requirements

- Google Colab environment
- API keys for:
  - Anthropic Claude
  - Google Gemini AI
- Google Drive access for storing learnings
- Python libraries:
  - anthropic
  - google-generativeai
  - fuzzywuzzy (for product name matching)

## Usage Guide

1. **Initialization**:
   - Run the notebook in Google Colab
   - Configure API keys using `userdata.get()`
   - Mount Google Drive for learning storage

2. **Customer Interactions**:
   - Use natural language queries to interact with the agent
   - The agent can handle product inquiries, order creation, status updates, and more
   - Sample queries are provided in the notebook comments

3. **Evaluation Process**:
   - Each interaction is automatically evaluated by Gemini
   - Evaluations include scoring and detailed reasoning
   - Data store consistency checks ensure correct handling of inventory

4. **Adding Learnings**:
   - After each interaction, you can add "learnings" to improve future responses
   - Learnings are processed, checked for conflicts, and saved to Drive
   - The system automatically retrieves relevant learnings for future interactions

## Architecture

The system follows a modular architecture:

1. **Storage Class**: Manages e-commerce data (customers, products, orders)
2. **ConversationContext**: Tracks conversation history and recent entities
3. **AgentEvaluator**: Core class that orchestrates agent interactions and evaluations
4. **Tool Functions**: Implement the actual business logic for e-commerce operations

## Sample Queries

```
- Show me all the products available
- I'd like to order 25 Perplexinators, please
- Show me the status of my order
- Please ship my order now
- How many Perplexinators are now left in stock?
- Add a new customer: Bill Leece, bill.leece@mail.com, +1.222.333.4444
- Add new product: Gizmo X, description: A fancy gizmo, price: 29.99, inventory: 50
- Update Gizzmo's price to 99.99 (test fuzzy matching with misspelling)
- I need to know the total value of all products in our inventory
```

## Technical Notes

- The system uses a sequence of tool calls to perform complex operations
- Fuzzy matching allows product identification even with misspellings
- Context awareness enables natural follow-up questions about orders or products
- The evaluation system can capture both the "before" and "after" states for accurate assessment

## Future Enhancements

- Add authentication and user-specific order history
- Implement more sophisticated learning conflict resolution
- Add support for more complex product queries (e.g., filtering, sorting)
- Integrate with actual e-commerce platforms via APIs

## Troubleshooting

- If experiencing Google Drive mounting issues, ensure proper permissions are granted
- For API key errors, verify keys are correctly stored in Colab's secrets manager
- If evaluations seem incorrect, check that the ground truth state is being properly captured

---

*This project demonstrates how large language models can be effectively used and evaluated in e-commerce contexts, providing a framework for building sophisticated AI-powered customer service systems.*
