# Ordinals DAO Protocol

## About

This is a protocol that allows anyone to set up a DAO (Decentralized Autonomous Organisation) and/or proposal for any set of ordinals and run it entirely on Bitcoin. That can be Bitmaps, one or more collections, galleries, a hand-picked list of inscriptions, BRC-20 tokens and Runes. 

As of now no protocol existed to allow any community of inscription holders to align, discuss organize and make decisions immutable, publicly, inscribed forever on the Bitcoin network. This Ordinals DAO protocol (dao) enables anyone to set up such an organisation onchain. 

The aim is to privide an infrastructure for communities to connect, exchange, organise, align, elaborate sentiment, find consensus, etc, on the bitcoin network.

This open source protocol leaves any ruleset open to be defined within DAO proposals. In its basic form, how this protocol used or not used is completely open for any community to decide. This means DAO voting results are not necessarily binding - but they can be, if in such way defined rules and contents of a proposal pass its defined thresholds. 

Such DAO can be inscribed by anyone: collection founders, asset holders, and even unrelated entities. The protocol does not define how or whether an offer is to be taken seriously. This will be decided by the group that is admitted to participation based on the "asset" definition.

Per default, ownership of any Ordinals DAO inscription does not grant any authority over any DAO, its deployment or proposal. Anyone can own a DAO inscription, but no one can own the DAO itself. 

Any of such rules though may or may not be defined via rules set by a respective proposal. This means, any DAO deployment or proposal might be inscribed by anyone. Bound/commited/associated with it will only ever be eligible asset inscription holders who voted for it. 

- **Everything is an inscription.** Creating a DAO (`deploy`), making a proposal (`propose`), voting (`vote`)
  and discussing (`msg`) are all inscriptions, so the whole history of a DAO lives on-chain. Discussions do not necessarily need to happen onchain.
- **No owner, no server.** Holding the deploy inscription gives no control. A DAO's rules can only change
  through proposals that the holders pass, and every change makes a new branch (section 7).
- **Holders decide.** Who can vote comes only from the `assets` in the deploy. Which DAO matters comes only
  from people taking part in its votes.
- **Anyone can verify.** Results follow from Bitcoin data alone, so any indexer or browser page can recount
  every vote and get the same answer.


## In this repo

| Path | What it is |
| --- | --- |
| this file | **The protocol**: deploy, proposals, rules and branches, votes, messages |
| [`examples/`](examples/) | One example inscription for every kind of op |
| [`tools/vote-builder.html`](tools/vote-builder.html) | Vote Builder: builds and checks a signed vote, all response types |
| [`tools/vote-builder-yes-no.html`](tools/vote-builder-yes-no.html) | The same, for yes / no / abstain only |
| [`tools/HOW-TO-VOTE.md`](tools/HOW-TO-VOTE.md) | Step by step: a correct vote with UniSat, Sparrow, ord and other wallets |

The Vote Builder pages are single HTML files with no libraries and no server. Download one and open it in your
browser; nothing you enter leaves the page.

**Status:** this protocol is a draft and may still change. You can create DAOs, proposals and votes today. An
indexer that counts the results is the next step.

## 1. What a deploy is

A DAO is created by one inscription that says only two things: the DAO's **name** and **who can vote**.

```json
{"p":"dao","op":"deploy","name":"bitmap.dao","assets":[{"type":"bitmap"}]}
```

- It contains **no rules**. Everything else comes with proposals (section 6).
- **Holding the deploy inscription gives no control** over the DAO. It's only the inscription.
- The **DAO ID** is the deploy's inscription ID.

## 2. Fields

All four fields are required. **Any other field makes the deploy invalid.**

| Field | Value |
| --- | --- |
| `p` | `"dao"` |
| `op` | `"deploy"` |
| `name` | The DAO's name; rules in section 3 |
| `assets` | Who can vote; rules in section 4 |

- Content type: `application/json` (`;charset=utf-8` allowed).
- The content must be one valid JSON object, UTF-8, without a byte order mark. Brotli content encoding is allowed.
- Duplicate keys make the deploy invalid.

## 3. `name`

Every name ends in `.dao` and matches one of two patterns:

