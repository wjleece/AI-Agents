# AI-Agents

# Dual Agent Evaluator System Documentation

## Overview

The Dual Agent Evaluator is a system that compares the effectiveness of two leading AI models—Anthropic's Claude 3.7 Sonnet and OpenAI's GPT-4—in customer service scenarios for an e-commerce platform. 
The system uses Google's Gemini 2.5 as an independent evaluator to assess the quality of responses from both models.

This document explains how the system works, how to configure and use it, and how to extend it for additional functionality.

## System Architecture

The system consists of the following core components:

1. **DualAgentEvaluator**: The main class that coordinates the evaluation process
2. **ConversationContext**: Maintains context for the conversation
3. **Business Logic Functions**: Implementations of e-commerce functions like product management and order processing
4. **Tool Handling**: Mechanisms for both Claude and GPT-4 to interact with the business functions
5. **Evaluation Logic**: Uses Gemini to evaluate the quality of responses
6. **Human Feedback Integration**: Collects and applies human feedback for ambiguous scenarios
7. **Test Harness**: Runs evaluation scenarios and collects metrics

## Configuration

Before running the system, you need to set up API keys for all three AI providers:

```python
# Configuration
ANTHROPIC_API_KEY = "your-anthropic-api-key"
OPENAI_API_KEY = "your-openai-api-key"
GOOGLE_API_KEY = "your-google-api-key"

# Model names (can be changed to use different models)
ANTHROPIC_MODEL = "claude-3-7-sonnet-20240229"
OPENAI_MODEL = "gpt-4-1106-preview"
EVAL_MODEL = "gemini-2.5-pro-preview-05-06"
```

## How It Works

### 1. Processing a User Request

When a user submits a request:

1. The request is added to the conversation context
2. The full context, including conversation history, is provided to both Claude and GPT-4
3. Each model processes the request, potentially using tools to interact with the e-commerce system
4. The responses from both models are collected
5. Gemini evaluates both responses based on accuracy, efficiency, context awareness, and helpfulness
6. The evaluation results are stored, and both responses are added to the conversation history

### 2. Tool Handling

Both models can use a set of tools to interact with the e-commerce system:

- Customer management: create_customer, get_customer_info
- Product management: create_product, update_product, get_product_info, list_all_products
- Order management: create_order, get_order_details, update_order_status

The system handles the different tool calling formats used by Anthropic and OpenAI.

### 3. Context Management

The system maintains context about:

- Previous conversation messages
- Recently viewed customers, products, and orders
- Recent actions taken

This context is provided to both models to help them understand references in user requests.

### 4. Evaluation Process

Gemini evaluates responses based on:

1. **Accuracy**: How correct and factual is the response?
2. **Efficiency**: Did the model get to the correct answer with minimal clarifying questions?
3. **Context Awareness**: Did the model correctly use conversation context?
4. **Helpfulness**: How well did the model address the user's needs?

Gemini assigns a score from 1-10 for each criterion and provides an overall evaluation.

### 5. Human Feedback Integration

When Gemini identifies ambiguity that neither model could reasonably resolve:

1. It flags this as requiring human clarification
2. Clearly states what information is needed
3. Asks an admin user for clarification
4. Stores this feedback as a "learning" for future similar situations

## Running the System

The test harness runs predefined scenarios through the evaluation process:

```python
# Initialize the dual agent evaluator
agent = DualAgentEvaluator()

# Run a test scenario
scenario_result = run_test_scenario(agent, "Basic Product Flow", [
    "Show me all available products",
    "I'd like to order 3 of the widget please",
    "What's the status of my order?"
])

# Display and save results
display_overall_results([scenario_result])
save_results([scenario_result])
```

## Extending the System

### Adding New Tools

To add a new tool:

1. Add the tool definition to the `tools` list in `initialize_business_logic()`
2. Implement the corresponding function in the `DualAgentEvaluator` class
3. Add handling for the tool in the `process_tool_call()` method
4. Add context updating for the tool in `update_context_from_tool_usage()`

Example:

```python
# 1. Add tool definition
{
    "name": "search_products",
    "description": "Searches products based on keywords",
    "input_schema": {
        "type": "object",
        "properties": {
            "keywords": {
                "type": "string",
                "description": "Keywords to search for in product names and descriptions."
            }
        },
        "required": ["keywords"]
    }
}

# 2. Implement the function
def search_products(self, keywords):
    results = {}
    for pid, product in self.products.items():
        if (keywords.lower() in product["name"].lower() or 
            keywords.lower() in product["description"].lower()):
            results[pid] = product
    return {
        "status": "success",
        "count": len(results),
        "products": results
    }

# 3. Add to process_tool_call
elif tool_name == "search_products":
    return self.search_products(**tool_input)

# 4. Add to update_context_from_tool_usage
elif tool_name == "search_products" and isinstance(tool_result, dict) and "status" in tool_result and tool_result["status"] == "success":
    # Maybe update context with searched products
    for pid, product in tool_result.get("products", {}).items():
        self.conversation_context.update_context("products", pid, product)
```

### Adding New Evaluation Criteria

To add a new evaluation criterion:

1. Update the `evaluator_system_prompt` to include the new criterion
2. Modify the `extract_score()` method to extract scores for the new criterion
3. Update the overall evaluation calculation in `evaluate_responses()`

### Creating New Test Scenarios

Define new scenarios in the `main()` function of the test harness:

```python
scenarios = [
    {
        "name": "Complex Order Scenario",
        "queries": [
            "I'd like to order 10 of each product you have",
            "Actually, let's make that 5 of each instead",
            "Can you ship half of my order today and the rest next week?",
            "What's the total cost of my entire order?",
            "Cancel the part of my order with the Gadget B"
        ]
    }
]
```

## Best Practices

1. **Realistic Scenarios**: Create test scenarios that reflect real customer interactions
2. **Ambiguity Testing**: Include intentionally ambiguous queries to test how models handle uncertainty
3. **Context Challenges**: Test the models' ability to maintain context across multiple interactions
4. **Edge Cases**: Include edge cases like unavailable products or invalid orders
5. **Human Feedback Collection**: Regularly review and refine human feedback learnings

## Troubleshooting

### Common Issues

1. **API Rate Limiting**: If hitting rate limits, add exponential backoff and retry logic
2. **Context Windows**: If hitting context window limits, implement more aggressive context pruning
3. **Tool Errors**: Check for mismatches between tool schemas and implementations
4. **Evaluation Consistency**: If Gemini evaluations seem inconsistent, try more structured prompting

## Future Enhancements

Potential enhancements to consider:

1. **Multi-Modal Support**: Add support for image-based products and queries
2. **Memory Optimization**: Implement more sophisticated context pruning for long conversations
3. **Custom Evaluation Metrics**: Add domain-specific evaluation metrics
4. **Active Learning**: Develop more sophisticated techniques to apply human feedback
5. **Expanded Tools**: Add more complex business functions like recommendations and promotions

## Conclusion

This system provides a comprehensive framework for comparing multiple AI assistants in a realistic e-commerce customer service setting. By leveraging Gemini as an independent evaluator and incorporating human feedback, the system can provide actionable insights about the relative strengths and weaknesses of different models.
    
    
    
  
