# Pair Programming with AI

# How to Vibe Code (Carefully)

## Conservative experiments

Until recently, my AI usage has been very conservative, particularly for coding. Like a lot of people, I had only been using the chatbot web interfaces rather than agents or APIs, and my main AI use cases were for ad hoc question answering, web searching, (human) language translation, and problem troubleshooting, *e.g.* I'd often open a chat session with an error message.

As for writing code, the most I attempted to generate were some occasional regexes or SQL queriess, or sometimes example scaffolding or usage code for some unfamiliar language or API. These attempts were usually successful, though sometimes required some prodding, especially with less popular and well documented topics. For example, my attempts to use ChatGPT for questions about Odin required a lot of skepticism and pushback on my part, such as in this [exchange about Odin types](https://chatgpt.com/share/69e4a23b-07c4-83e8-96ca-713b562acbb8): 

```
Me: No still very wrong. "type" cannot be used as a parameter type.
```

```
ChatGPT: You’re correct again. Thank you for the correction. Let’s discard the earlier framing and state this accurately, aligned with how Odin actually works...
```

```
Me: You are making up syntax. This is illegal:

My_Int :: type int

Read the manual.
```

```
ChatGPT: You are correct. Thank you for insisting on precision. I was indeed inventing syntax, and that is not acceptable here. Let’s reset fully and describe this exactly as Odin defines it, without extrapolation...
```

This same session also has a back-and-forth about parameter specialization of union types. This was a case where I knew the bot's explanations did not hold water, but I wasn't sure of the correct answer myself, so all I could do was poke holes in its explanations:

```
ChatGPT: Good catch. The behavior you’re seeing is correct, and the earlier explanation needs a key correction...
```

```
ChatGPT: ...I’ll answer it directly, without invented rules...
```

```
ChatGPT: ...You are exactly right — and this exposes the real rule, which earlier explanations blurred.

You’ve identified the contradiction correctly...
```

```
ChatGPT: You’re right to call this out, and thank you for sticking with it. The earlier explanations implicitly assumed compile-time knowledge that the Odin compiler does not and cannot have. Let’s fix this cleanly and accurately.

What follows is the correct model...
```

```
ChatGPT: This is the crux of the confusion, and now we can state the actual mechanism precisely...
```

```
ChatGPT: Good — this observation finally pins it down. The behavior you’re seeing is the definitive clue to the real semantics.

The correct explanation is...
```

```
ChatGPT: This is a good place to stop hand-waving and state the actual rule, because your example exposes it cleanly...
```

```
ChatGPT: Exactly — that’s correct. The “forward compatibility” idea I suggested is misleading in the context of parapoly unions. You’ve caught the subtlety. Let me clarify precisely...
```

Only after several challenges did the bot finally give a (seemingly) correct explanation. 

Then came the fanfare surrounding GPT-5.4 and Sonnet/Opus 4.6, but I still resisted trying agentic AI coding until more and more people whom I personally trusted said it was time to take it seriously (for example, [here](https://blog.s-schoener.com/2026-01-14-claude-optimism/) and [here](https://x.com/SebAaltonen/status/2044122213436063792)).
    
## First attempt at agentic coding

So in early March 2026 (way late to the party), I bought the $20 Claude subscription and started on a greenfield project, a vocabulary tracking and drilling program for learning Japanese. Within a week, I was already hooked enough to also get the $20 OpenAI and Gemini subscriptions.

Over the next couple weeks, I alternated between Claude (CLI, almost all Sonnet 4.6) and Codex (desktop app, almost all GPT-5.4), switching to the other every time my 5 hour window expired.

> [!NOTE]
> Gemini has proven way less capable for coding (stemming from its poor use of tools, perhaps), but I occasionally find it useful for word wrangling and brain storming tasks. I'm waiting to see if they can improve it reasonably soon, but I might just keep the subscription anyway for the 5TB of Google Drive storage.

My verdict? **Coding is not yet a "solved" problem (as some have claimed), but I can now see a future where I no longer write first draft code by hand, and it's coming probably sooner than expected.**

I consider this still an 'AI moderate' position, as you get overstated claims in both directions: opponents seem overly pessimistic of the current capabilities and the likely eventual future capabilities; proponents seem overly enthusiastic (to put it mildly).

In their recent podcast series, *Wading Through AI*, Casey Muratori and Demetri Spanos agree that the best current AIs are good for writing programs up to only a few thousand lines of code. From my recent experiences, this seems overly conservative, though we may be imagining different degrees of AI autonomy: "a few thousand lines" might be correct for one-shot vibe coding, but I've been having a lot of success with an incremental and closely monitored prompting strategy.

In general, I found the AI followed my directions very faithfully more than, perhaps, 97% of the time and produced results I could accept without change more than 95% of the time. Most of my prompts result in a few dozen to few hundred added or edited lines, a minority touch several hundred lines, and a few outliers add or edit one to two thousand lines.

### Micro level quality

On a micro level, the code has been consistently and impeccably boring and straightforward, with very sensible function decomposition, good naming sense, and a good eye for many (if not all) details. Over a month of prompting, I've barely seen any function logic bugs that didn't stem from under-specified prompting or requirements volatility (along the way, I've introduced many unplanned features and heavily redesigned other features, resulting in some serious code churn). 

By week two, I stopped skimming the generated function bodies of each prompt and just noted the function signatures. By week three, I was mostly watching which files got touched by each prompt and how much. 'Was the expected amount of code added or removed in the right places? If so, then probably good enough.' 

### Macro level quality

On a macro level, the key aspect of code I don't entrust to the AI is data design. Not that I found the AI's own data design ideas were necessarily way off the mark, but this is the aspect of code where the AI's proclivity to prefer adding over deleting is most dangerous. Keeping a tight hand on the data design is also the simplest concrete way for me to direct the scope and flow of the program features: there's only so far afield the AI might venture if it has to play within my data sandbox.

Aside from data design, it also seems critical for the human to exercise tight control over the tech stack and dependencies. At no point in my project did the AI attempt to sneak in new dependencies on its own accord, but this probably stemmed from my incremental prompting strategy and my having established a clear foundation early on. I imagine the AI is not so restrained in one-shot projects.

Another maybe crucial practice to to do periodic cleanup. Fortunately, I found you can offload this to the AI as well. For example:

```
Prompt: Review the backend code for dead code and redundancies. If issues are found, suggest some code deletions and consolidations.
```

```
Prompt: File foo might be getting overly large. Suggest a way to reorganize the code into new or existing files.
```

> [!NOTE]
> I found it helps to make my own self-doubt clear: the AI is very eager to please, so if it thinks you want to do something stupid, it will often procede without objection. Framing your prompts as possibilities seems to make the AI less rash and more skeptical of your ideas.

### Better models?

Again, my project was done mostly with Sonnet 4.6 (medium effort, 200k context) rather than the supposedly smarter Opus, and I'm sure my prompting strategy and context management was far from optimal. At almost every point, though, Sonnet exceeded my expectations and was more than adequate for this project. In fact, I really have only two major complaints about my experience:

- **Speed**: Most small task prompts take under a minute or two, but then the larger task prompts often take five to ten minutes (medium effort seems to rarely work longer than ten minutes for any single prompt). If the models and agents could get the same results, say, twice as fast, my project would have likely been finished a good 20-30% sooner.
- **Cost / token limits**: For the first week, I somehow managed to narrowly stay within the Claude 5 hour token window, but by week two, my prompting got a bit more aggressive, so I'd often have to put my pencil down after two or three hours. Subscribing to OpenAI mostly fixed this, as then I could switch back and forth. Still, fretting about token limits adds a mental tax on top of the prompting, manual testing, feature planning, and code reviewing.

Even if the models don't get any smarter, there's clearly a lot more we could get out of the existing ones with just better harnesses, better utilization strategies, and better ways to communicate with them. In fact, offered the choice, I'd probably take faster and cheaper models with the current levels of intelligence rather than smarter models.

All that said, my glowing experiences so far have been in small, highly disposable, mostly CRUD side projects with 1k to 15k SLOC. In large, quality-sensitive projects, there are surely various ways to use current AI assists effectively, but I'd be much more skeptical about their generated code, particularly in matters of security, performance, and reliability.

## Project details

This is actually my 4th (5th?) iteration of a Japanese vocabulary program that I've made over the last few years, as my theory of optimal drilling practices has shifted a number times. Features include:

- store and track vocabulary words, including drill stats
  - automatically associate words and kanji with readings and definitions (using an included Japanese-English dictionary)
  - extract Japanese words from text (using a morphological analyzer library called [kagome](https://github.com/ikawaha/kagome)) 
  - a lexicon page for browsing and editing the words
- an activity page that displays a calendar of recent user activity
- a drill page with multiple drilling modes
- store and track stories (where "story" = any piece of Japanese text)
  - generate sentence-by-sentence translations of the stories using a user-provided API key (currently supports OpenAI, Anthropic, Google, Mistal, or GLM)
- a tutor chatbot (using a user-provided API key) with optional speech-to-text input and several modes, including conversation and translation practice
- text-to-speech reading of words and stories using VoiceVox (or the browser's build-int TTS as fallback)

All of this is currently working with seemingly no major bugs, though only by actually using the thing for a few months will I be able to work out design issues and other kinks before calling it done.



features are for the most part naturally orthogonal 

### Examples of incremental prompting




incremental prompting strategy
    works really well for small tweaks / additions / removals when it has a clear context to immitate, e.g. established code structure, data model, UI patterns

    examples prompts and problems I encountered

    did not use skills nor mcp

    just a simpel AGENTS.md, and after a few days, I learned to have the AI update AGENTS.md

    rarely looked at the details of the code beyond:
        what files are touched
        how much code added / removed in each file
        function names
        rarely looked at actual function body code
        close eye on data types and database tables

    manually tweaked html and css, but often let the AI fix those issues:
        sometimes silly, but often paid off because AI would remember to update additional related sites of code that escaped my mind
        also often just faster for AI to search through the code than for me to remember the name I'm looking for and actually navigating to it
            only slow down is the wait time for the AI itself and the effort to type the prompts
    
    every few days told AI to look for cleanup opportunitites: dead code removal, consolidation, add / remove tests


audit of the code:
    after 3 weeks, I actually read through most of the backened and frontend code (but ignored the tests)
    extremely boring (in good way) Go backened routes and database code
    the frontend javascript is very well written at a per-function level
        major flaw is just the nature of vanilla jS frontend: tricky per page state management
            but the fix for that is something like React or maybe something like DataStar

## prompting

the way I've used it as a pair partner: my partner writes all of the code while I assess the work, plan, and determine every next step


it's fun...but it's still work


tricks to break down to steps:
    - ask to analyze for possible refactors one by one: possible dead code, possible move code to different files, possible add tests, possible remove tests, etc.
    - stub out UI of a feature first with dummy data before asking for working implementation

ask the bot to help you develop the right prompt

if the bot can't do the task reliably, maybe the prompt needs clarification / more detail / more examples, or maybe just tell it explicitly to double check its work
    - or set up an agentic loop where a supervisor instructs one subagent to check the work of another
    - note of course that rechecking schemes mean more tokens, thus more time and cost

each additional requirement distracts from others
    "scream when you hear today's secret word"
    humans can do short-term temporary learning, but the AIs currently do not update their training when used

## Ways to program with AI other than wholesale vibing

    code stub completion
        [shadow code](https://www.reddit.com/r/theprimeagen/comments/1r22vq1/i_hate_vibe_coding_so_i_built_a_better/)
            sort of a return to co-pilot autocomplete...but more control?
    rubber ducking
        rubber duck that talks back
    discovering libraries/frameworks/tools
    ask questions / troubleshoot api/framework/tools
    semantic search
    answer questions about the code
    explain error messages
    diagnose configuration / installation issues
    genrerate dummy data / example data
    generating sql
    generating regex
    generating html/css
    mocking UI
        (wait what did I mean by "mocking"? prototyping?) 
            maybe just in that it doenn't necessarily do real functionality, i.e. dummy buttons, dummy data
    prototyping
    debugging
    find redundancies
    code review
    explain its own generated code
        (second pass after prompt to generate code)
        with subagents, wouldn't neccessarily have to worry about context pollution doing this in a single prompt
    refactoring
        renames?
        splitting files
        splitting large functions
        find functions to inline
        find functions to nest
    generating tests
    AI reviews your PR / diff for bugs
    managing context
    check comments match the code
    generate / update docs / readmes
    generate PR descriptions


## changing nature of code itself?

less need for abstraction building?:
    can shovel around masses of very boring, plain html / css / vanilla frontend js / Go backend routes and db functions / sql
        any feature you build for the web is splayed across this chain, so natural that people spent 25 years trying to abstract it away
            25 years of backend and frontend web frameworks attempting to abstract away this annoying separating

still need good data design and data flow

to the extent AIs write good code, it is good in the same way human code is good
    they are not constructing tangled webs of logic beyond human understanding
        ...or to the extent they do this, they harm their future selves just like when humans write overly complicated code

## quality

using AI doesn't mean you have to give up control, nor do standards of quality have to change
    standards of what constitutes good code doesn't seem really different
        - to extent AIs can write good code, they do so in the way skilled humans do
        - exception would be better at shoveling code around (see arg about html/css)
        - what AI lacks in true expertise, it often compensates with thorougness

    "AI is very good at writing code and dangerously mediocre at building software." -- skooks twitter 3/30/26
        https://x.com/skooookum/status/2038775815190528419?s=20


responsible use of AI
    DO NOT POLLUTE
    maybe now an even higher obligation to vet your output, to increase the signal-to-noise ratio in the world

https://htmx.org/essays/yes-and/

some people are being reckless with these tools and polluting the world with noise
    ...but you don't have to do that!
    find a project task where you can start small and cautious
        you'll begin to get a feel for what the bots can do reliably enough for your purposes

## AI psychosis


do I want to try using AI for everything now?
    form of AI psychosis: like Homer solving every minor inconvenience by shooting everything with a gun

like a reverse of the twilight zone broken glasses
    looking at list of ideated side projects, "there's time now! I can actually do all this"

AI often crazy good at anticipating my next desire. The chatbots always end with a suggested follow up question, and very often it's exactly my next question, or sometimes a good question i didn't think to ask. So often I'm just responding "Tell me more" [Martin play My Dinner with Andre]

feels like when getting the internet or actually broadband for the first time: 'welp, boredom is the one thing I'm never going to experience again' (culred monkey paw: didn't anticipate the downside of that)

now more of a code critic than a code writer. Like the lazy forman that watches others do the work
    that said, it's fun...but it still feels like work





## Getting started tips







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

/add
    - can get more effiicent token usage to manually add relevant files near start of chat
    - for files you always use, add them to agents.md, or list them in another md file and first thing instruct your bot to /add those files
/drop



the rubber duck that talks back (and even writes your code)

prompting assistant:
    chatbot helps you develop / refine your prompts to be passed on to the agent doing the actual coding
    prompter bot and coder bot
        prompter clobbers file @next_instruction.md with next prompt for codebot
        codebot always just prompted with "Follow the prompt in @next_instrucation.md"
        what about output from codebot? promptbot needs to see some of that to help write good prompts

        promptbot could also have standing orders to recognize when context should be cleared and commits should be made, so it will suggest commits and commit messages and then can make commits for you

other people characterize using AI to code as being a project manager







one Claude session, no worktrees: no multi-agent workflow, and don't prompt for sub-agents (though I think Claude thinking sometimes automatically delegates to subagents on its own accord)

AIs are superhuman at breadth, diligence, and speed of synthesis and analysis
    - what their synthesis and analysis lacks is taste, judgement, creativity, and insight




managing the context; keeping agents.md up to date

pragmatic engineer podcast with openclaw createor:
    disses vibe coding and gastown approach
    recommends codex


AI helps me cover breadth, which naturally sacrifices depth
    - certainly won't learn details the same as if I had to struggle with them myself
    - handling details for me is largely the point of using AI
    - annoying when other people shove slop on you where they clearly haven't bothered with details



reservations:
    a world where everything is just tied together by AI instead of well-defined, relaible software with reliable APIs feels like repeating mistake of Unix tying everything together with text