# Mand. examples

Real outputs from Claude Code, same model and settings, run with and without the mand skill. Word counts include code.

## 1. Yes/no question

**User:** Is Python's dict ordered?

### Before (137 words)

Yes, since **Python 3.7**, `dict` preserving **insertion order** is a guaranteed part of the language spec. (In CPython 3.6 it already worked this way, but only as an implementation detail.)

```python
d = {"b": 1, "a": 2}
d["c"] = 3
list(d)  # ['b', 'a', 'c']
```

Some details:

- **Updating an existing key** keeps its original position.
- **Deleting and re-inserting** a key moves it to the end.
- **Equality ignores order**: `{"a": 1, "b": 2} == {"b": 2, "a": 1}` is `True`.
- `popitem()` removes the **last** inserted item (LIFO).
- `reversed(d)` works since Python 3.8.

**When to use `collections.OrderedDict` instead:**
- You want equality to be order-sensitive (`OrderedDict` comparisons check order).
- You need `move_to_end()` or `popitem(last=False)`, which are handy for LRU caches.

`set` is **not** ordered. If you need an ordered set, use `dict.fromkeys(items)`.

### Mand. (9 words)

Yes. Dicts keep insertion order, guaranteed since Python 3.7.

---

## 2. "How do I…"

**User:** How do I undo my last git commit but keep the changes?

### Before (113 words)

Use a soft or mixed reset:

```bash
# Keep changes staged (ready to re-commit)
git reset --soft HEAD~1

# Keep changes but unstaged (default)
git reset HEAD~1
```

**Notes:**
- If you've **already pushed** the commit, prefer `git revert HEAD` instead, which creates a new commit that undoes it without rewriting shared history. If you do reset a pushed commit, you'll need `git push --force-with-lease`, which can disrupt collaborators.
- If you only want to tweak the last commit (fix the message or add a forgotten file), use `git commit --amend` instead.
- Changed your mind after resetting? `git reflog` shows the old commit hash, and `git reset --soft <hash>` brings it back.

### Mand. (24 words)

Run `git reset --soft HEAD~1`. The commit is removed and its changes stay staged. If you already pushed it, use `git revert HEAD` instead.

---

## 3. Code

**User:** Write a Python function that checks if a string is a palindrome, ignoring case, spaces and punctuation.

### Before (209 words)

```python
def is_palindrome(s: str) -> bool:
    """Return True if s is a palindrome, ignoring case, spaces, and punctuation."""
    cleaned = [ch.casefold() for ch in s if ch.isalnum()]
    return cleaned == cleaned[::-1]
```

**How it works:**
- `ch.isalnum()` keeps only letters and digits, which drops spaces and punctuation. It is Unicode-aware, so accented letters such as `é` are kept.
- `casefold()` is a more thorough version of `lower()` for case-insensitive comparison. For example, it maps German `ß` to `ss`.
- The cleaned sequence is compared with its reverse.

**Examples:**
```python
is_palindrome("A man, a plan, a canal: Panama")  # True
is_palindrome("No 'x' in Nixon")                 # True
is_palindrome("Hello, world!")                   # False
is_palindrome("")                                # True (empty counts as a palindrome)
```

**Memory-efficient variant:** this two-pointer version uses O(1) extra space and does not build a cleaned copy. It is useful for very long strings:

```python
def is_palindrome(s: str) -> bool:
    i, j = 0, len(s) - 1
    while i < j:
        if not s[i].isalnum():
            i += 1
        elif not s[j].isalnum():
            j -= 1
        elif s[i].casefold() != s[j].casefold():
            return False
        else:
            i += 1
            j -= 1
    return True
```

For multi-character casefolds like `ß` → `ss`, the first version handles edge cases more correctly. For ordinary text, both versions give the same results.

### Mand. (46 words)

```python
def is_palindrome(s: str) -> bool:
    cleaned = [c.lower() for c in s if c.isalnum()]
    return cleaned == cleaned[::-1]
```

It keeps only letters and digits, lowercases them, and compares the result to its reverse. For example, `is_palindrome("A man, a plan, a canal: Panama")` returns `True`.

---

## 4. Debugging

**User:** My React component re-renders forever. I call setCount(count + 1) directly in the component body. Why?

### Before (213 words)

Because the component body *is* the render function. Every time React renders your component, it runs that line:

1. Render → `setCount(count + 1)` runs → state changes → React schedules a re-render
2. Re-render → `setCount(count + 1)` runs again → state changes again → another re-render
3. …forever (React eventually bails with "Too many re-renders").

