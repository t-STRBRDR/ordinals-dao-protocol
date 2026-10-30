# How to make a vote inscription

This guide shows how to get a correct vote inscription for an Ordinals DAO proposal, step by step, with
UniSat, Sparrow, ord and other wallets. The rules behind it are in the
[Ordinals DAO Protocol](../README.md), section 8.

## In short

1. Build the text to sign: easiest with the **Vote Builder** (`vote-builder.html`).
2. **Sign** that text with the address that holds the DAO's assets.
3. Put address and signature into the Vote Builder. It checks them and gives you **`vote.json`**.
4. **Inscribe** `vote.json` with any inscription service or wallet, inside the voting window.

You don't need to inscribe from the wallet that holds your assets. The signature proves who voted, so any
inscription service works.

## 1. What a vote looks like

```json
{"p":"dao","op":"vote","proposal":"<proposal ID>","choice":"yes","addr":"bc1p…","sig":"<signature>","n":1}
```

| Field | What to put in |
| --- | --- |
| `proposal` | the proposal's **inscription ID** (64 characters, then `i` and a number, e.g. `…i0`), not its inscription number |
| `choice` | one of the proposal's choices, e.g. `"yes"`, `"no"` or `"abstain"`. For a multiple-choice proposal: a list, e.g. `["Explorer","Art grant"]` |
| `addr` | the address that holds the DAO's assets |
| `sig` | the signature of that address over the text in section 2 |
| `n` | `1` for your first vote. To change your vote later, vote again with a higher `n` (2, 3, …) |

## 2. The text you sign

Exactly these four lines, separated by single line breaks, with **no line break at the end** and no extra
spaces:

```
DAO vote
proposal: <proposal ID>
choice: "yes"
n: 1
```

- The choice is written as JSON: **with double quotes** for one choice (`"yes"`), as a list for several
  (`["Explorer","Art grant"]`, no spaces).
- **One different character makes the vote invalid**: a space, a missing quote, a line break at the end. That's
  why the Vote Builder builds this text for you and checks the signature before you inscribe.

## 3. Which address to sign with

| The DAO's assets are … | Sign with |
| --- | --- |
| **Ordinals**: bitmaps, collections, inscriptions, runes, BRC-20 | the **Taproot address (bc1p…)** that holds them: the Ordinals address in Xverse or Leather, UniSat set to Taproot, a Sparrow Taproot wallet, or ord |
| **Bitcoin** (`bitcoin` DAOs) | any address with sats: **bc1q…**, **3…** or **bc1p…** |

The vote counts for exactly this address, with what it held at the proposal's block (section 4 of the
protocol). Assets on another address of the same wallet don't count for it.

The signature must be a **BIP-322** signature. Older "legacy" signatures (BIP-137, often the default for bc1q
and 3… addresses) are not accepted.

## 4. Signing, wallet by wallet

### Using the Vote Builder

Open `vote-builder.html` (or `vote-builder-yes-no.html` for yes/no/abstain only) in your browser:

1. Enter the **proposal ID**.
2. Pick your **choice**. For a multiple-choice proposal, choose "multiple choice" and write one choice per line.
3. Leave **n** at 1, unless you're replacing an earlier vote.
4. Step 3 now shows the text to sign. Sign it with one of the wallets below.
5. Paste your **address** and the **signature**. The page shows **✓ Signature is valid** or **✗** with the
   reason.
6. Only with ✓: click **Download** to save `vote.json`.

If you change the proposal ID, the choice or `n` after signing, the text changes and you have to sign again.

### UniSat

UniSat has no screen for signing your own text, so you sign it in the browser console.

1. Unlock UniSat and switch to the account that holds the assets. For ordinals, check
   **Settings → Address Type → Taproot (P2TR)**.
2. In the Vote Builder, click **Copy UniSat console command**.
3. Open a new tab with any normal website, e.g. `https://bitmapdao.org`. UniSat doesn't work on local files.
4. Open the console: `Cmd + Option + J` (Mac) or `Ctrl + Shift + J` (Windows).
5. Paste the command and press Enter. If Chrome blocks the paste, type `allow pasting`, press Enter, and paste
   again.
