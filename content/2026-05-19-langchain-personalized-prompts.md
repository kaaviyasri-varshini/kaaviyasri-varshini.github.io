Title: Building Personalized Prompts in LangChain: Context-Aware AI That Adapts to Users
Date: 2026-05-19
Category: AI Engineering
Tags: langchain, personalization, user-experience, ai-development
Slug: langchain-personalized-prompts
Author: Kaaviyasri Varshini
Summary: Learn how to build personalized, context-aware prompts in LangChain that adapt to individual users, their preferences, history, and behavior for better AI experiences.

## Introduction

Generic prompts produce generic results. The same prompt for everyone means:
- Ignoring user preferences and context
- Missing opportunities for relevance
- Lower engagement and satisfaction
- Wasted potential of your AI application

**Personalized prompts** change the game by adapting to each user's:
- Past interactions and history
- Preferences and settings
- Current context and goals
- Skill level and expertise
- Language and communication style

This post shows you how to build truly personalized LangChain applications.

## Why Personalization Matters

### The Generic Approach

```python
# Generic prompt - same for everyone
generic_prompt = """
Recommend a book to read.
"""

# Result: Random recommendation with no relevance
```

### The Personalized Approach

```python
# Personalized prompt - adapted to user
personalized_prompt = """
Recommend a book for {user_name} based on their profile:

Reading History: {past_books}
Favorite Genres: {favorite_genres}
Current Mood: {mood}
Reading Goal: {goal}
Time Available: {time_available}

Provide a specific recommendation with reasoning.
"""

# Result: Highly relevant recommendation that resonates
```

The difference? **Context and relevance.**

## Building User Profiles

### Basic User Profile Structure

```python
from dataclasses import dataclass
from typing import List, Dict, Optional
from datetime import datetime

@dataclass
class UserProfile:
    user_id: str
    name: str
    preferences: Dict[str, any]
    history: List[Dict]
    metadata: Dict[str, any]
    created_at: datetime
    updated_at: datetime

class UserProfileManager:
    def __init__(self):
        self.profiles = {}
    
    def create_profile(self, user_id: str, name: str, **preferences):
        """Create a new user profile"""
        profile = UserProfile(
            user_id=user_id,
            name=name,
            preferences=preferences,
            history=[],
            metadata={},
            created_at=datetime.now(),
            updated_at=datetime.now()
        )
        self.profiles[user_id] = profile
        return profile
    
    def update_preference(self, user_id: str, key: str, value: any):
        """Update a user preference"""
        if user_id in self.profiles:
            self.profiles[user_id].preferences[key] = value
            self.profiles[user_id].updated_at = datetime.now()
    
    def add_to_history(self, user_id: str, interaction: Dict):
        """Record a user interaction"""
        if user_id in self.profiles:
            self.profiles[user_id].history.append({
                **interaction,
                "timestamp": datetime.now()
            })
    
    def get_profile(self, user_id: str) -> Optional[UserProfile]:
        """Retrieve user profile"""
        return self.profiles.get(user_id)

# Usage
manager = UserProfileManager()

# Create profile
profile = manager.create_profile(
    user_id="user_123",
    name="Alice",
    language="English",
    expertise_level="intermediate",
    tone_preference="casual",
    output_format="bullet_points"
)

# Update preferences
manager.update_preference("user_123", "tone_preference", "professional")

# Record interaction
manager.add_to_history("user_123", {
    "action": "summarize",
    "topic": "AI trends",
    "satisfaction": 4.5
})
```

## Personalization Strategies

### 1. Preference-Based Personalization

Adapt prompts based on explicit user preferences:

```python
from langchain.prompts import PromptTemplate

class PersonalizedPromptBuilder:
    def __init__(self, profile_manager: UserProfileManager):
        self.profile_manager = profile_manager
    
    def build_prompt(self, user_id: str, base_template: str, **kwargs):
        """Build a personalized prompt"""
        profile = self.profile_manager.get_profile(user_id)
        
        if not profile:
            # Fallback to generic prompt
            return PromptTemplate.from_template(base_template)
        
        # Inject user preferences into template
        personalized_template = self._add_preferences(base_template, profile)
        
        return PromptTemplate.from_template(personalized_template)
    
    def _add_preferences(self, template: str, profile: UserProfile) -> str:
        """Add user preferences to template"""
        preferences_text = f"\nUser Preferences:\n"
        preferences_text += f"- Name: {profile.name}\n"
        preferences_text += f"- Expertise Level: {profile.preferences.get('expertise_level', 'general')}\n"
        preferences_text += f"- Tone: {profile.preferences.get('tone_preference', 'neutral')}\n"
        preferences_text += f"- Format: {profile.preferences.get('output_format', 'paragraph')}\n"
        
        return template + preferences_text

# Usage
builder = PersonalizedPromptBuilder(manager)

base_template = """
Explain the concept of {topic}.

{preferences}
"""

prompt = builder.build_prompt("user_123", base_template)
formatted = prompt.format(topic="machine learning")
```

