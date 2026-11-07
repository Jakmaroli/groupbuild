# Groupbuild — 2.5-minute demo script

**Scenario:** four friends plan a 2-day trip to Gokarna from Mangalore, under ₹3000 each.

**Cast:** four Mattermost accounts in the room, plus `bday-builder` and `bday-critic`.

| Person | Role in the demo |
|---|---|
| Rohan (you, the room's creator) | Starts the plan, so ties go to Rohan. Calls the deadline. |
| Priya | Replies fast. Wants the beach trek. Says `lock it`. |
| Sam | Replies with a lower budget. Wants a rest day. |
| Kiran | In the room, never replies. |

**Before recording:** rehearse once end to end. Check that both agents answer, and that the
Builder can create room documents and attach files. Speed up agent wait times in editing. Keep
the room on one side of the screen and the `Plan: Gokarna trip` document or the page on the other.

| Time | Who | Say / do | What the judges should see |
|---|---|---|---|
| 0:00 | — | Hook over the empty room | "Group plans die because nobody decides." |
| 0:08 | Rohan | `@bday-builder let's plan a 2-day Gokarna trip from Mangalore, under ₹3000 each` | The plan starts in the group chat |
| 0:15 | Builder | Opens the `Plan: Gokarna trip` document, @mentions Priya, Sam and Kiran for dates, budget and preferences in one message, and proposes the deadline rule | A room document with one row per person, every status `waiting` |
| 0:30 | Priya | `Sat-Sun works. ₹3000 max. I want the beach trek.` | Priya's row turns `replied` |
| 0:35 | Sam | `Sat-Sun ok, but ₹2500 max. I'd rather a rest day on the beach.` | A conflict appears. Kiran stays silent. |
| 0:45 | Builder | `A: beach trek on day 2 / B: rest day on day 2. Everyone reply A or B.` | The conflict becomes a vote, not an argument |
| 0:50 | Rohan, Priya, Sam | Rohan `A`, Priya `A`, Sam `B` | Votes counted in the plan document |
| 1:00 | Rohan | `deadline's up` | A person calls it, not a timer |
| 1:05 | Builder | `Decided: Sat-Sun, beach trek (A 2, B 1). Budget cap ₹2500 each. No reply: @kiran, going with the group.` | The decision, the counts and who didn't reply, in one message |
| 1:15 | Builder | Researches travel, stay and food on the web, builds **v1**, attaches `index.html`, hands it to `@bday-critic` | A real plan page: who's coming, day by day, travel, stay, budget split, packing list, sources |
| 1:30 | — | Open the attached page on screen | Budget table with every person's share against their budget; unverified numbers marked `estimate` |
| 1:40 | Critic | `FIX v1` with a budget error, for example the per-person shares don't add up to the total, or Sam's share is over ₹2500 | The Critic catches a real mistake, not a colour nitpick |
| 1:50 | Builder | Fixes it as **v2** and hands back. Critic: `PASS v2` | Agent-to-agent handoff in the room |
| 2:00 | Priya | `lock it` | Only a person can finalise |
| 2:05 | Builder | Posts the final page with release notes: decided, contributed, no reply (Kiran), critic caught (budget) | The whole story in one message |
| 2:15 | — | Open the `Decisions` document and scroll it | Every input, vote, decision, version and the lock, one line each with who |
| 2:25 | — | Close | "Groupbuild gets the group to a decision, then builds the plan." |

**If the Critic passes v1:** have Priya ask for something that breaks Sam's budget, for example
`add the sunset boat ride, ₹600 each`. If the Builder adds it, the Critic flags Sam's share as
over ₹2500. If the Builder flags it first, show that instead: the budget rule is working.

**If an agent stalls:** agents only act when a message arrives. @mention it again with the
same request.
