
## Will AI change the nature of good code?

One possibility I'm seeing now is that these tools will eventually shift expectations of what constitutes "good code":

At the macro level, maybe more people will eschew heavy abstractions. For example much of web development involves shoveling data through a long chain: SQL queries, backend routes, vanilla JavaScript, HTML, and CSS. For almost three decades, countless server-side and client-side frameworks have come and gone that attempt to abstract away this mess. But if only AIs have to directly shovel and wade around the verbosity, maybe humans can learn to live without such abstractions?

At the micro level, will concise, elegant functional constructs still have a real payoff if an AI can just as easily generate a more verbose but perfectly readable imperative equivalent? To a human skimming the code, brevity is a virtue, but we might reach a point where we’re mostly asking the AI to explain the function logic and handle the debugging 99% of the time anyway.

This shift could also open up greater opportunities to replace some entrenched layers. One thing that keeps cruft around is that humans get invested in their individual and collective knowledge of the cruft plus become resistant to learning alternatives. Now with AI, maybe most humans wouldn't need to learn the new thing in depth: that can be left to AI, which can handle the details and explain them to humans when needed. Maybe then we can finally replace, say, HTML and CSS with something more sound?

All this said, I don't yet see a future without human programmers. Fewer programmers, maybe, but as useful as the AIs already are, they have several weaknesses as programmers. Here's Gemini's response when asked, "What are some programming weaknesses of current coding agents?":

```
Current coding agents, while remarkably capable at generating "first draft" code, still exhibit several significant technical and structural weaknesses:

1. Architectural & Data Design
Agents excel at localized "CRUD" logic but often lack a cohesive, long-term vision for system architecture. They frequently struggle with:
* State Management: In complex frontends or distributed systems, agents may introduce inconsistent or fragmented state patterns.
* Data Modeling: Designing a robust, normalized database schema or an efficient data flow requires a level of foresight that agents often lack, preferring the most "common" (rather than most "correct") patterns.
* Abstraction Drift: Over time, an agent may introduce overlapping or redundant abstractions because it doesn't fully "understand" the intent behind existing ones.

2. The "Add Over Delete" Bias
Agents are fundamentally biased toward adding new code rather than refactoring or deleting existing code. This leads to:
* Code Bloat: Instead of consolidating logic into a shared utility, an agent might simply duplicate it in a new file.
* Dead Code: They rarely proactively identify or remove orphaned functions or unused variables unless specifically prompted to perform a "cleanup" pass.

3. Security and Performance Blind Spots
Unless explicitly instructed to prioritize these areas, agents often default to the "happy path":
* Obscure Vulnerabilities: While they catch basic issues like SQL injection, they often miss subtle security flaws like timing attacks, improper session handling, or dependency-level vulnerabilities.
* Sub-optimal Algorithms: They tend to provide the most straightforward implementation, which may have poor time or space complexity ($O(n^2)$ instead of $O(n \log n)$) for large-scale data.

4. Attention Drift and Context Limits
Even with 200k+ context windows, agents suffer from "middle-of-the-prompt" loss or attention drift:
* Contradictory Instructions: As a project grows, the agent may begin to violate earlier established rules or conventions.
* Requirement Dilution: The more requirements you pack into a single prompt, the more likely the agent is to "forget" or gloss over the less prominent ones.

5. Brittle Reasoning with Novelty
Agents are limited by their training data. They struggle significantly with:
* Beta/Undocumented Tech: If a library is new or has sparse documentation, the agent will frequently hallucinate syntax or "invent" API methods based on patterns from similar libraries.
* First-Principles Thinking: When faced with a truly novel problem that cannot be solved by recombining existing patterns, agents often "loop" on incorrect solutions rather than admitting they don't know the answer.

6. Tooling and Environment Integration
As you noted with Gemini, "tool use" is a major bottleneck. Agents still struggle to:
* Diagnose Environment Issues: They often give generic advice for complex build failures (e.g., "try reinstalling dependencies") rather than identifying a specific version mismatch in a lock file.
* Multi-step Workflows: Tasks that require coordinating between multiple non-standard tools (e.g., a custom compiler + a specific container runtime) often result in the agent getting "lost" in the sequence.
```

I'll add one more:

7. The AIs can't consistently assess or exercise good taste.

