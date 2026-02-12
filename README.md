# dataGazer-go
A RAD data (mostly for visualization) framework that a person can build in days instead of months, alternative implementation

## client description:
Same as of Ruby implementation but\
instead of Active Record for sqlite (mature), https://duckdb.org/docs/stable/clients/go (mature)\
instead TTY prompt (production ready), bubbletea (tend to think is prod. ready)\
instead `eval()` string parsing with injection defences, I might use: a full DSL, reflection (although I don't think it's sufficient to solve the problem), or an embedded LUA machine\
\
*let the race begin*

## Spec as **2026-02-08Z**

This design comes from the need to watch data quickly from a mobile phone, an SSH terminal, or basically on the go.

Soon we realized a client-server architecture has humongous advantages for easier development and security, so now is client–server.

The server must be in the same security ring as the database and the datafiles.

_Compiling_ means parsing and testing the grammars below and generating a signed watermark so it runs faster next time. It uses a signature that shouldn’t be crackable **too** easily.

_Session_ handshake is critical and done with a shared secret. There’s also a _SessionWatermark_ which, if it can’t be resolved with that secret, means the connection is considered broken.

We seek for a compression mechanism with encryption that runs really fast (tend to think those exist).

All data is filtered by the server, no leaks

Semantic _semaphores_ at design time help avoid complex locks for critical parts of the organization, nevertheless you always end up doing some of this anyway. The server is going to lock users, and maybe there will be an option to talk to the users that lock the resource, or whoever. In the grammar this is common:`LOCKS AND STOPS AT Semaphore1`, or for a foreign stop as `LOCKS SemaphoreAilleurs2`, and the other resource simply `STOPS AT SemaphoreAilleurs2`. This is quite good approach for organizations (or sub-systems used by) of about 200–500人, which is the scenario we’re aiming for.

N.B. on "nevertheless you always end up doing some of this anyway": databases have long used unlocking mechanisms such as MVCC in which a coherent version is always presented to users. While this is enough at logical level, experience say this is not the case at a organizational or business level.

Key bindings and other things are custom. The reason is that, computationally speaking, we come from 1992—maybe 1996—and we love to **invent things**, not just copy.

All operations are granted to the admin group. There will be folders for each role that include files with public keys; in the grammar, `GRANTED TO Admin` is the default even if it doesn’t appear explicitly.

Some runtime checks are a must, mostly to avoid affecting multiple rows at once. There will be mechanisms for that.

## Gazes

```
CREATE [ROOT] GAZE Gaze1 DESC 'Describing text' [GRANTED TO Role1] [LOCKS [AND STOPS AT] Semaphore1 | STOPS AT Semaphore1]
EXPORTS Param1, Param2 TO GAZE Gaze3 DESC 'Describing how to use that params in the next view'
EXPORTS Param2 TO GAZE Gaze5 DESC 'Describing how to use that params in the next view'
EXPORTS Param1 TO OPERATION Operation3 DESC 'Describing the same as above'
```

_Gazes_ also have default sorting and **picked** meaningful filters

Navigation is just a Tree.

Options list: source ID → target ID
Import: params as #, so they must fit

Push the query into the breadcrumb.

Now we have a navigation tree with a breadcrumb, state handling, a reload mechanism, etc. all compound in a **semantic way**

`.sql` files containing the queries for the view and marked `{ % 1 % }` the position of the exported parameter as the receiver
```
AS entry_distribution ON entry_distribution.id = { % 1 % };
```

Antecedents / State of the art: _"ORM Is an Offensive Anti-Pattern", Yegor Bugayenko 1 Dec. 2014_ — https://www.yegor256.com/2014/12/01/orm-offensive-anti-pattern.html

_Expression_ files are used for naming the elements of that `GAZE`. Notice that **there are no “columns” as such; what we aim for is semantic induction** — basically objects with operations. So this expression description uses `expr` (https://github.com/expr-lang/expr widely used and mature) for Golang and `eval()` for Ruby clients, that depict each element of the view. With this, the whole thing is considered a _Gaze_. It might be a good idea, and very cheap timewise, to define one for each client type.

Having said that, the maximum number of objects returned is _`60`_. If you need more, maybe this isn’t your paradigm — filter, sort, or even better, choose **another semantic goal**. this is the very thing we want to demonstrate.

_Here the first iteration of the development, and then..._

---

## Validation and / Operation wizards

Testing works like asserting invariants, to ensure the data good shape
```
CREATE VALIDATION Validation1 ...
```

And the upcoming design for operations, in the future
```
CREATE [ROOT] OPERATION Operation1 ...
CREATE [ROOT] WIZARD Wizard1
```
Are step wizards, and are going to use assertions

### key-bindings

`Esc / Left`: pull\
`Top / Down`: pagination, navigate line definition\
`Right / Enter`: option list; last position per node should be remembered for agility, even when navigating (pull) backwards\
\
If the list is bigger than _`60`_: warning — you may sort or filter to add relevance, or just read and print using the default sorting\
\
`I`: field expanded info, facilities for datatypes, dates, etc.\
`D`: details about the row (now, an object)\
`G / Ctrl‑C`: get, show window, prompt comment\
`P / Ctrl‑V`: put, select from window (mostly after F, filter)\
\
`S`: sort (done in‑lang, not SQL)\
`F`: filter (same, not SQL)\
`R`: reload current with S, F\
`U`: reload unsorted, unfiltered\
`E`: export\
\
Easy spec — isn’t it.\
\
Good to have: service for triggered notifications;
Vim/Emacs‑style navigation and/or custom bindings.
