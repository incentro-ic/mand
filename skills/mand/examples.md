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

---

## 9. Bad idea (rule 13)

**User:** How do I turn off SSL certificate verification in Python requests? The API I call gives a certificate error.

### Before (288 words)

Pass `verify=False`:

```python
import requests

resp = requests.get("https://api.example.com/data", verify=False)
```

That also prints an `InsecureRequestWarning` on every call. To silence it:

```python
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
```

For a whole session:

```python
s = requests.Session()
s.verify = False
```

**Only do this for local testing.** With `verify=False`, anyone who can intercept the traffic can read or change it, including tokens and credentials.

### Better fixes, depending on the cause

The error message usually tells you which one applies.

1. **Your company's network inspects HTTPS traffic** (Zscaler, Netskope, or a corporate proxy). This is the most common cause on a work laptop. Python doesn't use the macOS Keychain, so it doesn't trust your company's root certificate. The easiest fix is `truststore`, which makes Python use the operating system's certificate store:
   ```bash
   pip install truststore
   ```
   ```python
   import truststore
   truststore.inject_into_ssl()   # call once at startup, before making requests
   import requests
   requests.get("https://api.example.com/data")  # verification still on
   ```

2. **The API uses a self-signed or private certificate.** Export that certificate (or its CA) as a PEM file and point requests to it:
   ```python
   requests.get(url, verify="/path/to/ca.pem")
   ```
   You can also set it once for everything with `export REQUESTS_CA_BUNDLE=/path/to/ca.pem`.

3. **Your certificate bundle is out of date.** Run `pip install --upgrade certifi`. If you use python.org Python on macOS, also run `/Applications/Python\ 3.x/Install\ Certificates.command`.

4. **The server's certificate is expired, has the wrong hostname, or is missing its intermediate certificate.** That's a problem on the server side, and the API owner needs to fix it. Check with:
   ```bash
   openssl s_client -connect api.example.com:443 -servername api.example.com
   ```

If you paste the exact error (for example `CERTIFICATE_VERIFY_FAILED: self-signed certificate in certificate chain` or `unable to get local issuer certificate`), I can tell you which of these it is.

### Mand. (108 words)

Don't turn it off in production. It leaves the connection open to man-in-the-middle attacks. Fix the trust instead.

If the API uses a self-signed or internal CA certificate, point requests at it:

```python
import requests

r = requests.get("https://api.example.com", verify="/path/to/ca-bundle.pem")
```

If it's a public certificate failing on macOS, update your CA bundle with `pip install --upgrade certifi`. If you use python.org Python, also run `/Applications/Python 3.x/Install Certificates.command`.

For a quick local test only, this turns verification off and hides the warning:

```python
import requests
import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
r = requests.get("https://api.example.com", verify=False)
```

The `disable_warnings` line hides this warning for every request in the process, not just this one.

---

## 10. Pick one (rule 12)

**User:** Should I use PostgreSQL or MongoDB for a new web shop with customers, products and orders?

### Before (355 words)

**Use PostgreSQL.** A web shop is a good fit for a relational database.

**Why PostgreSQL fits:**

- **Your data is relational.** Customers place orders, orders contain line items, and line items point to products. That's foreign keys and joins, which is what relational databases are built for.
- **Transactions matter.** Checkout touches several records at once: create the order, reserve stock, record the payment. You want that to fully succeed or fully fail. Postgres has had solid ACID transactions across tables forever. MongoDB supports multi-document transactions now, but they're an add-on rather than the default way of working.
- **Integrity in the database.** Constraints like `NOT NULL`, `UNIQUE (email)`, `CHECK (stock >= 0)` and foreign keys stop bad data even when application code has a bug.
- **Reporting is easy.** Questions like "revenue per category last month" or "customers who haven't ordered in 90 days" are plain SQL. In MongoDB they need aggregation pipelines or copying data around.
- **It handles flexible data too.** Products with varying attributes (sizes, colors, specs) can go in a `JSONB` column with indexes, so you get MongoDB's main advantage where you need it.

**When MongoDB would make sense:**

- The catalog is the core of the product, with very different and fast-changing product shapes, and orders are minor or handled by an external platform.
- You expect to need horizontal sharding at very large scale from day one. Most shops never get there, and Postgres scales vertically and with read replicas a long way.
- Your team already knows MongoDB well and has no SQL experience.

**Practical tips if you go with Postgres:**

- On each order line, store a copy of the price and product name at the time of purchase. Don't just reference the product, because prices change.
- Use `numeric` for money (or integer cents), never `float`.
- Use `SELECT ... FOR UPDATE` or an atomic `UPDATE ... SET stock = stock - 1 WHERE stock > 0` so two people can't buy the last item.
- Before building a shop from scratch, check whether an existing platform (Shopify, Medusa, Saleor, etc.) already covers your needs.

### Mand. (33 words)

I recommend PostgreSQL. Customers, products and orders are related data, and orders need transactions and consistency, which a relational database handles best. For product attributes that differ per item, use a `jsonb` column.

---

## 11. Plan is fine (rule 17)

