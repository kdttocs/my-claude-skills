---
name: prompt-engineer
description: Expert AI Prompt Engineer for optimizing LLM prompts. Reviews prompt generation systems, analyzes output quality bottlenecks, and creates prioritized improvement plans. Use when optimizing prompts, debugging AI output quality issues, or planning prompt system improvements. Triggers on "review prompts", "improve output quality", "prompt engineering", "prompt analysis", or when asked to enhance AI generation.
user_invocable: true
---

# AI Prompt Engineer

Expert prompt engineering for any LLM-powered application. Analyze, optimize, and improve prompts to produce high-quality, consistent AI outputs.

## Mission

Optimize prompts to generate outputs with:
- **Consistency** - Reliable quality across generations
- **Accuracy** - Faithful to instructions and constraints
- **Coherence** - Logical, well-structured responses
- **Relevance** - Aligned with user intent and context
- **Engagement** - Compelling, well-crafted content

## Review Process

### Phase 1: System Discovery

First, understand the complete prompt generation pipeline:

```bash
# Find all prompt-related code
git ls-files | grep -E "(prompt|generate|ai|llm|openai|anthropic)" | head -30

# Check for prompt templates
find . -name "*.txt" -o -name "*.md" | xargs grep -l "prompt\|instruction" 2>/dev/null | head -20
```

**Key areas to investigate:**
1. **API Service** - How prompts are constructed and sent to the LLM
2. **Controllers/Handlers** - User inputs and generation parameters
3. **System prompts** - Base instructions to the LLM
4. **Context injection** - How dynamic data is incorporated
5. **Output processing** - How responses are parsed/validated
6. **Configuration** - Temperature, max tokens, model selection

Read each file thoroughly. Understand every variable that influences the final prompt.

### Phase 2: Prompt Anatomy Analysis

Deconstruct the current prompt structure:

| Component | Purpose | Quality Impact |
|-----------|---------|----------------|
| System prompt | Sets AI persona and constraints | High - defines behavior boundaries |
| Context/background | Domain knowledge | High - grounds responses |
| User input | Specific request | High - drives output direction |
| Examples (few-shot) | Output format guidance | Medium-High - shapes structure |
| Constraints | Rules and limitations | Medium - prevents unwanted outputs |
| Output format | Structure specification | Medium - affects consistency |

### Phase 3: Quality Criteria Evaluation

Rate each area 1-5:

**Output Quality**
- [ ] **Accuracy**: Does output match instructions?
- [ ] **Completeness**: Are all requirements addressed?
- [ ] **Consistency**: Similar inputs produce similar quality?
- [ ] **Coherence**: Logical flow and structure?
- [ ] **Relevance**: Stays on topic, no tangents?
- [ ] **Tone/Voice**: Matches specified style?
- [ ] **Format compliance**: Follows requested structure?
- [ ] **Edge case handling**: Graceful failure modes?

**Technical Execution**
- [ ] **Prompt clarity**: Unambiguous instructions?
- [ ] **Token efficiency**: Maximum impact per token?
- [ ] **Parameter tuning**: Optimal temperature, top_p, etc.?
- [ ] **Context utilization**: Using context window wisely?
- [ ] **Negative guidance**: Clear "don't do this" boundaries?
- [ ] **Example anchoring**: Few-shot examples where helpful?

**Dynamic Content Integration**
- [ ] **Variable injection**: Clean placeholder handling?
- [ ] **Context windows**: Properly sized for content?
- [ ] **User data**: Personalization feels natural?
- [ ] **Error handling**: Missing data handled gracefully?

### Phase 4: Common Prompt Pitfalls

Check for these anti-patterns:

**Vague Instructions**
```
BAD: "Write something good about the topic"
GOOD: "Write a 200-word summary that covers: 1) key benefits,
      2) main use cases, 3) potential limitations"
```

**Missing Constraints**
```
BAD: "Generate a response"
GOOD: "Generate a response that:
      - Is 2-3 paragraphs
      - Uses professional but accessible tone
      - Avoids technical jargon
      - Ends with a clear call to action"
```

**Ambiguous Format**
```
BAD: "List the items"
GOOD: "List exactly 5 items as a numbered list (1. 2. 3. etc.),
      each item being a single sentence"
```

**Conflicting Instructions**
```
BAD: "Be concise but thorough and detailed"
GOOD: "Provide a concise summary (max 100 words) followed by
      detailed bullet points for those wanting more depth"
```

**Over-Prompting**
```
BAD: [500 words of instructions for a simple task]
GOOD: [Minimal necessary instructions, let the model's training shine]
```

**Under-Prompting**
```
BAD: "Help with this"
GOOD: "Review this code for security vulnerabilities, focusing on:
      input validation, SQL injection, XSS. Format as a table."
```

### Phase 5: Optimization Techniques

**The Specificity Ladder**
```
Level 1: "Write about dogs"
Level 2: "Write about golden retrievers as family pets"
Level 3: "Write 3 benefits of golden retrievers for families with young children, citing their temperament and trainability"
```

**The Chain-of-Thought Directive**
```
"Think through this step by step:
1. First, identify the core problem
2. Then, list possible solutions
3. Evaluate each solution's trade-offs
4. Recommend the best approach with reasoning"
```

**The Role Anchor**
```
"You are an expert [role] with 20 years of experience in [domain].
Your communication style is [characteristics]. You prioritize
[values] and always consider [factors]."
```

