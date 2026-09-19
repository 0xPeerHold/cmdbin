# cmdbin

A searchable bin for the commands you keep forgetting.

cmdbin stores the commands you use — the `docker` invocation with six flags, the `ffmpeg` line you looked up twice last month, the `psql` connection string for staging — and lets you find them again by describing them badly. It runs as a single program on your own machine and keeps everything in one file. There is no account, no sign-in, and no server to connect to.

---

## Contents

* [Getting started](#getting-started)
* [Finding commands](#finding-commands)
* [Adding and editing commands](#adding-and-editing-commands)
* [Placeholders](#placeholders)
* [Projects](#projects)
* [Tags](#tags)
* [Dangerous commands](#dangerous-commands)
* [Trash](#trash)
* [Import and export](#import-and-export)
* [Sharing with another device](#sharing-with-another-device)
* [Running without a browser](#running-without-a-browser)
* [Local API](#local-api)
* [Your data and where it lives](#your-data-and-where-it-lives)
* [Backups](#backups)
* [Quitting](#quitting)
* [Settings and options](#settings-and-options)
* [Troubleshooting](#troubleshooting)
* [Privacy](#privacy)
* [Legal](#legal)


---

## Getting started

Run the program. It opens your browser to the bin automatically.

```bash
./cmdbin

```

On Windows, double-click `cmdbin.exe`.

Your browser opens at `[http://127.0.0.1:7717/](http://127.0.0.1:7717/)`. That address is your own machine — nothing is being sent anywhere. Leave the program running while you use it; closing the browser tab does not lose anything.

If you would rather start with example content, a feature tour is included:

```bash
./cmdbin -seed tour.json

```

That imports twelve commands covering every feature described below. It only loads when your bin is empty, so you cannot accidentally overwrite your own commands with it.

---

## Finding commands

Type in the search box. Search is built to tolerate the way people actually remember commands:

* **Describe it instead of naming it.** "see running containers" finds `docker ps` even though the words "see" and "running" never appear in the command itself.


* **Typos are fine.** `kubctl` finds `kubectl`. `docekr` finds `docker`.


* **Partial words work.** Typing `comp` matches `docker-compose` mid-word, not just at the start.


* **Compound names stay whole.** `docker-compose`, `node_modules` and `--force` are treated as single words rather than being split at the punctuation.



You do not need to match every word. A command matching more of your words ranks higher, but one matching some of them still appears.

Results are ordered by relevance blended with how often and how recently you use each command. A command you ran this morning outranks one you ran twenty times last quarter — recent use is weighted with a two-week half-life.

### Filtering

Narrow results using the sidebar: by project, by tag, by shell, or to favorites only. Filters combine with whatever you have typed in the search box.

### Keyboard

| Key | Does |
| --- | --- |
| `/` | Jump to the search box |
| `n` | New command |
| `j` / `k` | Move down / up through results |
| `↓` / `↑` | Same |
| `Enter` | From the search box, open the top result |

---

## Adding and editing commands

Press `n` or click the new-command button. A command can carry:

| Field | What it is for |
| --- | --- |
| **Title** | What you would call it out loud. This is weighted most heavily in search. |
| **Command** | The command itself. |
| **Description** | What it does, in your own words. Searchable, so write it the way you would ask for it later. |
| **Prerequisites** | What has to be true first — a VPN connected, a directory you must be in, a tool installed. |
| **Notes** | Anything else. Not weighted as heavily in search. |
| **Shell** | bash, zsh, fish, sh, PowerShell, cmd, nushell, SQL, Python, other, or any. |
| **Platform** | Linux, macOS, Windows, WSL, Docker, Android, iOS, or any. |
| **Language hint** | Stored for future syntax highlighting. |
| **Tags** | See [Tags](#tags). |
| **Project** | See [Projects](#projects). |
| **Default command** | See [Copying without filling the form](#copying-without-filling-the-form). |

Nothing except the command text is required.

As you type the command, the editor lists the placeholders it has found underneath — for example *2 placeholders: port:number image*. Check that line before saving: if a part you meant to be fillable isn't listed, cmdbin is going to treat it as plain text. See [How cmdbin recognises a placeholder](#how-cmdbin-recognises-a-placeholder).

Marking a command a **favorite** pins it and lets you filter to favorites only.

---

## Placeholders

Commands are rarely reusable verbatim — there is always a port, a container name, a filename that changes. Wrap those parts in braces and cmdbin turns them into a small form.

```
docker run --rm -it -p {port}:{port} {image}

```

Opening that command gives you two fields, `port` and `image`. Fill them in and the finished command is built underneath, ready to copy.

Note that `{port}` appears twice but produces **one** field. Repeats are occurrences of the same placeholder, not separate questions — fill it once and both update.

### How cmdbin recognises a placeholder

Not every pair of braces becomes a field. cmdbin reads the command one character at a time, and a brace only starts a placeholder when **all** of the following are true:

1. The `{` is not directly after a `$` or another `{`. That keeps shell variables such as `${PORT}` and doubled braces out.
2. A **name** comes straight after it. A name starts with a letter or underscore and may continue with letters, digits, underscores, hyphens and dots.
3. Optionally, a **type**: a colon followed by lowercase letters, such as `:number`.
4. Optionally, a **default**: `=` followed by anything up to the closing brace.
5. It ends with a single `}` — not `}}`.

So the full form is:

```
{name:type=default}
```

If any step fails, the braces and everything between them are left exactly as you wrote them. Nothing is reported as an error, because most braces in real commands are *meant* to be left alone — see [When you need literal braces](#when-you-need-literal-braces) — so the editor's placeholder line is how you tell which one you got.

The default is the one part that can contain almost anything, including colons, slashes and spaces, because it runs all the way to the closing brace. The only thing it cannot contain is a `}`.

#### Worked example: a URL in braces

Wrapping a whole URL in braces does **not** make it a placeholder:

```
curl -s {https://api.github.com/repos/golang/go/releases/latest}
```

cmdbin reads `https` as a name, then sees `:` and expects a type in lowercase letters. The next character is `/`, so there is no type, and the character after the name is neither `=` nor `}`. The braces fail the rule and the command is saved and copied exactly as typed, braces included. The editor shows no placeholders for it.

To make the URL fillable, give it a name and put the address after `=` as the default:

```
curl -s {url:url=https://api.github.com/repos/golang/go/releases/latest}
```

That is a `url` field, pre-filled with the address. Usually it is more useful to make only the part that changes into a field:

```
curl -s https://api.github.com/repos/{repo=golang/go}/releases/latest
```

Now the form asks for `repo` alone, pre-filled with `golang/go`, and you can type `docker/cli` without touching the rest of the URL.

### Placeholder types

Add a type after a colon to get a better input:

```
{port:number=8080}
{env:choice=dev|staging|prod}
{config:path}
{endpoint:url}
{force:boolean}
{token:secret}

```

| Type | What you get |
| --- | --- |
| `text` | A plain text box. This is the default, so `{name}` is a text field. |
| `number` | A numeric field. |
| `choice` | A dropdown. List the options after `=`, separated by `\|`. |
| `path` | A text box intended for a file or directory path. |
| `url` | A text box intended for a URL. |
| `boolean` | A dropdown offering `true` and `false`. |
| `secret` | A masked field. See the warning below. |

Types are lowercase. A type cmdbin doesn't recognise, such as `{port:int}`, gives you a plain text field. A type with a capital letter, such as `{port:Number}`, is not a type at all under rule 3 above, so the whole thing is not a placeholder and stays in the command as literal text. `bool` is accepted as a short form of `boolean`.

### Defaults

Anything after `=` is the starting value: `{port:number=8080}` opens with 8080 already filled in, and `{branch=main}` does the same without a type.

### Copying without filling the form

Some commands you run the same way almost every time. Tick **Default command** in the editor and any placeholder you leave empty falls back to the value written into the command text — so the command copies clean, with no braces, without you touching the form.

For that to work the value has to be in the command itself:

```
docker run -p {port:number=8080}:{port:number=8080} {image=nginx:latest}

```

With **Default command** ticked, that copies as `docker run -p 8080:8080 nginx:latest` straight away. Change `image` in the form and your value is used instead — the default only applies to fields you left alone.

Two limits worth knowing. A placeholder with no default has nothing to fall back to, so it still has to be filled and copy stays blocked until it is. Secrets are never defaulted, even with the box ticked, because a credential should not be copied by accident.

### Remembered values

Values you type into a placeholder are remembered and offered as suggestions next time, up to the ten most recent per field.

Secret fields are never remembered. A `{token:secret}` value is masked as you type, is not stored in your bin, and is not offered back to you later. Use `secret` for anything you would not want sitting in a plain text file — API tokens, passwords, connection strings with credentials in them.

### When you need literal braces

Some commands genuinely contain braces. cmdbin leaves these alone rather than mistaking them for placeholders:

| You write | Treated as |
| --- | --- |
| `${PORT}` | A shell variable, not a placeholder |
| `{{.Names}}` | A Go template, as used by `docker --format` |
| `{{port}}` | A literal `{port}` — double the braces to escape them |
| `@{Name='X';Expression={...}}` | PowerShell syntax, not a placeholder |
| `find . -exec rm {} \;` | Empty braces — there is no name, so they are left alone |
| `'{"level":"debug"}'` | JSON — a `"` cannot start a name |
| `cp file.{jpg,png} out/` | Shell brace expansion — the `,` fails the rule |
| `awk '{print $1}'` | awk code — the space after `print` fails the rule |
| `{https://example.com}` | A URL in braces — see the [worked example](#worked-example-a-url-in-braces) |

A placeholder name starts with a letter or underscore and may contain letters, digits, underscores, hyphens and dots — so `{qrcode.png}` and `{my-file}` are both valid names.

#### Braces that look like placeholders but aren't meant to be

A few tools use exactly the `{word}` shape that cmdbin looks for, so their code gets turned into fields you didn't ask for:

| You write | What happens | Write this instead |
| --- | --- | --- |
| `awk '{print}'` | A field named `print` | `awk '{{print}}'` |
| `jq '{name}'` (object shorthand) | A field named `name` | `jq '{{name}}'` |

Doubled braces are copied out as single ones, so each command on the right reaches your clipboard exactly as the tool expects it. The editor's placeholder line will show you when this has happened: if it lists a field you didn't intend, double those braces.

---

## Projects

Projects group related commands, and they nest — `Chroma RAG` can contain `Ingest`, which can contain its own children, up to six levels deep.

Create one with `+` beside the Projects heading in the sidebar. Hover any project row for:

* `+` — add a sub-project


* `R` — rename


* `x` — delete



Sibling projects must have unique names. Renaming a project updates the whole subtree beneath it, so search keeps finding commands by their project name.

### Deleting a project never silently deletes commands

The delete dialog always makes you choose what happens to the contents:

| Choose | Projects | Commands |
| --- | --- | --- |
| **Move commands to Ungrouped** | This project and its subtree go | All survive, unfiled |
| **Promote sub-projects** | Only this project goes | Its own commands become unfiled |
| **Delete the whole subtree** | This project and its subtree go | Go to trash, recoverable |

Even the third option is recoverable — those commands go to the trash rather than disappearing.

---

## Tags

Tags cut across projects. A command can carry as many as you like, and you can filter by one tag or several.

Filtering by several tags offers two modes: **ANY** (show commands with at least one of these tags) and **ALL** (show only commands carrying every one).

Tags support aliases, so a tag named `kubernetes` can also answer to `k8s` in search. Aliases currently arrive through import rather than being editable in the interface.

---

## Dangerous commands

Any command can be flagged **dangerous**, with a short note explaining why. Flagged commands are visibly marked in the list and in the detail pane.

This is worth using for anything destructive, irreversible, or run against production — a `DROP`, a `--force` push, a recursive delete, a deploy. The note is the useful part: "this drops the table, it does not truncate it" is the thing you want in front of you at the moment you are about to press copy.

The flag is advisory. cmdbin marks these commands but never blocks or executes anything — see [What cmdbin does not do(*What cmdbin does not do)

---

## Trash

Deleting a command moves it to the trash rather than removing it. The trash view lets you restore anything, or empty it to delete permanently.

Emptying the trash cannot be undone. Restore anything you want to keep first.

---

## Import and export

Export writes a single JSON file containing whatever you select. Import reads one back.

### Import only ever adds

This is a guarantee, not a setting: import cannot modify, replace, or delete a command already in your bin. There is no option to make it do so.

An entry in the file being imported is skipped when it is already present, which is judged two ways: the same command, or the same command text already in the same project. That second rule is per-project deliberately, because the same command legitimately appears in two projects under different titles.

In practice this means re-importing a file you exported last week creates nothing, and if you have edited one of those commands since, your edit stands.

The risk runs the other way: if you edit a command and then import a file containing a *differently-identified* copy of the original, you get a near-duplicate rather than an overwrite. Duplicates are easy to delete; overwritten work is not, which is why it errs in this direction.

### Choosing what to export

Export opens a dialog with checkboxes for projects and tags, a live "N of M commands" count, and toggles for including sub-projects, ANY-vs-ALL tag matching, favorites only, and whether to carry usage statistics. Selecting nothing exports everything.

A partial export always includes the parent projects of anything it contains, so nesting survives the round trip.

### Choosing what to import

Import happens in two steps. The file is summarized first — how many projects, tags and commands it holds — and **nothing is written** at this stage. Those become checkboxes, so you can take a colleague's entire bin and pull in only their Kubernetes commands. Only the projects those commands actually need get created.

---

## Sharing with another device

Sharing lets you open your bin from a phone or another computer on the same network — useful for copying a command to a device that is not the one holding your bin.

Sharing is off by default. When it is off, no network listener exists at all; the program is reachable only from the machine it runs on.

To use it, switch sharing on in the interface. You will be shown an address and a six-digit PIN. On the other device, open the address and enter the PIN.

If your machine is on more than one network — Wi-Fi and Ethernet, or with a VPN, Docker or a virtual machine running — you will see several addresses. They are listed most-likely-first, so try the top one and work down.

Things to know before you switch it on:

* Guests get full access, not read-only. A paired device can create, edit and delete commands just as you can. Only sharing itself, the PIN lockout reset, and quitting the program are restricted to the machine running it.


* Anyone on your network can reach the address. The PIN is what stops them getting in. Treat it like a password and prefer a network you trust.


* Your browser will warn about the certificate. The connection is encrypted, but with a certificate your own machine generated rather than one bought from an authority, so the browser cannot vouch for it and says so. Accepting the warning once on your own device is expected here.


* **The warning comes back if you move networks.** The certificate has to name the addresses you connect to, so taking the host machine from home to the office replaces it and paired devices see the warning once more. Ordinary use does not trigger this.


* **Pairing lasts 12 hours**, then the device must enter the PIN again.


* **Ten wrong PIN attempts locks pairing**, and only the host machine can reset it.


* Switching sharing off immediately drops every paired device.



By default the PIN is different every time the program starts.

---

## Running without a browser

cmdbin's interface is a web page, so a browser is how you use it. There is no terminal interface and no command-line subcommands — the only things you can do from a shell are the startup options below.

You can, however, stop it launching a browser for you:

```bash
./cmdbin -no-browser

```

It starts normally and prints the address; it simply opens nothing. This is safe to leave running unattended. The 5-second `-exit-on-close` default only applies when a tab actively reports that it is closing, so with no tab ever open, nothing triggers it and the program stays up. `-exit-on-idle` is off unless you set it.

### On a machine with no browser at all

On a headless server or a Raspberry Pi, start it without a browser and switch sharing on, then use it from your laptop or phone:

```bash
./cmdbin -no-browser -share

```

It prints an address and a pairing PIN. Note that a paired device gets full access — it can create, edit and delete — so on a machine sitting on a shared network the PIN is the only thing protecting your commands. Add `-pin` if you want a fixed one instead of a new one each time it starts. Do not use `-host` for this; see the warning in [Settings and options](#settings-and-options).

### Reading your commands from a terminal

Your bin is plain JSON, so you can look things up without starting cmdbin at all:

```bash
jq -r '.commands[] | "\(.title)\t\(.commandText)"' ~/.cmdbin/bin.json

```

That is read-only. It will not update usage counts, remembered values, or anything else, so what you see in the app afterwards is unchanged.

---

## Local API

A local HTTP endpoint that a program on the same machine can use to read your commands and add new ones. Claude Code, Ollama, a shell script, or you with `curl`.

**cmdbin answers requests. It never makes them.** It holds no API key, no model endpoint and no vendor relationship, and it opens no connection of its own. Which model you point at this, and whether that model runs locally or somewhere else, happens entirely in your client. cmdbin is not part of that conversation.

**It is off until you arm it.** There is no ambient endpoint sitting on your loopback port waiting to be found by whatever else runs as you.

### Arming it

Three states. Reading and writing are armed separately, so an agent doing read-only work never holds a capability it does not use.

| State | `/api/llm`, `/api/llm/pack` | `POST /api/llm/add` |
| --- | --- | --- |
| **off** (default) | refused | refused |
| **read** | allowed | refused |
| **write** | allowed | allowed |

**In the app:** the API button in the header. Host only — a paired phone cannot open a write path into your bin.

**At startup:**

```bash
cmdbin -api     # armed read-only
cmdbin -api-write    # armed for reading and writing
cmdbin -api -api-timeout 2h # a longer window
cmdbin -api -api-timeout 0  # armed until you switch it off

```



`-api` arms read-only on purpose. Writing needs its own flag, so the state that can change your bin is never one keystroke away from the state that cannot.

The token is printed to the terminal at startup, the same way the pairing PIN is. On a shared machine that puts a live credential in your scrollback.

#### The window closes

`-api-timeout` is a ceiling, default **30 minutes**, measured from the moment you arm it. It is not an idle timeout: a busy session is cut off just the same. That is deliberate — the rule is short enough to state in one sentence, and `0` is there for a dedicated box where you would rather not have one.

A long working session will hit it eventually. When it does, the next call is refused with `api-off` and you arm it again, which mints a new token.

**Arming state does not survive a restart.** Restarting cmdbin disarms it, exactly as restarting drops every paired sharing session.

### Authentication

Send the token as a header:

```
X-Cmdbin-Token: <token>

```



`Authorization: Bearer <token>` also works.

The token is shown beside the switch in the app, and printed at startup if a flag armed it. It is **minted when you arm and invalidated the moment you disarm**, so it never outlives its window. Re-arming rotates it; the previous one stops working immediately.

`-api-token` pins a fixed value across arms, for a machine where a config file has to hold it:

```bash
cmdbin -api-write -api-token "$(cat ~/.config/cmdbin/token)"

```



**Not in the URL.** A secret in a path lands in shell history, in `ps` output, and in any transcript of the session — including, for this use case, the model's own context. A header keeps it out of all three.

Being on loopback is no longer sufficient by itself. Any process on your machine can reach the port, including a browser extension, and a web page you visit can issue a cross-origin POST to `localhost` even though CORS stops it reading the reply. So:

* Requests carrying an `Origin` header are refused outright.
* `POST` must send `Content-Type: application/json`.

Between them, a browser cannot reach this at all.

Over network sharing, a caller needs a paired session **and** the token.

### The loop

```bash
TOKEN='...'   # from the app, or from the startup banner
H="X-Cmdbin-Token: $TOKEN"

# 1. learn the contract
curl -s -H "$H" localhost:7717/api/llm > contract.json

# 2. read a project back, in a shape a model can edit
curl -s -H "$H" 'localhost:7717/api/llm/pack?groups=3&limit=20' > pack.json

# 3. hand both to a model, collect its JSON

# 4. check it without writing anything
curl -s -H "$H" -H 'Content-Type: application/json' \
 -X POST 'localhost:7717/api/llm/add?dry=1' --data-binary @generated.json | jq

# 5. write it
curl -s -H "$H" -H 'Content-Type: application/json' \
 -X POST localhost:7717/api/llm/add --data-binary @generated.json | jq

```



GET describes and POST executes on the same path, so a caller that has lost the thread can re-fetch the contract from the path it is about to call.

Step 4 writes nothing and returns the same response minus the counts. Its `problems` array names the command index, the field, and what is wrong — specific enough to hand straight back as a correction.

### Endpoints

#### `GET /api/llm`

The entrance. Returns the shared writing rules, the live enums, the ability list, and the current access state.

Enums come from the binary's own constants rather than from this document, so they cannot go stale. If a placeholder type is added to cmdbin, it appears here the same day.

#### `GET /api/llm/pack`

Existing commands, in the same shape you are asked to send back.

| Parameter | Meaning |
| --- | --- |
| `groups` | comma-separated project ids |
| `tags` | comma-separated tag ids |
| `tagMode` | `or` (default) or `and` |
| `subgroups` | `0` to exclude sub-projects |
| `favorites` | `1` for favorites only |
| `limit` | default 50 |

The projection drops ids, uuids, timestamps and usage events. On the bundled tour set that is roughly 103 tokens per command instead of 286, which decides whether a project fits in a small context window at all.

It carries no handle, deliberately. Adding is safe without one; modifying is not, and modifying is not an ability here.

`truncated: true` means the limit cut the selection. Narrow the scope rather than raising the limit — a caller handed four hundred commands does worse than one handed the right twenty.

#### `GET /api/llm/add`

The template: the envelope, the add-specific rules, and a worked example. The placeholder grammar is not repeated here; it came from the entrance, and two copies drift. Available under a read arm.

#### `POST /api/llm/add`

Adds commands. **Needs a write arm.** `?dry=1` composes and validates without writing.

### Writing commands via API

You never write cmdbin's stored placeholder syntax. Send bare `{name}` markers plus a separate spec list, and the server composes.

That is not a style preference. An inline default containing a brace reparses into a different placeholder plus a stray brace, silently, and no amount of instruction reliably stops a generator producing one. Made unrepresentable, it stops being a failure mode.

```json
{
 "group": "Databases",
 "commands": [{
 "title": "Back up a database to a file",
 "command_text": "pg_dump -h {host} -U {user} -d {dbname} > {backup_file}",
 "description": "Write a plain SQL dump of a database to a file.",
 "tags": ["postgres", "backup"],
 "shell": "bash",
 "placeholders": [
  { "name": "host", "type": "text", "default": "localhost" },
  { "name": "user", "type": "text", "default": "postgres" },
  { "name": "dbname", "type": "text" },
  { "name": "backup_file", "type": "path" }
 ],
 "is_dangerous": true,
 "danger_note": "The > redirect overwrites the target file without asking."
 }]
}

```



Only `command_text` is required; a missing title falls back to the first line.

**Rules that get things rejected:**

* Every marker needs exactly one spec, and every spec needs a marker. No extras on either side.
* A marker holds only a name. No type, colon, equals sign, default, or nested marker.
* `{{name}}` means a literal `{name}` and creates no field. `${VAR}` is a shell variable and is left alone.
* A default is literal text and cannot contain a brace or a pipe.
* `choice` needs a non-empty `choices` array; nothing else may have one.
* `is_dangerous: true` requires a `danger_note`.

Use `secret` for passwords, tokens and connection strings. Those values are never written to disk.

**The last check is the load-bearing one.** After composing, the server re-parses the result with cmdbin's own parser — not a second implementation — and requires the fields that come out to match the specs that went in. Anything the parser reads differently is rejected rather than guessed at.

### API Errors

#### Refused at the gate — HTTP 403

```json
{ "error": "the local API is switched off; arm it in the app, or start with -api",
 "reason": "api-off",
 "state": "off" }

```



| `reason` | What happened |
| --- | --- |
| `api-off` | Not armed, or the window closed |
| `api-read-only` | Armed for reading; this call needs a write arm |
| `api-token-missing` | No `X-Cmdbin-Token` header |
| `api-token-stale` | Not the current token — it changed when the API was re-armed |
| `api-origin` | Request carried an `Origin` header |
| `api-content-type` | POST without `Content-Type: application/json` |

**Never a 404.** A caller that gets a 404 retries, invents a different path, or reports success. A named reason surfaces in the transcript where the person can see it and act.

#### Rejected on content — HTTP 400

```json
{
 "problems": [
 { "index": 0, "title": "Backup", "field": "placeholders",
  "issue": "the default for \"out\" contains a brace; a default is literal text" }
 ],
 "warnings": []
}

```



Nothing is written when `problems` is non-empty, even for the commands in the batch that were fine.

#### Accepted — HTTP 200

```json
{ "composed": ["..."], "warnings": [...], "added": 3, "skipped": 1, "projectsCreated": 1 }

```



`warnings` never blocks a write. The destructive heuristic warns; it does not reject. A heuristic that blocks on its own false positives is one people learn to switch off.

### What a generated command can and cannot do (API)

* **It can add. It cannot modify or delete.** Those abilities do not exist on this API, and a caller cannot invoke an ability that is not there. Adds go through the same additive path as the Import dialog, so sending a batch twice skips rather than overwrites.
* **Everything added is tagged `llm`.** Applied by the server, not optional, and it survives export — so a batch can be reviewed, or selected and deleted, as a batch.
* **Everything added carries a note.** Every command, not only the ones the heuristic noticed:
> Added through the local API by a program. Nobody has read it.


* A command that trips the destructive heuristic carries that plus what the heuristic found. The note is applied by the server and cannot be suppressed, and it renders in the detail pane next to where the danger marker renders — the screen somebody is looking at in the second before they copy something into a shell.
* **It never sets the danger flag.** That flag means a person judged this command, and a guess must not be able to impersonate one. The heuristic writes a note instead.

---

## Your data and where it lives

Everything is in one file:

| System | Location |
| --- | --- |
| Linux, macOS | `~/.cmdbin/bin.json` |
| Windows | `.cmdbin\bin.json` in your user folder |

It is ordinary JSON. You can read it, copy it, put it in version control, sync it, or move it to another machine. Use `-bin` to point at a different file:

```bash
./cmdbin -bin ~/Dropbox/work-bin.json

```

Saves are written to a temporary file and then renamed over the original, so the bin is never left half-written if something goes wrong mid-save.

One exception to "one file": the first time you switch sharing on, `share-cert.pem` and `share-key.pem` are written into the same folder, both readable only by you. Neither exists until you use sharing, and deleting them just makes a new pair next time.

---

## Backups

cmdbin does not take automatic backups. Since the bin is a single file, any of these work:

* Copy `bin.json` somewhere safe periodically.


* Keep it in a synced folder and point `-bin` at it.


* Keep it in a Git repository and commit it — it is JSON, so changes are readable in a diff.


* Use **Export** to write a portable file you can re-import anywhere.



Export is the most durable option, since the export format is designed to be read back by any version.

---

## Quitting

Three ways, all of which save first:

* **Quit** in the header. Saves, confirms, and tells you the tab is safe to close.


* **Ctrl-C** in the terminal window.


* **Automatically**, if you started it with `-exit-on-idle`.



Avoid force-killing the program (`kill -9`, End Task). Saves are batched by a second or so, so a forced kill can lose your most recent edit.

To find the process if you have lost the window:

```bash
lsof -ti:7717 | xargs -r kill

```

---

## Settings and options

Options are given on the command line when starting the program.

| Option | Default | What it does |
| --- | --- | --- |
| `-bin` | `~/.cmdbin/bin.json` | Which bin file to use |
| `-port` | `7717` | Which port to listen on |
| `-host` | `127.0.0.1` | Which address to bind. Leave this alone unless you understand the warning below |
| `-seed` | none | Import this file if the bin is empty |
| `-no-browser` | off | Do not open a browser at startup |
| `-share` | off | Switch sharing on at startup |
| `-share-port` | `7718` | Port used for sharing |
| `-pin` | random | Use a fixed pairing PIN instead of a new one each run |
| `-exit-on-idle` | never | Quit after this long with no browser in touch, e.g. `10m` |
| `-exit-on-close` | `5s` | Quit this long after the last tab closes |
| `-api` | off | Armed read-only for local LLM/program API integration |
| `-api-write` | off | Armed for reading and writing via the local API |
| `-api-token` | random | Pin a fixed token value across arms for the local API |
| `-api-timeout` | `30m` | Ceiling window for local API access (`0` for indefinite) |

`CMDBIN_FILE`, `PORT`, `SHARE_PORT` and `PIN` work as environment variables.

If you set `-exit-on-idle`, keep it generous. Browsers slow down timers in background tabs, so a value under about two minutes can quit the program while a tab is still sitting open. The program warns you at startup if you set it that low.

**About `-host`:** binding to anything other than `127.0.0.1` exposes the bin with **no PIN and no encryption** — that protection belongs to the sharing feature, not to the main address. Anyone who can reach the port can read and edit everything. Copy buttons also stop working properly, because a plain `http` address that is not `localhost` is not treated as a secure context by browsers. If you want another device to have access, use [sharing](#sharing-with-another-device) instead.

When you do bind wider, startup prints the addresses other devices can use alongside the loopback one, so you do not have to look up your own IP.

---

## Troubleshooting

* **The browser didn't open.** Go to `[http://127.0.0.1:7717/](http://127.0.0.1:7717/)` yourself. If you changed `-port`, use that number.


* **"Port already in use."** Either cmdbin is already running — check your other browser tabs — or something else has the port. Use `-port 7800` or whatever is free.


* **My phone can't reach the shared address.** If more than one address is listed, try each in turn — the top one is the most likely but not guaranteed. Otherwise, both devices must be on the same network, and some public and guest networks block devices from seeing each other entirely, which cannot be worked around from inside the app.


* **The certificate warning worries me.** It is expected. See [sharing](#sharing-with-another-device) for what it means.


* **I'm locked out of pairing.** Ten wrong PIN attempts locks it. Reset it from the machine running cmdbin, or restart the program for a fresh PIN.


* **I lost a command.** Check the trash — deletes go there first and can be restored, unless the trash has been emptied.


* **My bin won't load.** The file is JSON; if it has been edited by hand or truncated by a disk problem, it may not parse. Restore your most recent backup or export.


* **A command with braces isn't showing the fields I expected.** See [when you need literal braces](#when-you-need-literal-braces).


* **Local API calls are refused with `api-off`.** The API is either not armed or its timeout window has closed; arm it again or check startup flags.

---

## What cmdbin does not do

Worth being explicit, because it shapes what the program is safe to be used for:

* **It never runs anything.** cmdbin stores, searches and copies command text. Executing it is up to you, in your own terminal, deliberately. Nothing in the app runs a shell.


* **It does not check whether a command is correct or safe.** Text you save is text you get back. The dangerous flag is a note you write to yourself, not an analysis the program performed.


* **It does not manage secrets.** The `secret` placeholder type keeps values out of your bin and off the screen, but cmdbin is not a password manager and the bin file is not encrypted. Anything you type into a non-secret field, or paste into the command text itself, is stored in plain text.


* **It has no terminal interface.** The interface is a web page and needs a browser. There are no command-line subcommands for searching or copying — see [Running without a browser](#running-without-a-browser) for the headless options.



---

## Privacy

cmdbin runs entirely on your own machine.

* There is no account, no sign-in, and no server belonging to anyone else.


* It does not send usage data, analytics, crash reports or telemetry anywhere.


* It does not check for updates over the network.


* With sharing off, the program listens only on a loopback address, which means only your own machine can reach it. This is the shipped default.


* The only network activity that ever happens is sharing, which you switch on yourself, and which stays within your local network.



What is recorded, locally, in your bin file:

| Recorded | Retained |
| --- | --- |
| That you used or copied a command, and when | 365 days, up to 100 events per command |
| Values typed into non-secret placeholders | The 10 most recent per field |
| Values typed into `secret` placeholders | Never recorded |

These retention figures are stated in the privacy notice shipped with the application, which is the authoritative version.

You can delete all of it at any time by deleting your bin file.

---

## Legal

The full legal documents — end-user license agreement, privacy notice, publisher information and third-party licenses — ship inside the application. Open the **About** panel to read them. They carry a version number and an effective date so you can tell which text you agreed to.

Those documents are the controlling versions. Anything in this guide is a plain-language summary written to help you, and where the two differ the shipped documents govern.

### License

Your use of cmdbin is governed by the end-user license agreement in the About panel. Please read it before use.

### No warranty

cmdbin is provided without warranty, to the extent permitted by law. You are responsible for the commands you store and for what happens when you run them. The program does not execute anything itself, does not validate what you save, and cannot tell whether a command is safe in your environment. Treat everything in your bin as text you wrote, because it is.

Consumer protection law in your country may give you rights that cannot be excluded by an agreement. Nothing here is intended to remove them.

### Third-party software

Third-party components and their licenses are listed in the About panel.

### Publisher

© 2026 0xPeerHold LLC

Website: [http://0xPeerHold.com](https://0xpeerhold.com)

General contact: support@0xPeerHold.com

The registered legal name appears in full in the About panel.