| Style | Name before `.dao` | Pattern for the full name | Example |
| --- | --- | --- | --- |
| lowercase | lowercase letters, digits, `-`; 1–32 characters | `^[a-z0-9-]{1,32}\.dao$` | `bitmap.dao` |
| rune style | uppercase A–Z with the spacer `•` | `^[A-Z](•?[A-Z])*\.dao$` | `DOG•GO•TO•THE•MOON.dao` |

- **A DAO is name + assets.** The first valid deploy of a combination of name and assets, in Bitcoin order
  (section 6.2), is the DAO. Every later deploy with the same name and the same assets is invalid.
- **Same name, other assets:** valid, but a **different DAO**. Being inscribed first does not make it the DAO
  that people mean by that name. Example: `nodemonke.dao` with the NodeMonke collection and `nodemonke.dao`
  with another collection are two DAOs. `monke.dao` with the NodeMonke collection is a third one.
- **Names are compared case-insensitively.** `dog.dao` (lowercase) and `DOG.dao` (rune style) with the same
  assets are the same DAO. Spacers count: `DOG•GO.dao` and `DOGGO.dao` are different names. Similar names are allowed.
- **Assets are compared by what they are, not how they are written** (section 4.1).
- **Who can vote is decided only by `assets`** (section 4), not by the name. The protocol can't tell which
  combination of name and assets is the "right" one; the community decides by taking part in a DAO's votes.
  Tools therefore always show the name **together with** the assets (e.g. "nodemonke.dao · Parent: NodeMonke"),
  never the name alone.
- The **DAO ID** is the deploy's inscription ID.
- **Rune style is a convention** for DAOs around a rune: use the rune's full name exactly as etched, with its
  spacers, e.g. `DOG•GO•TO•THE•MOON.dao` for rune `840000:3`. It's expected that the correctly written name
  becomes the one people use, but it is not checked against `assets`.
- Rune style uses **uppercase A–Z** and the spacer **`•` (U+2022 BULLET)**. No other characters, and no spacer
  at the start, at the end, or twice in a row. In JSON, the spacer may be written as the character `•` or as
  `\u2022`. Both are the same once the JSON is parsed.

## 4. `assets`

`assets` is a **non-empty list of entries**. Each entry names one group of inscriptions or tokens, and a DAO
can combine any number of them: several collections, galleries, bitmaps, tokens, or a plain list of
inscriptions. That way a DAO can be made for any set of ordinals.

```json
"assets": [
  {"type": "bitmap"},
  {"type": "gallery", "ids": ["<gallery inscription ID>"]},
  {"type": "inscriptions", "ids": ["<inscription ID>", "<inscription ID>"]}
]
```

Each entry is an object with `type` and, except for `bitmap`, `all` and `bitcoin`, `ids`. `ids` is always a **non-empty list of
strings**, even when there is only one. No other keys are allowed.

