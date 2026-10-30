# Examples

One file per kind of inscription, written exactly as it would be inscribed (content type `application/json`).
The rules for every field are in the [standard](../standard/README.md).

| File | What it is |
| --- | --- |
| `deploy-*.json` | DAO deploys: bitmap, gallery (NodeMonkes), parent (Quantum Cats, OCM Genesis with two parents), combined assets, a list of inscriptions, all ordinals, bitcoin, BRC-20 (ORDI), rune (DOG•GO•TO•THE•MOON) |
| `propose-discussion.json` | a discussion: only a title and text, no voting |
| `propose-signal.json` | a non-binding draft that refers to the discussion |
| `propose-multi.json` | a multiple-choice proposal: pick up to three |
| `rules.json` | a rules inscription: a proposal donation and a requirement for binding proposals |
| `propose-adopt-rules.json` | a proposal that adopts those rules as the DAO's first branch |
| `propose-binding.json` | a binding proposal that follows that branch |
| `vote-paid-yourself.json` | a vote without signature (way 1): counts for the address that paid the commit transaction |
| `vote-signed.json` | a signed vote (way 2); make your own with the Vote Builder in [`tools/`](../tools/) |
| `vote-multi.json` | a vote on the multiple-choice proposal |
| `msg.json`, `msg-reply.json` | a message on the discussion and an answer to another message |

The deploys use real collection, token and rune IDs. All other IDs are placeholders so you can follow the
links between the files:

| Placeholder | Stands for |
| --- | --- |
| `aaaa…aaaai0` | the DAO (its deploy inscription) |
| `bbbb…bbb1i0` … `bbbb…bbb5i0` | proposals #1 (discussion) to #5 (binding proposal) |
| `cccc…cccci0` | the rules inscription, which is also the branch ID once it is adopted |
| `dddd…ddddi0` | a message |

`bc1p…` and the signature in `vote-signed.json` are placeholders too. A real proposal in a branch with
a `donation` setting must also pay that donation in its transaction.
