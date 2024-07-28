+++
title = 'Blogging Github Style'
date = 2024-07-27T13:00:00-05:00
draft = false
tags = ["general", "hugo", "github", "blog"]
summary = "may as well"
+++
# Intro

I've decided to take advantage of the github pages hosting once again and fire up a blog.

What inspired me? Honestly, seeing the [github style theme](https://github.com/MeiK2333/github-style) for [hugo](https://gohugo.io/). It's pretty clever. What better way to have a github hosted pages blog than to use a style that imitate github?

There is the obvious confusing that could happen, like folks not appreciating the brilliant theme and thinking that I was too lazy to do *anything* at all, and just write markdown files. Of course, in a round about way that's sort of what's happening, but a bit of configuration and leveraging hugo and the theme.

My purpose here is to post snippets of things I learn as quickly as possible, so don't expect long prose.

# A Hugo Shortcode for Callouts

Oh, my one significant enhancement to the theme was adding support for callouts since Hugo's markdown is not quite fully github markdown 1:1. In github markdown, you can do a callout in markdown like this:

```markdown
some content here...

> [!NOTE]
> your note content goes here!

...
```

So I decided to add my own callout support with a Hugo shortcode ([see code here](https://github.com/swilcox/swilcox.github.io)). So for callouts in my markdown, I have to write them like this:

```markdown
{{</* callout note */>}}
your note content goes here!
{{</* /callout */>}}
```

which renders as:
{{< callout note >}}
your note content goes here!
{{< /callout >}}

A lot of the magic is via css, in particular some shenanigans with `::before` to inject the appropriate `svg` file. I only built support for `note`, `important`, and `warning` into the css. But new types could be added with new styles and new svg files. One additional I did add, though, was renaming of the title of the callout. There's an optional 2nd parameter of the callout shortcode, so:

```markdown
{{</* callout warning 경고문 */>}}
My ability to read 한글 is limited!
{{</* /callout */>}}
```

{{< callout warning 경고문 >}}
My ability to read 한글 is limited!
{{< /callout >}}

So there ya go.