For parent and gallery collections, copy the anchor from the registry
[`collections.json`](https://github.com/TheWizardsOfOrd/ordinals-collections). It uses the same type names,
and a single `id` there becomes `ids: [id]` here.

| `type` | `ids` | Eligible on a proposal confirmed at block `N` |
| --- | --- | --- |
| `bitmap` | none (omit `ids`) | valid bitmaps (section 4.2) with block number `n ≤ N`, confirmed at height ≤ `N` |
| `parent` | 1 or more collection parent inscription IDs | children of any listed parent, confirmed at height ≤ `N` |
| `gallery` | exactly 1 gallery inscription ID | inscriptions listed in that gallery's items |
| `multi_gallery` | 2 or more gallery inscription IDs | inscriptions listed in any of these galleries |
| `inscriptions` | 1 or more inscription IDs | exactly the listed inscriptions |
| `all` | none (omit `ids`) | **every inscription** confirmed at height ≤ `N`: a DAO for all of ordinals |
| `brc20` | exactly 1: the token's **deploy inscription** ID | addresses holding a balance of that token at the end of block `N` |
| `rune` | exactly 1: the **rune ID** `block:tx` of its etching | addresses holding a balance of that rune at the end of block `N` |
| `bitcoin` | none (omit `ids`) | **every address holding at least 1 satoshi** at the end of block `N` |

- **The eligible set is the union of all entries.** An inscription that is eligible through more than one
  entry (e.g. it is in a gallery and also in the list) counts **once**.
- **How much each item's vote is worth is not part of the deploy.** The suggested default is **1 vote per
  eligible item**, and it applies until a branch's rules define something else (section 7).
- Inscription IDs match `^[0-9a-f]{64}i[0-9]+$`. Rune IDs match `^[1-9][0-9]*:[0-9]+$`.
- Gallery anchors are pinned. Anyone can inscribe another gallery for the same items, but only the IDs in the
  deploy count.
- **Balances** (`brc20`, `rune`, `bitcoin`) are taken at the end of block `N`, because tokens and sats can't be told apart
  and would otherwise be counted again after a transfer:
  - BRC-20: the address's total balance (available plus transferable) under the standard BRC-20 indexing rules.
  - Runes: the address's balance of that rune as ord indexes it.
  - Bitcoin: the address's total balance in sats, i.e. the sum of all its unspent outputs.
  - By default an address gets **1 vote per whole token** (rounded down), and for `bitcoin` **1 vote per
    satoshi**, until the rules set something else (e.g. 1 vote per address).
- **`all`** makes every inscription eligible. Whoever holds many cheap inscriptions then has many votes;
  `max_votes_per_address` or the DAO's rules can limit that.
- **`bitcoin`** makes every Bitcoin holder eligible: a DAO for all of Bitcoin. Counting it needs an index of
  address balances (e.g. electrs), which Bitcoin Core and ord don't keep. Splitting sats over many addresses
  doesn't add votes with the default of 1 vote per satoshi; a rule like 1 vote per address would invite that.

### 4.1 When two `assets` are the same

The underlying assets count, not the way they are written. Two `assets` are the same if they are equal after
these steps:

1. The order of entries and of IDs doesn't matter, and an ID listed twice counts once.
2. Entries of the same type are merged: `parent [A]` + `parent [B]` is the same as `parent [A, B]`.
3. `gallery` and `multi_gallery` are both lists of galleries: `gallery X` + `gallery Y` is the same as
   `multi_gallery [X, Y]`.
4. `all` already covers every inscription: `all` combined with `bitmap`, `parent`, `gallery`, `multi_gallery` or
   `inscriptions` entries is the same as `all` alone. Token and `bitcoin` entries stay separate.

A parent and a gallery for the same collection stay **different**, because their items can differ over time
(new children, a changed gallery).

### 4.2 Valid bitmaps

A bitmap inscription for block `n` is **valid** when:

- its content type is `text/plain` (`;charset=utf-8` allowed);
- its content is exactly `<n>.bitmap`, matching `^(0|[1-9][0-9]*)\.bitmap$` (no leading zeros, no spaces, no
  line break);
- `n` is at most the height of the block that confirmed the inscription;
- it is the **first** inscription in Bitcoin order (section 6.2) that meets these rules for that `n`.

This follows the bitmap community standard. Later inscriptions of the same `<n>.bitmap` are not bitmaps.

## 5. Examples for name/asset aligned DAOs

**Bitmap**
```json
{"p":"dao","op":"deploy","name":"bitmap.dao","assets":[{"type":"bitmap"}]}
```

**Gallery collection: STARBREEDERS** (ID from `collections.json`)
```json
{"p":"dao","op":"deploy","name":"starbreeder.dao","assets":[{"type":"gallery","ids":["d40289abd05b3194ddea74fe396fecac7fd6b07aa0ddb826bee1c66d3e75d1bfi0"]}]}
```

**Parent collection: Quantum Cats** (ID from `collections.json`)
```json
{"p":"dao","op":"deploy","name":"quantumcats.dao","assets":[{"type":"parent","ids":["0e383b8af3e7f8767bc9ec0a48fbf837d82b0d537f4dbc7a8853e6828112ea41i0"]}]}
```

**Parent collection with two parents: OCM Genesis by OnChainMonkey** (IDs from `collections.json`)
```json
{"p":"dao","op":"deploy","name":"ocm.dao","assets":[{"type":"parent","ids":["fb162a46943e5d7d31d72ee2c8c850e66c1ca5d0d453068aa63883528285ed21i0","14bc4de4887332f8d5a5a274874a33e6c7f4b9e5c972e1ce0d41fd450f7660fdi0"]}]}
```

**Gallery collection: NodeMonkes** (ID from `collections.json`)
```json
{"p":"dao","op":"deploy","name":"nodemonkes.dao","assets":[{"type":"gallery","ids":["2ebbd9b93006b69714dd517fbe0d4bf7f8462ffe213105e03048d49ed46eba04i0"]}]}
```

**Several groups combined: bitmaps, NodeMonkes and Quantum Cats**
```json
{"p":"dao","op":"deploy","name":"blocks-and-friends.dao","assets":[{"type":"bitmap"},{"type":"gallery","ids":["2ebbd9b93006b69714dd517fbe0d4bf7f8462ffe213105e03048d49ed46eba04i0"]},{"type":"parent","ids":["0e383b8af3e7f8767bc9ec0a48fbf837d82b0d537f4dbc7a8853e6828112ea41i0"]}]}
```

**A plain list of inscriptions** (IDs shortened here; in a deploy they are written out in full)
```json
{"p":"dao","op":"deploy","name":"my-picks.dao","assets":[{"type":"inscriptions","ids":["6fb9…a1i0","0c3e…77i0","b245…19i3"]}]}
```

**BRC-20: ORDI** (the ORDI deploy inscription; its content is
`{"p":"brc-20","op":"deploy","tick":"ordi","max":"21000000","lim":"1000"}`)
```json
{"p":"dao","op":"deploy","name":"ordi.dao","assets":[{"type":"brc20","ids":["b61b0172d95e266c18aea0c624db987e971a5d6d4ebc2aaed85da4642d635735i0"]}]}
```

**Bitcoin: every holder of at least 1 satoshi**
```json
{"p":"dao","op":"deploy","name":"bitcoin.dao","assets":[{"type":"bitcoin"}]}
```

**Rune: DOG•GO•TO•THE•MOON** (etched in block 840000, transaction 3:
`e79134080a83fe3e0e06ed6990c5a9b63b362313341745707a2bff7d788a1375`)
```json
{"p":"dao","op":"deploy","name":"DOG•GO•TO•THE•MOON.dao","assets":[{"type":"rune","ids":["840000:3"]}]}
```

## 6. Proposals

A proposal opens a discussion or puts something to a vote. It takes its vote settings from the rules
of a branch, and can adopt new rules (section 7).

```json
{
  "p": "dao",
  "op": "propose",
  "dao": "<DAO deploy inscription ID>",
  "kind": "binding",
  "title": "Fair claiming of new blocks",
  "body": "## Summary\nFull proposal text in Markdown …",
  "refs": ["<discussion ID>", "<draft ID>"],
  "choices": ["yes", "no", "abstain"],
  "duration": 1008,
  "quorum_pct": "5"
}
```

### 6.1 Fields

| Field | Required | Value |
| --- | --- | --- |
| `p`, `op` | yes | `"dao"`, `"propose"` |
| `dao` | yes | the DAO's ID (its deploy inscription ID) |
| `kind` | no | `"discussion"`, `"signal"` or `"binding"` (section 6.5). Missing means `"signal"` |
| `title` | yes | 1–200 characters |
| `body` | yes | the proposal text, Markdown, at most 100,000 bytes (UTF-8, before compression) |
| `refs` | no | 1–20 IDs of proposals or messages of the same DAO that this proposal builds on (section 6.6) |
| `type` | no | how votes are given: `"single"` or `"multi"` (section 6.7). Missing means `"single"` |
| `choices` | yes, except discussion | 2–10 different strings, each 1–32 characters. `"abstain"`, if present, counts toward quorum but never wins |
| `max_choices` | only with `"multi"` | how many choices a voter may pick, an integer from 2 to the number of choices |
| `duration` | yes, unless the branch sets it; never in a discussion | voting window length in blocks, an integer ≥ 1 (1008 ≈ one week) |
| `quorum_pct` | yes, unless the branch sets it; never in a discussion | the minimum share of all possible votes that must take part, as a decimal string from `"0"` to `"100"` |
| `max_votes_per_address` | no, and not if the branch sets it | an integer ≥ 1, or `null` / missing for no cap |
| `branch` | no | the branch whose rules this proposal follows (section 7.4). Missing means no rules |
| `adopt` | no | a rules ID; if the proposal passes, those rules become a new branch (section 7.2) |

- Content type `application/json`. Brotli content encoding is recommended, because long texts become much cheaper.
- Any other field makes the proposal invalid, and so does a `dao` that isn't a valid deploy.
- A `discussion` has **no** `type`, `choices`, `max_choices`, `duration`, `quorum_pct`, `max_votes_per_address`,
  or `adopt`. If any of them is present, it is invalid.
- **Vote settings come from the branch.** `duration`, `quorum_pct` and `max_votes_per_address` are taken from
  the rules of the proposal's branch (section 7.5), because those rules were already accepted. A proposal
  states a vote setting itself only if its branch doesn't set it, e.g. a proposal without a branch.
- If a proposal states a vote setting that its branch sets, the proposal is **invalid**, so a proposal can
  never show different values than the ones that apply.
- The content stays with the proposal: `title`, `body`, `kind`, `type`, `choices` and `max_choices`.
- Where nothing is set, there are no minimums; the holders decide what to take seriously.
- The **proposal ID** is its inscription ID. Votes and messages reference this ID.

### 6.2 Order and numbering

1. All valid proposals of a DAO, of every kind, are put in **Bitcoin order**: block height, then the
   transaction's position in the block, then the order of inscriptions inside the transaction. For every
   inscription since block 824,544 this is the same as inscription-number order, but anyone can check it from
   Bitcoin alone.
2. That order gives the proposal numbers: the DAO's first valid proposal is **#1**, the next is **#2**, and so on.
   Proposals carry no self-chosen number, so numbers can't be faked or duplicated.
3. Proposal #1 is the DAO's **genesis proposal**, whatever its kind.

### 6.3 Voting window and result

- `N` = the block in which the proposal is confirmed.
- Votes count if they are confirmed in blocks `N+1` to `N+duration`.
- Eligibility is decided at `N` (section 4).
- The result is final 6 blocks after the window closes.
- **Quorum is met** when the eligible items that took part (including `abstain`) are at least `quorum_pct` percent
  of all items eligible at `N`.
- **Single choice:** the choice with the most votes **wins**, if quorum is met, it isn't `abstain`, and no other
  choice has the same number of votes. Otherwise there is no winner.
- **Multiple choice:** the result is the number of votes for each choice. Choices are ranked by votes. How many
  of them are taken (e.g. "the top three") is up to the proposal's text.
- A proposal **passes** when quorum is met and its winner is `"yes"`.

### 6.4 Rules proposals

Proposals that change rules are described in section 7: rules are inscribed separately, a proposal adopts them,
and each adopted set of rules is a new branch.

### 6.5 Proposal kinds and binding

| `kind` | What it is | Voting |
| --- | --- | --- |
| `discussion` | Opens a topic: a question or an idea. Only text; people answer with messages (section 9) and drafts. | none |
| `signal` | A non-binding vote: drafts, opinion polls, temperature checks. | yes, never binding |
| `binding` | A decision that is meant to bind the DAO. | yes, binding only under the rules |

- **Nothing is binding by itself.** A `binding` proposal is binding only if its branch's rules (section 7) have a
  `binding` setting, the proposal meets all its requirements, and it passes. Otherwise it is counted and shown
  as **non-binding**.
- **Binding for whom:** a binding proposal binds its own branch. What that means for the DAO as a whole is up to
  the DAO's own rules and its holders (section 7.6).
- The `binding` setting can be, for example:
  ```json
  "binding": {"min_quorum_pct": "10", "min_duration": 1008, "requires_signal": true}
  ```
  `requires_signal: true` means a binding proposal must list in `refs` a `signal` proposal of the same branch
  that passed and whose result was final before `N`. That way every binding decision goes through a discussion and
  a draft first.
- **Adopting rules without a branch is the exception:** with no rules, a plain pass is enough (section 7.2), so
  the first branches can start at all.

### 6.6 Links (`refs`)

`refs` connects a proposal or a message to the discussion, drafts and messages it builds on, so the path from
an idea to a decision is traceable on-chain.

```
#12 discussion   "How should we fund development?"
 ├─ message        "I think a donation per proposal …"          (proposal: #12)
 │   └─ message    "Agreed, but 10k sats is too much"           (reply)
 ├─ #13 signal     Draft A: 10k sats per proposal               (refs: [#12])
 ├─ #14 signal     Draft B: 5k sats + max 3 per address         (refs: [#12, <message ID>])
 └─ #16 binding    Final version of Draft B                     (refs: [#12, #14])
```

- Each ref is an inscription ID (`^[0-9a-f]{64}i[0-9]+$`). Anything else makes the proposal invalid.
- Refs that don't point to a proposal or message of the same DAO are ignored. They only matter where the rules
  use them (`requires_signal`).
- A contribution that should be voted on is inscribed as a **`signal` proposal** (a draft) that refers to the
  discussion, not as a message.

### 6.7 Response types

| `type` | Voter picks | `choice` in the vote | Counting | Status |
| --- | --- | --- | --- | --- |
| `single` | exactly one choice | `"yes"` | each item counts for that one choice | defined |
| `multi` | 1 to `max_choices` choices | `["Explorer", "Art grant"]` | each item counts **fully for every** picked choice | defined |
| `ranked` | an order of choices | `["b", "a", "c"]` | ranked choice: the weakest is dropped round by round | reserved |
| `split` | votes divided between choices | `{"a": 7, "b": 5}` | the voter's items are split as given | reserved |

A multiple-choice proposal:
```json
{"p":"dao","op":"propose","dao":"<DAO ID>","kind":"signal","title":"Which three projects should we fund?","body":"…","type":"multi","choices":["Explorer","Wallet plugin","Art grant","Education","Meetup"],"max_choices":3,"duration":1008,"quorum_pct":"5"}
```

- `abstain` can't be combined with other choices.
- An item that votes for three choices still counts **once** toward quorum.

## 7. Rules and branches

**The protocol gives a framework for making rules, not a set of rules.** It defines how rules are written, how
they are adopted, how they build on each other, and how proposals refer to them. What the rules say, what
majorities they need, and which branch a community follows is up to each DAO.

Rules and proposals are two different things:

- **Rules** (`op:"rules"`) are the content: which settings are added or changed, and which branch they build
  on. A rules inscription on its own is only a draft and has no effect.
- **Proposals** decide. A proposal belongs to a **branch** and follows its rules. A proposal can **adopt** a
  rules inscription; if it passes, those rules become a new branch.

Together they form a tree of branches, like a family tree:

```
(no rules)
 ├─ R1  adopted: "donation 10k sats"                    → branch R1 = {donation 10k}
 │   ├─ R2  adopted in R1: "+ binding needs 10%"         → branch R2 = {donation 10k, binding 10%}
 │   │   └─ R4 adopted in R2: "donation 5k instead"      → branch R4 = {donation 5k, binding 10%}
 │   └─ R3  adopted in R1: "+ max 3 votes per address"   → branch R3 = {donation 10k, max 3}
 └─ R5  adopted: "1 vote per address"                   → branch R5 = {1 per address}
```

### 7.1 The rules inscription

```json
{"p":"dao","op":"rules","dao":"<DAO ID>","base":"<branch ID>","set":{"donation":{"address":"bc1p…","min_sats":10000}}}
```

| Field | Required | Value |
| --- | --- | --- |
| `p`, `op` | yes | `"dao"`, `"rules"` |
| `dao` | yes | the DAO's ID |
| `base` | yes | the branch these rules build on (its ID, section 7.2), or `null` to build on no rules |
| `set` | one of the two | settings to add or overwrite: an object of `name: value` |
| `unset` | one of the two | names of settings to remove: a list of strings |

- Setting names match `^[a-z0-9_]{1,32}$`. The known settings are in section 7.5. Other names are allowed and
  carried along; the protocol ignores them, but communities and tools can use them.
- Content type `application/json`. Any other field makes the rules inscription invalid.
- **Anyone can inscribe rules.** They do nothing until a proposal adopts them.
- The **rules ID** is its inscription ID.
- **Recursion:** a setting's value can be an inscription ID, e.g. the tally code `core`. Tools then load that
  inscription with `/content/<ID>`, so it is inscribed once and used by many branches without being copied.

### 7.2 Adopting rules: how a branch is made

A proposal adopts rules with `"adopt": "<rules ID>"`. It must:

- have `kind` `"binding"`, `type` `"single"`, and both `"yes"` and `"no"` among its choices;
- be in the branch the rules build on: its `branch` equals the rules' `base` (both missing / `null` for no
  rules);
