# Bingo Cards → JSON (instructions for the AI)

You are given one or more photos of **75-ball bingo cards** and this file. Read every card and produce **one single JSON file** that follows the format below exactly. The JSON will be imported by a program, so the format must be exact.

**Important:** the order of the cards must be kept, and the cards must be **grouped per photo**. Photo 1 = group 1, photo 2 = group 2, and so on. Inside a group, the cards stay in the same order as they sit in the photo. Do not sort them by card number.

## Prompt the user will send with this file
> Basahin ang attached na photos ng bingo cards at sundin ang BINGO_CARDS_AI_INSTRUCTIONS.md. Ibigay ang isang JSON file lang. Bawat photo ay isang group, at panatilihin ang order ng cards. (Optional: dapat may ___ cards.)

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
- **Never guess.** If a digit is unreadable (glare, blur, cut off), write `null` for that cell and add an entry to `issues` (see section 5).
- Do not invent cards. Only output cards that are actually visible.
- Cards partly cut off at the edge of a photo: include them only if all 25 cells can be read; otherwise list them under `issues` and leave them out of `cards`.

## 3. Groups and order (keep this exactly)
- **One photo = one group.** The first photo the user sends is `group` 1, the second is `group` 2, and so on, in the order the photos were given.
- Inside a group, order the cards by how they sit in the photo: **left to right, then top to bottom** (finish the top row of cards first, then the next row). Give each card a `pos` starting at 1 for the first card in that photo.
- `pos` counts every physical card slot in the photo. If a card in the photo is left out (unreadable or cut off), skip its `pos` number instead of renumbering, so the other cards keep their real positions. Gaps in `pos` are fine.
- **Never reorder** cards by card number, by value, or for any other reason. The order in the photo is the order in the JSON.
- If the photo has cards in a single column or a stack, use top to bottom. If the layout is unclear, use the most natural reading order and add a short note to `issues`.
- The user may give names for the groups (for example "Pack A"). If so, use them in `groups[].name`. Otherwise use `"Photo 1"`, `"Photo 2"`, and so on.

## 4. Duplicates and card numbers
**Duplicates**
- The same physical card can appear in more than one photo.
- Two cards are duplicates if they have the **same printed card number**, or if **all 24 non-free numbers match in the same positions**. Keep only one.
- Keep the card in the group (photo) where it is clearest. Leave its `pos` gap in the other group and add an entry to `issues`, for example `"Card 7 appears in photo 1 and photo 2; kept in group 1"`. If the readings differ slightly, keep the clearer reading and note the conflict.

**Card numbers (`no`)**
- If a card has a **printed card number or ID**, use it as an integer.
- If cards have **no printed number**, assign numbers `1, 2, 3, …` following the final order: group 1 in `pos` order, then group 2 in `pos` order, and so on.
- Never use the same `no` twice.

## 5. Output format

Return **only** this JSON. No markdown commentary, no explanation outside the JSON. If you can create files, save it as **`bingo_cards.json`** and give the file. If you cannot create files, output one single JSON code block and nothing else.

The `cards` list is ordered by `group` ascending, then `pos` ascending. It is **not** ordered by `no`.

```json
{
  "format": "bingo-cards-v2",
  "count": 4,
  "groups": [
    { "id": 1, "name": "Photo 1", "count": 2 },
    { "id": 2, "name": "Photo 2", "count": 2 }
  ],
  "cards": [
    {
      "no": 12,
      "group": 1,
      "pos": 1,
      "grid": [
        [4, 18, 33, 52, 70],
        [11, 27, 41, 48, 63],
        [2, 16, 0, 59, 75],
        [15, 22, 36, 55, 61],
        [9, 30, 44, 47, 68]
      ]
    },
    {
      "no": 7,
      "group": 1,
      "pos": 2,
      "grid": [
        [1, 20, 31, 46, 62],
        [7, 25, 38, 51, 66],
        [13, 29, 0, 57, 71],
        [5, 17, 43, 60, 64],
        [14, 23, 45, 49, 74]
      ]
    },
    {
      "no": 3,
      "group": 2,
      "pos": 1,
      "grid": [
        [3, 19, 32, 47, 65],
        [8, 21, 40, 50, 72],
        [12, 26, 0, 58, 69],
        [6, 30, 35, 53, 62],
        [10, 17, 43, 56, 74]
      ]
    },
    {
      "no": 25,
      "group": 2,
      "pos": 2,
      "grid": [
        [15, 24, 34, 49, 61],
        [2, 16, 39, 54, 67],
        [9, 28, 0, 60, 73],
        [14, 18, 42, 46, 63],
        [1, 22, 45, 51, 75]
      ]
    }
  ],
  "issues": []
}
```

(In the example, the printed card numbers are 12, 7, 3 and 25. They stay in photo order, so card 12 comes before card 7.)

Field rules:
- `format`: always the string `"bingo-cards-v2"`.
- `count`: number of items in `cards`.
- `groups`: one item per photo that has at least one card in `cards`. `id` is the photo number (same as `cards[].group`), `name` is the group name, `count` is how many cards from `cards` belong to that group.
- `cards[].no`: positive integer, unique.
- `cards[].group`: positive integer, the `id` of its group (the photo number).
- `cards[].pos`: positive integer, the card's position in its photo (reading order, starting at 1). Unique inside the same group.
- `cards[].grid`: **5 rows**, each with **5 numbers**, in reading order. Row index = top to bottom, column index = B, I, N, G, O. The center `grid[2][2]` is always `0`. All other cells are integers within their column range, or `null` only if truly unreadable (a card with any `null` will be skipped by the importer, so list it in `issues`).
- `issues`: array of short strings describing anything the user should check, for example `"Card 14: row 3 column G unreadable (glare)"`, `"Card 7 appears in photo 1 and photo 2; kept in group 1"`, `"Expected 50 cards, found 48"`. Use an empty array if there is nothing to report.

## 6. Self-check before answering
Go through this list before you output:
1. Every `grid` has exactly 5 rows of 5 values, and `grid[2][2]` is `0`.
2. Every number is in the correct column range.
3. No number repeats inside the same card.
4. No two cards share the same `no`, and no two cards have identical grids.
5. Every card has a `group` and a `pos`. Inside each group the `pos` values are unique and increase in the order of the list.
6. The cards are in the same order as in the photos (left to right, then top to bottom), grouped per photo, and not sorted by `no`.
7. `count` equals the length of `cards`, and each `groups[].count` equals the number of cards with that `group`.
8. If the user told you the expected number of cards, compare it with `count` and add an `issues` entry if it does not match.
9. The output is valid JSON (double quotes, no trailing commas, no comments).
