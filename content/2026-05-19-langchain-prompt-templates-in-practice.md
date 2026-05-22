Title: LangChain Prompt Templates in Practice: Building Reusable and Dynamic Prompts
Date: 2026-05-19
Category: AI Engineering
Tags: langchain, prompt-engineering, python, ai-development
Slug: langchain-prompt-templates-in-practice


## Introduction

In previous posts, we explored prompt anatomy and hygiene. Now it's time to put that knowledge into practice by building **reusable prompt templates** that can adapt to different scenarios without code duplication.

Think of prompt templates as blueprints—you design them once, then fill in the details dynamically based on your application's needs.

## Why Prompt Templates Matter

Without templates, you'd be writing prompts like this:

```python
# Bad: Hardcoded prompts everywhere
response1 = llm.invoke("Translate 'Hello' to Spanish")
response2 = llm.invoke("Translate 'Goodbye' to Spanish")
response3 = llm.invoke("Translate 'Thank you' to French")
```

This approach:
- Creates code duplication
- Makes updates difficult (change one, miss others)
- Doesn't scale as your application grows
- Mixes prompt logic with business logic

**Prompt templates solve all of these problems.**

## Reusable Prompt Blueprints

LangChain's `PromptTemplate` class lets you create reusable blueprints with placeholders:

```python
from langchain.prompts import PromptTemplate

# Create a reusable template
translation_template = PromptTemplate(
    input_variables=["text", "target_language"],
    template="Translate the following text to {target_language}:\n\n{text}"
)

# Use it multiple times with different inputs
prompt1 = translation_template.format(text="Hello", target_language="Spanish")
prompt2 = translation_template.format(text="Goodbye", target_language="French")
prompt3 = translation_template.format(text="Thank you", target_language="German")
```

### Benefits of This Approach

1. **Single Source of Truth**: Update the template once, all usages reflect the change
2. **Type Safety**: LangChain validates that you provide all required variables
3. **Testability**: Test your template separately from your business logic
4. **Maintainability**: Prompts are separated from code logic

## Dynamic Prompt Inputs

Templates become powerful when you combine them with dynamic data sources:

```python
from langchain.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

# Template for product descriptions
product_template = PromptTemplate(
    input_variables=["product_name", "features", "target_audience"],
    template="""
Create a compelling product description for {product_name}.

Key Features:
{features}

Target Audience: {target_audience}

Write a description that highlights the benefits and appeals to the target audience.
"""
)

# Dynamic data from your database or API
products = [
    {
        "product_name": "SmartWatch Pro",
        "features": "Heart rate monitoring, GPS tracking, 7-day battery life",
        "target_audience": "Fitness enthusiasts"
    },
    {
        "product_name": "CloudSync Storage",
        "features": "Unlimited storage, end-to-end encryption, cross-platform sync",
        "target_audience": "Remote teams"
    }
]

llm = ChatOpenAI(model="gpt-4")

# Generate descriptions dynamically
for product in products:
    prompt = product_template.format(**product)
    description = llm.invoke(prompt)
    print(f"\n{product['product_name']}:\n{description.content}\n")
```

### Real-World Use Cases

- **Customer Support**: Template for handling different types of inquiries
- **Content Generation**: Blog posts, social media, email campaigns
- **Data Analysis**: Consistent analysis prompts across different datasets
- **Code Review**: Standardized review prompts for different programming languages

## Conditional Prompt Sections

Sometimes you need prompts that adapt based on context. LangChain supports conditional logic:

```python
from langchain.prompts import PromptTemplate

def create_email_template(include_urgency=False, include_attachments=False):
    base = "Write a professional email with the following details:\n\n"
    base += "Subject: {subject}\n"
    base += "Recipient: {recipient}\n"
    base += "Main Message: {message}\n"
    
    if include_urgency:
        base += "\nUrgency Level: {urgency}\n"
        base += "Emphasize the time-sensitive nature of this request.\n"
    
    if include_attachments:
        base += "\nAttachments: {attachments}\n"
        base += "Mention the attachments in the email body.\n"
    
    base += "\nTone: {tone}"
    
    variables = ["subject", "recipient", "message", "tone"]
    if include_urgency:
        variables.append("urgency")
    if include_attachments:
        variables.append("attachments")
    
    return PromptTemplate(
        input_variables=variables,
        template=base
    )

# Use different versions based on needs
urgent_template = create_email_template(include_urgency=True)
attachment_template = create_email_template(include_attachments=True)
full_template = create_email_template(include_urgency=True, include_attachments=True)
```

### Using Partial Variables

LangChain also supports partial templates—pre-filling some variables:

```python
from langchain.prompts import PromptTemplate

# Base template
base_template = PromptTemplate(
    input_variables=["company", "product", "tone"],
    template="Write a {tone} announcement for {company} about {product}."
)

# Create a partial template with company pre-filled
acme_template = base_template.partial(company="ACME Corp")

# Now you only need to provide product and tone
prompt = acme_template.format(product="new AI assistant", tone="exciting")
```

## Prompt Composition

For complex applications, you can compose multiple templates together:

```python
from langchain.prompts import PromptTemplate

# Individual components
context_template = PromptTemplate(
    input_variables=["context"],
    template="Context: {context}\n"
)

question_template = PromptTemplate(
    input_variables=["question"],
    template="Question: {question}\n"
)

instruction_template = PromptTemplate(
    input_variables=["instruction"],
    template="Instructions: {instruction}\n"
)

# Compose them
def create_qa_prompt(context, question, instruction="Answer based on the context only."):
    parts = [
        context_template.format(context=context),
        question_template.format(question=question),
        instruction_template.format(instruction=instruction)
    ]
    return "\n".join(parts)

# Use the composed prompt
full_prompt = create_qa_prompt(
    context="LangChain is a framework for building LLM applications.",
    question="What is LangChain?",
    instruction="Provide a concise answer in one sentence."
)
```

### Pipeline Composition

You can also chain templates in a pipeline:

```python
from langchain.prompts import PromptTemplate
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain

# Step 1: Generate ideas
idea_template = PromptTemplate(
    input_variables=["topic"],
    template="Generate 3 blog post ideas about {topic}. List them as bullet points."
)

# Step 2: Expand one idea
expansion_template = PromptTemplate(
    input_variables=["idea"],
    template="Expand this blog post idea into a detailed outline:\n\n{idea}"
)

llm = ChatOpenAI(model="gpt-4")

# Create chains
idea_chain = LLMChain(llm=llm, prompt=idea_template)
expansion_chain = LLMChain(llm=llm, prompt=expansion_template)

# Execute pipeline
ideas = idea_chain.run(topic="AI prompt engineering")
outline = expansion_chain.run(idea=ideas.split('\n')[0])  # Take first idea

print(outline)
```

## Best Practices

### 1. Keep Templates Focused

Each template should have a single, clear purpose:

```python
# Good: Focused template
summarize_template = PromptTemplate(
    input_variables=["text", "max_words"],
    template="Summarize the following text in {max_words} words or less:\n\n{text}"
)

# Bad: Template trying to do too much
multi_purpose_template = PromptTemplate(
    input_variables=["text", "task", "language", "style", "length"],
    template="Do {task} on this text in {language} with {style} style, {length} length:\n\n{text}"
)
```

### 2. Use Descriptive Variable Names

```python
# Good
PromptTemplate(
    input_variables=["customer_name", "order_id", "issue_description"],
    template="..."
)

# Bad
PromptTemplate(
    input_variables=["x", "y", "z"],
    template="..."
)
```

### 3. Store Templates Separately

Following the project guidelines, keep prompts in separate `.txt` files:

```
project/
├── prompts/
│   ├── translation.txt
│   ├── summarization.txt
│   └── email_generation.txt
└── app.py
```

```python
# app.py
from langchain.prompts import PromptTemplate

def load_template(filename):
    with open(f"prompts/{filename}", "r") as f:
        template_text = f.read()
    return PromptTemplate.from_template(template_text)

translation_template = load_template("translation.txt")
```

### 4. Version Your Templates

Track template changes like code:

```python
# prompts/summarization_v2.txt
# Version: 2.0
# Changed: Added tone parameter
# Date: 2026-05-19

Summarize the following text in {max_words} words with a {tone} tone:

{text}
```

## Practical Example: Customer Support Bot

Let's build a complete example:

```python
from langchain.prompts import PromptTemplate
from langchain_openai import ChatOpenAI

# Load template from file
with open("prompts/customer_support.txt", "r") as f:
    template_text = f.read()

support_template = PromptTemplate(
    input_variables=["customer_name", "issue_type", "issue_details", "customer_tier"],
    template=template_text
)

llm = ChatOpenAI(model="gpt-4", temperature=0.7)

def handle_support_ticket(ticket):
    """Process a customer support ticket"""
    prompt = support_template.format(
        customer_name=ticket["name"],
        issue_type=ticket["type"],
        issue_details=ticket["details"],
        customer_tier=ticket["tier"]
    )
    
    response = llm.invoke(prompt)
    return response.content

# Example usage
ticket = {
    "name": "Alice Johnson",
    "type": "Billing Issue",
    "details": "I was charged twice for my subscription this month.",
    "tier": "Premium"
}

response = handle_support_ticket(ticket)
print(response)
```

**prompts/customer_support.txt:**
```
You are a helpful customer support agent. Respond to the following customer inquiry:

Customer Name: {customer_name}
Customer Tier: {customer_tier}
Issue Type: {issue_type}

Issue Details:
{issue_details}

Provide a professional, empathetic response that:
1. Acknowledges the issue
2. Explains the next steps
3. Provides a timeline for resolution
4. Thanks the customer for their patience

For Premium tier customers, prioritize urgency and offer additional compensation if appropriate.
```

## Summary

Prompt templates are essential for building maintainable LangChain applications:

- **Reusable Blueprints**: Define once, use everywhere
- **Dynamic Inputs**: Adapt to different data sources
- **Conditional Logic**: Templates that change based on context
- **Composition**: Build complex prompts from simple components
- **Best Practices**: Keep focused, use descriptive names, version control

In the next post, we'll explore **managing prompt changes safely** and strategies for testing and monitoring prompt performance in production.