- come after the rules inscription in Bitcoin order, and belong to the same DAO.

If the proposal **passes** and meets the `binding` setting of its branch (if there is one), the rules become a
**branch** once the result is final. The **branch ID** is the rules ID.

- With no rules (`base: null`), a plain pass is enough: quorum is met and "yes" wins. That's how the first
  branches can start at all.
- If several proposals adopt the same rules, the first one to pass (in order of final results) makes the
  branch. Later ones change nothing.
- Rules that are never adopted stay on-chain as drafts. A later proposal can still adopt them.
- **A branch never changes.** New rules always make a new branch, so a proposal's rules can't change while it
  runs.

### 7.3 A branch's rules

A branch's rules are all settings along its path from the root, applied in order: each rules inscription
**adds or overwrites** the settings in `set` and removes those in `unset`. In the tree above, R4 has
`donation 5k` (R4 overwrites R1) and `binding 10%` (from R2).

### 7.4 Proposals in a branch

- A proposal names its branch with `"branch": "<branch ID>"`. The branch must have become final **before** `N`
  and belong to the same DAO; otherwise the proposal is invalid.
- The proposal is decided under that branch's rules.
- **Without `branch`**, a proposal has no rules: no donation, 1 vote per eligible item, nothing binding, and it
  states its own vote settings.
  Only adopting rules with `base: null` works there.

