# Notes
Layer-by-layer breakdown
#  Layer  What happens
## 1 Perception - Raw user input is parsed — intent extracted, context assembled
## 2 Memory (STM/LTM) - Short-term (in-context history) and long-term (vector DB / knowledge base) are looked up and merged
## 3 RAG Pipeline - Query is embedded → top-K chunks retrieved → re-ranked → injected into the prompt
## 4 Goal & Planning - Agent identifies the goal, decomposes it into subtasks using CoT / ReAct / Plan-and-Execute
## 5 LLM Reasoning Core - The LLM reasons over the augmented prompt and decides if it has enough info
## 6 Action & Tools (MCP) - If more info is needed, the Tool Selector dispatches calls via the Model Context Protocol server to web search, DB, APIs, code executor, file system, etc.
## 7 Observe → Reflect Loop - Tool results are observed, STM is updated, agent re-plans — loops until goal is satisfied (ReAct style)
## 8 Response Synthesis - Final answer is synthesised, self-critiqued for hallucinations/safety, then formatted
## 9 Memory Update - The turn is written back to STM and key learnings persisted to LTM
## 10 Feedback Loop - User thumbs up/down feeds back into LTM for continuous improvement
