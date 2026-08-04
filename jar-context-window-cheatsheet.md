# The Context Window Cheatsheet

*You commented "jar" so here it is. From @theprocrastihacker*

---

## The short version

Your AI has a memory jar. It is a fixed size. Everything you say goes in it.

When the jar fills up, the oldest stuff falls out the bottom. Usually that is the instructions you
gave at the start.

So it did not get dumber. It just cannot see the beginning of your chat anymore.

---

## What is actually taking up space

All of this fights for the same jar:

| What goes in | How much space |
|---|---|
| Your custom instructions | small, but always there |
| Your first message | small |
| Any file you pasted | usually the biggest problem |
| Every reply the AI gave you | grows fast |
| Error logs and search results | huge, and mostly junk |
| Your latest question | small |

Here is the part people miss. The AI re reads the whole jar every single time you send a message.
That is why long chats also get slower and cost more, not just worse.

---

## 6 things that fix it

### 1. Just start a new chat
Boring advice. Still the best thing on this list by a mile.

If the chat has gone bad, nothing you type will fix it. The bad stuff is already in the jar. Start
fresh.

### 2. Stop pasting whole files

Paste the 20 lines that matter, not the whole 2000 line file.

One giant error log can eat half your jar and push out the instructions you actually cared about.

Bad: paste the entire stack trace
Good: paste the 5 lines around the actual error

### 3. Put your rules in a file, not in the chat

Chat messages fall out of the jar. Files get loaded fresh every time.

- Claude Code: use `CLAUDE.md` (keep it under 200 lines, longer files actually get followed less)
- Cursor: use `.cursorrules`
- ChatGPT: use Custom Instructions or Project instructions

### 4. Ask for a summary, then restart

When the chat is going well but starting to slip, do this:

```
Summarise everything we decided so far as a bullet list.
Include the decisions, the rules, and what is still left to do.
```

Copy that. Open a new chat. Paste it as your first message.

You keep the useful stuff and throw away the noise.

### 5. Do your research somewhere else

If a task means reading a lot of stuff you will never look at again, do it in a separate chat and
bring back only the answer.

In Claude Code that is a subagent. It fills up its own jar instead of yours.

### 6. Learn the warning signs

Your jar is full when the AI:

- repeats something it already said
- ignores a rule you gave earlier
- forgets a file you pasted
- argues against something you both already agreed on

Do not try to correct it. Correcting it just adds more to the jar. Restart.

---

## The thing nobody tells you

Bigger context windows did not fix this problem. They just made the jar bigger.

AI models still get less reliable as the jar fills up, even before it is completely full. Stuff in
the middle of a long chat gets ignored more than stuff at the start or end. So a huge context
window does not mean the AI actually remembers all of it properly.

The skill is not finding the AI with the biggest jar. It is keeping your jar clean.

---

## Save this bit

```
CHAT GOING BAD?

repeating itself        -> start a new chat
ignoring your rules     -> move rules into a file, restart
forgot your file        -> paste only the part that matters
chat is good but slipping -> ask for a summary, restart with it
about to paste a log    -> paste 20 lines, not 2000
```

---

*More stuff like this at @theprocrastihacker. Daily tech and AI, minus the jargon.*