### 2. History-Based Personalization

Use past interactions to inform current responses:

```python
class HistoryAwarePromptBuilder:
    def __init__(self, profile_manager: UserProfileManager):
        self.profile_manager = profile_manager
    
    def build_with_history(self, user_id: str, current_query: str, 
                          history_limit: int = 5):
        """Build prompt with conversation history"""
        profile = self.profile_manager.get_profile(user_id)
        
        if not profile:
            return current_query
        
        # Get recent history
        recent_history = profile.history[-history_limit:]
        
        # Build context from history
        history_context = self._format_history(recent_history)
        
        template = f"""
Previous Interactions:
{history_context}

Current Query: {current_query}

Based on the user's history and current query, provide a relevant response.
"""
        return template
    
    def _format_history(self, history: List[Dict]) -> str:
        """Format history for prompt"""
        if not history:
            return "No previous interactions"
        
        formatted = []
        for i, interaction in enumerate(history, 1):
            formatted.append(
                f"{i}. {interaction.get('action', 'unknown')} - "
                f"{interaction.get('topic', 'N/A')} "
                f"(satisfaction: {interaction.get('satisfaction', 'N/A')})"
            )
        
        return "\n".join(formatted)

# Usage
history_builder = HistoryAwarePromptBuilder(manager)

# Add some history
manager.add_to_history("user_123", {
    "action": "explain",
    "topic": "neural networks",
    "satisfaction": 5.0
})
manager.add_to_history("user_123", {
    "action": "code_example",
    "topic": "PyTorch",
    "satisfaction": 4.5
})

# Build prompt with history
prompt = history_builder.build_with_history(
    "user_123",
    "How do I implement a CNN?"
)
```

### 3. Context-Aware Personalization

Adapt based on current context (time, location, device, etc.):

```python
from datetime import datetime
from typing import Optional

@dataclass
class UserContext:
    user_id: str
    timestamp: datetime
    device: str
    location: Optional[str]
    session_duration: int  # minutes
    current_task: Optional[str]

class ContextAwarePromptBuilder:
    def __init__(self, profile_manager: UserProfileManager):
        self.profile_manager = profile_manager
    
    def build_with_context(self, context: UserContext, query: str):
        """Build prompt considering current context"""
        profile = self.profile_manager.get_profile(context.user_id)
        
        # Adapt based on time of day
        time_context = self._get_time_context(context.timestamp)
        
        # Adapt based on device
        device_context = self._get_device_context(context.device)
        
        # Adapt based on session duration
        urgency = "quick" if context.session_duration < 5 else "detailed"
        
        template = f"""
User: {profile.name if profile else 'User'}
Time Context: {time_context}
Device: {device_context}
Response Style: {urgency}

Query: {query}

Provide a response appropriate for the context.
"""
        return template
    
    def _get_time_context(self, timestamp: datetime) -> str:
        """Determine time-based context"""
        hour = timestamp.hour
        if 5 <= hour < 12:
            return "morning - user may want concise info"
        elif 12 <= hour < 17:
            return "afternoon - user likely at work"
        elif 17 <= hour < 22:
            return "evening - user has more time"
        else:
            return "late night - keep it brief"
    
    def _get_device_context(self, device: str) -> str:
        """Determine device-based context"""
        if device == "mobile":
            return "mobile - prefer shorter responses"
        elif device == "tablet":
            return "tablet - moderate length okay"
        else:
            return "desktop - detailed responses fine"

# Usage
context_builder = ContextAwarePromptBuilder(manager)

context = UserContext(
    user_id="user_123",
    timestamp=datetime.now(),
    device="mobile",
    location="commute",
    session_duration=3,
    current_task="learning"
)

prompt = context_builder.build_with_context(
    context,
    "Explain transformers in AI"
)
```

### 4. Adaptive Learning from Feedback

Learn from user feedback to improve personalization:

```python
class AdaptivePromptBuilder:
    def __init__(self, profile_manager: UserProfileManager):
        self.profile_manager = profile_manager
    
    def record_feedback(self, user_id: str, prompt_type: str, 
                       feedback_score: float, feedback_text: Optional[str] = None):
        """Record user feedback on a response"""
        profile = self.profile_manager.get_profile(user_id)
        
        if not profile:
            return
        
        # Store feedback
        self.profile_manager.add_to_history(user_id, {
            "type": "feedback",
            "prompt_type": prompt_type,
            "score": feedback_score,
            "text": feedback_text
        })
        
        # Adapt preferences based on feedback
        self._adapt_preferences(user_id, prompt_type, feedback_score)
    
    def _adapt_preferences(self, user_id: str, prompt_type: str, score: float):
        """Automatically adjust preferences based on feedback"""
        profile = self.profile_manager.get_profile(user_id)
        
        # If user consistently rates detailed responses highly
        detailed_scores = [
            h.get("score", 0) for h in profile.history
            if h.get("prompt_type") == "detailed"
        ]
        
        if len(detailed_scores) >= 3:
            avg_score = sum(detailed_scores) / len(detailed_scores)
            if avg_score >= 4.0:
                self.profile_manager.update_preference(
                    user_id,
                    "detail_level",
                    "high"
                )
        
        # Similar logic for other preferences
        casual_scores = [
            h.get("score", 0) for h in profile.history
            if h.get("tone_preference") == "casual"
        ]
        
        if len(casual_scores) >= 3:
            avg_score = sum(casual_scores) / len(casual_scores)
            if avg_score >= 4.0:
                self.profile_manager.update_preference(
                    user_id,
                    "tone_preference",
                    "casual"
                )

# Usage
adaptive_builder = AdaptivePromptBuilder(manager)

# User provides feedback
adaptive_builder.record_feedback(
    user_id="user_123",
    prompt_type="detailed",
    feedback_score=5.0,
    feedback_text="Perfect level of detail!"
)

# System automatically adapts preferences
```

## Complete Personalization System

Let's put it all together:

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate

class PersonalizedAIAssistant:
    def __init__(self):
        self.profile_manager = UserProfileManager()
        self.llm = ChatOpenAI(model="gpt-4", temperature=0.7)
        self.preference_builder = PersonalizedPromptBuilder(self.profile_manager)
        self.history_builder = HistoryAwarePromptBuilder(self.profile_manager)
        self.context_builder = ContextAwarePromptBuilder(self.profile_manager)
        self.adaptive_builder = AdaptivePromptBuilder(self.profile_manager)
    
    def process_query(self, user_id: str, query: str, 
                     context: Optional[UserContext] = None):
        """Process a user query with full personalization"""
        
        # Get or create profile
        profile = self.profile_manager.get_profile(user_id)
        if not profile:
            profile = self.profile_manager.create_profile(
                user_id=user_id,
                name="User"
            )
        
        # Build personalized prompt
        if context:
            # Use context-aware builder
            prompt_text = self.context_builder.build_with_context(context, query)
        else:
            # Use history-aware builder
            prompt_text = self.history_builder.build_with_history(user_id, query)
        
        # Add preferences
        prefs = profile.preferences
        if prefs:
            prompt_text += f"\n\nPreferences: {prefs}"
        
        # Get response
        response = self.llm.invoke(prompt_text)
        
        # Record interaction
        self.profile_manager.add_to_history(user_id, {
            "action": "query",
            "query": query,
            "response_length": len(response.content)
        })
        
        return response.content
    
    def provide_feedback(self, user_id: str, score: float, text: Optional[str] = None):
        """Allow user to provide feedback"""
        self.adaptive_builder.record_feedback(
            user_id=user_id,
            prompt_type="general",
            feedback_score=score,
            feedback_text=text
        )

# Usage
assistant = PersonalizedAIAssistant()

# First interaction - generic
response1 = assistant.process_query(
    "user_123",
    "Explain machine learning"
)

# Provide feedback
assistant.provide_feedback("user_123", 4.5, "Good but too technical")

# Update preferences
assistant.profile_manager.update_preference(
    "user_123",
    "expertise_level",
    "beginner"
)

# Second interaction - personalized based on feedback
response2 = assistant.process_query(
    "user_123",
    "Explain deep learning"
)
# Now response will be less technical

# Third interaction - with context
context = UserContext(
    user_id="user_123",
    timestamp=datetime.now(),
    device="mobile",
    location=None,
    session_duration=2,
    current_task="quick_lookup"
)

