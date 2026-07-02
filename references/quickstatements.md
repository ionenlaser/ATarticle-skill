# Wikidata enrichment via QuickStatements (V1 commands)

After writing the article you will usually have learned facts that are **not yet on
the Wikidata item** — most often the troupe's founding year, alternative names, and
its social-media profiles. This file is the reference for turning those into
QuickStatements **V1** commands and a ready-to-run batch URL.

The target is QuickStatements 3.0 (`qs-dev.toolforge.org`), which runs V1 commands
through the Wikibase REST API. Everything below is the V1 subset that tool accepts.

## When to generate statements

Only propose a statement when **all** of these hold:

1. The fact is supported by a source you actually read (the same sources you cited
   in the article).
2. The item does **not already carry it** — re-read the `get_statements` output
   from Step 1 and skip anything already present (QuickStatements silently ignores
   exact duplicates, but proposing them just adds noise for the reviewer).
3. The value is a stable property of the group, not a transient detail.

Never invent an identifier or a date to fill a slot. A missing statement is fine; a
wrong one is a defect a human now has to find and revert.

## V1 line format

One command per line. Fields are separated by a TAB (or a `|`). The first field is
the **item id** (you already have it — the input QID), then the property, then the
value, then optional qualifier and source pairs.

```
<QID>	<PROPERTY>	<VALUE>	[<SOURCE-PROP>	<SOURCE-VALUE> ...]
```

Because the item already exists, every line starts with its QID. Do **not** use the
`CREATE` / `LAST` chaining shortcut here — `LAST` is rejected by the QS 3.0 engine,
so repeat the QID on each line instead.

### Value formats

| Value kind | Format | Example |
|---|---|---|
| Item (Q-id) | bare id, no quotes | `Q1726` |
| String / URL / external id | double-quoted | `"https://www.laimerbrettl.de/"` |
| Monolingual text | `lang:"text"` | `de:"Laimer Brett'l e. V."` |
| Time | `+YYYY-MM-DDT00:00:00Z/<precision>` | `+1993-00-00T00:00:00Z/9` |

Time precision: `/9` = year, `/10` = month, `/11` = day. Use **year precision**
(`+1993-00-00T00:00:00Z/9`) unless a source gives a trustworthy exact date — and be
wary of social-media "founded on 1 January" values, which are almost always a
year-only fact dressed up as a full date. Use `+` for CE years and at least four
digits.

### Labels, aliases, descriptions

Use a language-prefixed code in the property slot, value in quotes:

- Label: `Lde "..."`, `Len "..."`
- Alias: `Ade "..."`, `Aen "..."` (several at once with a pipe: `Ade "A|B|C"`)
- Description: `Dde "..."`, `Den "..."`

These are **not statements and cannot carry references** — Wikidata does not
reference labels/aliases/descriptions. Only add an alias when it is a real
alternative name (an official short form, a former name, a common spelling), not a
typo or a translation. Don't overwrite the primary label; add variants as aliases.

### Statements with references

Append source pairs after the value, using `S` + the property number instead of
`P`. The standard web-source block is **reference URL (P854)** plus **retrieved
(P813)**:

```
Q122798558	P571	+1993-00-00T00:00:00Z/9	S854	"https://www.laimerbrettl.de/chronik/"	S813	+2026-06-30T00:00:00Z/11
```

- Use the real retrieval date (today) at day precision `/11`.
- For a second, independent reference block on the same statement, prefix the new
  block's first source with `!` (e.g. `!S854`).
- Richer reference parts exist if you need them: `S248` stated in (an item),
  `S1476` title, `S123` publisher, `S407` language of work. Keep it to URL +
  retrieved unless a part genuinely adds provenance.

**Reference the factual claims; leave identifiers and names unreferenced.**
Inception, street address, and similar sourced facts must carry a reference — the
point of the enrichment step is verifiable data. But **social-media profile
identifiers do not need a reference** (the profile is self-evidencing, and a
profile URL referencing itself is circular), and aliases/labels cannot be
referenced at all. So in practice: reference `P571` and a street-address
qualifier; do not attach `S…` pairs to social-media statements or aliases.

## Default enrichment set for amateur-theatre troupes

These are the statements worth generating by default when the research turns them
up and the item lacks them:

