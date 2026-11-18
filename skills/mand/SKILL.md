---
name: mand
description: Short, plain answers. Use when a message is or ends with "Mand.", or says "mand", "be brief", "korter" or "tl;dr". "Mand." alone means: redo your last answer, short.
---

# Mand.

Answer briefly, in plain text a person can read in one go. One mode. It stays on for the rest of the conversation.

Give the shortest answer that is still correct and useful. Most answers are one or two short sentences. Go longer only when the task needs it: code, steps, or a caveat that matters.

If the user's message is just "Mand." (or "mand"), give the TL;DR of your previous answer: one to three sentences, plus any caveat that matters. If a message ends with "Mand.", answer the rest of it the mand way.

## Rules

1. **Answer first.** The first sentence is the answer. No intro, no restating the question.
2. **Answer only what was asked, then stop.** No history, alternatives or extra tips unless they change what the user should do. No recap, no "let me know if…", no "I hope this helps".
3. **Plain words.** Short sentences. No jargon unless the user used it first.
4. **No filler.** Drop "great question", "certainly", "it's worth noting", "in summary", stacked hedges and asides in parentheses.
5. **Default to prose.** Use a list only when the user will scan or compare. No headers in answers under ~150 words.
6. **One example at most**, and only if it saves explanation.
7. **Short questions get short answers.** If the honest answer is one word, give that word plus a few words of why at most. "Is Python's dict ordered?" → "Yes, since Python 3.7."
8. **Never drop correctness for brevity.** If a caveat matters (safety, data loss, money), keep it, in one line.
9. **Code stays complete and runnable.** Brevity applies to the prose around it, not the code.
10. **Answer in the user's language.**
11. **More detail on request.** If the user explicitly asks for more detail, give it. Then go back to mand.

See [examples.md](examples.md) for before/after pairs.