### 7.5 Known settings

A DAO doesn't have to use any of these. They are shared names, so that every tool reads a setting the same way
when a branch does use it. A branch can set them to anything, change them, or remove them again with `unset`,
e.g. by a simple majority if its own rules allow that.

| Setting | Value | Effect on proposals in the branch |
| --- | --- | --- |
| `donation` | `{"address": "bc1p…", "min_sats": 10000}` | the proposal's transaction must pay at least `min_sats` to `address`, otherwise the proposal is invalid |
| `binding` | `{"min_quorum_pct": "10", "min_duration": 1008, "requires_signal": true}` | the requirements a `binding` proposal must meet to be binding (section 6.5) |
| `duration` | an integer ≥ 1 | the voting window in blocks for every proposal in the branch |
| `quorum_pct` | a decimal string from `"0"` to `"100"` | the quorum for every proposal in the branch |
| `max_votes_per_address` | an integer ≥ 1 | the cap per address for every proposal in the branch |
| `weights` | to be defined (section 10) | votes per `assets` entry, instead of 1 vote per item |
| `core` | an inscription ID | the tally code used for results in this branch |

### 7.6 Which branch counts

The protocol doesn't decide which branch is the "right" one, and doesn't rank branches.

- Several branches can exist side by side, and anyone can build on any of them.
- A DAO can define in its own rules what makes a branch binding for everyone, e.g. a threshold of participation
  or of yes votes. The protocol only counts the votes; the meaning comes from the rules.
