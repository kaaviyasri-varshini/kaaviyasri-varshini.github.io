Title: LangChain Prompting Fundamentals: Anatomy and Variables
Date: 2026-05-18
Category: Tutorials
Tags: LangChain, Prompting, AI, LLM, Tutorial
Slug: langchain-prompt-anatomy-and-variables

## Understanding Prompt Structure

When working with LangChain, understanding how to structure prompts is fundamental to getting consistent, high-quality outputs from language models. A well-structured prompt is the difference between vague responses and precise, actionable results.

## Anatomy of a Prompt

Every effective prompt has three core components that work together:

**1. Instruction** — The task you want the model to perform. This is the "what" of your prompt.

```python
"Translate the following text to French."
"Summarize this article in 3 bullet points."
"Extract all email addresses from the text."
```

**2. Context** — Background information that helps the model understand the situation. This is the "why" and "how" of your prompt.

```python
"You are a professional translator working on legal documents."
"This article is from a technical blog about machine learning."
"You are helping a user clean up their contact list."
```

**3. Input** — The actual data the model needs to process. This is the "what to work on."

```python
"Text: Bonjour, comment allez-vous?"
"Article: [full article text here]"
"Contact info: John Doe, john@example.com, 555-1234..."
```

## Instruction vs Context vs Input

Understanding the distinction between these three elements is crucial:

- **Instruction** tells the model what action to take
- **Context** shapes how the model should approach the task
- **Input** provides the raw material to work with

Here's a complete example:

```python
from langchain.prompts import PromptTemplate

prompt = PromptTemplate(
    template="""
    Context: You are a technical writer creating documentation for developers.
    
    Instruction: Explain the following code snippet in simple terms.
    
    Input:
    {code}
    
    Explanation:
    """,
    input_variables=["code"]
)
```

## Prompt Variables

Prompt variables make your prompts reusable and dynamic. Instead of hardcoding values, you define placeholders that get filled in at runtime.

**Basic Variable Usage:**

```python
from langchain.prompts import PromptTemplate

template = "Tell me a {adjective} joke about {topic}."

prompt = PromptTemplate(
    template=template,
    input_variables=["adjective", "topic"]
)

# Generate the prompt
final_prompt = prompt.format(adjective="funny", topic="programming")
print(final_prompt)
# Output: "Tell me a funny joke about programming."
```

**Multiple Variables in Context:**

```python
template = """
You are a {role} with {years} years of experience.

Task: {task}

Input: {input_data}

Please provide your expert analysis.
"""

prompt = PromptTemplate(
    template=template,
    input_variables=["role", "years", "task", "input_data"]
)

result = prompt.format(
    role="data scientist",
    years="10",
    task="Analyze this dataset for anomalies",
    input_data="[dataset here]"
)
```

## Why This Matters

Separating instruction, context, and input with proper variables gives you:

- **Reusability** — Write once, use many times with different inputs
- **Clarity** — Clear structure makes debugging easier
- **Consistency** — Same format produces more predictable results
- **Maintainability** — Update prompts without touching application code

## Practical Example

Here's a real-world example combining all concepts:

```python
from langchain.prompts import PromptTemplate
from langchain.llms import OpenAI

# Define the prompt template
email_template = PromptTemplate(
    template="""
    Context: You are a {tone} customer service representative.
    
    Instruction: Write a response to the following customer inquiry.
    
    Customer Name: {customer_name}
    Inquiry: {inquiry}
    
    Response:
    """,
    input_variables=["tone", "customer_name", "inquiry"]
)

# Use it
llm = OpenAI(temperature=0.7)
prompt = email_template.format(
    tone="friendly and professional",
    customer_name="Sarah",
    inquiry="I haven't received my order yet. Order #12345."
)

response = llm(prompt)
print(response)
```

## Key Takeaways

1. Every prompt should have clear instruction, context, and input sections
2. Use variables to make prompts dynamic and reusable
3. Structure matters — well-organized prompts get better results
4. LangChain's `PromptTemplate` makes this pattern easy to implement

Understanding these fundamentals sets the foundation for more advanced prompting techniques like few-shot learning and chain-of-thought reasoning, which we'll explore in upcoming posts.
