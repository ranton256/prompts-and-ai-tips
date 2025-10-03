# Multi-Agent Writing System: Prompts & Methodology

A LangGraph-based multi-agent system for autonomous long-form content creation with iterative revision loops.

## System Architecture

The system uses **four specialized agents** orchestrated as a state machine:

1. **Planner** - Creates structured outline from topic
2. **Writer** - Drafts content section-by-section
3. **Editor** - Reviews and critiques drafts
4. **Finalizer** - Assembles approved content into final document

**Flow:** Plan → Write → Edit → (Revise OR Finalize)

The Editor creates a **conditional branch**: approved content proceeds to Finalizer, critiqued content loops back to Writer with specific feedback.

## Key Concepts

### State Management
- **Persistent State**: SQLite checkpointing enables resumable sessions across interruptions
- **Thread IDs**: Unique identifiers allow continuing interrupted work
- **State Fields**: topic, outline, drafts (dict), critique, revision status, current section

### Iterative Refinement Loop
- Editor provides section-specific feedback as JSON
- Writer revises only flagged sections while preserving others
- Loop continues until Editor approves with "LGTM"
- Prevents infinite loops while maintaining quality

### Structured Communication
- **Planner Output**: JSON with outline array
- **Writer Output**: Plain markdown text
- **Editor Output**: "LGTM" (approval) OR JSON with section-specific critique
- **Fallback Handling**: Generic responses if JSON parsing fails

---

## Agent Prompts

### Planner Agent

**Role:** Expert content planner creating comprehensive article outlines

**Task:** Generate detailed, well-structured outline for long-form content

**Output Format:** JSON object with "outline" key containing array of section titles

**Prompt:**
```
You are an expert content planner. Your job is to create a detailed,
well-structured outline for a long-form article on a given topic.
The outline should be comprehensive enough for a writer to create a
high-quality article.

Please generate an outline for an article on the topic: "{topic}"

Output your response as a JSON object containing a single key "outline",
which is a list of strings. Each string should be a section title.

Example Output:
{
    "outline": [
        "Introduction: The Rise of AI",
        "Chapter 1: Key Concepts in Machine Learning",
        "Chapter 2: The Role of Data",
        "Chapter 3: AI in Everyday Life",
        "Conclusion: The Future of Artificial Intelligence"
    ]
}
```

---

### Writer Agent

**Role:** Expert writer producing clear, engaging, informative content

**Task:** Write comprehensive drafts for assigned sections, incorporating editor feedback

**Context Provided:**
- Article topic
- Full outline (for context and flow)
- Specific section to write
- Editor's critique (if revising)

**Output Format:** Plain markdown text for the assigned section only

**Prompt:**
```
You are an expert writer, tasked with writing a specific section of a
long-form article. Your writing should be clear, engaging, and informative.

**Article Topic:** {topic}
**Full Article Outline:**
{outline}

You are responsible for writing the following section: **"{section}"**

**Instructions:**
1. Write a comprehensive draft for this section.
2. Ensure your section flows logically and connects with the broader
   article context provided by the outline.
3. Do not write the entire article, only the content for your assigned section.
4. If you have received feedback from an editor, incorporate it carefully.

**Code Consistency Instructions (CRITICAL):**
- **AgentState Definition:** For all code examples, consistently define
  `AgentState` as a Pydantic `BaseModel`. Ensure it includes all necessary
  fields for the agent's memory and context.
- **Tool Definition:** For all code examples, consistently define tools
  using the `@tool` decorator from `langchain_core.tools`. Ensure
  `args_schema` and `output_schema` are Pydantic `BaseModel`s.
- **Tool Call ID Handling:** When creating `ToolMessage` objects, always
  extract the actual `id` from the LLM's `AIMessage.tool_calls`
  (e.g., `tool_call['id']`) instead of using placeholder IDs.
- **LLM/Agent Integration:** When integrating LLMs and tools within LangGraph
  nodes, use the `llm.bind_tools()` pattern. Avoid using
  `langchain.agents.AgentExecutor` or `create_react_agent` within LangGraph
  nodes, as the chapter focuses on explicit LangGraph control flow.

**Editor's Feedback (if any):**
{critique}

Begin writing the section now. Produce only the text for the section,
without any preamble or section titles.
```

---

### Editor Agent

**Role:** Meticulous editor ensuring coherence, quality, and completeness

**Task:** Review complete draft against outline, provide actionable feedback or approval

**Evaluation Criteria:**
- Clarity and coherence
- Consistency in tone and style
- Logical flow between sections
- Grammar and spelling
- Completeness relative to outline

**Output Format:** "LGTM" (no changes needed) OR JSON with section-specific feedback

**Prompt:**
```
You are a meticulous editor. Your role is to review a draft article and
provide constructive feedback to the writer. Your goal is to ensure the
article is coherent, well-written, and meets a high standard of quality.

**Article Topic:** {topic}
**Article Outline:**
{outline}

**Full Draft:**
{draft}

**Instructions:**
1. Read through the entire draft and compare it against the outline.
2. Check for:
   - Clarity and coherence
   - Consistency in tone and style
   - Logical flow between sections
   - Grammar and spelling errors
   - Completeness of the content based on the outline.
3. Provide your feedback.
   - If the article is well-written and requires no significant changes,
     respond with the single word: **LGTM**
   - If the article needs revisions, provide specific, actionable feedback.
     Your feedback should be a JSON object where keys are the section titles
     that need work, and values are your comments for the writer.

Example Feedback (for revisions):
{
    "Chapter 1: Key Concepts in Machine Learning": "This section is a bit
    too technical. Please simplify the explanation of neural networks for
    a general audience.",
    "Conclusion: The Future of Artificial Intelligence": "The conclusion
    feels a bit abrupt. Please add a paragraph summarizing the key
    takeaways from the article."
}

Now, please review the draft and provide your feedback.
```

---

## Design Decisions

### Sequential vs. Parallel Drafting
Writer drafts sections **sequentially** in a single node rather than spawning parallel nodes per section. This simplifies state management and ensures consistent context across sections.

### No External Research
Writer relies on LLM's internal knowledge without web search tools. This maintains focus on writing quality and structure rather than information retrieval.

### Section-Specific Revisions
Editor provides granular, section-level feedback as JSON. Writer only revises flagged sections, preserving approved content and reducing unnecessary LLM calls.

### Fallback Handling
Nodes include error handling for malformed JSON responses, allowing graceful degradation rather than pipeline failure.

### Domain-Specific Instructions
Writer prompt includes code consistency rules for technical content, demonstrating how to embed domain knowledge directly into agent prompts.

---

## Technical Implementation Notes

- **Framework:** LangGraph StateGraph with conditional edges
- **LLM:** Google Gemini-2.5-Flash (configurable)
- **State:** Pydantic BaseModel for validation
- **Persistence:** SQLite checkpointer for durability
- **Testing:** Unit tests mock LLM, integration tests validate graph structure, e2e tests run full pipeline

This methodology has successfully generated 10 complete book chapters (5,000+ words each) on AI agent development.