- Legitimacy grows naturally: branches gain legitimacy by naturally forming long chains of adopted rules and many proposals.
- Tools show the whole tree: each branch with its chain of rules, its proposals and their participation, so
  everyone can see which branches are alive.

## 8. Votes

```json
{"p":"dao","op":"vote","proposal":"<proposal ID>","choice":"yes"}
```

### 8.1 Fields

| Field | Required | Value |
| --- | --- | --- |
| `p`, `op` | yes | `"dao"`, `"vote"` |
| `proposal` | yes | the proposal's inscription ID (not its inscription number) |
| `choice` | yes | for `single`: one of the proposal's choices. For `multi`: a list of 1 to `max_choices` different choices |
| `addr` | only with `sig` | the voter's address |
| `sig` | no | BIP-322 signature by `addr` (section 7.2, way 2) |
| `n` | no | an integer ≥ 1, only with `sig`. Missing means `1` |

- Content type `application/json` or `text/plain` (`;charset=utf-8` allowed), so inscription services that only
  offer text work too.
- Any other field makes the vote invalid. So does a `proposal` that is a `discussion` or not a valid proposal,
  or a choice that doesn't fit the proposal's `type` (unknown choice, too many, a choice twice, `abstain`
  combined with others).
- A vote counts only if it is confirmed inside the proposal's window (section 6.3).

