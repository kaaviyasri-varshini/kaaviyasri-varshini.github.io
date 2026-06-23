Title: LangChain Structured Outputs: Building Reliable Validation Pipelines
Date: 2026-06-23
Category: AI Engineering
Tags: langchain, structured-outputs, validation, python, ai-development
Slug: langchain-structured-outputs-validation
Summary: Learn why structured outputs matter and how to build reliable validation pipelines in LangChain with typed outputs, JSON schemas, and error handling strategies.

## Introduction

When you ask an AI to generate text, you typically get back... text. Freeform, unpredictable, and hard to work with programmatically. But what if you need:
- A list of items to display in a UI
- Structured data to insert into a database
- Specific fields like name, email, and phone number
- JSON to pass to another API

**Structured outputs** solve this problem by forcing the model to return data in a predictable format you can parse, validate, and use reliably in your application.

This post shows you how to build robust validation pipelines in LangChain.

## Why Structure Matters

### The Problem with Unstructured Outputs

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# Ask for contact info
response = llm.invoke("Extract the contact details from: John Doe, john@example.com, 555-1234")

print(response.content)
# Output: "The contact details are: Name: John Doe, Email: john@example.com, Phone: 555-1234"
```

This looks fine to humans, but try parsing it:

```python
# How do you extract the email reliably?
# What if the format changes?
# What if the model adds extra text?
```

### The Solution: Structured Outputs

```python
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI

class Contact(BaseModel):
    """Contact information"""
    name: str = Field(description="Full name")
    email: str = Field(description="Email address")
    phone: str = Field(description="Phone number")

llm = ChatOpenAI(model="gpt-4")
structured_llm = llm.with_structured_output(Contact)

response = structured_llm.invoke("Extract: John Doe, john@example.com, 555-1234")

print(type(response))  # <class '__main__.Contact'>
print(response.name)   # "John Doe"
print(response.email)  # "john@example.com"
print(response.phone)  # "555-1234"
```

**Benefits:**
- Type-safe data access
- Automatic validation
- No parsing errors
- Database-ready objects
- Clear contracts between AI and code

## Typed Outputs with Pydantic

Pydantic provides Python type annotations with runtime validation. LangChain uses it to define output schemas.

### Basic Typed Output

```python
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI

class BlogPost(BaseModel):
    """A blog post structure"""
    title: str = Field(description="Catchy blog title")
    summary: str = Field(description="One sentence summary")
    tags: list[str] = Field(description="List of relevant tags")
    word_count: int = Field(description="Target word count")

llm = ChatOpenAI(model="gpt-4", temperature=0.7)
structured_llm = llm.with_structured_output(BlogPost)

result = structured_llm.invoke(
    "Create a blog post outline about Python asyncio"
)

print(f"Title: {result.title}")
print(f"Tags: {', '.join(result.tags)}")
```

### Complex Nested Structures

You can create nested models for complex data:

```python
from typing import List, Optional

class Address(BaseModel):
    street: str
    city: str
    state: str

class Person(BaseModel):
    name: str = Field(description="Full name")
    age: Optional[int] = Field(description="Age if mentioned")
    addresses: List[Address] = Field(description="List of addresses")
```

### Field Validation

Pydantic allows custom validators for data quality:

```python
from langchain_core.pydantic_v1 import validator
import re

class User(BaseModel):
    username: str
    email: str
    age: int
    
    @validator("email")
    def email_valid(cls, v):
        if not re.match(r'^[\w\.-]+@[\w\.-]+\.\w+$', v):
            raise ValueError("Invalid email format")
        return v
    
    @validator("age")
    def age_reasonable(cls, v):
        if v < 0 or v > 150:
            raise ValueError("Age must be between 0 and 150")
        return v
