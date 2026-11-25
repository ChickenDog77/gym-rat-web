# Gym Rat website

Members sign in with the same Apple ID as the Gym Rat app and look back at their workouts,
PRs, progress per exercise, and the group's week. Read-only.

The page talks to the `gym-rat` Supabase Edge Function (actions `web_login` and `web_data`).
The key in the page is Supabase's public key; all data access is checked on the server.
Source of truth: `web/` in the private gym-rat repository.

## Sharing (build 47)

- **Shared workouts** (`index.html?w=<key>`): one workout anyone with the link can see, no sign-in (`web_workout`).
  First names only and no clock times (how long it took is shown); the gym stays (Ben's call, Nov 8). At the bottom:
  "Want in? Ask Ben for an invite", with the first name of whoever runs the sharer's group. Links are 32 random letters
  and digits, and the lifter can stop sharing one from the app (the page then says the link isn't valid anymore).
- **Join page** (`join.html?i=<key>`): where an invite link goes (`web_invite`). Who invited them, the group, the
  TestFlight link, and the code with a Copy button, with "Joining a group shows your workouts to everyone in it" up
  front. A used or expired invite says to ask for a new one. The key is made with the invite and can't be found by
  trying codes. `index.html?i=...` forwards here. Try it locally with `join.html?demo=ok` (or `used`, `expired`, `bad`).
- **Link previews**: every page carries the same Open Graph card (`og.png`, 1200×630, "Gym Rat"; the join page's title
  is "Lift with me on Gym Rat"), so a link in a text shows a branded card instead of a bare address. The site is plain
  files, so the card can't show a workout's own numbers.
- **Counts**: shared-page opens, join-page opens (by whether the invite still works) and its Copy and TestFlight taps
  are counted per day (`share_counts`, through `web_workout`, `web_invite` and `web_count`). `gr_share_stats(30)` reads
  them back with the app's own counts (links and shares by card type, Stop sharing, invite uses, Tell the group).

## Review page (`review.html`, the app's owner only)

`review.html` is where Ben replays a past workout at his desk: the audio and the lines together,
with his notes on the lines teaching the coach. Nothing links to it from the stats page, so
everyone else's site looks the same; open `https://gymrat.bensealey.com/review.html`.

- **Who can use it.** It uses the stats page's sign-in (the same `gymrat.session` in the
  browser, so signing in on either page works for both). Every call is
  `{ action: "web_review", session, op, ... }`, and the function checks the session and
  `gr_is_admin` on every call. A member who isn't an admin sees only "This page is only for
  the app's owner." Sign in with Apple on this page uses the site's root address
  (`https://gymrat.bensealey.com/`) as Apple's return address, the one the stats page uses.
- **What it calls.** `op: "list"` (his workouts with the coach talking, newest first, 40 at a
  time, "Show older" for more), `"workout"` (the lines from the phone's own `review_workout`,
  plus the numbers, the words-per-set trend and where his notes stand), `"turn_note"` (save,
  change or remove a note, with its tag and better line), `"audio_playback"` (signed links to
  the audio, good for an hour), and `"teach"` (a notes pass now).
- **The player.** One timeline stitched from the workout's 2-minute pieces in the server's order,
  with gaps between pieces left out, the same timeline as each line's `audio_timeline_ms`.
  Play/pause, back and ahead 10 seconds, 1×/1.25×/1.5×/2× (remembered in this browser), and a
  scrubber with a mark where each note is. The line being heard lights up and the list follows
  it, unless he scrolled away in the last few seconds. Clicking a line plays from 1.5 seconds
  before it. Lines with no audio (said between pieces, or after the last) still show. Links are
  renewed five minutes before the hour is up, and once more if a piece fails to load; a piece
  that still won't play is skipped with a message. Keys: space plays or pauses, ← and → go back
  or ahead 5 seconds (not while typing a note).
- **Notes.** Beside each line: on the coach's lines "Should've been silent", "Too long",
  "Wrong" and "Say it like this…"; on his own lines "Wrong" and "Say it like this…" (what the
  coach should have said after it). One tap adds the tag; a second tap takes it off. "Say it
  like this…" opens a box for the better line (up to 300 characters) and an optional note.
  "Note" is a free note. Every note can be edited (its tag too) or deleted (tap Delete twice).
  ⌘↩ saves, Esc cancels.
- **Numbers.** Sets, coach replies, words per set, "Got it", lines tagged should've been
  silent, and the times he cut the coach off, from the self-review's numbers where it ran;
  then words per set for his last 10 workouts with the coach talking, this one in green.
- **Teach the coach from my notes.** Starts a notes pass for that workout and shows what it
  changed and drafted when it's done (it checks every 3 seconds for up to 4 minutes). Undoing a
  change, or deciding a draft, is in What changed on the phone. Notes not taught by hand are
  taught at 3 a.m. his time.
- **Safety.** Everything said in a workout and every note is put on the page as text, never as
  HTML. The page loads no libraries; Apple's sign-in script is loaded only on the sign-in screen.

### Demo

`review.html?demo=1` runs on the made-up workouts in `review-demo.json` with no sign-in and no
server: the notes, the player (a made-up recording with beeps where he talks and a tone where the
coach talks), the trend and the teach button all work in the page alone. One piece's audio link
has already run out the first time, and links run out after 90 seconds, so renewing links gets
used. The banner switches between views: `&as=guest` (not the owner), `&as=signedout` and
`&as=empty` (no workouts). Put `#fail` in a note to see a save that fails. To try it locally:
`cd web && python3 -m http.server 8745`, then open `http://127.0.0.1:8745/review.html?demo=1`.
