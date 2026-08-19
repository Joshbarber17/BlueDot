[README.md](https://github.com/user-attachments/files/31235989/README.md)
# Blue Dot

A reconciliation worklist for people who keep their budget in a spreadsheet.

Banks used to mark unread transactions with a dot that cleared once you looked at
them. When America First redesigned their portal, that indicator went away, and
with it any way to know which charges had already been entered into a spreadsheet.

Blue Dot puts that state back, on your side, where it should have been all along.
It never talks to a bank. You feed it the CSV you already download, and it tells
you what you haven't accounted for yet.

## Why it survives portal redesigns

The old dot was read-state stored at the bank. This computes the same thing
locally by fingerprinting each transaction, so no amount of redesigning on their
end can take it away.

Each row gets a fingerprint made of:

    posting date | signed amount | first 40 chars of raw description | occurrence index

The occurrence index is what keeps two identical $4.50 coffees on the same day
from collapsing into one entry. Naive deduplication silently eats the second one,
which is worse than having no indicator at all.

The fingerprint uses the **raw** bank description, not the cleaned-up merchant
name shown on screen. That way improving the display logic never invalidates
history you've already marked.

Practical upshot: re-import the same date range as often as you want. Overlap
costs nothing. Only genuinely new transactions surface.

## Setup

1. Create a repo and drop `index.html` at the root.
2. Settings → Pages → Source: Deploy from a branch → `main` / `root`.
3. Wait a minute, then open `https://<username>.github.io/<repo>/`.
4. Bookmark it.

The repo can be public. The file contains no data of any kind. Your CSV is read
in the browser and never uploaded, and everything the app remembers lives in
`localStorage` on your own machine.

## Daily loop

1. Download the transaction CSV from your bank.
2. Drop it on the page.
3. Work the queue in **One at a time**, oldest first.
4. For each one: type it into the right table in your spreadsheet, then press
   **Accounted for**.

Nothing is ever cleared by looking at it, scrolling past it, or clicking on the
card. Only the button clears a dot. **Skip for now** advances and leaves the dot
lit.

Keyboard: `Enter` accounts and advances, `↓` skips.

Clicking the merchant, amount, or date copies that single value, for the fields
that are faster to paste than retype. The amount copies as a plain number with no
currency symbol.

### First run

Don't click through months of history one at a time. Switch to **Full list**,
check off everything already entered in your spreadsheet, and mark the batch.
That seeds the memory in one pass. After that it's a handful of rows a day.

## Category memory

The first time it sees a merchant it asks which table the charge belongs in.
After that the field pre-fills itself and tells you where that merchant went last
time.

Matching is exact first, then a two-word prefix, then a one-word prefix, so
`PUBLIX SUPER MKT 1234` and `Publix Super Market` resolve to the same table.

No setup. It learns your table names as you type them.

## Pending transactions

Leave **Skip pending** checked.

Pending charges change when they post. Restaurant tips get added, gas stations
drop the authorization hold and replace it with the real amount. A different
amount is a different fingerprint, so a pending row you mark today reappears as
new once it settles. Filtering them out avoids the whole class of problem.

## Back up your memory

`localStorage` is per-browser and per-device. Clearing site data, switching
machines, or a new browser profile all wipe it.

**Back up** downloads a JSON file with every accounted fingerprint and every
learned table assignment. Keep it wherever your spreadsheet lives.

**Restore** merges a backup back in. It only ever adds, never removes, so
restoring onto a populated install is safe and restoring onto an empty one is a
full recovery.

Worth doing monthly, and worth doing before you touch anything in browser
settings.

## Column mapping

Headers are detected automatically. Correct it once and it remembers that
correction for that exact header layout.

If your bank exports separate debit and credit columns rather than one signed
amount, set the credits dropdown and the two get combined, with debits negative.

## Footer controls

| Control | What it does |
|---|---|
| Undo last | Puts the most recent batch back in the queue |
| Back up | Downloads accounted history and learned tables as JSON |
| Restore | Merges a backup file back in |
| Forget categories | Clears learned table assignments, keeps accounting history |
| Clear memory | Wipes everything. Every transaction returns as new |

## Files

    index.html    the whole application, no dependencies, no build step
    README.md     this file

One external request, for the Archivo webfont. It falls back to system fonts if
that's blocked, and nothing else about the page depends on the network.