```

This ensures your data meets business requirements before it enters your system.

## JSON & Schema-Based Responses

Sometimes you want raw JSON instead of Python objects, especially when integrating with APIs or databases.

### Basic JSON Schema

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# Define JSON schema
json_schema = {
    "title": "Product",
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "price": {"type": "number"},
        "in_stock": {"type": "boolean"},
        "categories": {
            "type": "array",
            "items": {"type": "string"}
        }
    },
    "required": ["name", "price", "in_stock"]
}

structured_llm = llm.with_structured_output(json_schema)
result = structured_llm.invoke("Create a product: Wireless Mouse, $29.99, in stock")
```

### Converting Between Pydantic and JSON Schema

You can easily convert between Pydantic models and JSON schemas:

```python
from langchain_core.pydantic_v1 import BaseModel

class Product(BaseModel):
    name: str
    price: float
    in_stock: bool

# Get JSON schema from Pydantic model
json_schema = Product.schema()

# Use either approach
structured_llm_pydantic = llm.with_structured_output(Product)
structured_llm_json = llm.with_structured_output(json_schema)
```

Both approaches work identically—choose based on your needs.

## Handling Invalid Model Outputs

Models don't always return valid structured data. You need error handling strategies.

### Try-Except Pattern

```python
from langchain_core.pydantic_v1 import ValidationError
from typing import Optional

def extract_person_safe(text: str) -> Optional[Person]:
    """Safely extract person data"""
    try:
        result = structured_llm.invoke(f"Extract person from: {text}")
        return result
    except ValidationError as e:
        print(f"Validation error: {e}")
        return None
    except Exception as e:
        print(f"Unexpected error: {e}")
        return None
```

### Fallback Strategies

When structured extraction fails, have a backup plan:

```python
def extract_with_fallback(text: str):
    """Try structured extraction, fall back to unstructured"""
    try:
        return structured_llm.invoke(f"Extract: {text}")
    except:
        # Fall back to basic LLM call
        response = llm.invoke(f"Extract name, age, and email from: {text}")
        return {"raw_response": response.content, "parsed": False}
```

### Default Values

Use Pydantic defaults for optional or missing fields:

```python
class PersonWithDefaults(BaseModel):
    name: str
    age: int = 0  # Default if not provided
    email: str = "unknown@example.com"
    phone: Optional[str] = None  # Truly optional
```

This makes your system resilient to incomplete data.

## Beginner-Friendly Validation Loops

Build a validation pipeline that automatically retries on failures.

### Simple Retry Loop

```python
class ValidationPipeline:
    def __init__(self, llm, output_model, max_retries=3):
        self.structured_llm = llm.with_structured_output(output_model)
        self.max_retries = max_retries
    
    def extract_with_retry(self, prompt: str):
        """Extract with automatic retries"""
        for attempt in range(self.max_retries):
            try:
                result = self.structured_llm.invoke(prompt)
                print(f"✓ Success on attempt {attempt + 1}")
                return result
            except ValidationError as e:
                print(f"✗ Attempt {attempt + 1} failed")
                if attempt < self.max_retries - 1:
                    prompt += "\n\nIMPORTANT: Ensure all required fields are provided."
        return None
```

### Self-Correcting Pipeline

Ask the model to fix its own validation errors:

```python
class SelfCorrectingPipeline:
    def __init__(self, llm, output_model):
        self.structured_llm = llm.with_structured_output(output_model)
        self.llm = llm
    
    def extract_with_correction(self, prompt: str):
        """Extract with self-correction on validation errors"""
        try:
            return self.structured_llm.invoke(prompt)
        except ValidationError as e:
            # Ask model to correct itself
            correction_prompt = f"""
The previous attempt failed validation with: {str(e)}
Original task: {prompt}
Please provide correct structured output that satisfies all requirements.
"""
            try:
                return self.structured_llm.invoke(correction_prompt)
            except:
                return None
```

### Progressive Validation

Try extracting at different completeness levels:

```python
class MinimalPerson(BaseModel):
    name: str

class CompletePerson(MinimalPerson):
    age: int
    email: str
    phone: str

class ProgressiveValidator:
    def __init__(self, llm):
        self.llm = llm
    
    def extract_best_effort(self, text: str):
        """Extract at the highest possible validation level"""
        # Try complete first, fall back to minimal
        for model in [CompletePerson, MinimalPerson]:
            try:
                structured = self.llm.with_structured_output(model)
                return structured.invoke(f"Extract: {text}")
            except:
                continue
        return None
```

These patterns make your extraction robust and production-ready.

## Complete Example: User Registration System

Here's a practical example showing all concepts together:

```python
from langchain_core.pydantic_v1 import BaseModel, Field, validator
from langchain_openai import ChatOpenAI
import re

class UserRegistration(BaseModel):
    """User registration information"""
    username: str = Field(description="Unique username")
    email: str = Field(description="Valid email address")
    full_name: str = Field(description="Full name")
    age: int = Field(description="Age in years")
    
    @validator("username")
    def validate_username(cls, v):
        if not re.match(r'^[a-zA-Z0-9_]{3,20}$', v):
            raise ValueError("Username must be 3-20 alphanumeric characters")
        return v
    
    @validator("email")
    def validate_email(cls, v):
        if not re.match(r'^[\w\.-]+@[\w\.-]+\.\w+$', v):
            raise ValueError("Invalid email format")
        return v

class UserRegistrationSystem:
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.structured_llm = self.llm.with_structured_output(UserRegistration)
    
    def register_user(self, user_input: str, max_retries=3):
        """Process user registration with validation"""
        prompt = f"Extract user registration from: {user_input}"
        
        for attempt in range(max_retries):
            try:
                user = self.structured_llm.invoke(prompt)
                print(f"✓ Registration successful: {user.username}")
                return user
            except ValidationError as e:
                print(f"✗ Attempt {attempt + 1} failed")
                if attempt < max_retries - 1:
                    prompt += "\n\nEnsure all requirements are met."
        
        return None
```

This system handles validation errors gracefully and retries with improved guidance.

## Best Practices

### 1. Start Simple, Add Complexity

Begin with basic structures and add validation as needed:

```python
# Start basic
class Product(BaseModel):
    name: str
    price: float

# Add validation when required
class ValidatedProduct(Product):
    @validator("price")
    def price_positive(cls, v):
        if v <= 0:
            raise ValueError("Price must be positive")
        return v
```

### 2. Provide Clear Field Descriptions

Good descriptions help the model understand what you need:

```python
class GoodModel(BaseModel):
    email: str = Field(description="Valid email address in format user@domain.com")
    age: int = Field(description="Age in years, between 0 and 150")
```

### 3. Use Appropriate Types

Choose the right type for each field:

```python
from datetime import datetime
from enum import Enum

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

class Task(BaseModel):
    title: str
    priority: Priority  # Enum ensures valid values
    due_date: datetime  # Proper date type
    completed: bool     # Boolean, not string
```

### 4. Handle Edge Cases

Plan for incomplete or invalid data:

```python
class RobustModel(BaseModel):
    required_field: str
    optional_field: Optional[str] = None
    list_field: List[str] = []  # Default to empty list
    numeric_field: float = 0.0  # Sensible default
```

## Summary

Structured outputs transform unreliable text into reliable data:

1. **Pydantic Models**: Type-safe Python objects with validation
2. **JSON Schemas**: Raw JSON for APIs and databases
3. **Error Handling**: Try-except, fallbacks, and defaults
4. **Validation Pipelines**: Retry loops and self-correction
5. **Best Practices**: Clear schemas, good descriptions, proper types

The result? AI applications that integrate seamlessly with your code and never break production.

## Next Steps

Try building a structured output system:
1. Define a Pydantic model for your use case
2. Add field validation with `@validator`
3. Implement a retry pipeline
4. Test with various inputs
5. Monitor validation success rates

What structured data will you extract?

