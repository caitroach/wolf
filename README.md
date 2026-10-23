# wolf
Welcome to 2026, where data brokers have crept into every facet of your life. Like your period! 

A few years ago, I used Flo to track my periods. When it was revealed that the company was selling my *menstrual data* to Meta, I switched to Stardust, a woman-owned and data-encrypted period tracking app. Turns out "data encrypted" actually means "Hey Meta and Google, here's the sensitive health information of millions of users". 

While the idea of period apps stealing your menstrual cycle data and selling it to data brokers and advertising firms is abstractly creepy and invasive, this practice now has a tangible risk to women living in areas where abortion is criminalized (like, recently, our neighbours to the south: the states).

Flo and Stardust both are vulnerable to warrants and subpoenas by federal governments. If you live in an area with no access to abortion, a sudden gap in menstrual data followed by the return of a normal cycle creates a clear, timestamped record that prosecutors could use in court to argue that an illegal abortion took place.

Period tracking data can act as the starting piece of evidence that prompts authorities to demand warrants for license plate reader data, location histories, and decrypted personal text messages and e-mails.

<img width="1920" height="1080" alt="wolf" src="https://github.com/user-attachments/assets/d768ce51-5571-42a8-b93a-cedd7f44685b" />

*try the demo [here](https://caitroach.github.io/wolf/)!*

## this project
Wolf (Flow backwards... creative, I know) is an open-source alternative to creepy, money-hungry apps that sell your menstrual data to shady brokers. It's a single static HTML file with zero dependencies, zero backend, and no account. Your data is stored in your browser and can be downloaded locally as plain text. 

## Privacy 
All entries are saved in your own browser. This program does not have the ability to send any information anywhere. Ever. 

### menstrual data 
Menstrual data is stored locally in your browser in `localStorage`. Google physically can't get into it because it's on your own computer. It's stored as plain JSON for simplicity and is not encrypted because it's a local file. 

### your IP address
- One of the fonts I'm using is by Google Fonts in `<head>`. This reveals your IP to Google, but they can't see what you actually log.
- I'm hosting this with GitHub pages, so GitHub (owned by Microsoft) can see your IP and the repo you're on, but not any of the information you put into it.
- I recommend using a VPN for everything, but there's little risk here.

### going off the grid? 
Download this as a HTML file rather than clicking the README link and open it on your computer whenever you want.

## Data model 
Stored under `localStorage["localcycle.v1"]`:
 
```json
{
  "days": {
    "2026-10-14": { "flow": 2, "symptoms": ["Cramps"], "note": "optional text" }
  }
}
```
 
| Field | Type | Rules |
|---|---|---|
| key | `YYYY-MM-DD` string | Local calendar date. No time or timezone is stored |
| `flow` | integer `0`-`3` | `0` none, `1` light, `2` medium, `3` heavy. A day counts as a period day when `flow > 0` |
| `symptoms` | string array | Values from the fixed list: Cramps, Headache, Bloating, Fatigue, Mood swings, Spotting, Acne, Tender chest |
| `note` | string | Truncated to 500 characters |
 
Rules enforced on load and on restore (`normalize`):
 
- Keys that do not match `YYYY-MM-DD` are dropped.
- Dates later than today are dropped.
- Invalid `flow` becomes `0`. Unknown symptoms are removed. Notes are truncated.
- An entry with no flow, no symptoms and no note is deleted rather than stored empty.

## Backup format
 
The backup box produces and accepts:
 
```json
{ "app": "local-cycle", "version": 1, "days": { "...": {} } }
```
 
Restore parses the text, runs it through `normalize`, and **replaces** all current data. Anything that is not valid JSON with a `days` object is rejected with a message and changes nothing. Restore and erase need two taps within 4 seconds, because the host viewer may block `confirm()` dialogs.

## Prediction algorithm 
All date math uses local dates. Day differences keep results correct across daylight-saving changes.
1. **Group days into periods.** Take all days with `flow > 0`, sorted. A day joins the current period if it is within 7 days of that period's last day. Otherwise it starts a new period. Period length is `end - start + 1`.
2. **Cycle lengths.** The gap in days between consecutive period start dates.
3. **Average cycle.** Mean of the last 6 cycles whose length is between 15 and 60 days, rounded. Falls back to **28** when none qualify.
4. **Average period length.** Mean of the last 6 completed periods (a period is completed once its last day is at least 2 days before today), rounded and clamped to 1-10. Falls back to **5**.
5. **Next period.** `last period start + average cycle`.
6. **Predicted days.** For cycles 1 to 3 ahead, mark `average period length` days starting at `last start + k * average cycle`. Only days on or after today that are not already logged as period days are drawn.
7. **Cycle day.** `today - last period start + 1`.
Headline states: not logged yet, period day N, next period in N days, expected today, N days past the estimate.

## Features 
- Month calendar with period days, predicted days, today, and a note/symptom marker.
- Per-day logging flow, symptoms, and free-text note
- Plain-language status headline
- Predictions for the next three cycles
- Stats including average cycle length, average period length, periods logged, and last period start
- Recent-cycles bar list
- Backup and restore as copy-paste JSON text
- Erase-everything button
- Light and dark themes
- Keyboard navigation and screen-reader labels on every day 

## UI and accessibility
 
- Future days cannot be selected or logged. Predicted future days are display-only.
- Arrow keys move the selected day by 1 (left/right) or 7 (up/down) days and never past today. Focus is restored after each re-render.
- Every calendar day is a `<button>` with a full `aria-label` (date, flow, predicted, has notes, today) and `aria-pressed`.
- Flow and symptom controls are toggle buttons with `aria-pressed`. Backup messages use `role="status"`.
- Layout is responsive down to about 400px. The calendar and the day editor stack under 780px.
- Theming uses CSS custom properties. Dark values apply through `prefers-color-scheme`, and a `data-theme="light|dark"` attribute on the root element overrides it.
- `prefers-reduced-motion` disables transitions.
- Fonts: Bricolage Grotesque (headings), Figtree (body), each with system fallbacks.

There is nothing to install or build, and the program is too small and dumb to store or send anything. 