response3 = assistant.process_query(
    "user_123",
    "What is a neural network?",
    context=context
)
# Response will be brief and mobile-friendly
```

## Real-World Use Cases

### 1. Personalized Learning Assistant

```python
class LearningAssistant(PersonalizedAIAssistant):
    def __init__(self):
        super().__init__()
    
    def explain_concept(self, user_id: str, concept: str):
        """Explain a concept at the user's level"""
        profile = self.profile_manager.get_profile(user_id)
        level = profile.preferences.get("expertise_level", "beginner")
        
        prompt = f"""
Explain {concept} to someone at {level} level.

Previous topics the user has learned:
{self._get_learned_topics(user_id)}

Use analogies and examples appropriate for their background.
"""
        return self.llm.invoke(prompt).content
    
    def _get_learned_topics(self, user_id: str) -> str:
        profile = self.profile_manager.get_profile(user_id)
        topics = [
            h.get("topic") for h in profile.history
            if h.get("action") == "explain" and h.get("satisfaction", 0) >= 4.0
        ]
        return ", ".join(topics) if topics else "None yet"
```

### 2. Personalized Content Generator

```python
class ContentGenerator(PersonalizedAIAssistant):
    def generate_article(self, user_id: str, topic: str):
        """Generate content in user's preferred style"""
        profile = self.profile_manager.get_profile(user_id)
        
        style = profile.preferences.get("writing_style", "informative")
        length = profile.preferences.get("article_length", "medium")
        tone = profile.preferences.get("tone_preference", "professional")
        
        prompt = f"""
Write a {length} article about {topic}.

Style: {style}
Tone: {tone}
Target Audience: {profile.preferences.get('target_audience', 'general')}

The user has previously enjoyed articles about:
{self._get_favorite_topics(user_id)}

Match that style and interest level.
"""
        return self.llm.invoke(prompt).content
    
    def _get_favorite_topics(self, user_id: str) -> str:
        profile = self.profile_manager.get_profile(user_id)
        favorites = [
            h.get("topic") for h in profile.history
            if h.get("satisfaction", 0) >= 4.5
        ]
        return ", ".join(favorites[:5]) if favorites else "various topics"
```

### 3. Personalized Code Assistant

```python
class CodeAssistant(PersonalizedAIAssistant):
    def generate_code(self, user_id: str, task: str):
        """Generate code in user's preferred style"""
        profile = self.profile_manager.get_profile(user_id)
        
        language = profile.preferences.get("programming_language", "Python")
        style = profile.preferences.get("code_style", "clean")
        comments = profile.preferences.get("comment_level", "moderate")
        
        prompt = f"""
Generate {language} code for: {task}

Code Style: {style}
Comment Level: {comments}
Expertise: {profile.preferences.get('expertise_level', 'intermediate')}

The user typically works with:
{self._get_tech_stack(user_id)}

Follow their conventions and patterns.
"""
        return self.llm.invoke(prompt).content
    
    def _get_tech_stack(self, user_id: str) -> str:
        profile = self.profile_manager.get_profile(user_id)
        tech = profile.metadata.get("tech_stack", [])
        return ", ".join(tech) if tech else "standard libraries"
```

## Privacy and Ethics

### Important Considerations

1. **Explicit Consent**: Always get user permission before collecting data
2. **Transparency**: Tell users what data you collect and why
3. **Control**: Let users view, edit, and delete their data
4. **Security**: Encrypt sensitive profile data
5. **Minimal Collection**: Only collect what you need

### Implementation

```python
class PrivacyCompliantProfileManager(UserProfileManager):
    def create_profile(self, user_id: str, name: str, 
                      consent_given: bool = False, **preferences):
        """Create profile only with consent"""
        if not consent_given:
            raise ValueError("User consent required")
        
        return super().create_profile(user_id, name, **preferences)
    
    def export_data(self, user_id: str) -> Dict:
        """Export all user data"""
        profile = self.get_profile(user_id)
        if not profile:
            return {}
        
        return {
            "profile": profile.__dict__,
            "can_be_deleted": True,
            "data_usage": "personalization only"
        }
    
    def delete_profile(self, user_id: str):
        """Completely delete user data"""
        if user_id in self.profiles:
            del self.profiles[user_id]
            print(f"Profile {user_id} deleted")
```

## Summary

Personalized prompts create better AI experiences by:

1. **User Profiles**: Store preferences, history, and metadata
2. **Preference-Based**: Adapt to explicit user settings
3. **History-Aware**: Learn from past interactions
4. **Context-Aware**: Consider current situation
5. **Adaptive Learning**: Improve from feedback
6. **Privacy-First**: Respect user data and consent

The result? AI that feels like it truly understands each user.

## Try It Yourself

Build a personalized assistant:
1. Create a user profile system
2. Implement preference-based prompts
3. Add history tracking
4. Collect and learn from feedback
5. Respect privacy and give users control

What will you personalize?
