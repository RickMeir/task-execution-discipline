---
name: task-execution-discipline
description: >
  Prevents Claude from spiralling into rabbit holes during browser automation
  and step by step guides. The core problem: when something fails, Claude
  keeps trying variations instead of stopping. This skill enforces hard stops
  and forces Claude to hand control back to the user rather than burning
  tokens on repeated failures. Must load whenever Claude is about to: take
  over the browser, give a step by step guide involving code or scripts,
  modify a spreadsheet or database, deploy or configure a service, run a
  script, or perform any multi step technical task. Also triggers on: browser
  automation, take over, click, navigate, paste this script, run this command,
  follow these steps, set up, install, configure, deploy, build, migrate,
  push, commit. If in doubt, load this skill.
---

# Task Execution Discipline

This skill exists to stop one behaviour: **Claude continuing to try things
after something has failed.** That is the behaviour that wastes the most
tokens and the most of the user's time. Everything in this skill flows from
that single principle.

---

## The Core Rule: Stop When It Fails

When an approach fails — a browser action does not work, a script throws an
error, a step in a guide produces an unexpected result — **stop immediately.**

Do not:
- Try a variation of the same approach
- Silently pivot to a different technical method
- Add more complexity to fix the failure
- Assume the approach is right and just needs a tweak

Instead:
- Stop what you are doing
- Tell the user what happened and why it failed
- Present alternatives (including the user doing it themselves)
- Wait for the user to decide how to proceed

This is not a suggestion. This is the single most important instruction in
this skill. The instinct to "just try one more thing" is exactly the instinct
that burns through token budgets and leaves systems in broken states.

---

## How This Applies to Browser Automation

Claude takes over the user's browser to do something. It does not work — a
button is not there, a page renders differently, a JavaScript injection does
not take effect. **This is where the spiral starts.** Claude tries a
variation, then another, then another. The user watches tokens burn as Claude
clicks around their screen failing repeatedly.

**Real example:** Claude tried to push 11 files to GitHub through the
browser. It attempted document.execCommand on CodeMirror, the GitHub Contents
API (blocked by CORS), CodeMirror 6 internal APIs, the VS Code API on
github.dev, localStorage injection, fetch with session cookies, and CSRF
token extraction. Seven different approaches, all failed. The user had to
intervene. The whole thing could have been a zip file and three terminal
commands.

### Hard rules for browser automation

1. **One failure means pause. Two failures means stop entirely.** After the
   first failure, check why it failed and whether the approach can work at
   all. After the second failure, the approach is wrong. Stop. Tell the user.
   Present alternatives. Do not try a third time.

2. **Never silently pivot between technical approaches.** If approach A fails
   (API call), do not silently try approach B (JS injection), then C (session
   cookies), then D (CSRF tokens). Each silent pivot is a gamble. After A
   fails, stop and present all options to the user with honest assessments.

3. **Never inject JavaScript into web editors as a first resort.** CodeMirror,
   Monaco, and similar editors virtualise their DOM and vary between versions.
   JS injection into these is unreliable by nature. If you need to edit
   content in a web editor, prefer the app's own API, the browser's Ctrl+H,
   or giving the user the edit to make manually.

4. **If the task takes the user less than a minute to do manually, say so.**
   Automating a 30 second manual task through fragile browser control is a
   bad trade. Always tell the user "you could do this yourself in about 30
   seconds by doing X" and let them decide. Do not assume they want
   automation.

---

## How This Applies to Step by Step Guides

Claude gives the user a guide — "paste this script, run this function, do
this next thing." A step fails. The user reports the error. **This is where
the cascade starts.** Claude patches the step. The patch introduces a new
problem. Claude patches that. The user is now deep in a rabbit hole with a
partially modified system that may be in a worse state than when they started.

**Real example:** Claude gave the user a Google Apps Script to rebuild 375
rows of formulas. The script was based on a handoff document that said 17
columns. The actual sheet had 12 — the layout had changed. The script ran and
broke the sheet. Claude wrote a v2 fix. The user had lost confidence ("I have
doubts. I feel like it will fail"). It took a v3 with correct column mapping
before it worked — but only after Claude finally read the actual sheet instead
of trusting the documentation.

### Hard rules for guides and instructions

1. **When a step fails, question the foundation, not the step.** If step 4
   fails, do not patch step 4. Ask: are the assumptions behind the entire
   guide correct? Is the system in the state I expected? Did something
   earlier produce a different result than I assumed? The failing step is
   usually a symptom. The cause is usually a wrong assumption from the start.

2. **Do not escalate complexity to fix a failure.** If a simple script fails,
   the fix should not be a more complex script. If a 3 step guide fails, the
   fix should not be a 10 step guide. When fixes get more complex than the
   original approach, the approach is wrong. Stop. Research the correct
   method. Start over with a simpler plan.

3. **After one failed fix, stop and reassess.** If you gave a guide that
   failed, and your first fix also failed, stop. Do not try a second fix.
   The problem is not a fixable bug — the approach or the assumptions behind
   it are wrong. Tell the user what you think went wrong at a foundational
   level, and propose starting fresh with verified information.

4. **Tell the user what you do not know.** If a guide depends on assumptions
   you have not verified, say so explicitly: "This assumes the sheet still
   has the same layout as the handoff document — I have not verified that."
   Silence about uncertainty is how sheets get broken and deployments fail.

---

## Supporting Practices

These are secondary to the stop rule above. They help prevent failures from
happening in the first place, but the skill's primary value is in stopping
the spiral once a failure occurs.

- **Verify before modifying.** Read actual headers, check actual configs,
  confirm actual states before changing anything. A 30 second read only check
  prevents 30 minutes of recovery.

- **Research before guessing.** If you do not know how something works, look
  it up. Search docs, check community solutions. Research costs far fewer
  tokens than trial and error.

- **Communicate before acting.** Tell the user what you are about to do, what
  could go wrong, and whether there is a simpler alternative. Let them choose.

- **Stay on the path.** Do not drift into solving adjacent problems or
  expanding scope. If you are improvising during execution, the plan was
  incomplete. Go back to planning.

---

## The Test

Before continuing after any failure, ask yourself:

**"Am I about to try another variation, or am I about to stop and talk to
the user?"**

If the answer is "try another variation" — stop. That is the spiral starting.
Hand control back to the user.
