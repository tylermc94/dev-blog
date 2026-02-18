+++
title = 'Workshop Assistant Phase One Update'
date = 2026-02-17
draft = false
toc = true
description = "I built a voice-controlled workshop assistant from scratch in 2 weeks. Here are the challenges I faced, what worked, and what I learned about speech recognition on Raspberry Pi."
tags = ["voice-assistant", "raspberry-pi", "python", "speech-recognition", "maker-projects", "workshop-automation"]
categories = ["Projects"]
cover = "/images/workshop-forge-phase1.jpg"
+++

### I. Phase One Complete!

Two months ago, I set myself a goal to waste less time doom scrolling and spend more time being productive. My first big project was a Workshop Assistant app that I now call "The Forge." My goal was to create a voice assistant tailored to my workflow and hobbies. Since I'm building this as an actual tool, as well as using it as a learning opportunity, I broke the project into phases, each one adding on a new concept to figure out. As I wrap up Phase 1, I'm feeling really good about the project as a whole. It's been tough, like learning any new thing, but I feel really good about what I've accomplished so far. 

Even though it's just a basic framework with limited functionality right now, the first time I was in a situation where my hands were busy and I could just say "Hey Forge, set a timer for 5 minutes" and have it actually set the timer and remind me was a really cool moment. Even though I could do the exact same thing with Siri on my phone, the fact that this was something I put together myself (with a little help from Claude), made it the most impressive timer I've used.

---

### Quick Stats

| Metric | Value |
|--------|-------|
| Development Time | ~2 weeks |
| Platform | Raspberry Pi 5 |
| Wake Word Accuracy | ~95% |
| Speech Recognition Accuracy | ~95% |
| Response Time | <3 seconds |
| Total Cost | $0 (used existing hardware) |

---

### II. What I Built

My goals for phase one were relatively simple:

- Wake word detection (“Hey Forge”)
- Speech-to-text with dynamic recording (stops when you stop talking)
- Three working skills: timers, calculator, calendar
- Text-to-speech responses
- All running locally on a Raspberry Pi 5

My goal for phase one was to start with the basic framework. At it's most basic, I need to be able to ask Forge a question and get an answer back. This way as I move onto more complex components, it all runs through the voice pipeline I made here in Phase One. 

---

### III. The Technical Stack 

Since I have an abundance of Raspberry Pi's, my goal was to make this program run on a Pi 5. The Raspberry Pi 5 has plenty of resources to run the basic workshop assistant functionality and overhead for adding more complex features down the road. For audio I/O, my initial plan was to use a Focusrite Scarlett 2i4 that I've had for years collecting dust in the basement. It works perfectly for audio input with an Audiotechnica condenser mic, but I wasn't able to get it to output to any speakers, so I had to also use a separate USB speaker set. At some point in the future I'm planning on exploring other audio devices for this setup.

