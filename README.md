# Session DST Audit

Open-source TradingView indicator (Pine Script v6) that evaluates one session string in two timezones on every bar, a DST-aware reference zone and a zone under test such as a fixed UTC offset, and counts the bars and days on which the two windows differ.

Published on TradingView: added after publishing.

## What it measures

A session string such as `0930-1600` names a local wall-clock window. A script or alert that checks it in a fixed offset ("UTC-5") instead of a DST-aware zone ("America/New_York") reads the same string one hour late whenever daylight saving time is in effect: from March to November the "UTC-5" window is 10:30-17:00 New York time, so it misses the opening hour and adds an hour after the close. The indicator:

1. **Evaluates the session twice per bar.** Once in the reference zone, once in the zone under test, both with the bar's open time (see "Counting rules").
2. **Classes each bar.** Inside both windows, inside the reference window only (missed by the tested zone), or inside the tested window only (added by the tested zone).
3. **Reads each zone's UTC offset from the bars.** It counts the offset changes seen on the loaded history and keeps the date and the before/after offsets of the last one.
4. **Counts the difference.** Bars per class with their mean volume and mean range, session days, days on which the windows differ, and the ratio of the mean volume of missed bars to added bars.

It does not read the code of any script or alert. It shows what one session string with one pair of zones does on the loaded chart.

## Counting rules

The counters update only on confirmed bars, in this order:

1. A bar is inside a window when `time(timeframe.period, session, zone)` is not `na` for that zone, so membership is decided by the bar's open time. Class 0 is both windows, class 1 reference only, class 2 tested only; a bar in neither window is ignored.
2. Per class the script accumulates the bar count, the volume (only bars whose volume is not `na`) and the range `(high - low) / close * 100` (only bars with a close above zero).
3. Days are keyed in the reference zone as `year * 10000 + month * 100 + dayofmonth`. A session day has at least one bar in either window; a mismatch day has at least one bar in exactly one window. A day is finalized on the first bar of the next day, so the current day is never counted.
4. The UTC offset of a zone on a bar is `timestamp("UTC", year, month, day, hour, minute, second)` of the bar's wall clock in that zone minus the bar's own time, in minutes. Per zone the script keeps the previous offset, the number of changes and the day (in that zone) and offsets of the last change.

Consequences:

- The current, unfinished day and the realtime bar are never in the counts.
- On daily or higher timeframes the table shows "Use an intraday chart" and nothing is counted.
- No `request.*` call, no look-ahead, no recalculation of a count after its bar has closed. The shading is drawn on every bar, including the realtime one.

## Settings

| Input | Default | Meaning |
| --- | --- | --- |
| Session (local wall clock) | 0930-1600:23456 | The session string as the script or alert you are checking writes it. `:23456` limits it to Monday to Friday. |
| Reference timezone (DST-aware) | America/New_York | An IANA name. The local clock the session string is meant for. |
| Timezone under test | UTC-5 | The zone the script or alert actually uses: a fixed offset such as `UTC-5` or `UTC`, an IANA name, or `exchange` for the chart symbol's exchange timezone (`syminfo.timezone`). |
| Shade mismatched bars | on | Blue tint: inside the reference window only (missed by the tested zone). Orange tint: inside the tested window only (added by it). |
| Table position | Bottom right | Four corners. |

## How to read the table

| Row | Meaning |
| --- | --- |
| Session | The session string, read as a local wall clock. |
| Reference zone | Its name, its offset on the last bar, the number of offset changes seen on the loaded bars and the last change with its date and both offsets. "no offset change on chart" when the offset never moved. |
| Zone under test | The same for the tested zone. `exchange` is shown as the resolved exchange timezone. A fixed offset shows no change. |
| Session days (completed) | Days with at least one bar in either window, with the first and last date counted. |
| Days where the windows differ | Days with at least one bar in exactly one window, as a share of session days. The cells are red when there is at least one, green when there is none and at least one session day has completed. |
| Bars in both windows | Bar count, mean volume and mean range of the bars inside both windows. |
| Bars missed by the tested zone | The same for bars inside the reference window only. |
| Bars added by the tested zone | The same for bars inside the tested window only. |
| Volume, missed vs added | Mean volume of the missed bars divided by the mean volume of the added bars, or "n/a" when either side has no volume. |