**User:** I'm adding node_modules to my .gitignore so it doesn't end up in git. That's the right approach, isn't it?

### Before (99 words)

Yes, that's the standard approach. Add this line to `.gitignore`:

```
node_modules/
```

A few related points:

- **If `node_modules` was already committed, ignoring it now won't remove it.** Git keeps tracking files it already knows about. Stop tracking it with:
  ```sh
  git rm -r --cached node_modules
  git commit -m "Stop tracking node_modules"
  ```
- **Commit your lockfile.** That's `package-lock.json`, `yarn.lock` or `pnpm-lock.yaml`. With the lockfile in git, `npm ci` (or the equivalent) rebuilds the exact same `node_modules` on any machine.
- **Some other common entries:** `.env` (to keep secrets out), build output like `dist/` or `build/`, and `npm-debug.log*`.

### Mand. (43 words)

Yes, and commit your lockfile (`package-lock.json` or similar) so installs stay the same everywhere. If `node_modules` is already committed, `.gitignore` won't remove it, so run `git rm -r --cached node_modules` once and commit. That only removes it from git; your local files stay.

---

## 12. Far more than needed, in Dutch (rules 13 and 10)

**User:** Ik wil Kubernetes gebruiken voor mijn persoonlijke blog met zo'n 50 bezoekers per maand. Hoe zet ik een cluster op?

### Before (449 words)

Eerlijk gezegd is Kubernetes voor een blog met 50 bezoekers per maand flink overkill. Wil je het vooral om **Kubernetes te leren**, dan is dit een prima oefenproject. Gaat het je alleen om het **hosten van de blog**, dan zijn er veel eenvoudigere en goedkopere opties.

## Als het alleen om de blog gaat

- **Statische site** (Hugo, Astro, Eleventy) op **GitHub Pages, Cloudflare Pages of Netlify**: gratis, geen onderhoud, en het blijft draaien als je ineens veel bezoekers krijgt.
- Gebruik je een CMS zoals Ghost of WordPress, dan volstaat **één kleine VPS** (Hetzner, ~€4/maand) met Docker Compose.

Een cluster betekent juist meer werk: updates, certificaten, storage, monitoring en meestal hogere kosten. Bij jouw bezoekersaantal levert dat niets op.

## Als je Kubernetes wilt leren: zo zet je het simpel op

**1. Kies een lichte distributie op één VPS**
Gebruik **k3s**: één binary, en Traefik (ingress) en local-path storage zitten er al in.

```bash
# Op een VPS (bijv. Hetzner CX22, Ubuntu)
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
```

Kopieer `/etc/rancher/k3s/k3s.yaml` naar `~/.kube/config` op je laptop en vervang `127.0.0.1` door het IP-adres van je server.

**2. Zet cert-manager op voor HTTPS**
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```
Maak daarna een `ClusterIssuer` voor Let's Encrypt aan.

**3. Deploy je blog**
Bijvoorbeeld een nginx-container met je statische Hugo-build:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: blog }
spec:
  replicas: 1
  selector: { matchLabels: { app: blog } }
  template:
    metadata: { labels: { app: blog } }
    spec:
      containers:
        - name: blog
          image: ghcr.io/<jouw-user>/blog:latest
          ports: [{ containerPort: 80 }]
---
apiVersion: v1
kind: Service
metadata: { name: blog }
spec:
  selector: { app: blog }
  ports: [{ port: 80 }]
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: blog
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls: [{ hosts: [jouwblog.nl], secretName: blog-tls }]
  rules:
    - host: jouwblog.nl
      http:
        paths:
          - path: /
            pathType: Prefix
            backend: { service: { name: blog, port: { number: 80 } } }
```

**4. Wil je verder, dan kun je dit toevoegen**
- **GitOps** met Argo CD of Flux, zodat een push naar git automatisch deployt.
- **CI**: een GitHub Action die je image bouwt en naar ghcr.io pusht.

**Alternatieven:**
- **Lokaal experimenteren:** kind, minikube of k3d, gratis en op je eigen laptop.
- **Managed Kubernetes:** DigitalOcean of Scaleway, maar dat is duurder dan nodig.

**Mijn advies:** host de blog statisch op Cloudflare Pages of Netlify, en speel met k3s op een goedkope VPS als je Kubernetes wilt leren. Zo blijft je blog online, ook als je het cluster een keer stukmaakt.

Wat is je doel: Kubernetes leren of gewoon een blog online hebben? Dan kan ik je verder helpen met de juiste route.

### Mand. (89 words)

Doe het niet: Kubernetes is voor 50 bezoekers per maand veel te zwaar. Je betaalt meer, en je bent meer tijd kwijt aan onderhoud dan aan je blog.

Gebruik een statische sitegenerator zoals Hugo of Astro, en host gratis op Cloudflare Pages, Netlify of GitHub Pages. Je pusht naar Git en de site staat live, met HTTPS en zonder servers.

Wil je Kubernetes vooral leren, gebruik dan k3s op één goedkope VPS van €5 per maand, of kind lokaal op je laptop. Houd die oefening los van je blog.
