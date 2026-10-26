# Bingo Cards → JSON (instructions for the AI)

You are given one or more photos of **75-ball bingo cards** and this file. Read every card and produce **one single JSON file** that follows the format below exactly. The JSON will be imported by a program, so the format must be exact.

## Prompt the user will send with this file
> Basahin ang attached na photos ng bingo cards at sundin ang BINGO_CARDS_AI_INSTRUCTIONS.md. Ibigay ang isang JSON file lang. (Optional: dapat may ___ cards.)

---

## 1. Card layout
- Each card is a 5x5 grid with header columns **B, I, N, G, O** (left to right).
- The **center cell (row 3, column N) is FREE**. Always write it as `0`.
- Number ranges per column (a number outside its column's range is a misread):

| Column | Range |
|---|---|
| B | 1–15 |
| I | 16–30 |
| N | 31–45 |
| G | 46–60 |
| O | 61–75 |

- If a photo is rotated or skewed, mentally straighten it first. Read the header letters to confirm orientation.

## 2. Reading rules
- Read each card **row by row, left to right, top to bottom** (row 1 = top row).
- Use the column ranges above to double-check every digit (for example a "B" cell cannot be 52; re-read it, as it is probably 5 or 12).
- **Never guess.** If a digit is unreadable (glare, blur, cut off), write `null` for that cell and add an entry to `issues` (see section 4).
- Do not invent cards. Only output cards that are actually visible.
- Cards partly cut off at the edge of a photo: include them only if all 25 cells can be read; otherwise list them under `issues` and leave them out of `cards`.

## 3. Multiple photos and duplicates
- The user may send several photos, and the same physical card can appear in more than one photo.
- Merge everything into **one** `cards` list with **no duplicates**.
- Two cards are duplicates if they have the **same printed card number**, or if **all 24 non-free numbers match in the same positions**. Keep only one. If the readings differ slightly, keep the clearer reading and note the conflict in `issues`.
- Order the final list by `no` ascending.

## 4. Card numbers (`no`)
- If a card has a **printed card number or ID**, use it as an integer.
- If cards have **no printed number**, assign numbers `1, 2, 3, …` in the order the cards first appear (photo 1 first, left to right, top to bottom; then photo 2, and so on).
- Never use the same `no` twice.

## 5. Output format

Return **only** this JSON. No markdown commentary, no explanation outside the JSON. If you can create files, save it as **`bingo_cards.json`** and give the file. If you cannot create files, output one single JSON code block and nothing else.

```json
{
  "format": "bingo-cards-v1",
  "count": 2,
  "cards": [
    {
      "no": 1,
      "grid": [
        [4, 18, 33, 52, 70],
        [11, 27, 41, 48, 63],
        [2, 16, 0, 59, 75],
        [15, 22, 36, 55, 61],
        [9, 30, 44, 47, 68]
      ]
    },
    {
      "no": 2,
      "grid": [
        [1, 20, 31, 46, 62],
        [7, 25, 38, 51, 66],
        [13, 29, 0, 57, 71],
        [5, 17, 43, 60, 64],
        [14, 23, 45, 49, 74]
      ]
    }
  ],
  "issues": []
}
```

Field rules:
- `format`: always the string `"bingo-cards-v1"`.
- `count`: number of items in `cards`.
- `cards[].no`: positive integer, unique.
- `cards[].grid`: **5 rows**, each with **5 numbers**, in reading order. Row index = top to bottom, column index = B, I, N, G, O. The center `grid[2][2]` is always `0`. All other cells are integers within their column range, or `null` only if truly unreadable (a card with any `null` will be skipped by the importer, so list it in `issues`).
- `issues`: array of short strings describing anything the user should check, for example `"Card 14: row 3 column G unreadable (glare)"`, `"Photo 2 had a duplicate of card 7; kept photo 1 reading"`, `"Expected 50 cards, found 48"`. Use an empty array if there is nothing to report.

## 6. Self-check before answering
Go through this list before you output:
1. Every `grid` has exactly 5 rows of 5 values, and `grid[2][2]` is `0`.
2. Every number is in the correct column range.
3. No number repeats inside the same card.
4. No two cards share the same `no`, and no two cards have identical grids.
5. `count` equals the length of `cards`.
6. If the user told you the expected number of cards, compare it with `count` and add an `issues` entry if it does not match.
7. The output is valid JSON (double quotes, no trailing commas, no comments).