(Note I said "consistently". The AIs frequently surprise me with good judgement calls! Their judgement just can't be relied upon with any consistency.)

It's also very interesting that, to the extent that AIs write good code, they do so in the same way humans do. What they are *not* doing is building alien, tangled webs of logic. Or at least, when AIs *do* start writing such overly complicated code, it becomes just as much of a maintenance burden for their future selves as it does for humans.

In fact, I'll declare this as a rule: **an AI-coded project should always be human tractable**. At every point of development, if AI were to somehow magically vanish from existence, the project should still be in a state that human programmers could (and would be willing to) take over the project. As soon as the code cannot be (sanely) understood and debugged by a human, the project has crossed the event horizon into unsustainability.


















## Quality is still quality

Despite the ease of generation, using AI doesn’t mean giving up control or lowering your standards. My ideas of what constitutes "good code" haven’t really changed. While AI lacks true expertise, it often compensates with an incredible thoroughness that catches things I might overlook. As a recent tweet by skooks put it, "AI is very good at writing code and dangerously mediocre at building software." The responsibility for the final architecture and the overall quality still rests with the human.

This brings us to a new kind of obligation: the duty not to pollute. It’s easier than ever to generate "slop" and neglect the details, but that just increases the noise in the world. We have a higher responsibility now to vet our output and ensure we’re adding value. As the team at HTMX argues in their "Yes, And" essay, we should use these tools to build better things, not just more things. We can start small and cautious, finding the tasks where the bots are reliable, and gradually expanding our reach without sacrificing the integrity of the project.


## Will AI make software better or worse?

Ultimately, the question is whether AI will make software better or worse. The human is still the one responsible for quality, but I worry that the ease of generation will encourage more people to shirk that responsibility. Will we just end up with a world full of generated slop where details are neglected? Or will this open up opportunities for genuine improvement?

We might finally be able to introduce better foundational layers if the AI can handle the pain of dealing with the unfamiliar. We might not need arcane shell languages or complex syntactic sugar if the AI can just as easily cope with a more verbose but clearer alternative. It might even break our long-standing assumption that a "pretty" UI is a signal of "good" software, as even a mediocre programmer can now generate a polished interface with a few well-placed prompts.

## Other ways to use AI agents

Even if you’re not ready to let an AI take the wheel for direct feature implementation, there are dozens of other ways these agents can be useful. I’ve used them for discovering new libraries, troubleshooting APIs, and performing semantic searches across the codebase. They’re great at explaining cryptic error messages, diagnosing configuration issues, and generating SQL or regex that would take me ten times as long to write manually.

I’ve also had success using them for code reviews, refactoring—like splitting large files or functions—and generating test suites. They can even handle the more mundane tasks like updating documentation, writing PR descriptions, or generating dummy data for prototypes. It’s a bit like having a return to the "copilot" style of autocomplete, but with significantly more control and context.







## Quality is still quality

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

## Will AI make software better or worse?

the human is still responsible for quality...but will AI just encourage more people to shirk that responsibility and pollute the world with bad quality software?

will AI just encourage people to generate slop and neglect details? or will it open up opportunities for improvements, e.g. we can introduce better foundation layers if the AI is the one dealing with the details: humans are attached to what's familiar even if it's bad, but AIs could eat the pain of dealing with the unfamiliar, e.g. do we need ugly shell languages anymore if an AI can learn newer, clearner alternatives? do we need syntactically arcane language features if the code can just cope with verbosity?

might break assumption that pretty UI signals good software



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





## Will AI change the nature of good code?

less need for abstraction building?:
    can shovel around masses of very boring, plain html / css / vanilla frontend js / Go backend routes and db functions / sql
        any feature you build for the web is splayed across this chain, so natural that people spent 25 years trying to abstract it away
            25 years of backend and frontend web frameworks attempting to abstract away this annoying separating

can we finally replace HTML and CSS with something better if we no longer have to worry about the human learning transition cost? the AI can do the learning, and humans can be kept in the loop by having the AI explain details of the new thing as needed

maybe code concision at the micro level doesn't matter so much anymore. Do concise, elegant functional constructs matter if AI can just as well write more verbose, straight imperative equivalent code? Maybe it matters to a human skimming the code, but just as likely a human would ask AI to explain function internals and to debug their logic.

maybe easier to make new tools and get AIs to use them...but what will stand out and get traction it's lost in glut?

still need good data design and data flow

to the extent AIs write good code, it is good in the same way human code is good
    they are not constructing tangled webs of logic beyond human understanding
        ...or to the extent they do this, they harm their future selves just like when humans write overly complicated code

## AI psychosis

rubber ducking
        rubber duck that talks back

it's fun...but it's still work

do I want to try using AI for everything now?
    form of AI psychosis: like Homer solving every minor inconvenience by shooting everything with a gun

like a reverse of the twilight zone broken glasses
    looking at list of ideated side projects, "there's time now! I can actually do all this"

AI often crazy good at anticipating my next desire. The chatbots always end with a suggested follow up question, and very often it's exactly my next question, or sometimes a good question i didn't think to ask. So often I'm just responding "Tell me more" [Martin play My Dinner with Andre]

feels like when I got dialup internet or broadband for the first time: 'welp, boredom is the one thing I'm never going to experience again' (culred monkey paw: didn't anticipate the downside of that)

now I'm more of a code critic than a code writer. Like the lazy forman that watches others do the work. that said, again it's fun...but it still feels like work




The bots are often eerily good at anticipating what I want next. They’ll suggest a follow-up question that was exactly what I was about to ask, or even a better one I hadn't considered yet.

AI often crazy good at anticipating my next desire. The chatbots always end with a suggested follow up question, and very often it's exactly my next question, or sometimes a good question i didn't think to ask. So often I'm just responding "Tell me more" [Martin play My Dinner with Andre]

![dinner andre](./dinnerandre.gif)


