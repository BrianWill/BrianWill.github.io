# Pair Programming with AI

# How to Vibe Code (Carefully)



Prior experience using AI

    semantic search

    scafolding examples for libraries APIs

    questions about APIs

    error message diagnostics

    questions about language syntax / semantics

        arguing with chatgpt about Odin

    generate regex / sql queries

    Japanese translation and grammaticaly analysis of Japanese sentences


More and more people I trust said it's gotten serious

    aaltonen
    https://blog.s-schoener.com/2026-01-14-claude-optimism/


March 2026, way late to the party

Finally got a claude pro ($20) subscription. Happened to be during their promotion that doubled the 5 hour window allowance.

Used claude code for first time

verdict:
    coding is not yet solved, as some have claimed, but I can see the day I stop writing any new code by hand seems to be coming sooner than I expected

    Muratori podcast claim: good for up to a few thousand lines of code seems to undersell the current capabilities
        I probably wouldn't trust today's best AI to write thousand of lines in one shot, but you can guide an AI incrementally to build quite large programs. More importantly, an AI can greatly assist in large code bases, though this requires both programming expertise and some skill in using the AI, e.g. managing its context and tasking it correctly.

    enormous potential with current AI smarts if just got faster and cheaper, better harnesses / tool use and better ways of communicating with the human

My Japanese vocab drilling app.

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

## AI assists other than just writing the code

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