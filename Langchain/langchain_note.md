
```
2:47:09
```

Agentic AI and AI Agent difference ->


AI Agent -?
LLM -> knows how to generates the text
if we configure or add some specifications to LLM to do certain task -> AI Agent
speciifications in some sense -> tools


when we enable communication between the AI agents => Agentic AI
example: business agent communicating with advertising agent
i.e orchestration of ai agents

Goal:
AI Agents = fulfill subgoal/ task
Agentic AI = fullfil entire goal


frameworks are nothing but just like a python module which has many classes, methods etc package them where someone can use it 

Langchain -> Agentic framework

Langchain has built the python classes for your agentic solutions that you can use and modify as per your use cases.


why langchain?
if i want to use gpt-5 model, we need to have openAI SDK and initialize it, if i want to use claude models, we need to have claude SDK and initialize it, if i want to use gemini model, we need to have google SDK and initialize it... so this make code redundancy

instead we can use langchain sdk/framework which has all the models



## Messages and Prompts

messages are like a static way to provide information for llm
prompts are like a dynamic way to provide information for llm

Prompts are the messages that are sent to the LLM. Prompts are more user-friendly than messages.

## Messages and Prompts
### The core mental model

Think of it as **template vs. runtime data**.

A **prompt** (`PromptTemplate`, `ChatPromptTemplate`) is a _reusable blueprint with holes in it_. You author it once, with `{placeholders}`, and fill those holes with variables at call time. It's static structure you control.

**Messages** (`SystemMessage`, `HumanMessage`, `AIMessage`, `ToolMessage`) are the _concrete, live units of conversation_ that actually flow into and out of the model. In an agent, they're the running state — they accumulate as the agent thinks, calls a tool, gets a result, and responds again.

A prompt template, when invoked, _produces_ messages. So they're not competing concepts — one is the mold, the other is what comes out of it and then grows during the loop.

### When to reach for each

Use a **prompt template** for the parts _you_ author and want to parameterize:

- The system prompt / role definition ("You are a research assistant that...")
- Output-format instructions, tone, constraints
- Few-shot examples
- Anything reusable across runs where only some values change

Use **messages** for everything that's _runtime conversation state_:

- The growing back-and-forth during the agent loop
- Tool calls (`AIMessage` with `tool_calls`) and especially tool results (`ToolMessage`) — this part is agent-specific and you essentially never template it
- Conversation history you're carrying between turns

The rule of thumb: if you're writing it by hand and it has variables, it's a prompt. If the framework is appending it as the agent runs, it's a message.

### How this looks in current LangChain

The official docs now center on `create_agent`, a minimal, highly configurable agent harness where you compose the agent from model, tools, prompt, and middleware. In this model, the agent's state _is_ a list of messages, and you typically hand it the system instructions as a prompt while the harness manages the message list for you: [Langchain](https://docs.langchain.com/oss/python/langchain/overview)

python

```python
from langchain.agents import create_agent

agent = create_agent(
    model="anthropic:claude-sonnet-4-5",
    tools=[search, calculator],
    system_prompt="You are a helpful research assistant. Cite sources.",  # the authored, static part
)

# messages = the runtime state that grows through the loop
result = agent.invoke({
    "messages": [{"role": "user", "content": "What's the GDP of Japan?"}]
})
```

Here the system prompt is your template layer; the `messages` list is the live state the agent appends to (its tool calls, the `ToolMessage` results, the final answer) as it iterates.

The **bridge** between the two worlds is `MessagesPlaceholder` — when you build a `ChatPromptTemplate` and need to reserve a slot for the dynamic message list to be injected:

python

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}. Be concise."),   # templated, has a variable
    MessagesPlaceholder("history"),                  # where live messages get dropped in
    ("human", "{input}"),
])
```

### The short version

Build your _instruction layer_ with prompt templates because you want reusability and variables. Let _messages_ be the agent's working memory — the thing that flows through the loop and that the framework grows for you, particularly the tool-call/tool-result exchange. If you find yourself manually assembling a long message list for an agent, that's usually a sign something belongs in a template instead.

One caveat worth flagging: LangChain's agent API has churned a lot (the older `AgentExecutor` + `MessagesPlaceholder("agent_scratchpad")` pattern is being superseded by `create_agent` and LangGraph). If you tell me which version you're on, I can make the examples exact for your setup.


Using PromptTemplate
generating llm response using Pydantic
generating llm response using typed dict


difference between pydantic and typed dict

when workflows (results of llm) are very strict use pydantic, if necessary
when workflows (results of llm) are not much strict use typed dict, if necessary

means you are not suppose to break workflows because of some keys miss matched from llm

pydantic will return pydantic obejct
typed dict will return dictionary

pydantic will through error at runtime if anything miss matched
where as typed dict will mark errors while writing code but doesnt break anything at run time


Pipeline: Sequence of task


Chains: Output of one node will the input of next node