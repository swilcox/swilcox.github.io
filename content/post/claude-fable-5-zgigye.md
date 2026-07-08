+++
title = 'Claude Fable 5 and zgigye'
date = 2026-07-07T12:00:00-05:00
draft = false
summary = 'Claude Fable 5 wrote most of a Z-machine in Zig, then helped turn it into a terminal, web, and WebAssembly project.'
tags = ['ai', 'claude', 'zig', 'z-machine', 'wasm']
+++
# Claude Fable 5 and `zgigye`

## Overview

I've had some pretty successful experiments with Claude over the last couple of years. Sometimes it's been small stuff and lately some slightly bigger projects.

My recent experiment with Claude Fable 5 and [zgigye](https://github.com/swilcox/zgigye) feels like something different.

`zgigye` is a [Z-machine](https://en.wikipedia.org/wiki/Z-machine) interpreter written in [Zig](https://ziglang.org/). The short version is that it runs old Infocom-style interactive fiction story files, currently targeting version 3 `.z3` stories. The name is a little joke: **기계** (*gigye*) is Korean for "machine", so `zgigye` is "z-machine" with a little bit of Korean mixed in.

I had some useful context going into this because I had tried roughly the same experiment back in November. At that point I asked Codex, Gemini, and Grok to write a Z-machine interpreter in Python. Python should have been the easier target language. But none of them got to anything that could run even a simple no-input test story.

So this was not my first "can an AI write a Z-machine?" experiment. The previous answer had been: not really.

## The One Shot That Wasn't Supposed To Work

One claim about LLMs/AI coding tools is that they're not particularly good at coding Zig. Zig is still a moving target in places, the documentation and examples are thinner than something like Python or Go, and a Z-machine is not just "parse a file and print some things." It has memory layout rules, packed addresses, opcodes, an object tree, a dictionary, text encoding, a stack, call frames, branching, input handling, and lots of tiny spec details where being almost right is the same as being wrong.

So shockingly, the result was almost perfect. It was so close to perfect that I seriously haven't felt the need to dig deep into the code other than to check the overall layout and organization. The initial implementation was a working Z-machine v3 interpreter. It had the basic module boundaries, the core memory handling, instruction decoding, object/property handling, text decoding, opcode dispatch, and a plain frontend. Very quickly, it was able to run real story files and then pass the `czech.z3` Z-machine emulation checker with:

```text
Passed: 349, Failed: 0
```

That is the part that still feels a little unreal to me. This was not a todo app. It was not "write a CLI that calls an API." It was a niche virtual machine in a language that LLMs are rumored to be bad at writing.

It also makes the contrast with November hard to ignore. In something like 6-8 months, this went from "several models cannot get a Python version far enough to load and run a basic story" to "Claude Fable 5 can produce a working Zig implementation and then keep extending it." That is a dramatic shift.

## What It Turned Into

The project did not stop at "prints some Zork text in the terminal."

At this point `zgigye` has four useful ways to run stories:

* a full-screen terminal UI using [libvaxis](https://github.com/rockorager/libvaxis).
* a plain text mode, which also makes tests and piping commands much easier.
* a browser version compiled to WebAssembly, with a static demo available at [swilcox.github.io/zgigye](https://swilcox.github.io/zgigye/).
* an HTTP frontend and web backend that runs one request per turn which involves saving state and restoring state for each request/response loop.


The core interpreter is kept free of terminal, file, and network access (which is one of the few instructions I had given it when creating the interpreter). Frontends provide input and output through a small interface. So adding the browser version was not a rewrite; it was another frontend around the same machine.

## It Still Needed Judgment

I don't want to overstate this into "AI wrote a whole project and I just watched." But it was some very gentle steering on my part. My initial guidanace on architecture and separation of concerns was important, but this was definitely a whole new level of AI coding because I feel like it really understood the assignment.

With `zgigye`, the model seemed able to keep a lot more of the system in its head. It could adjust the interpreter, tests, terminal UI, web session layer, and eventually the WebAssembly build without constantly losing the thread. That matters a lot for this sort of project, because a small local change can have weird consequences elsewhere. If instruction decoding changes, opcode behavior can change. If input handling changes, tests, terminal play, and the web frontend can all be affected.

## Final Thoughts

`zgigye` is probably the most successful AI-assisted coding experiment I've had so far. It took an idea that I would normally file under "interesting, but likely too much effort for a weekend project" and made it feel tractable almost immediately.

The part I keep coming back to is that it crossed several boundaries at once: niche spec, Zig, interpreter, terminal UI, web server, WebAssembly. Any one of those would have been a reasonable place for the model to stumble. Instead, it kept moving.

I'm sure there are still rough edges and probably some bugs hiding in there. It only targets Z-machine version 3 right now, and there are features like in-band save/restore and more advanced screen behavior that are not implemented. But as an experiment in what these tools can now do, it has changed my expectations.