### 8.2 Who voted: two ways

**Way 1, pay yourself.** Without `sig`, the vote counts for the address that paid for the inscription: the
address of the **first input of the commit transaction** (the transaction that created the output the reveal
spends to make the inscription). Inscribe it from your own wallet. Inscription services don't work for this,
because they pay from their own address.

**Way 2, signature.** With `addr` and `sig`, the vote counts for `addr`, whoever paid. This works with
inscription services and any wallet that can sign a BIP-322 message. The signed message is exactly these four
lines, joined with `\n` and without a trailing newline:

```
DAO vote
proposal: <proposal ID>
choice: <choice>
n: <n>
```

- `<choice>` is the choice as compact JSON, exactly as `JSON.stringify` writes it: `"yes"` or
  `["Explorer","Art grant"]`.
- An invalid signature makes the vote invalid. It does not fall back to way 1.
- **Replay protection:** a signed vote counts only if its `n` is higher than the `n` of every earlier signed vote
  from the same address for the same proposal. So nobody can re-inscribe an old signature to undo a newer vote.

### 8.3 Counting

1. Votes are taken in Bitcoin order (section 6.2).
2. **Per address, only the latest valid vote counts.** A new vote replaces the earlier one completely.
3. **Each eligible item counts once:** for the latest of these counted votes whose address held the item at the
   end of the block in which the vote was confirmed.
