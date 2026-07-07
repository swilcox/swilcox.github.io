+++
title = 'Claude Fable 5 and zgigye'
date = 2026-07-07T12:00:00-05:00
draft = false
summary = 'Claude Fable 5 wrote most of a Z-machine in Zig, then helped turn it into a terminal, web, and WebAssembly project.'
tags = ['ai', 'claude', 'zig', 'z-machine', 'wasm']
+++
# Claude Fable 5 and `zgigye`

## Overview

I've had some pretty successful experiments with Claude over the last couple of years. Sometimes it's been small stuff: tests, project scaffolding, or helping me remember the shape of a library. Sometimes it's been more ambitious, but still in the realm of "this gave me a good start and then I took over."

My recent experiment with Claude Fable 5 and [zgigye](https://github.com/swilcox/zgigye) feels like something different.

`zgigye` is a [Z-machine](https://en.wikipedia.org/wiki/Z-machine) interpreter written in [Zig](https://ziglang.org/). The short version is that it runs old Infocom-style interactive fiction story files, currently targeting version 3 `.z3` stories. The name is a little joke: **기계** (*gigye*) is Korean for "machine", so `zgigye` is "z-machine" with a little bit of Korean mixed in.

The remarkable part is not just that Claude helped with it. The remarkable part is how much of it it got right so quickly.

I had some useful context going into this because I had tried roughly the same experiment back in November. At that point I asked Codex, Gemini, and Grok to write a Z-machine interpreter in Python. Python should have been the easier target language. But none of them got to anything that could run even a simple no-input test story. Some attempts could not even get a story properly loaded into memory.

So this was not my first "can an AI write a Z-machine?" experiment. The previous answer had been: not really.

## The One Shot That Wasn't Supposed To Work

There are a couple of common claims you see about LLM coding:

* they are not great at Zig.
* they are not great at highly stateful interpreter-type programs.
* they can fake a plausible architecture, but the details fall apart once you start running real inputs.

And honestly, those claims still sound pretty reasonable to me. Zig is still a moving target in places, the documentation and examples are thinner than something like Python or Go, and a Z-machine is not just "parse a file and print some things." It has memory layout rules, packed addresses, opcodes, an object tree, a dictionary, text encoding, a stack, call frames, branching, input handling, and lots of tiny spec details where being almost right is the same as being wrong.

So naturally, I asked Claude Fable 5 to write one.

And it essentially did.

Not perfectly. Not in a way where I would say, "cool, no need to understand this." But the initial implementation was shockingly close to a working Z-machine v3 interpreter. It had the basic module boundaries, the core memory handling, instruction decoding, object/property handling, text decoding, opcode dispatch, and a plain frontend. Very quickly, it was able to run real story files and then pass the `czech.z3` Z-machine emulation checker with:

```text
Passed: 349, Failed: 0
```

That is the part that still feels a little unreal to me. This was not a todo app. It was not "write a CLI that calls an API." It was a niche virtual machine in a language that LLMs are rumored to be bad at writing.

It also makes the contrast with November hard to ignore. In something like 6-8 months, this went from "several models cannot get a Python version far enough to load and run a basic story" to "Claude Fable 5 can produce the bulk of a working Zig implementation and then keep extending it." That is a pretty dramatic shift.

## What It Turned Into

The project did not stop at "prints some Zork text in the terminal."

At this point `zgigye` has three useful ways to run stories:

* a full-screen terminal UI using [libvaxis](https://github.com/rockorager/libvaxis).
* a plain text mode, which also makes tests and piping commands much easier.
* a browser version compiled to WebAssembly, with a static demo available at [swilcox.github.io/zgigye](https://swilcox.github.io/zgigye/).

There is also a demo HTTP frontend that runs one request per turn. That led to one of the more interesting design parts of the project: suspend/resume. A web frontend cannot just block waiting for input the way a terminal program can, so the interpreter can stop at an input instruction, save the mutable machine state, and resume later with the next command.

That same shape made the WebAssembly version much more approachable. The core interpreter is kept free of terminal, file, and network access. Frontends provide input and output through a small interface. So adding the browser version was not a rewrite; it was another frontend around the same machine.

That is exactly the kind of architecture I would have wanted if I had written it all myself. The strange part is how quickly Claude got there and how well it kept extending the design once the constraints were stated clearly.

## It Still Needed Judgment

I don't want to overstate this into "AI wrote a whole project and I just watched."

There was still a lot of steering. There were places where I had to tell it that the structure was wrong, that a detail of the Zig version mattered, or that the core needed to remain independent from the frontend. There were also the usual moments where the generated code was plausible but not quite the code I wanted to own.

But compared to my earlier experiences, the ratio felt different. With some previous AI-assisted projects, the first pass was useful mostly as a starting point. I would get the shape, learn a bit from it, and then replace or heavily edit major pieces.

With `zgigye`, the model seemed able to keep a lot more of the system in its head. It could adjust the interpreter, tests, terminal UI, web session layer, and eventually the WebAssembly build without constantly losing the thread. That matters a lot for this sort of project, because a small local change can have weird consequences elsewhere. If instruction decoding changes, opcode behavior can change. If input handling changes, tests, terminal play, and the web frontend can all be affected.

The most impressive part may not have been the first implementation. It may have been how quickly it could expand the project without collapsing the earlier design.

## Why This Felt Different

I have used AI tools before where the limit shows up pretty quickly. They can write a chunk of code, but they do not really seem to understand the project as a project. You ask for the next feature and suddenly the old design is ignored, or dependencies get mixed into layers where they do not belong, or tests become a performance of testing rather than a useful check.

This was much better than that.

The project now has a pretty clean split:

* core Z-machine logic in reusable modules.
* terminal UI concerns outside the core.
* a session layer for turn-at-a-time frontends.
* debug commands that work through the same UI abstraction.
* integration tests using real story files.
* a WebAssembly target that can run the interpreter entirely in the browser.

Again, not magic. I still had to know enough to evaluate what was happening. In fact, this sort of thing probably makes developer judgment more important rather than less. If you cannot tell when the model is making a bad architectural decision, it can generate a lot of code very quickly in the wrong direction.

But when the direction is good, it is startling.

## Final Thoughts

`zgigye` is probably the most successful AI-assisted coding experiment I've had so far. It took an idea that I would normally file under "interesting, but likely too much effort for a weekend project" and made it feel tractable almost immediately.

The part I keep coming back to is that it crossed several boundaries at once: niche spec, Zig, interpreter, terminal UI, web server, WebAssembly. Any one of those would have been a reasonable place for the model to stumble. Instead, it kept moving.

I'm sure there are still rough edges and probably some bugs hiding in there. It only targets Z-machine version 3 right now, and there are features like in-band save/restore and more advanced screen behavior that are not implemented. But as an experiment in what these tools can now do, it has changed my expectations.

I don't think this means "developers are done" or any of the more breathless versions of this conversation. But I do think it means the line has moved. A project that would have been a long slog became a fast, fascinating collaboration.

And now I can play Mini-Zork in a Zig-written Z-machine that was largely bootstrapped by Claude, in the terminal or in the browser.

That still seems wild.
