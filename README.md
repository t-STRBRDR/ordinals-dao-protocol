# Ordinals DAO Protocol

An open protocol to create a DAO for any set of ordinals and run it entirely on Bitcoin: bitmaps, collections,
galleries, hand-picked inscriptions, BRC-20 tokens, runes, or bitcoin itself. Creating a DAO, proposals,
rules, votes and messages are all inscriptions.

- **No owner, no server.** Holding a DAO's deploy inscription gives no control.
- **Holders decide.** Who can vote comes only from the DAO's assets. The protocol is a framework for making
  rules, not a set of rules.
- **Everything is on-chain**, so anyone can follow and recount every DAO.

## Contents

| Path | What it is |
| --- | --- |
| [`standard/README.md`](standard/README.md) | **The standard**: deploy, proposals, rules and branches, votes, messages |
| [`examples/`](examples/) | One example inscription for every kind of op |
| [`tools/vote-builder.html`](tools/vote-builder.html) | Vote Builder: builds and checks a signed vote, all response types |
| [`tools/vote-builder-yes-no.html`](tools/vote-builder-yes-no.html) | The same, for yes / no / abstain only |
| [`tools/HOW-TO-VOTE.md`](tools/HOW-TO-VOTE.md) | Step by step: a correct vote with UniSat, Sparrow, ord and other wallets |

The Vote Builder pages are single HTML files with no libraries and no server. Download one and open it in your
browser; nothing you enter leaves the page.

## Status

The standard is a draft and may still change. You can create DAOs, proposals and votes today. An indexer that
counts the results is the next step.

## License

[MIT](LICENSE)