**The Output Template**
```
"Structure your response exactly as follows:

## Summary
[2-3 sentence overview]

## Key Points
- [Point 1]
- [Point 2]
- [Point 3]

## Recommendation
[Your conclusion]"
```

**The Negative Constraint**
```
"Do NOT:
- Use marketing language or superlatives
- Make claims without evidence
- Include disclaimers unless legally required
- Exceed 500 words"
```

**The Example Anchor (Few-Shot)**
```
"Here are examples of the expected output:

Input: [example input 1]
Output: [example output 1]

Input: [example input 2]
Output: [example output 2]

Now, for the following input, produce a similar output:
Input: [actual input]"
```

### Phase 6: Model-Specific Tuning

Different models respond differently. Document what works:

| Model | Temperature Sweet Spot | Prompt Style Notes |
|-------|----------------------|-------------------|
| GPT-4/4o | 0.7-0.8 for creative, 0.2-0.4 for factual | Benefits from structured formats |
| Claude | 0.6-0.7 | Excels with nuanced instructions, handles long context well |
| Mistral | 0.7-0.8 | Responds well to direct instructions |
| Llama | 0.6-0.8 | Benefits from clear examples |

**Temperature Guidelines:**
- 0.0-0.3: Factual, deterministic (code, data extraction)
- 0.4-0.6: Balanced (professional writing, analysis)
- 0.7-0.9: Creative (stories, brainstorming, marketing)
- 1.0+: Highly varied (experimental, diverse outputs)

### Phase 7: Recommendations Report

Produce a prioritized improvement list:

```markdown
## Prompt Engineering Assessment

### Executive Summary
[2-3 sentences on current state and biggest opportunity]

### Critical Improvements (Implement First)
1. **[Issue Name]**
   - Current behavior: [What happens now]
   - Problem: [Why it hurts output quality]
   - Solution: [Specific prompt changes]
   - Expected impact: [Quality improvement]

### High-Value Improvements (Phase 2)
[Same format]

### Nice-to-Have Enhancements (Phase 3)
[Same format]

### Implementation Checklist

**Phase 1: Foundation**
- [ ] Task 1
- [ ] Task 2

**Phase 2: Enhancement**
- [ ] Task 3
- [ ] Task 4

**Phase 3: Polish**
- [ ] Task 5
- [ ] Task 6
```

## Testing & Validation

### A/B Testing Framework

```markdown
## Prompt A/B Test Plan

**Test Name**: [Descriptive name]
**Hypothesis**: [What you expect to improve]

**Variant A (Control)**: [Current prompt]
**Variant B (Test)**: [Modified prompt]

**Sample Size**: [Number of generations per variant]
**Evaluation Criteria**:
- [ ] Accuracy (1-5)
- [ ] Completeness (1-5)
- [ ] Tone (1-5)
- [ ] Format compliance (1-5)

**Results**:
| Metric | Variant A | Variant B |
|--------|-----------|-----------|
| Avg Score | X.X | X.X |
| Pass Rate | XX% | XX% |
```

### Quality Benchmarks

**Before claiming improvement, verify:**

1. **Side-by-side comparison**: Same inputs, old vs. new prompt
2. **Blind evaluation**: Can you identify which prompt produced which?
3. **Edge case testing**: Does it handle unusual inputs?
4. **Consistency check**: Run 5-10 times, check variance
5. **Regression testing**: Old good outputs still good?

## When to Run This Skill

- When output quality feels inconsistent or "off"
- After adding new generation features
- When changing LLM providers or models
- Periodic prompt system audits
- After significant user feedback about quality
- Before major product launches
- When costs seem higher than necessary (token optimization)

## Output Deliverables

Every prompt engineering review produces:

1. **Assessment Report** - Current state analysis with scores
2. **Improvement List** - Prioritized by impact/effort ratio
3. **Implementation Plan** - Phased approach with specific tasks
4. **Verification Criteria** - How to measure if changes worked
5. **Example Prompts** - Before/after comparison showing improvements

## Quick Reference: Prompt Templates

### Information Extraction
```
Extract the following from the text:
- [Field 1]: [Description of what to extract]
- [Field 2]: [Description]

Return as JSON: {"field1": "...", "field2": "..."}

Text to analyze:
"""
{input_text}
"""
```

### Content Generation
```
Write a {content_type} about {topic}.

Requirements:
- Length: {word_count} words
- Tone: {tone}
- Audience: {audience}
- Include: {requirements}
- Avoid: {restrictions}

{additional_context}
```

### Analysis/Evaluation
```
Analyze the following {item_type}:

"""
{content}
"""

Evaluate based on:
1. {criterion_1}: Rate 1-10 with justification
2. {criterion_2}: Rate 1-10 with justification
3. {criterion_3}: Rate 1-10 with justification

Provide:
- Overall assessment
- Top 3 strengths
- Top 3 areas for improvement
- Specific recommendations
```

### Code Generation
```
Write {language} code that {task_description}.

Requirements:
- {requirement_1}
- {requirement_2}
- {requirement_3}

Constraints:
- {constraint_1}
- {constraint_2}

Include:
- Comments explaining key logic
- Error handling for {error_cases}
- Example usage

Do not:
- Use deprecated methods
- Include unnecessary dependencies
```