| Fact | Property | Value kind | Notes |
|---|---|---|---|
| Inception / founding | `P571` | time | Year precision unless an exact date is solid. Referenced. |
| Official name | `P1448` | monolingual | e.g. `de:"… e. V."`. Often already present. |
| Alternative names | `Ade` / `Aen` | alias | Short forms, former names, common spellings. No reference. |
| Official website | `P856` | string (URL) | Only if missing. |
| Street address | `P6375` | monolingual | **As a qualifier of headquarters location (`P159`)**, not a standalone statement — see below. |
| Facebook | `P2013` | string | The username in `facebook.com/<id>`. No reference. |
| Instagram | `P2003` | string | The handle without `@`. No reference. |
| X / Twitter | `P2002` | string | The handle without `@`. No reference. |
| YouTube channel | `P2397` | string | The `UC…` channel id (not the @handle). No reference. |
| Mastodon | `P4033` | string | `user@instance` form. No reference. |

Add a social-media profile only if you actually found it; do not guess a handle
from the group's name. These statements carry **no reference** (see above).

**Street address as a qualifier.** Model the address as `located at street address
(P6375)` qualifying the existing `headquarters location (P159)` statement, rather
than as its own claim. In V1 you add a qualifier by repeating the main
property/value and appending the qualifier property/value (then the reference):

```
Q122798558	P159	Q1726	P6375	de:"Sudetendeutsche Straße 40, 80937 München"	S854	"<source URL>"	S813	+2026-06-30T00:00:00Z/11
```

QuickStatements matches the existing `P159 = Q1726` statement and adds the
qualifier and reference to it. Only do this when the address is genuinely the
group's **registered seat / headquarters** — not merely a performance venue or a
rehearsal hall, which are a different thing and do not belong on `P159`. If you
only know a playing venue, leave the address out.

**Do not add person-valued statements** (founders `P112`, directors, individual
members, etc.). The people involved in a small amateur troupe almost never have
their own Wikidata items, so the value would be an unlinkable string or a redlink
that pollutes the graph. Stick to facts whose value is a date, a string/identifier,
or an item that already exists.

If during research you find a useful fact that does **not** fit the table above
(for example described at URL `P973`, a parent or member organisation, or an
identifier in an authority file), propose it for the current item **and ask the
user whether that property should become part of the skill's default set** before
baking it in. (Person-valued facts are out of scope — see above — so do not ask
about founders or members.)

## Building the batch URL

QuickStatements 3.0 accepts a pre-filled batch via:

```
https://qs-dev.toolforge.org/batch/new?v1=<percent-encoded commands>
```

Percent-encode the **entire** command block. The characters that break a URL if
left raw are the ones that matter most: `+` → `%2B` (critical — a raw `+` decodes
to a space and corrupts every date), `/` → `%2F`, `"` → `%22`, `:` → `%3A`, TAB →
`%09`, newline → `%0A`. The reliable way is to encode programmatically rather than
by hand:

```python
import urllib.parse
url = "https://qs-dev.toolforge.org/batch/new?v1=" + urllib.parse.quote(commands, safe="")
```

where `commands` is the TAB-and-newline command block. Verify the encoded `+`
appears as `%2B`, not `%20` or a literal `+`.

### Worked example

Two enrichment statements for Laimer Brett'l (`Q122798558`) — founding year (from
the chronicle) and the Facebook page — each with a reference:

```
Q122798558	P571	+1993-00-00T00:00:00Z/9	S854	"https://www.laimerbrettl.de/chronik/"	S813	+2026-06-30T00:00:00Z/11
Q122798558	P2013	"LaimerBrettl"	S854	"https://www.laimerbrettl.de/"	S813	+2026-06-30T00:00:00Z/11
```

Encoded as a batch URL:

```
https://qs-dev.toolforge.org/batch/new?v1=Q122798558%09P571%09%2B1993-00-00T00%3A00%3A00Z%2F9%09S854%09%22https%3A%2F%2Fwww.laimerbrettl.de%2Fchronik%2F%22%09S813%09%2B2026-06-30T00%3A00%3A00Z%2F11%0AQ122798558%09P2013%09%22LaimerBrettl%22%09S854%09%22https%3A%2F%2Fwww.laimerbrettl.de%2F%22%09S813%09%2B2026-06-30T00%3A00%3A00Z%2F11
```

## Presenting the result

When file tools are available, also write the human-readable command block to a
`.qs` (or `.txt`) file alongside the article so the user can inspect or paste it
into the QS text area directly. Always **show the readable commands** (so the user
can see exactly what will be written) **and** the batch URL (so they can open,
review, and run it). Make clear the user must be logged in and that QS previews the
batch before anything is saved — the URL opens a draft, it does not auto-run.
