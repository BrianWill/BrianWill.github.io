# How to Vibe Code (Carefully)

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

Over the next couple weeks, I alternated between Claude (CLI, almost all Sonnet 4.6 medium) and Codex (desktop app, almost all GPT-5.4 medium), switching to the other every time I ran out of my 5-hour-window allotted tokens.

> [!NOTE]
> Gemini has proven way less capable for coding (stemming from its poor use of tools, perhaps), but I occasionally find it useful for word wrangling and brain storming tasks. I'm waiting to see if they can improve it reasonably soon, but I might just keep the subscription anyway for the 5TB of Google Drive storage.

My verdict? **Coding is not yet, as some have claimed, a "solved" problem, but I can now see a future where I no longer write first draft code by hand, and it's coming probably sooner than expected.**

I consider this still an 'AI moderate' position, as you get overstated claims in both directions: opponents seem overly pessimistic of the current capabilities and the likely eventual future capabilities; proponents seem overly enthusiastic (to put it mildly).

In their recent podcast series, *Wading Through AI*, Casey Muratori and Demetri Spanos agree that the best current AIs are good for writing programs up to only a few thousand lines of code. From my recent experiences, this seems overly conservative, though we may be imagining different degrees of AI autonomy: "a few thousand lines" might be correct for one-shot vibe coding, but I've been having a lot of success with an incremental and heavily guided prompting strategy.

In general, I found the AI followed my directions very faithfully more than ~97% of the time and produced results I could accept without change more than ~95% of the time. Most of my prompts added or edited a few dozen to few hundred lines, a minority added or edited several hundred lines, and a few outliers added or edited one to two thousand lines.

### Micro level quality

