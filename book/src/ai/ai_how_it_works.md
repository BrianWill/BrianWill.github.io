What follows are notes for an explainer about AI. The target audience is a techinically sophisticated user who wants to understand the terminology and broad concepts. The explainer may get into some recent (2010's on) history to better explain the sequence of how these ideas were developed, but it will not get into mathematical details of neural networks and training.

Your task is to write an expanded outline in @ai_how_it_works_outline.md based on this document. Expand this outline into a better structure and fill in many details for these topics:

- Keep the prose to mostly bullet points. This is not yet meant to be full prose with transitions and connective phrasing.
- Avoid conversational language. 
- Avoid loose metaphors. Keep the information and concept explanations concrete.
- The list below very loosely represents the desired order of topics. Don't feel obligated to follow it too strictly.


[start of the notes]

## AI Basics

recent AI history:

neural network

model

gradient descent

layers

perceptrons

tensor

attention

transformer

LLM

chatbot

input vs output tokens
    sub-word tokens

output tokens fed back in as input (but they are normally just called "output tokens")

pre-training / post-training (training in between)

tokens

documents

base model


VLM


fine-tuning

reinforcement learning
    has a technical definition and a more popular loose definition


context

    "transcript" is the sequence of text or tokens that were fed into the model to produce the context, whereas the context is a bunch of activations

KV caching
    session token stream is split into **stack** of blocks
        useless to have missing holes in block list:
            whichever earliest block is missing, that's where replay must resume from
            effectively, if a block is vacated from cache, then all of the session blocks after it are useless and so should be vacated too
        strategy: earlier blocks cached longer than more recent blocks
        so your most recent prompt may require replaying some amount of tokens before your prompt, but usually not the whole session unless you haven't  
        

scenarios where the whole chat history has to be re-sent / "replayed", in whole or in part


hallucination

RAG

tools

agents

sub-agents / agent orchestration / agentic loops
    MoE (more of a training thing?)

in-context learning

skills

compacting

system prompts

memory

agents.md (claude.md / gemini.md / etc)

mcp

open models

distillation

diffusion (image models before moving to transformers)



[end of notes; ignore everything past here]


### 1. "Tokens are split sequentially into blocks"
**Verdict: Correct.**
In systems like vLLM (which powers many backends) or Anthropic’s Prompt Caching, the conversation isn't stored as one giant file. It is broken into **Blocks** (often 16, 32, or 64 tokens each).
* **Why?** It prevents "memory fragmentation." If every user had one giant contiguous block of memory, the GPU would run out of space very quickly. By using small, fixed-size blocks, the system can scatter your "memory" across different physical parts of the GPU RAM, much like how a computer's RAM handles multiple open tabs.
* **Hashed Prefixes:** Each block is assigned a unique "hash" based on the text it contains **plus every token that came before it**. This ensures that the block for the word "apple" in a recipe is cached differently than the word "apple" in a tech review.

---

### 2. "These blocks effectively form a stack... if a middle block is vacated, all blocks after it must be reconstructed."
**Verdict: Correct (with a 2026 "Research" Caveat).**
You hit on the most important constraint of the Transformer architecture: **Causal Attention.**

* **The "Stack" Logic:** To understand token #101, the model *must* have the mathematical state (the KV Cache) of tokens #1 through #100. Because the math is sequential, you cannot "teleport" to the middle of a conversation. 
* **The "Middle Block" Problem:** If Block B (tokens 33–64) is deleted but Block A (1–32) and Block C (65–96) remain:
    * The model can use Block A.
    * It **must re-calculate** Block B.
    * **Crucially:** Even though you "have" the data for Block C, the model usually cannot use it yet because the mathematical "hidden state" that flows *into* Block C depends on the result of Block B. 

> **The 2026 Exception:** There is a new technique called **RelayCaching** (and similar "perforated cache" research) currently being tested. It attempts to "stitch" non-sequential blocks together by approximating the missing middle. However, in production tools like **Claude Code** or **Gemini Advanced**, your "Stack" analogy is the golden rule: **A break in the chain forces a replay from that point forward.**

---

### Summary Table: Cache Dynamics

| Feature | Reality | Analogy |
| :--- | :--- | :--- |
| **Storage** | Discrete Blocks (Hashed). | Lego bricks. |
| **Structure** | Sequential Chain (Prefix-based). | A freight train (cars must be in order). |
| **Eviction** | If a middle "car" is removed... | The rest of the train behind it is uncoupled. |
| **Result** | Re-computation ("Prefill") starts at the break. | You have to re-drive that stretch of road. |

**Does this help clarify why maintaining a "clean" prefix (like keeping your system prompt and large files at the very top of the chat) is so important for saving money?**





Your intuition is spot on. In the world of LLM architecture and tools like Claude Code, the "Session" is a high-level **User Experience** wrapper (managing things like file permissions, history files on disk, and tool access), while the actual "thing" being fed to the model is something else entirely.

The proper technical term for the sequence of tokens that makes up the current context is the **Context Window** (or simply the **Context**). However, depending on the layer of the stack you are discussing, there are more precise terms:

### 1. **The Context Window** (The "Physical" Limit)
This is the most common term. It refers to the total "slots" available in the model's working memory at a single point in time. When you run `/clear`, you aren't ending the "Session," but you are emptying the **Context Window**.

### 2. **The Transcript** or **History** (The Logical Sequence)
If you are talking about the literal string of text (prompts + responses) that Claude Code has recorded, developers call this the **Transcript**. 
* In Claude Code, the "Transcript" is stored in your local `~/.claude/projects/` directory. 
* When you hit Enter, the CLI "packages" the relevant parts of this transcript into a single **Prompt Construction** to send to the API.

### 3. **The Prompt Prefix** (The "Immutable" Start)
Since you've been thinking about the "Stack" of blocks, this is a crucial term. 
* The **Prefix** consists of the tokens at the very beginning of the sequence (System Prompt + Tools + `CLAUDE.md`).
* As long as the **Prefix** stays identical, the KV Cache remains valid. 
* In Claude Code, when you `/clear`, you are essentially stripping the "Messages" from the end of the stack but keeping the **Prefix** (System instructions) intact for the next turn.

### 4. **The Active Window** (The "Sliding" View)
When a conversation gets extremely long (e.g., 300k tokens) and exceeds the 200k limit, the model uses a **Sliding Window**. 
* The tokens that are currently "inside" the 200k limit are the **Active Context**.
* Anything that has fallen off the back of the stack is **Evicted Context**.

---

### Summary of Terminology

| Term | What it specifically means |
| :--- | :--- |
| **Session** | The high-level "job" (permissions, local logs, project state). |
| **Context Window** | The actual "bucket" of tokens the model is currently looking at. |
| **Transcript** | The total history of what has been said (even if it's too big to fit in context). |
| **Prefill** | The *process* of the model reading the token stream to build its internal math. |
| **State** | The mathematical result (KV Cache) of having processed a specific token stream. |

**In short:** If you want to sound like a compiler engineer or an LLM researcher, use **"The Context"** or **"The Contextual Prefix."** If you are talking about the file that stores your conversation on your hard drive, use **"The Transcript."**

**Would you like me to show you how the `/compact` command actually "rewrites" the Transcript to create a shorter Context for the next turn?**