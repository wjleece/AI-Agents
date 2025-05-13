# AI-Agents

Basic AI Agent
* Uses gpt-4.1-latest and claude-3-7-sonnet-latest to perform tasks
* Uses gemini-2.5-pro-preview-05-06 (see if that can be replaced with -latest) as an evaluator
* Functions include:
* tools = [
    {
        "name": "create_customer",
        "description": "Adds a new customer to the database. Includes customer ID, name, email, and (optional) phone number.",
        "input_schema": {
            "type": "object",
            "properties": {
                "name": {
                    "type": "string",
                    "description": "The name of the customer."
                },
                "email": {
                    "type": "string",
                    "description": "The email address of the customer."
                },
                "phone": {
                    "type": "string",
                    "description": "The phone number of the customer."
                }
            },
            "required": ["name", "email"]
        }
    },
    {
        "name": "get_customer_info",
        "description": "Retrieves customer information based on their customer ID. Returns the customer's name, email, and (optional) phone number.",
        "input_schema": {
            "type": "object",
            "properties": {
                "customer_id": {
                    "type": "string",
                    "description": "The unique identifier for the customer."
                }
            },
            "required": ["customer_id"]
        }
    },
    {
        "name": "create_product",
        "description": "Adds a new product to the product database. Includes ID, name, description, price, and initial inventory count.",
        "input_schema": {
            "type": "object",
            "properties": {
                "name": {
                    "type": "string",
                    "description": "The name of the product."
                },
                "description": {
                    "type": "string",
                    "description": "A description of the product."
                },
                "price": {
                    "type": "number",
                    "description": "The price of the product."
                },
                "inventory_count": {
                    "type": "integer",
                    "description": "The amount of the product that is currently in inventory."
                }
            },
            "required": ["name", "description", "price", "inventory_count"]
        }
    },
    {
        "name": "update_product",
        "description": "Updates an existing product with new information. Only fields that are provided will be updated; other fields remain unchanged.",
        "input_schema": {
            "type": "object",
            "properties": {
                "product_id": {
                    "type": "string",
                    "description": "The unique identifier for the product to update."
                },
                "name": {
                    "type": "string",
                    "description": "The new name for the product (optional)."
                },
                "description": {
                    "type": "string",
                    "description": "The new description for the product (optional)."
                },
                "price": {
                    "type": "number",
                    "description": "The new price for the product (optional)."
                },
                "inventory_count": {
                    "type": "integer",
                    "description": "The new inventory count for the product (optional)."
                }
            },
            "required": ["product_id"]
        }
    },
    {
        "name": "get_product_info",
        "description": "Retrieves product information based on product ID or product name (with fuzzy matching for misspellings). Returns product details including name, description, price, and inventory count.",
        "input_schema": {
            "type": "object",
            "properties": {
                "product_id_or_name": {
                    "type": "string",
                    "description": "The product ID or name (can be approximate)."
                }
            },
            "required": ["product_id_or_name"]
        }
    },
    {
        "name": "list_all_products",
        "description": "Lists all available products in the inventory.",
        "input_schema": {
            "type": "object",
            "properties": {},
            "required": []
        }
    },
    {
        "name": "create_order",
        "description": """Creates an order using the product's current price.
                      If requested quantity exceeds available inventory, no order is created and available quantity is returned.
                      Orders can only be created for products that are in stock.
                      Supports specifying products by either ID or name with fuzzy matching for misspellings.""",
        "input_schema": {
            "type": "object",
            "properties": {
                "product_id_or_name": {
                    "type": "string",
                    "description": "The ID or name of the product to order (supports fuzzy matching)."
                },
                "quantity": {
                    "type": "integer",
                    "description": "The quantity of the product in the order."
                },
                "status": {
                    "type": "string",
                    "description": "The initial status of the order (e.g., 'Processing', 'Shipped')."
                }
            },
            "required": ["product_id_or_name", "quantity", "status"]
        }
    },
    {
        "name": "get_order_details",
        "description": "Retrieves the details of a specific order based on the order ID. Returns the order ID, product name, quantity, price, and order status.",
        "input_schema": {
            "type": "object",
            "properties": {
                "order_id": {
                    "type": "string",
                    "description": "The unique identifier for the order."
                }
            },
            "required": ["order_id"]
        }
    },
    {
        "name": "update_order_status",
        "description": """Updates the status of an order and adjusts inventory accordingly.
                      Changing to "Shipped" decreases inventory.
                      Changing to "Returned" or "Canceled" from "Shipped" increases inventory.
                      Status can be "Processing", "Shipped", "Delivered", "Returned", or "Canceled".""",
        "input_schema": {
            "type": "object",
            "properties": {
                "order_id": {
                    "type": "string",
                    "description": "The unique identifier for the order."
                },
                "new_status": {
                    "type": "string",
                    "description": "The new status to set for the order.",
                    "enum": ["Processing", "Shipped", "Delivered", "Returned", "Canceled"]
                }
            },
            "required": ["order_id", "new_status"]
        }
    },
    {
        "name": "cancel_order",
        "description": """Cancels an order based on the provided order ID. Sets the order status to "Canceled" and returns a confirmation message if the cancellation is successful. If the order was already shipped, this will also return the items to inventory.""",
        "input_schema": {
            "type": "object",
            "properties": {
                "order_id": {
                    "type": "string",
                    "description": "The unique identifier for the order to be cancelled."
                }
            },
            "required": ["order_id"]
        }
    }
]