As for the software, I made some modifications to my original plan, mainly switching my speech-to-text to [`faster-whisper`](https://GitHub.com/SYSTRAN/faster-whisper) for better output than I could get with `vosk`. Since I wanted dynamic recording, where it continues listening until I'm done speaking, `faster-whisper` was the best choice. I also changed to using [`Piper`](https://GitHub.com/rhasspy/piper) for my text-to-speech. I had originally planned to use `pyttsx3`, but before I implemented it I found Piper, which sounds much more realistic and is designed to run on edge devices like a Raspberry Pi. 

I still went with [`Porcupine`](https://picovoice.ai/platform/porcupine/) for wake word detection and it works extremely well! I did run into one issue with it. When setting up porcupine, you record yourself speaking your wake word to train a model. I recorded myself in a quiet room on a high quality mic and used that model. Once I started working with the tool in my noisy basement workshop, I had to fight against the background noise in the room. I'm planning on training a new model, but I want to make sure I'm using the hardware that I plan to stay with before I do that.

### System Architecture
```
┌─────────────────────────────────────────────────────┐
│              Main Loop (Async)                       │
└─────────────────────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
┌──────────────┐  ┌─────────────┐  ┌──────────────┐
│  Wake Word   │  │ Speech-to-  │  │    Intent    │
│  Detection   │→ │    Text     │→ │ Recognition  │
│ (Porcupine)  │  │  (Whisper)  │  │   (Routing)  │
└──────────────┘  └─────────────┘  └──────────────┘
      ↑                   ↑                 │
      │                   │                 │
Scarlett 2i4         Scarlett 2i4          │
  (Card 2)            (Card 2)              │
   Mic Input           Mic Input            │
                                            │
                         ┌──────────────────┼──────────────────┐
                         ▼                  ▼                  ▼
                  ┌───────────┐      ┌────────────┐    ┌──────────┐
                  │   Timer   │      │ Calculator │    │ Calendar │
                  │   Skill   │      │    Skill   │    │   Skill  │
                  └───────────┘      └────────────┘    └──────────┘
                         │                  │                  │
                         └──────────────────┼──────────────────┘
                                            ▼
                                   ┌─────────────────┐
                                   │  Text-to-Speech │
                                   │     (Piper)     │
                                   └─────────────────┘
                                            │
                                            ▼
                                     USB Speakers
                                       (Card 3)
                                    Audio Output
```
---

### IV. The Journey - Key Challenges 

While building everything out, I ran into a few issues. Working with the voice pipeline was more challenging than I expected. The first problem I ran into was dealing with audio routing in linux. I'm so used to Windows, I wanted everything to just be plug and play. Unfortunately, I had to play around with it a bunch to get the app to record from and play to the right devices. As of now, I have to run `aplay-l` and then hard code the card numbers for the speakers and mic in the Forge config. it's not the best solution, but I plan on adding a device selection menu when I get to the GUI phase.

Once I got the devices working and started working with `vosk` for speech to text, I ran into my next major issue. With background noise in the basement and limited context in my short questions, it really struggled to accurately output what I was saying. If I said "What's four plus four," it would usually come back with "whats for plus for". Once that gets passed to the calculator, it would throw an error because it didn't receive any numbers. I played around a lot with the `vosk` configuration, changing how often it sampled my voice during a question, changing mic settings to get rid of background noise, and adding logic to the code to correct specific homophones.
```python
def parse_calc_expression(expression):
    # Fix common speech recognition errors for math
    homophones = {
        'for': 'four',
        'to': 'two',
        'too': 'two',
        'ate': 'eight',
        'won': 'one'
    }
    
    for wrong, right in homophones.items():
        expression = expression.replace(wrong, right)
```
 In the end, I found `faster-whisper` and it worked almost perfectly with little configuration. I scrapped `vosk` and moved the STT to `faster-whisper` and haven't had a problem since.

 One of the key improvements in Phase 1 was implementing dynamic recording that stops when I stop talking, rather than cutting off after a fixed duration.
```python
# Configuration settings for dynamic recording
DYNAMIC_SILENCE_THRESHOLD = 1.5   # Seconds of silence before stopping
DYNAMIC_MAX_DURATION = 30         # Maximum recording time (safety)
DYNAMIC_ENERGY_THRESHOLD = 500    # Audio level to detect speech vs silence

# The recording loop monitors audio energy levels
def transcribe_speech_dynamic():
    audio_buffer = []
    silence_duration = 0
    
    with sd.InputStream(device=AUDIO_INPUT_DEVICE) as stream:
        while True:
            chunk, _ = stream.read(DYNAMIC_CHUNK_SIZE)
            audio_buffer.append(chunk)
            
            # Calculate loudness of this chunk
            energy = np.abs(chunk).mean()
            
            # Track silence duration
            if energy < DYNAMIC_ENERGY_THRESHOLD:
                silence_duration += chunk_duration
            else:
                silence_duration = 0  # Reset if speech detected
            
            # Stop when silence threshold reached
            if silence_duration >= DYNAMIC_SILENCE_THRESHOLD:
                break
    
    # Process complete audio with Whisper
    return transcribe_with_whisper(audio_buffer)
```

This approach allows natural speech patterns - I can take brief pauses to think without the recording cutting off, but it stops quickly once I'm done speaking.

The last major issue I ran into was dynamic recording. In the beginning I had it set up to listen for 5 seconds after the wake word was detected and then parse what was said and pass it to the interpreter. This was fine, but if I talked for longer than 5 seconds, it would cut me off, and if I just had a quick question that only took a second, I would be waiting while it finished recording and then parsed. I wanted to change to a dynamic recording length where it would record until I was done speaking and then parse it and send it to the interpreter. I made this adjustable in config and eventually got to a point that felt really natural. It listens until I'm done speaking, doesn't cut me off, and understands me almost all of the time!

---

### V. What Actually Works Now 

Overall, I think I'm in a really good place with the Forge! It might seem super basic to someone else trying it out (and feel free to try it out, [the repo](https://GitHub.com/tylermc94/workshop-assistant) is public and decently well documented), but I'm really happy with it for my first big coding project. I'm able to set timers, do basic math, and ask for the current date and time. While the feature set is limited, the next steps are to grow it with integrations for AI, home assistant, and whatever else I can think of to make it as useful as possible for me. 

The performance is honestly better than I had hoped for. The wake word is recognized ~95% of the time, even in the noisy basement, and response time after I finish a query is under 3 seconds for the simple skills it has currently. I think as a framework to build onto going forward, I'm really happy with it!

---

### VI. What Doesn't Work Right Now

Since the goal of Phase One was to just have a working voice pipeline and a few basic skills for testing, there are definitely some shortcomings to address later. 
- Timers run completely in the background with no way of modifying them or knowing how much time is left.
- Killing the program takes forever because of the wake word threads needing to time out.
- There's no automatic recovery from crashes. If I just leave it running in the background and it crashes, I don't know it until I try to ask it something and don't get an answer.

I think these are all relatively simple issues that I can fix later on. These aren't blockers for daily use, just polish items for later phases. Overall, there are no major bugs at this point that I've found. I'm sure I'll run into more later, but for now I'm happy with what I have. 

---

### VII. The Development Process 
Since this was my first big coding project, I've had to create my development workflow on the fly. I started out keeping all of my notes and ideas in a Claude chat, asking it for information when I couldn't remember. This was fine, but I eventually moved to Notion to track the overall project and my progress and notes. Keeping everything as organized as possible has been difficult, but well worth it. While writing this blog post, I have Notion open and it's been incredibly easy to look back and reference everything that I've done.

My main goal with the planning of Phase One was to break it down into chunks that were easy to focus on one at a time. This made sure that I stayed on track while working on each component and helped keep my momentum up. Since each component had to be implemented and working before the next one could be started, it meant that I got to build something up, get it working, and then feel good about the accomplishment before moving on to the next piece. This kept my ADHD brain happy and interested so I didn't just give up on the project before it even had a chance.

As my first major coding project, this was my first time really working with GitHub. I've used GitHub in the past to access things made by others and to save my little learning projects, but this project was the first time I had a multi tiered project file to update. I had to learn how gitignore works, and when I adapted the initial voice pipeline to `async` functions to allow timers to run in the background, I learned how branching works. I learned a ton about source control and change tracking in this project and it really wasn't as difficult as I expected.

On the point of the `async` conversion, that wasn't something I had initially planned for. If I set a 10-minute timer, I didn't want the entire system to freeze for 10 minutes waiting for it to finish - I needed to be able to ask other questions while the timer ran.

This required converting the entire codebase from synchronous to asynchronous Python using `async`/`await`. I used Git branching for the first time to create an `async-refactor` branch, which let me make major changes safely without breaking the working code on my main branch. Once I had the async version working, I merged it back.

The key was using `asyncio.create_task()` to spawn background timer tasks:
```python
async def set_timer(duration):
    """Run timer in background while main loop continues"""
    await asyncio.sleep(duration)
    play_alarm_sound()  # Beep when complete

# In main loop - this doesn't block!
asyncio.create_task(set_timer(300))  # 5 minute timer runs in background
```

This was my first real exposure to asynchronous programming, and it turned out to be less intimidating than I expected. The ability to handle multiple concurrent operations (like running timers while listening for new commands) made the whole system feel much more responsive.

---

### VIII. Lessons Learned

The biggest lesson I learned from this was that breaking up a large project into smaller, more digestible chunks really works well for the way I work. There were definitely a few times that I ran into issues and felt the urge to give up, but these were minor roadblocks and once I pushed through a few, I built up momentum and those small issues became less of a problem.

I'm not the most patient person, so the amount of time I had to spend researching different libraries and tools for the voice pipeline, only to later scrap those tools for something else was something I really struggled with the first time it happened. When I realized that `vosk` was just not going to work for me and I would have to rewrite almost the entire speech to text component, I almost gave up. I definitely leaned on Claude AI for those moments, letting it do a bit more of the research and write the initial test code to make sure I wouldn't waste more time on another non-viable tool. I think I let myself get a bit overambitious with the first bit's, like building the whole STT component with `vosk` before making sure it would even work. For the later pieces, like TTS and skills, I wrote minimal testing functions just to make sure it would actually work the way I needed it to, before fully building out the code.

---

### IX. What’s Next - Phase 2 Preview

I've gotten a little distracted from the Forge project, but I'll be getting into Phase Two soon. The main goal of Phase Two is integration with Claude AI. I have two main goals with this integration:

1. Natural language understanding
    - To test the voice pipeline, I built an intent recognition program to look for key phrases in the STT transcription and route the command to the proper skill program. The problem with this is I have to be very specific in my wording of questions. I can say "Hey Forge, what time is it?" and it will tell me the time, but if I say "Hey Forge, what's the time?", it wont know what to do with that. Once I build out the Claude integration, I'll be able to pass unrecognized queries to Claude to handle. So if I ask it "What's the time?" and it doesn't recognize that, Forge will pass that to Claude and Claude will give me the time. This means that when I'm working with epoxy and I need to know the ratio of resin to hardener, rather than saying "What is 25% of 840?" I can say “Hey Forge, I’m mixing epoxy. I need 25% hardener for 840 grams of resin. How much hardener?”

2. Answering complex questions.
    - If I ask Forge something outside of the current skill set, it will just return "Sorry, I didn't understand that." I could keep building out more and more skills to answer any query that I could possibly ask it, but in reality, thats a huge effort. To have a really fluid experience where I never run into a query it doesn't know how to handle, I can pass any unrecognized queries to Claude. I can say "Hey Forge, I'm working with *X Brand* resin and I need 900g total, how much resin and hardener do I need and what is the working time?"

There are a few things I want to implement alongside those features:

1. Budget Tracking
    - Since I'll be using the Claude API, I don't want to accidentally use a ton of tokens and end up with a massive bill at the end of the month. I'll add a token tracker and configurable thresholds to warn me when I've spent half my budget and then shut down the Claude integration once I hit the limit. 

2. Frequent Query Tracking
    - Since every message to Claude costs money, I want to minimize how much I have to use it as much as I can. In my Claude integration, I want to have it track the queries that it handles the most and give me reports so I can find ways to handle some of those queries locally. It will track the intention of what I'm asking, so they don't have to be worded the same to be grouped together, and give me information on the output, ways that I've phrased the questions, and anything else that would be helpful in programming a local skill handler.


Phase One proved to me that this project is doable if I just put in the effort. Along with developing the Forge tool it'self, I've also developed and adapted my workflow and found that programming isn't as difficult and scary as I thought. I'm really excited to keep moving forward with this project and see how it evolves. I've already put the skills I learned with Forge to use in other projects, I built a Discord bot to help schedule D&D games with my group using the programming skills I started here. Follow along on my [Instagram](https://www.instagram.com/elevation1505/) to track my progress and see the other cool stuff I'm working on!

---
