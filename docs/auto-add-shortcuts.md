# Auto add: Apple Pay & MAE QR (iOS Shortcuts)

iOS won't let any app read notifications from another app, so the Wansem app can't
see MAE alerts by itself. The **Shortcuts** app can do it, and posts straight to your Apps
Script link, exactly like the app's Save button does. The app doesn't need to be open.

There are 3 shortcuts:

| Shortcut | What it does | How it runs |
|---|---|---|
| **Wansem Add** | Asks for a category, sends to the Sheet | Called by the other two |
| **Wansem QR** | Reads the MAE notification/receipt from a screenshot | Back Tap (2× tap the back of the phone) |
| Automation **Transaction** | Apple Pay tap → straight into the Sheet | Automatic |

---

## 1. Wansem Add (main shortcut)

Input: text in the form `4.75|NASI KANDAR DE MAJU MAXIM` (amount|merchant).

1. **Receive** *Text* input from *Shortcuts*.
2. **Split Text** `Shortcut Input` by *Custom* `|`
3. **Get Item from List**: *First Item* → rename it to `Amount`
4. **Get Item from List**: *Last Item* → rename it to `Merchant`
5. **Choose from Menu**, prompt: `Merchant · RM Amount`
   options: `🍔 Food`, `🧾 Bills`, `🛍️ Shopping`, `🚗 Transport`, `🎮 Entertainment`, `📦 Others`
   In each option: **Text** with the same name (e.g. `🍔 Food`).
   (Spelling must match the app exactly, including the emoji.)
6. Rename the result of Choose from Menu → `Category`
7. **Current Date** → **Format Date** *Custom* `dd/MM/yyyy` → `Date`
8. **Format Date** (Current Date) *Custom* `H:mm` → `Time`
9. **Format Date** (Current Date) *Custom* `yyyy-MM MMMM` → `Month`
   > MMMM gives the month name in the iPhone's language. If your iPhone is set to Malay it
   > gives "Ogos" instead of "August". Check the result by running it once.
10. **Get Contents of URL**
    - URL: *your Apps Script link* (same as in Settings in the app)
    - Method: **POST**
    - Request Body: **JSON**
      | Key | Type | Value |
      |---|---|---|
      | date | Text | `Date` |
      | time | Text | `Time` |
      | category | Text | `Category` |
      | amount | Number | `Amount` |
      | remark | Text | `Merchant` |
      | month | Text | `Month` |
11. **Show Notification**: `✅ RM Amount · Merchant saved`

Test: run with the input `1.00|TEST` → check the Sheet → delete the row.

---

## 2. Wansem QR (MAE Scan & Pay)

Flow: after paying, **screenshot** the notification (or the MAE receipt) → **tap the back of the phone 2×** → choose a category → done.

1. **Get Latest Screenshots** (count 1)
2. **Extract Text from Image** (`Latest Screenshots`)
3. **Replace Text**: find `\s+` replace with ` ` (a space) — turn on **Regular Expression**
   (OCR splits the lines, this joins them back)
4. **Match Text** `RM ?([0-9,]+\.[0-9]{2})` in `Updated Text`
5. **Get Group from Matched Text**: group at index *1*
6. **Replace Text** `,` → (empty) — so `1,250.00` becomes `1250.00` → `Amount`
7. **Match Text** ` to (.+?)\.? REF` in `Updated Text` (from step 3)
8. **Get Group from Matched Text**: group at index *1* → `Merchant`
9. **If** `Amount` *does not have any value* → **Show Alert** "No RM amount found in the screenshot" → **Stop This Shortcut** · **End If**
10. **Text** `Amount|Merchant`
11. **Run Shortcut** `Wansem Add` with input `Text`
12. (optional) **Delete Photos** `Latest Screenshots` — so your photos don't fill up with screenshots

Back Tap: **Settings → Accessibility → Touch → Back Tap → Double Tap → Wansem QR**.

Example from the notification:

```
Successful payment of RM 4.75 to NASI KANDAR DE MAJU MAXIM. REF: 384429882Q.
  → Amount = 4.75, Merchant = NASI KANDAR DE MAJU MAXIM
Successful payment of RM 11.40 to LAUGH TALE SDN. BHD.. REF: 377476253Q.
  → Amount = 11.40, Merchant = LAUGH TALE SDN. BHD.
```

---

## 3. Apple Pay automation (fully automatic)

Only for payments **tapped with iPhone / Watch via Apple Wallet** (Maybank Visa Debit card in Wallet).
MAE QR doesn't go through Wallet, so use #2 for that.

1. Shortcuts → **Automation** → **+** → **Transaction**
2. Card: choose **Maybank Visa Debit** · Category: all · choose **Run Immediately**
3. Actions:
   1. **Text** `Shortcut Input` → tap the variable, choose **Amount**
   2. **Replace Text** `[^0-9.]` → (empty), Regular Expression on — removes `RM` and commas → `Amount`
   3. **Text** `Amount|Merchant` (Merchant = `Shortcut Input` → **Merchant**)
   4. **Run Shortcut** `Wansem Add` with input `Text`

Because Wansem Add asks for a category, a notification will pop up right after you tap to pay. Tap it,
choose a category, done. If you don't want to be asked at all, replace the Choose from Menu step in
Wansem Add with a fixed category (e.g. `📦 Others`) and change it later in the app.
