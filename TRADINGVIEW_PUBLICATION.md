# Session DST Audit: Does a Fixed-Offset Session Drift?

Description to paste as is (between the rules):

---
WHAT IT DOES
Session DST Audit takes one session string, such as 0930-1600, and checks it twice on every bar of an intraday chart: once in a DST-aware reference zone, for example America/New_York, and once in the zone under test, for example the fixed offset UTC-5. A session string is a local wall-clock window, so the two readings agree only while the two zones share the same UTC offset. When daylight saving time moves the reference clock and the fixed offset stays put, the tested window drifts: with the defaults, from March to November the UTC-5 window runs from 10:30 to 17:00 New York time, so it skips the first hour of the session and adds an hour after the close. The table counts the bars and days affected on the loaded history and compares the missed bars with the added ones.

WHAT IS ORIGINAL HERE
It does not inspect any script's code. It takes the session and the two zones you give it and shows what that pair does on this chart: the same string evaluated in both zones bar by bar, each bar classed as inside both windows, inside the reference window only (missed) or inside the tested window only (added). Each zone's UTC offset is computed at every bar's time, so the offset changes it reports are the ones that fall inside the loaded history. Missed and added bars are kept apart, each with its mean volume and mean range, because an hour at the open and an hour after the close are not the same hour.

HOW THE WINDOWS ARE COMPARED
- On every bar the session is tested in both zones with the bar's open time: a bar is inside a window when that time falls in the session in that zone.
- Three classes: both windows, reference only (missed), tested only (added). Bars in neither window are ignored.
- The UTC offset of a zone on a bar is the bar's wall clock in that zone minus its UTC time, in minutes. Each change across the loaded bars is counted; the last change is shown with its date and both offsets.
- Days are keyed in the reference zone. A session day has at least one bar in either window; a mismatch day has at least one bar in exactly one window. A day is counted when the next one begins, so the current day never is.
- Per class: bar count, mean volume over the bars that have volume, and mean range as (high - low) / close in percent.
- Counters move only on confirmed bars; no count changes after a bar closes, and no other timeframe is requested. Shading is drawn on every bar, including the realtime one.
- The word exchange as the zone under test stands for the chart symbol's exchange timezone.

HOW TO USE IT
1. Open an intraday chart. On daily or higher timeframes the table only says to use an intraday chart.
2. Enter the session string as the script or alert you are checking writes it, and the zone it is meant for.
3. Enter the zone that script or alert actually uses: a fixed offset such as UTC-5, UTC, an IANA name, or exchange.
4. Read the table. Blue bars are inside the reference window only, orange bars inside the tested window only. With both zones equal every count is zero.

HOW TO READ THE TABLE
Session: the string under test, read as a local wall clock.
Reference zone and Zone under test: name, offset on the last bar, number of offset changes on the loaded bars, and the last change with its date and both offsets.
Session days (completed): days with at least one bar in either window, with the first and last date.
Days where the windows differ: days with at least one bar in exactly one window, as a share of session days. Red when there is at least one, green when there are session days and none differ.
Bars in both windows, missed by the tested zone, added by the tested zone: bar counts per class, each with its mean volume and mean range.
Volume, missed vs added: mean volume of the missed bars over that of the added bars.

LIMITATIONS
- The result depends on the loaded history and on the symbol's trading hours: a symbol that does not trade in the added hour shows no added bars, and a short history may hold no offset change.
- Volume is the data vendor's volume for this symbol and feed.
- An unrecognized timezone string stops the script with an error; it is the user's input to check.
- The tool does not read the code of any script or alert. It only shows what one session string with one pair of zones does on this chart.
- Bars are classed by their open time, so a bar that straddles a window boundary counts on the side of its open.
- Counts on loaded bars, not a trading rule. Not financial advice.

The chart shows ES1! (CME) on the 15 minute timeframe with the default settings. The table is in the bottom right.
---

## Publishing checklist

- [ ] Compile `session_dst_audit.pine` in the Pine Editor with no errors or warnings.
- [ ] Set the chart: ES1! (CME), 15 minutes, default inputs, table visible, a DST date in view so that both blue and orange bars show.
- [ ] Check the identities: `Both + Missed + Added` equals the confirmed bars inside at least one window, blue bars are the missed ones, orange bars the added ones.
- [ ] Set the tested zone equal to the reference zone: no runtime error, every mismatch count is 0, the mismatch row is green, no bar is shaded. Restore `UTC-5`.
- [ ] Open a daily chart: the table shows only "Use an intraday chart" and no bar is shaded.
- [ ] Check one blue bar by hand against the clock: between the March and November DST dates it opens between 09:30 and 10:15 New York time (steps in `README.md`).
- [ ] Check the title is at most 60 characters.
- [ ] Publish as open source, with the MIT license text if the form allows it.
- [ ] Check that the description contains no links, contact info, or performance claims, and is between 3,000 and 4,600 characters.
- [ ] Choose the category and tags (for example: session, timezone, DST, daylight saving, UTC offset, time, audit).
- [ ] Add the TradingView script URL to `README.md` after publishing.