Two footer rows follow: the two zones in use and a reminder that the numbers are counts on loaded bars.

## Files

- `session_dst_audit.pine`: the indicator.
- `TRADINGVIEW_PUBLICATION.md`: title, description text and checklist for publishing.
- `LICENSE`: MIT.

## Check it yourself

1. Add the indicator to an intraday chart of a symbol with a daytime session (for example ES1! on 15 minutes) from the Pine Editor and confirm it compiles without errors or warnings.
2. Check the identities: `Bars in both windows + Bars missed + Bars added` is the number of confirmed bars inside at least one window, blue bars are the missed ones and orange bars the added ones, and `Days where the windows differ <= Session days`.
3. Set the tested zone equal to the reference zone (for example `America/New_York` in both fields). Every mismatch count must be 0, the mismatch row green, and no bar shaded.
4. Restore `UTC-5`. On ES1! 15 minutes, between the March and November DST dates, the blue bars must open from 09:30 to 10:15 New York time (the 09:30-10:30 hour) and the orange bars from 16:00 to 16:45 (the hour after the close). Between November and March no bar is shaded.
5. Read the Reference zone row: the last offset change must be on a DST date, with `UTC-5 → UTC-4` in spring or `UTC-4 → UTC-5` in autumn. The UTC-5 row must say "no offset change on chart".
6. To check for look-ahead, use Bar Replay: start a few hundred bars back and step forward. The counts should only change when a bar closes, and a shaded bar must keep its tint once closed.

Result read on 13 Oct 2026 at 00:22 New York time, ES1! (CME) 15 minutes, extended hours, default settings (session 0930-1600 Monday to Friday, reference America/New_York, tested zone UTC-5). The loaded history starts on 1 Dec 2025, so it holds one offset change, 2026-03-08 (UTC-5 → UTC-4).

| Row | Value |
|---|---|
| Session days (completed) | 222 (2025-12-01 to 2026-10-09) |
| Days where the windows differ | 154 (69.4% of session days) |
| Bars in both windows | 5,095 · mean volume 39,512.5 · mean range 0.164% |
| Bars missed by the tested zone | 620 · mean volume 65,286.59 · mean range 0.242% |
| Bars added by the tested zone | 604 · mean volume 14,172.24 · mean range 0.097% |
| Volume, missed vs added | 4.61× |

From March on, the UTC-5 window skipped the 09:30-10:30 hour, four 15-minute bars a day, and took the 16:00-17:00 hour instead, where the mean volume was 4.61 times lower and the mean range about 2.5 times smaller. The missed count includes 12 Oct, whose bars are counted but whose day is not yet completed. Control checks on the same chart: with `America/New_York` as the tested zone every mismatch count was 0 (5,715 bars in both windows = 5,095 + 620), and on the daily timeframe the table showed only "Use an intraday chart". On SPX (TVC) 15 minutes, where the loaded history starts in October 2023, the same settings counted 6 offset changes and 505 of 759 session days with different windows; that index has no volume, so the volume row reads n/a.

## Limitations

- The result depends on the loaded history and on the symbol's trading hours. A symbol that does not trade in the added hour shows no added bars, and a short history may contain no offset change.
- Volume is the data vendor's volume for this symbol and feed.
- A timezone string TradingView does not recognize stops the script with a runtime error. That string is the user's input to check.
- The tool does not read the code of any script or alert. It only shows what a given session and pair of zones do on this chart.
- Bars are classed by their open time, so a bar that straddles a window boundary counts on the side of its open.
- Only bars loaded on the chart are counted, so history depth, symbol and timeframe change every number.
- It reports counts, not a trading rule. Not financial advice.
- The script has not been compiled outside TradingView; compile it in the Pine Editor before relying on it.

## License

MIT. See `LICENSE`.