4. An item is worth 1 vote, unless the proposal's branch sets `weights` (section 7.5).
5. **Balances** (`brc20`, `rune`, `bitcoin`): each address counts with its balance at the end of block `N` (section 4), for
   its latest counted vote.
6. `max_votes_per_address`, if set, caps how many votes one address can count with.

## 9. Messages

A message is an on-chain contribution to a discussion: a comment, a position, or an answer to another message.
Messages are not counted in any vote.

```json
{"p":"dao","op":"msg","proposal":"<proposal ID>","text":"I support this, but the duration is too short."}
```

As an answer to another message:
```json
{"p":"dao","op":"msg","proposal":"<proposal ID>","reply":"<message ID>","text":"Agreed, 2016 blocks would be better."}
```

| Field | Required | Value |
| --- | --- | --- |
| `p`, `op` | yes | `"dao"`, `"msg"` |
| `proposal` | one of the two | the proposal (of any kind) the message is about |
| `dao` | one of the two | the DAO ID, for a message that isn't about one proposal |
| `reply` | no | the message ID this one answers |
| `refs` | no | 1–20 IDs it builds on, as in section 6.6 |
| `text` | yes | plain text or Markdown, at most 10,000 bytes (UTF-8, before compression) |
| `addr`, `sig` | no | a BIP-322 signature, as for votes |

- Content type `application/json` or `text/plain`. Any other field makes the message invalid.
- **Anyone can write messages.** The author is found the same two ways as for votes (section 8.2). For way 2 the
  signed message is:
  ```
  DAO message
  <"proposal" or "dao">: <ID>
  text: <text>
  ```
- Each message costs fees, which limits spam. Because the author's address is known, tools can show whether the
  author holds eligible items, and can show only holders' messages if the reader wants.
- The **message ID** is its inscription ID. Messages are shown in Bitcoin order.
- Off-chain boards (like the bitmapdao.org board) stay the place for everyday talk. Messages are the permanent,
  public record.

## 10. Open

- **Who may propose.** Options:
  - (a) **anyone**, including people who hold none of the DAO's assets;
  - (b) only holders, proven by inscribing the proposal as a child of one of the DAO's eligible assets.
  Until decided, indexers accept proposals from anyone and show whether the proposer is a holder.
- **The `weights` setting:** how votes per `assets` entry are written (section 7.5).
- **The `core` setting:** the format of the tally code and how tools run it.
- **Ranked and split voting:** the exact counting rules for `ranked` and `split` (section 6.7). Until they are
  defined, proposals using them are invalid.
- **Voting on messages:** whether messages can get simple support votes. Until then, a contribution that should
  be voted on is inscribed as a `signal` proposal.

## License

[MIT](LICENSE)