On a micro level, the code has been consistently and impeccably boring and straightforward, with very sensible function decomposition, good naming sense, and a good eye for many (if not all) details. Over a month of prompting, I've barely seen any function logic bugs that didn't stem from under-specified prompting or requirements volatility (along the way, I've introduced many unplanned features and heavily redesigned other features, resulting in some serious code churn). 

By week two, I stopped skimming the generated function bodies of each prompt and just noted the function signatures. By week three, I was mostly watching which files got touched by each prompt and by how much. 'Was the expected amount of code added or removed in the right places? If so, then the code is probably good enough.' 

### Macro level quality

On a macro level, the key aspect of code I don't entrust to the AI is data design. Not that the AI's own data design ideas are necessarily way off the mark, but this is the aspect of code where the AI's proclivity to prefer adding over deleting is most dangerous. Keeping a tight hand on the data design is also the simplest concrete way for me to direct the scope and flow of the program features: there's only so far afield the AI might venture if it has to play within my data sandbox.

Aside from data design, it also seems critical for the human to exercise tight control over the tech stack and dependencies. At no point in my project did the AI attempt to sneak in new dependencies on its own accord, but this probably stemmed from my incremental prompting strategy and my having established a clear foundation early on. I imagine the AI is not so restrained in one-shot projects.

Another maybe crucial practice is to do periodic cleanup. Fortunately, this can be largely offloaded to the AI as well. For example:

```
Prompt: Review the backend code for dead code and redundancies. If issues are found, suggest some code deletions and consolidations.
```

```
Prompt: Review the backend test coverage for missing and outdated tests. Suggest additions and removals.
```

```
Prompt: File foo might be getting overly large. Suggest a way to reorganize the code into new or existing files.
```

> [!NOTE]
> I found it helps to make my own self-doubt clear in the prompts: the AI is very eager to please, so if it thinks you want to do something stupid, it will often procede without objection. Framing your prompts as possibilities seems to make the AI more opinionated and potentially skeptical of your ideas.

### Better models?

My project was done mostly with Sonnet 4.6 (medium effort, 200k context) rather than the supposedly smarter Opus. I'm sure my prompting strategy and context management were far from optimal, however at almost every point, Sonnet exceeded my expectations and was more than adequate for this project. In fact, I really have only two major complaints:

- **Speed**: Most small-task prompts take under a minute or two, but then the larger-task prompts often take five to ten minutes (medium effort seems to rarely work longer than ten minutes for any single prompt). If the models and agents could get the same results, say, twice as fast, my project would have likely been finished a good 20-30% sooner.
- **Cost / token limits**: For the first week, I somehow managed to consistently stay narrowly within the Claude 5 hour token limits, but by week two, my prompting got a bit more aggressive, so I'd often have to put my pencil down after two or three hours. Subscribing to OpenAI mostly fixed this as then I could switch back and forth. Still, fretting about token limits adds a mental tax on top of the prompting, manual testing, feature planning, and code reviewing.

Even if the models don't get any smarter, there's clearly a lot more we could get out of the existing ones with just better harnesses, better utilization strategies, and better ways to communicate with them. Offered the choice, I'm quite sure I'd take a faster and cheaper model with the Sonnet 4.6 level of intelligence over a smarter model.

All this said, my glowing experiences so far have been in small, highly disposable, mostly CRUD side projects with 1k to 15k SLOC. In large, quality-sensitive projects, there are surely various ways to use current AI assists effectively, but I'd be much more skeptical about the code they generate, particularly in regards to security, performance, and reliability.

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
    - the chatbot prompts can incoporate your tracked vocabulary *(not yet implemented)*
- text-to-speech reading of words and stories using VoiceVox (or the browser's build-int TTS as fallback)

All of this is currently working with seemingly no major bugs. Still, only by actually using the thing for a few months will I be able to finalize the design and work out the kinks before calling it done.

> [!WARNING]
> The AI tutor chatbot in particular is half-baked design-wise. I need to spend some time refining the prompts and still need to implement integration of the user's tracked vocabulary.

As for the code, the program is implemented as a single-user, locally-served web app: 

    - Go backend using Chi routing
    - SQLite DB
    - kagome (Go library for Japanese morphological analysis)
    - vanilla JS frontend
    - ES6 modules

> [!NOTE]
> I originally used [Wails v3](https://wails.io/) (a web view library for Go applications) so the app could run "standalone" version, but its zoom scaling behaviour was borking the layout, so I reverted the app to simply run in just the user's browser.

It should be acknowledged that this kind of app is very low stakes in terms of performance and security; nor does the app do anything sophisticated algorithmically, as it's mostly CRUD, which lends itself to boilerplate, orthogonal code. Hence, this is not the most demanding test case for AI's reasoning capabilities.

On the other hand, most every feature in a web app naturally gets split into several pieces, *i.e.*:

    - backend route handling
    - backend database queries
    - frontend logic and state management in Javascript
    - frontend HTML
    - frontend CSS

With few exceptions, both Claude and Codex were remarkably good at accounting for and coordinating all of these pieces for every requested change, big or small.

I was also surprised how well the AI agents could produce decent UI designs despite me not using a proper browser testing harness (meaning they were shoveling around HTML and CSS without a way to verify their changes). In fact, though I often made small HTML and CSS tweaks by hand, I eventually found the AI could do even these small changes quicker and more reliably: the AI can often edit the relevant code faster than I can find it, and the AI would often remember to account for the rest of the frontend in ways that would often slip my mind (such as remembering to update CSS class name strings in the Javascript).

> [!NOTE]
> Making small, trivial tweaks through prompting instead of hand edits might seem wasteful and a dubious boost of productivity, but looking forward a few years, if the models and agents can do the same work multiple times faster and cheaper, then the advantage will become unambiguous. If we also assume a future where quick prompting through voice and pointing with the cursor becomes the norm, then there will be no comparison.

At the three week mark, I wasn't quite done with every feature, but I decided to manually audit the non-test code by reading every line of every function:

- *Backend*: The backend Go route handlers and database queries were, as I said up front, impeccably boring (complementary) and straightforward. In the ~8k SLOC, I found barely a single thing worth manually tweaking.
- *Frontend*: On an individual function level, there was also very little to complain about in the ~8k SLOC of Javascript. At a macro level, however, the *ad hoc* state management patterns on some pages were a little concerning, but I think this is just the nature of vanilla frontend Javascript.

On both fronts, I suspect my habit of running cleanup prompts every few days payed off by excising dead code and bad structure. Without this periodic cleanup, would the AI have gotten more and more confused about proper style and intent as the codebase collected junk? I'm not sure, as it's not clear to me how much the AIs cue off the existing code.

> [!NOTE]
> Interestingly, neither the Go or Javascript code contained any methods or other signs of object-oriented design. I didn't explicitly express this preference, but perhaps my early prompts steered the AI towards a certain style?

As for file structure, I normally tolerate large individual source files, but for this project I generally told the AI to split files larger than several hundred lines, e.g. file `foo` becomes `foo_x` and `foo_y`. You can just ask the AI to suggest a logical split, and usually it can come up with something.

In theory, the AI will have an easier time navigating smaller files, as it can help avoid context pollution. Does this theory actually pay off? Or like a lot of prompting theory, is it just superstition? I'm not sure, but I decided I like the smaller files anyway: as long as I don't have to do the refactoring and the files are meaningfully named, I find that an upward bound of several hundred lines per file is nice when navigating code I haven't written.

### Example prompts

To give you an idea of the incremental prompting style, here are some examples from my session logs. Most are requests for single feature additions, tweaks, or bug fixes:

```
The AI provider/modal selection and reroll buttons should [sic missing word] at top of the right sidebar. The reroll buttons should be "Generate alternate meaning", "Generate alternate examples"
```

```
If the user has no AI keys available, this should be indicated with a message and instructions in the area that normally says "Click a generate button to see alternatives"
```

```
In the lexicon word list, the buttons revelead by hover are shifting layout a bit. They should probably be visiblity hidden instead of display none when not shown.
```

```
In the lexicon, each word should have a play button next to it that plays audio saying the word using the browser's TTS.
```

```
The settings modal should be much wider (take up most of the viewport), and the content should be split into two columns: the drill defaults on teh left; the tts voices and voicevox settings on the right
```

A minority of prompts ask for several things at once and ask the AI to contribute more design:


```
The top right icon in the header should be a link to new "welcome.html". The welcome page includes the header with navlinks like the lexicon, drill, and activity pages. The content of the page explains the functionality of the app. Make the text layout interesting (like a magazine or borchure [sic]), and find some interesting Japanese-themed images to accompany the text.
```

```
In the regular (non matching pairs) drill mode, let's make the last word info better match the word info in the matching pairs mode:

1) Rows each with two columns: content of the left column is left aligned; content of the right column is right aligned
2) first row is the word in left column (text aligned to bottom of its column); right column is the image (if any)
3) second row is the meaning in left column; pos in right column
4) third row is reading in the left column; kanji with meanings in the right column
5) fourth row is the japanese example sentence in the left column; english example sentence in the right
```

```
the regular (non matching pairs) drill mode is now broken in a few ways:

non-matching pairs mode not displaying word info correctly 

non-matching pairs mode not persisting state correctly between reloads

the "Next" button is not being displayed when "skip answer reveal" is disabled

These issues may all stem from the matching pairs mode interfering with the non-matching pairs mode. As their logic is quite different, make sure they don't erroneously share code paths and state.
```
        
Notice that I mostly stuck to full sentences and a polite but direct tone, as if the bot were a pair programming partner who does all the driving while I navigate. I'm sure this was generally unnecessary effort on my part, but I stuck with it so as to not introduce a new variable into my prompting experiments.

Only in the fourth week did I begin to use planning mode more extensively. I'm not sure it helped the bot produce better results, but I do appreciate getting more details about what the bot is going to do before it does it.

Here are a few agentic coding tricks I picked up:

- Instead of updating your AGENTS.md file manually, tell the bot to do it.
- Tell the bot to stub out a new feature's UI before implementing the functionality, *e.g.* add buttons that don't yet do anything. This is one natural way to break down a new feature into smaller increments (arguably no different from manual coding).
- Have the bot generate dummy data and example data.
- Ask the bot (or another AI) to refine your prompt and tell it to ask you questions before starts implemention.

## Will AI change the nature of good code?

less need for abstraction building?:
    can shovel around masses of very boring, plain html / css / vanilla frontend js / Go backend routes and db functions / sql
        any feature you build for the web is splayed across this chain, so natural that people spent 25 years trying to abstract it away
            25 years of backend and frontend web frameworks attempting to abstract away this annoying separating

can we finally replace HTML and CSS with something better if we no longer have to worry about the human learning transition cost? the AI can do the learning, and humans can be kept in the loop by having the AI explain details of the new thing as needed

maybe easier to make new tools and get AIs to use them...but what will stand out and get traction it's lost in glut?

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

rubber ducking
        rubber duck that talks back

it's fun...but it's still work

do I want to try using AI for everything now?
    form of AI psychosis: like Homer solving every minor inconvenience by shooting everything with a gun

like a reverse of the twilight zone broken glasses
    looking at list of ideated side projects, "there's time now! I can actually do all this"

AI often crazy good at anticipating my next desire. The chatbots always end with a suggested follow up question, and very often it's exactly my next question, or sometimes a good question i didn't think to ask. So often I'm just responding "Tell me more" [Martin play My Dinner with Andre]

feels like when getting the internet or actually broadband for the first time: 'welp, boredom is the one thing I'm never going to experience again' (culred monkey paw: didn't anticipate the downside of that)

now more of a code critic than a code writer. Like the lazy forman that watches others do the work
    that said, it's fun...but it still feels like work


## Will AI make software better or worse?

the human is still responsible for quality...but will AI just encourage more people to shirk that responsibility and pollute the world with bad quality software?

will AI just encourage people to generate slop and neglect details? or will it open up opportunities for improvements, e.g. we can introduce better foundation layers if the AI is the one dealing with the details: humans are attached to what's familiar even if it's bad, but AIs could eat the pain of dealing with the unfamiliar, e.g. do we need ugly shell languages anymore if an AI can learn newer, clearner alternatives? do we need syntactically arcane language features if the code can just cope with verbosity?

might break assumption that pretty UI signals good software










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













```
Instead of a separate button to play, clicking the word itself should play the audio. Signal this with the pointer finger cursor on hover and a slight color change
```


```
The settings modal should have an option to pick the TTS voices for japanese and English
```

```
When the user picks a voice in the settings pulldowns, they should here a short example of the voice.
```

```
OK good. Now we have a bit of a UI design problem: the user should be able to preview the Japanese sentence in the edit words modal, but currently clicking the sentence is for editing. I suppose there could be a little play button next to the japanese flag. Any suggestions?
```

```
Different words seem to sit at a bit different vertical positions within the main-area. Specifically, it seems like katakana words sit a bit higher than words with kanji
```


```
When the user generates audio from the settings menu, the new audio should immediately be "live" on the page, i.e. anything that triggers playing audio should play the latest generated audio for it.
```





```
In the settings, under drill defaults, add a number spinner (styled like the others) for 'New word target drills' (the initial value of the target drill count for a newly added word). Like other user settings, this should be persisted. The initial default value is 8.
```



each additional requirement distracts from others
    "scream when you hear today's secret word"
    humans can do short-term temporary learning, but the AIs currently do not update their training when used



## Ways to use AI agents other than direct feature implementation and bug fixing

Even if you're not ready to let an AI to directly generate new code or fix existing code, there are dozens of other ways AI agents might be helpful in your greenfield or brownfield projects:

    discovering libraries/frameworks/tools
    ask questions / troubleshoot api/framework/tools
    semantic search
    answer questions about the code
    explain error messages
    diagnose configuration / installation issues
    generating sql
    generating regex
    generating html/css
    mocking UI
        (wait what did I mean by "mocking"? prototyping?) 
            maybe just in that it doenn't necessarily do real functionality, i.e. dummy buttons, dummy data
    find redundancies
    code review
    refactoring
        renames?
        splitting files
        splitting large functions
        find functions to inline
        find functions to nest
    generating tests
    AI reviews your PR / diff for bugs
    check comments match the code
    generate / update docs / readmes
    generate PR descriptions


code stub completion
        [shadow code](https://www.reddit.com/r/theprimeagen/comments/1r22vq1/i_hate_vibe_coding_so_i_built_a_better/)
            sort of a return to co-pilot autocomplete...but more control?
managing context

explain its own generated code
        (second pass after prompt to generate code)
        with subagents, wouldn't neccessarily have to worry about context pollution doing this in a single prompt

genrerate dummy data / example data

debugging

prototyping


## Getting started tips