6. In UniSat, click **Connect**, check the text, then **Sign**.
7. The console shows the signature in quotes, e.g. `'AUAX…Z8bh'`. Copy it **without the quotes**: right-click
   it and choose **Copy string contents**. If you don't see it, type `s` and press Enter.
8. Paste your address (from UniSat) and the signature into the Vote Builder.

When the Vote Builder runs on a website instead of a local file, it shows a **Sign with UniSat** button that
does steps 2–8 for you.

### Sparrow

Sparrow (version 2.5.1 or newer) signs BIP-322 messages for Taproot, SegWit and nested SegWit addresses. You
don't need your own node.

1. Open the wallet that holds the assets.
2. In the Vote Builder, click **Copy** next to the text to sign.
3. In Sparrow, go to the **Addresses** tab, right-click the address that holds the assets, and choose
   **Sign/Verify Message**.
4. Paste the text into the message field. Don't add a line break at the end.
5. If Sparrow offers a signature format, choose **BIP322**.
6. Click **Sign** and copy the signature.
7. Paste the address and the signature into the Vote Builder.

Hardware wallets connected to Sparrow may not be able to sign BIP-322 messages. If the Sign button is
missing or fails, use a software wallet.

### ord

`ord wallet` needs your own Bitcoin Core node and ord index.

1. Find the address that holds the assets: `ord wallet inscriptions` or `ord wallet addresses`.
2. Sign the text from the Vote Builder (replace the ID and choice with yours):
   ```
   ord wallet sign --address bc1p… --text "$(printf 'DAO vote\nproposal: <proposal ID>\nchoice: "yes"\nn: 1')"
   ```
   `printf` adds the line breaks and no line break at the end. Check `ord wallet sign --help` for the
   options of your ord version.
3. ord prints the signature as base64 (the `witness` value). Paste it with the address into the Vote Builder.
4. You can inscribe right away: `ord wallet inscribe --fee-rate <sats/vB> --file vote.json`

### Xverse and Leather

Xverse and Leather can sign BIP-322 messages, but only when a website asks them to. They have no screen for
signing your own text. The Vote Builder doesn't connect to these wallets yet, so for now sign with UniSat,
Sparrow or ord. You can import the same seed phrase into Sparrow to sign there.

### Other wallets

Any wallet works if it can sign your own text with **BIP-322** for the address that holds the assets. Check
the result in the Vote Builder: ✓ means it works. Electrum and Bitcoin Core's `signmessage` only make legacy
signatures, which are not accepted.

## 5. Inscribing `vote.json`

- **Any inscription service** works, e.g. UniSat Inscribe (→ **File**), OrdinalsBot, or `ord wallet inscribe`.
  Upload the file `vote.json` as it is.
- **Content type** must be `application/json` or `text/plain`. Services set this from the `.json` file name.
- **Timing:** the inscription has to be **confirmed** inside the voting window: from the block after the
  proposal was confirmed until `duration` blocks later. Leave room for slow confirmation and pick a fee rate
  that confirms in time.
- Where the inscription ends up doesn't matter. It doesn't have to go to the address that holds the assets.

## 6. Changing your vote

Make a new vote with **`n` one higher** than your last signed vote (2, 3, …) and inscribe it inside the voting
window. Your latest valid vote counts. A vote with the same or a lower `n` than an earlier signed vote is
ignored, so nobody can re-inscribe an old signature of yours to undo a newer vote.

## 7. Checklist before you pay

- [ ] The proposal ID is the inscription ID (`…i0`), copied in full.
- [ ] The choice is one of the proposal's choices, spelled exactly the same.
- [ ] The address is the one that **holds the DAO's assets**.
- [ ] The Vote Builder shows **✓ Signature is valid**.
- [ ] `n` is higher than in your earlier votes on this proposal.
- [ ] The voting window is still open long enough for the inscription to confirm.

## Common problems

| Message in the Vote Builder | What to do |
| --- | --- |
| `unsupported address` | Use a bc1p…, bc1q… or 3… address. Old 1… addresses are not supported. |
| `not a BIP-322 signature` | The wallet made a legacy signature. Use BIP-322 (in UniSat: the console command from the Vote Builder). |
| `signature does not match this address and text` | The text, the address or the account in the wallet differs from what you signed. Copy the text again and sign again with the right address. |
| `The proposal ID is not valid` | Copy the full inscription ID: 64 characters, then `i` and a number. |
