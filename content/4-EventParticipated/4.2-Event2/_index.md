---
title: "Event 2"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---



# Summary Report: “BUILDING AGENTIC AI”

### Event Objectives

- Optimizing Context with Amazon Bedrock
- Build AI agent automation with Amazon Bedrock through practical techniques and real-world use cases
### Speakers

- **Nguyen Gia Hung** - Head of Solutions Architect, AWS
- **Kien Nguyen** - Solutions Architect, AWS
- **Viet Pham** - Founder & CEO, Diagflow
- **Kha Van** - Community Leader, AWS
- **Thang Ton** - Co-Founder & COO, Cloud Thinker
- **Henry Bui** - Head of Engineering, Cloud Thinker

### Key Highlights

#### Techniques to optimize costs and performance for AI agent systems

- Long product release times → Lost revenue/missed opportunities
- Poor performance → Lost productivity, costly costs
- Non-compliance with security regulations → Lost security, reputation

#### Four “Quick Wins” techniques to optimize

#### Prompt Caching
This is the most important technique mentioned, which can reduce costs by 70-90% and increase processing speed: Context structure: The Context window is divided into 3 parts: (1) System & Tool Schema, (2) Conversation History, and (3) Objective Prompt. Common mistake: Most people only cache the System Prompt and Tool parts. However, the Conversation History part is the most expensive part (can account for 80-90% of the cost) and is often overlooked. The right strategy: Set a “checkpoint” so that the entire conversation history is also cached. Although the first run (cache write) costs 25% more, subsequent runs will save 90%

#### Context Compaction

Cloud Thinker has pointed out a smart summary technique to avoid losing cache The old way: Create a new agent to summarize the old conversation. This method loses all previous cache and often reduces quality (performance degradation). The new way (Cloudthinker technique): Keep the agent and conversation history intact (to take advantage of the existing cache), only change the “Objective Prompt” part to a new task “summarize this conversation”. This approach helps to take advantage of cache hits, reduce the summarization cost from e.g. $0.30 to e.g. $0.030 (~90% reduction) and improve the output quality

#### Tool Consolidation

The problem with newer protocols like MCP (Model Context Protocol) is that putting too many tools (e.g. 50 tools) into the context will cause context flooding. Solution: Instead of putting the entire complex schema (data structure) into the prompt, use a simple “dictionary” and group the instructions. Just-in-time Instruction: The agent can use a special command (get instruction) to get detailed instructions on how to use the tool only when needed. This reduces the number of tokens that must be fed into the input continuously

#### Parallel Tool Calling

Instead of running sequentially like the old ReAct model (2022), modern models allow multiple tools to run in parallel to save time. However, this feature is often not enabled by default; programmers need to add specific instructions (e.g., “maximize efficiency”) to force the model to run in parallel  

### Key Takeaways

#### Cost Management Strategy

- **Input cost:** accounts for the majority of the operating costs of AI agents running in a loop. That is, each round, the Agent must reload the entire context (Conversation history, system prompt, memory). So the longer the Agent loop runs, the more money it will burn -> The longer the History, the more input tokens each round will increase.

- Solution: use Prompt Caching and checkpointing to minimize this cost. Reduce costs by up to 80-90%.

#### Smart “Summarization” technique

Summarization: Keep the current agent and conversation history intact to take advantage of the existing cache, and maintain better context quality. With this technique, the summarization cost is reduced from $0.3 to $0.03 (reduced by ~90%) and the output quality is improved.

#### Tool Design: Avoid context flooding
- Tool Design:
- Problem: Too many tools are included (e.g., MCP with 50 tools) will cause context flooding.
- Solution: Create a special command for the Agent to call to get detailed instructions when needed, instead of stuffing the entire schema into the prompt.
- Make the context compact, reduce token usage and increase performance.

### Performance optimization: Force parallel running (Parallel Tool Calling)
- Maximize Efficiency: Add specific instructions to the prompt to force the model to implement tasks in parallel instead of sequentially.

### Event Experience

Participating in the “Building Agentic AI” workshop was a very interesting experience, improving knowledge about Agentic AI. Some outstanding experiences:

#### Learn from highly specialized speakers
- Speakers from AWS, Cloudthinker, Diaflow shared best practices in modern application design.

#### Some event photos
![event2](/images/5.12-event.jpg)
