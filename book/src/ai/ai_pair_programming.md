# Pair Programming with AI

# How to Vibe Code (Carefully)


## AI Basics

[add some recent history]

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

KV caching

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

memory

agents.md (claude.md / gemini.md / etc)

mcp

open models

distillation

diffusion (image models before moving to transformers)

# prompting strategies

ask the bot to help you develop the right prompt

if the bot can't do the task reliably, maybe the prompt needs clarification / more detail / more examples, or maybe just tell it explicitly to double check its work
    - or set up an agentic loop where a supervisor instructs one subagent to check the work of another
    - note of course that rechecking schemes mean more tokens, thus more time and cost

## Using Claude

claude CLI

    uses API key (or subscription)
    sends your prompts to Anthropic data center where the model processes your input
    runs with associated project directory...but isn't reliably sandboxed to the dir
        typically you ask the bot not to stray from the directory without your permission
            ...but Claude cannot strictly enforce this, e.g. allowing claude to use find:* isn't sandboxed to just the project directory
    receives output back from data center
    output can include tool tags, which trigger your local Claude to invoke local actions, such as file reads / writes, and running shell commands
    Cladue will ask for permission before acting locally on your machine
    

### session management

sessions

choosing models

switching models

effort

/clear
/context
/usage





the rubber duck that talks back (and even writes your code)

prompting assistant:
    chatbot helps you develop / refine your prompts to be passed on to the agent doing the actual coding
    prompter bot and coder bot
        prompter clobbers file @next_instruction.md with next prompt for codebot
        codebot always just prompted with "Follow the prompt in @next_instrucation.md"
        what about output from codebot? promptbot needs to see some of that to help write good prompts

        promptbot could also have standing orders to recognize when context should be cleared and commits should be made, so it will suggest commits and commit messages and then can make commits for you

other people characterize using AI to code as being a project manager

the way I've used it as a pair partner: my partner writes all of the code while I assess the work, plan, and determine every next step

describe my experience with Japanese vocab app

responsible use of AI
    DO NOT POLLUTE
    maybe now an even higher obligation to vet your output, to increase the signal-to-noise ratio in the world

it's fun...but it's still work

do I want to try using AI for everything now?
    form of AI psychosis: like Homer solving every minor inconvenience by shooting everything with a gun

one Claude session, no worktrees: no multi-agent workflow, and don't prompt for sub-agents (though I think Claude thinking sometimes automatically delegates to subagents on its own accord)

works really well for small tweaks / additions / removals when it has a clear context to immitate, e.g. established code structure, data model, UI patterns

managing the context; keeping agents.md up to date

standards of what constitutes good code doesn't seem really different
    - to extent AIs can write good code, they do so in the way skilled humans do
    - exception would be better at shoveling code around (see arg about html/css)
    - what AI lacks in true expertise, it often compensates with thorougness

less need for abstraction building:
    can shovel around masses of very boring, plain html / css / vanilla frontend js / Go backend routes and db functions / sql
        any feature you build for the web is splayed across this chain, so natural that people spent 25 years trying to abstract it away
            25 years of backend and frontend web frameworks attempting to abstract away this annoying separating

haven't yet defined skills nor used mcp

AI helps me cover breadth, which naturally sacrifices depth
    - certainly won't learn details the same as if I had to struggle with them myself
    - handling details for me is largely the point of using AI
    - annoying when other people shove slop on you where they clearly haven't bothered with details

each additional requirement distracts from others
    "scream when you hear today's secret word"
    humans can do short-term temporary learning, but the AIs currently do not update their training when used

tricks to break down to steps:
    - ask to analyze for possible refactors one by one: possible dead code, possible move code to different files, possible add tests, possible remove tests, etc.
    - stub out UI of a feature first with dummy data before asking for working implementation