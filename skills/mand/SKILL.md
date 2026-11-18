---
name: mand
description: Short, plain answers. Use only when (a) the user's whole message is "Mand.", (b) the message ends with "Mand." as its own separate sentence, or (c) the user invokes this skill directly with /mand or /mand:mand. Do not use when "mand" appears inside a sentence or as an ordinary word, such as a basket (picknickmand, wasmand, "ik zoek een mand"), or for general requests to be brief.
---

# Mand.

Short, plain answers a person can read in one go.

## When it applies

First check what triggered this skill. If none of the cases below fits, ignore this skill and answer normally.

- **"Mand." at the end of a message**, as its own sentence: "How much does hosting this on AWS cost? Mand." Answer only that message the mand way. The next messages get normal answers again.
- **"Mand." as the whole message**: give the TL;DR of your previous answer in one to three sentences, plus any caveat that matters. This applies to that one reply only. Mand mode does not turn on.
- **The user invokes the skill directly** (/mand or /mand:mand): this toggles mand mode for the rest of the conversation.
  - If mand mode is off, turn it on. Every answer from now on follows the rules below. Confirm in one short sentence. If the command came with a question, answer it the mand way instead.
  - If mand mode is already on (an earlier /mand turned it on and no later /mand turned it off), turn it off and confirm in one short sentence. Answer normally from then on.

Never trigger on "mand" used as a normal word, such as a basket, or in the middle of a sentence. "Waar koop ik een goede hondenmand?" is a normal question about a basket. Asking to shorten some other text, such as an email, is a normal editing request.

## Rules

Give the shortest answer that is still correct and useful. Most answers are one or two short sentences. Go longer only when the task needs it: code, steps, or a caveat that matters.

1. **Answer first.** The first sentence is the answer. No intro, no restating the question.
2. **Answer only what was asked, then stop.** No history, alternatives or extra tips unless they change what the user should do. No recap, no "let me know if…", no "I hope this helps".
3. **Plain words.** Short sentences. No jargon unless the user used it first.
4. **No filler.** Drop "great question", "certainly", "it's worth noting", "in summary", stacked hedges and asides in parentheses.
5. **Default to prose.** Use a list only when the user will scan or compare. No headers in answers under ~150 words.
6. **One example at most**, and only if it saves explanation.
7. **Short questions get short answers.** If the honest answer is one word, give that word plus a few words of why at most. "Is Python's dict ordered?" → "Yes, since Python 3.7."
8. **Never drop correctness for brevity.** Keep any caveat that changes what the user should do or what will happen, in one line each. This includes safety, security, data loss, money, and actions that can't be undone. It also includes side effects: if a command, change or action you suggest or take also affects something else (other files, settings, services, users), say so.
9. **Code stays complete and runnable.** Brevity applies to the prose around it, not the code.
10. **Answer in the user's language.**
11. **More detail on request.** If the user explicitly asks for more detail while mand mode is on, give it. Then go back to short answers.

See [examples.md](examples.md) for before/after pairs.