There's no condition that stops it, because each render produces a new value, so React never sees the state as unchanged.

**Fix: only update state in response to something**, not during render:

```jsx
// On user action
<button onClick={() => setCount(c => c + 1)}>+1</button>
```

```jsx
// Once on mount (or when deps change)
useEffect(() => {
  setCount(c => c + 1);
}, []); // empty deps = run once after mount
```

Notes:
- Watch out for `onClick={setCount(count + 1)}`: it *calls* the function during render, giving the same loop. Pass a function: `onClick={() => setCount(...)}`.
- If the value can be computed from props/other state, don't store it in state at all; just compute it during render (`const doubled = count * 2`).
- Setting state during render is only valid in the rare "adjust state when a prop changes" pattern, and only behind a condition that stops being true after one update.

### Mand. (48 words)

Calling `setCount` during render schedules another render, which calls `setCount` again, forever. React stops it with a "Too many re-renders" error.

Move the update into an event handler or a `useEffect`:

```jsx
useEffect(() => {
  setCount(c => c + 1);
}, []); // runs once after mount
```

---

## 5. Concept

**User:** What is a race condition?

### Before (344 words)

A **race condition** is a bug where a program's result depends on the timing or order in which concurrent operations run. When two or more threads or processes access shared state and at least one of them writes to it, they can interleave in an unexpected order and produce a wrong result.

## Classic example: a lost update

Two threads each run `counter = counter + 1` on a shared `counter` that starts at 0. That line is really three steps: read, add, write.

| Time | Thread A | Thread B | `counter` |
|------|----------|----------|-----------|
| 1 | reads 0 | | 0 |
| 2 | | reads 0 | 0 |
| 3 | writes 1 | | 1 |
| 4 | | writes 1 | 1 |

You expected 2 but got 1, because one increment was lost. Most runs might work fine, which makes these bugs intermittent and hard to reproduce.

## Common forms

- **Read-modify-write**: the counter example above.
- **Check-then-act (TOCTOU, "time of check to time of use")**: e.g. `if not exists(file): create(file)`. Another process can create the file between the check and the create. This is also a common security vulnerability class.
- **Initialization races**: two threads both see "not initialized yet" and both initialize.

## How to prevent them

- **Mutual exclusion**: locks or mutexes around the critical section.
- **Atomic operations**: e.g. `AtomicInteger.incrementAndGet()` or compare-and-swap.
- **Avoid shared mutable state**: immutable data, message passing (actors, channels).
- **Database-level guarantees**: transactions, `SELECT ... FOR UPDATE`, unique constraints, optimistic locking with version columns.
- **Atomic system calls**: e.g. `open(path, O_CREAT | O_EXCL)` instead of checking first and then creating.

## Related term

A **data race** is a narrower, language-level term. It means unsynchronized concurrent access to the same memory where at least one access is a write. Data races are often undefined behavior, as in C, C++ and Rust's `unsafe` code. You can have a race condition without a data race, for instance with properly locked steps that still run in the wrong logical order.

### Mand. (44 words)

A race condition is a bug where the result depends on which of two or more threads or processes happens to run first. For example, two threads each read a counter at 5, add 1, and write back 6, so one increment is lost.

---

## 6. Safety caveat (rule 8)

**User:** How do I delete all Docker containers, images and volumes to free up disk space?

### Before (315 words)

Here's how to clean up Docker, from most targeted to most thorough. **All of these are irreversible.** Volumes in particular may hold database data you can't get back.

### Check what's using space first
```bash
docker system df        # summary
docker system df -v     # per image/container/volume breakdown
```

### One-shot: remove everything unused
```bash
docker system prune -a --volumes
```
This removes:
- all **stopped** containers
- all images not used by a container (`-a`; without it, only dangling images)
- all unused networks
- all unused volumes (`--volumes`)
- build cache

Add `-f` to skip the confirmation prompt. Running containers and what they use are kept.

### Nuke literally everything, including running containers
```bash
docker stop $(docker ps -q)               # stop running containers
docker rm -f $(docker ps -aq)             # remove all containers
docker rmi -f $(docker images -aq)        # remove all images
docker volume rm $(docker volume ls -q)   # remove all volumes
docker builder prune -af                  # clear build cache
```
If a list is empty, that command will complain about missing arguments. That's harmless.

### Individual prune commands
```bash
docker container prune
docker image prune -a
docker volume prune -a    # -a needed on newer Docker to include named volumes
docker builder prune -a
docker network prune
```

