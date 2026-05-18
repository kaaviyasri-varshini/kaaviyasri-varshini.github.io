Title: LangChain Few-Shot Learning: Teaching by Example
Date: 2026-05-18
Category: Tutorials
Tags: LangChain, Few-Shot Learning, AI, LLM, Tutorial
Slug: langchain-few-shot-learning

## Learning Through Examples

Few-shot learning is one of the most powerful techniques in prompt engineering. Instead of just telling the model what to do, you show it examples of the desired behavior. This dramatically improves output quality and consistency.

## Few-Shot Learning Basics

The concept is simple: provide the model with a few examples of input-output pairs before asking it to process new input. The model learns the pattern from your examples and applies it to new cases.

**Zero-Shot (No Examples):**
```
Translate to French: "Hello, how are you?"
```

**Few-Shot (With Examples):**
```
Translate to French:

Example 1:
English: "Good morning"
French: "Bonjour"

Example 2:
English: "Thank you"
French: "Merci"

Now translate:
English: "Hello, how are you?"
French:
```

The few-shot version gives the model clear context about the format and style you expect.

## When to Use Few-Shot Learning

Few-shot prompting is especially useful when:

- You need consistent output formatting
- The task requires domain-specific knowledge
- You want to establish a particular tone or style
- Zero-shot results are inconsistent or incorrect
- You're working with structured data extraction

## FewShotPromptTemplate in LangChain

LangChain provides `FewShotPromptTemplate` to make few-shot learning easy and maintainable.

**Basic Structure:**

```python
from langchain.prompts import FewShotPromptTemplate, PromptTemplate

# Define the example format
example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\nOutput: {output}"
)

# Provide examples
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "hot", "output": "cold"},
]

# Create the few-shot prompt
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    prefix="Give the antonym of each word:",
    suffix="Input: {word}\nOutput:",
    input_variables=["word"]
)

# Use it
print(few_shot_prompt.format(word="big"))
```

**Output:**
```
Give the antonym of each word:
Input: happy
Output: sad
Input: tall
Output: short
Input: hot
Output: cold
Input: big
Output:
```

## Real-World Example: Sentiment Classification

Here's a practical example for sentiment analysis:

```python
from langchain.prompts import FewShotPromptTemplate, PromptTemplate

# Define how each example should look
example_template = """
Review: {review}
Sentiment: {sentiment}
"""

example_prompt = PromptTemplate(
    input_variables=["review", "sentiment"],
    template=example_template
)

# Training examples
examples = [
    {
        "review": "The product exceeded my expectations. Highly recommend!",
        "sentiment": "Positive"
    },
    {
        "review": "Terrible quality. Broke after one use.",
        "sentiment": "Negative"
    },
    {
        "review": "It's okay. Nothing special but does the job.",
        "sentiment": "Neutral"
    }
]

# Build the few-shot prompt
sentiment_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    prefix="Classify the sentiment of customer reviews as Positive, Negative, or Neutral.\n\n",
    suffix="Review: {review}\nSentiment:",
    input_variables=["review"]
)

# Test it
new_review = "Amazing service! Will definitely buy again."
prompt = sentiment_prompt.format(review=new_review)
print(prompt)
```

## Dynamic Example Selection

For advanced use cases, you can dynamically select which examples to include based on the input:

```python
from langchain.prompts.example_selector import LengthBasedExampleSelector

# Create an example selector
example_selector = LengthBasedExampleSelector(
    examples=examples,
    example_prompt=example_prompt,
    max_length=200  # Limit total prompt length
)

# Use it in the template
dynamic_prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="Classify sentiment:\n\n",
    suffix="Review: {review}\nSentiment:",
    input_variables=["review"]
)
```

This automatically adjusts the number of examples based on available token space.

## Structured Data Extraction Example

Few-shot learning excels at teaching models to extract structured data:

```python
examples = [
    {
        "text": "John Smith works at Google in Mountain View. Email: john@google.com",
        "output": "Name: John Smith | Company: Google | Location: Mountain View | Email: john@google.com"
    },
    {
        "text": "Sarah Johnson is a developer at Microsoft, Seattle office.",
        "output": "Name: Sarah Johnson | Company: Microsoft | Location: Seattle | Email: N/A"
    }
]

extraction_prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=PromptTemplate(
        input_variables=["text", "output"],
        template="Text: {text}\nExtracted: {output}"
    ),
    prefix="Extract structured information from text:\n\n",
    suffix="Text: {text}\nExtracted:",
    input_variables=["text"]
)
```

## Best Practices

1. **Use 3-5 examples** — More isn't always better; quality over quantity
2. **Show edge cases** — Include examples that cover different scenarios
3. **Keep examples diverse** — Vary the inputs to show the full pattern
4. **Match your use case** — Examples should closely resemble real inputs
5. **Test and iterate** — Adjust examples based on model performance

## Key Takeaways

- Few-shot learning teaches models by example rather than instruction alone
- `FewShotPromptTemplate` makes it easy to structure and reuse examples
- This technique dramatically improves consistency and accuracy
- Dynamic example selection helps manage token limits
- Few-shot prompting is essential for structured data tasks

Few-shot learning bridges the gap between generic model capabilities and your specific use case. Master this technique, and you'll see immediate improvements in your LangChain applications.
