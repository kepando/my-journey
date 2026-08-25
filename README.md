# my-journey

An app which lets you create a timeline of your life's events.

Enter things as you remember them — buying a house, a kid being born, starting a new job —
and the app catalogues them onto a timeline that runs from the day you were born to today.

## Views

**Timeline** — a vertical scroll from birth to today, ruled by year, with each event
coloured by category and stamped with your age at the time. Tap an event to see how long
ago it was, and to edit it.

**Life line** — the same events on one horizontal axis, one lane per category, so
overlapping stretches of life are visible at a glance. Zooms between whole-life, decades,
and years.

**Add** — title, date, category, optional note. The date has a precision switch: exact
day, month, or year only. You never have to invent a date you don't remember.

**Compare** — the time between any two events, or between an event and today. This is how
duration questions get answered: *have we lived in this house longer than the last one?*

## Setup

1. **Azure app registration.** The app uses the shared **KenApps-Common** registration
   (scopes `User.Read`, `Files.ReadWrite`). Add this app's redirect URIs to it:
   - `https://kepando.github.io/my-journey/`
   - `http://localhost:5500/` (or whatever port you test on)

2. **Client ID.** Replace the placeholder in `index.html`:

   ```javascript
   const CONFIG = {
     clientId: 'REPLACE_WITH_KENAPPS_COMMON_CLIENT_ID',
     ...
   ```

3. **Birth date.** Set near the top of the script in `index.html`. Every age and the start
   of the horizontal axis derive from it.

   ```javascript
   const BIRTH = '1988-03-18';
   ```

4. **Deploy.** GitHub Pages from the `main` branch root; the app lives at the
   `/my-journey/` subpath.

## Data

Events live in `/MyJourney/events.json` in OneDrive — plain, readable JSON you own and can
back up or edit by hand. Saves are debounced 1500ms and are last-write-wins, which is
fine for a single user across devices.

Events are moments, not spans: each has one date. Durations come from Compare rather than
from start/end pairs.

Categories: **home**, **career**, **family**, **milestone**, **health**, **travel**.