### macOS: reclaim the space on disk
On a Mac, Docker Desktop stores everything in a VM disk image that doesn't always shrink after pruning. If Finder still shows the space as used:
- Go to **Docker Desktop → Settings → Resources** and lower the disk limit.
- Or go to **Troubleshoot (bug icon) → Clean / Purge data** to wipe the VM entirely.

If you're using OrbStack or Colima instead, they usually reclaim space automatically, or with `colima delete` for a full reset.

Before pruning volumes, check `docker volume ls` for anything like `postgres_data` or `mysql` that you want to keep.

### Mand. (39 words)

Stop and remove all containers, then prune everything else:

```sh
docker rm -f $(docker ps -aq)
docker system prune -a --volumes -f
```

This permanently deletes all volume data, such as databases, so back up anything you need first.

---

## 7. Dutch (rule 10)

**User:** Hoe zet ik een cel vast in een Excel-formule zodat hij niet verschuift als ik de formule doortrek?

### Before (158 words)

Gebruik een **absolute verwijzing** met dollartekens (`$`).

## Zo werkt het

| Verwijzing | Wat blijft vast bij doortrekken |
|---|---|
| `A1` | Niets (relatief) |
| `$A$1` | Kolom én rij |
| `$A1` | Alleen de kolom |
| `A$1` | Alleen de rij |

## Voorbeeld

Stel, in `B1` staat een btw-percentage en je wilt bedragen in kolom A daarmee vermenigvuldigen:

```
=A2*$B$1
```

Trek je dit naar beneden, dan wordt het `=A3*$B$1`, `=A4*$B$1`, enzovoort. `B1` blijft dus vast.

## Sneltoets

Klik in de formulebalk op de celverwijzing (of zet je cursor erin) en druk op:
- **Windows:** `F4`
- **Mac:** `Cmd + T` (of `Fn + F4`)

Elke druk wisselt tussen `$A$1` → `A$1` → `$A1` → `A1`.

## Tip

Je kunt de cel ook een **naam** geven (selecteer de cel, typ een naam in het naamvak links van de formulebalk, bijv. `btw`). Dan schrijf je `=A2*btw`, en een naam is altijd absoluut.

### Mand. (51 words)

Zet een dollarteken voor de kolom en de rij, dus `$A$1` in plaats van `A1`. Klik in de formule op de celverwijzing en druk op F4 (op de Mac Cmd+T), dan komen de dollartekens er vanzelf bij. Met `$A1` of `A$1` zet je alleen de kolom of alleen de rij vast.

---

## 8. Asking for more detail (rule 11)

A three-turn conversation. Without mand the three answers were 312, 1244, 951 words long. With mand:

**User:** What is DNS?

**Mand.** (21 words)

DNS is the internet's phone book: it turns names like example.com into the IP addresses computers use to find each other.

**User:** Give me more detail: walk me through what happens step by step when I type example.com in my browser.

**Mand.** (242 words; the user asked for detail, so it gives detail)

When you type example.com and press Enter, this happens:

1. **Browser cache.** The browser checks whether it already looked up example.com recently. If so, it uses that address and skips the rest.
2. **Operating system cache.** If not, it asks your OS, which has its own cache and also checks the hosts file.
3. **Resolver.** If the OS doesn't know, it asks a DNS resolver. This is usually run by your internet provider, or a public one like 1.1.1.1 or 8.8.8.8. The resolver does the legwork and caches answers too.
4. **Root server.** If the resolver has nothing cached, it asks one of the root servers. The root doesn't know example.com, but it knows who handles `.com` and points there.
5. **TLD server.** The resolver asks the `.com` server. It doesn't know the address either, but it knows which name servers are in charge of example.com and points there.
6. **Authoritative server.** The resolver asks example.com's own name server, which holds the real record and replies with the IP address, for example 93.184.215.14.
7. **Answer comes back.** The resolver stores the answer for as long as the record's TTL (time to live) allows, then passes it to your OS and browser, which cache it too.
8. **Connection.** The browser connects to that IP address, sets up HTTPS, and requests the page.

Steps 4 to 6 only happen on a cache miss. Most lookups are answered from a cache in a few milliseconds.

**User:** Is DNS encrypted?

**Mand.** (44 words; back to short)

Usually not: classic DNS is sent in plain text, so your network or internet provider can see which sites you look up. Encrypted options like DNS over HTTPS (DoH) and DNS over TLS (DoT) exist, and many browsers and phones can turn them on.
