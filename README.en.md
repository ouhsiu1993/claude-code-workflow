# Claude Code Workflow

[![繁體中文](https://img.shields.io/badge/%E7%B9%81%E9%AB%94%E4%B8%AD%E6%96%87-README-e9ecef?style=for-the-badge)](README.md)
![English](https://img.shields.io/badge/English-current-0b7285?style=for-the-badge)

Once the AI has finished writing the code, the time goes into working out what it actually did.

This repo holds a Claude Code output style that moves an existing project-management reporting discipline into the space between a person and an AI.

---

## What an output style is

Claude Code lets you define how it replies using a single markdown file.

It governs reporting only, not engineering. How it reads code, uses tools, and solves problems is untouched — the only thing that changes is how it accounts for the work afterwards.

Several styles ship with the tool, and you can drop your own into a designated folder. This repo holds mine.

---

## Why

Models keep getting stronger, and the thinking time keeps getting longer. The longer you wait, the more you expect back.

But across recent versions, reporting quality has not kept pace with reasoning quality. Walls of jargon, answers that miss the point, runs that finish without a conclusion — and then you have to ask again: so is it fixed? what does this sentence mean?

The cost is double. Each follow-up burns another round of tokens, and every round now takes longer than it used to. When you are rushing a bug fix, a release, or a report, that lands directly on the schedule.

Over time it turns into something else: you stop trusting the report. It once told me "completed and tests passed." There were no test files in that project. Once is nothing. After a few times, you stop taking it at face value even when it is right.

Two more patterns keep recurring. You point at one thing to fix, and it tidies up whatever else looked wrong nearby — by the time you notice, the diff is too large to review line by line. Or the reply is all jargon: readable to you, but it has to be rewritten before anyone else can use it.

None of this is about code quality. It happens after delivery.

---

## Three rules, one problem

> **The report is written for engineers.**

It assumes the reader can fill in the context: knows the jargon, knows what that function used to do, can judge whether the change was the right one. Not everyone using it is standing in that position.

Not following it costs more than one missing detail. You cannot tell whether the work was done correctly, so you either accept it wholesale or ask again.

So the three rules serve a single goal: **make what happened legible to someone without an engineering background.**

| Rule | The discipline it borrows from |
|---|---|
| STAR report | The format used in performance reviews and incident reports — readable without an engineering background |
| Findings in a separate section | Change management: raise what falls outside scope, do not act on it unilaterally |
| Jargon explained in plain language | Removes the comprehension barrier — you cannot judge what you cannot read |

Engineers get less out of these three: they can read the output, judge the scope themselves, and answer to no one. The people who need them are the ones writing code with AI every day whose trade is not engineering.

---

## The three rules

### 1. Every code change goes out as STAR

Whenever code is touched, the reply ends with this block:

```
### Change report
What was wrong: (how the old behaviour misbehaved; what the user would actually see)
What we wanted: (the state this change was aiming for)
What was done: (what was touched, and how it was solved)
What happened: (the behaviour now; whether it was verified, and how)
```

STAR is the format performance reviews and incident reports already use. Its value here is that it forces the account to begin with the problem rather than the action — the most common failure in technical reporting is skipping why and going straight to what I did.

Two supporting rules matter more than the format itself:

**"What was wrong" starts from something the user can see.** Write "the screen freezes after pressing submit", not "the promise never resolves". The first is readable without an engineering background. The second is not.

**"What happened" must not be inflated.** No tests run, say so. Tests failed, show the result. "Confirmed" only when it was genuinely verified. This is the load-bearing rule — whether a report can be trusted comes down to this line.

### 2. Problems noticed but out of scope go in their own section

When something else turns up, it is neither fixed silently nor dropped silently:

```
### Also noticed
What was found: (the symptom)
Likely / confirmed cause: (state plainly whether this is confirmed or a guess)
Suggested next step: (what it would do; if it needs a decision, ask directly)
```

This is ordinary change management: **raise what falls outside scope rather than acting on it.** Once raised, I decide whether it happens now, later, or not at all. The decision stays on my side.

Kept in its own section, scope does not quietly expand, and findings do not quietly disappear.

### 3. Jargon gets explained in plain language

Explained on first use, covering two things: what it is, and why it is being used here.

> This uses debounce (collapsing a rapid burst of triggers into a single one at the end.
> Every keystroke would otherwise hit the server, which is wasteful, so it waits 300ms
> after the user stops typing).

This rule is what makes the other two work. However complete the four STAR lines are, they are unreadable if every second word is jargon — and what you cannot read, you cannot evaluate. All that is left is to take it on faith.

The reader has no engineering background. If the explanation grows new jargon of its own, it is broken — rewrite it.

---

## Install

### 1. Pick a file

| OS | File |
|---|---|
| Windows (PowerShell) | `output-styles/en/star-report-windows.md` |
| macOS / Linux (bash, zsh) | `output-styles/en/star-report-unix.md` |

Identical content. The only difference is which shell syntax it suggests commands in.

Traditional Chinese versions live in `output-styles/zh-TW/`.

### 2. Drop it in the right folder

To apply it across every project, use the user level:

macOS / Linux:
```bash
mkdir -p ~/.claude/output-styles
```

Windows PowerShell:
```bash
New-Item -ItemType Directory -Force "$HOME\.claude\output-styles"
```

Copy the file in, keeping the `.md` extension.

For a single project, create `.claude/output-styles/` in the project root and put it there instead. Project level overrides user level.

### 3. Turn it on

In Claude Code:

```bash
/output-style
```

Pick `star-report` from the list, or name it directly:

```bash
/output-style star-report
```

Back to default:

```bash
/output-style default
```

### 4. Check it took

Ask it to change any file. If the reply ends with a "### Change report" block, it is live.

---

## Before and after

**Before**

> Fixed the `useEffect` dependency array, adding `userId` to avoid a stale closure.
> Also refactored `fetchProfile` to use `useCallback` for memoization and added an error boundary.
> Tests complete.

**After**

> Fixed the profile page.
>
> ### Change report
> **What was wrong**: After switching users, the profile page still showed the previous user's data.
> **What we wanted**: Switching users should update the page to the new user's data.
> **What was done**: The fetch only ran on first page load; it now also re-runs when the user changes.
> **What happened**: Switched between three accounts locally, all updated correctly. **No unit tests were run** — this project has none for it.
>
> ### Also noticed
> **What was found**: The page goes blank when the fetch fails.
> **Confirmed cause**: There is no error handling here; the component throws when the request fails.
> **Suggested next step**: An error state would help, but that is outside this change. Want it handled too?

The first version reads as professional, yet gives no way to tell how far it was verified, or whether that error boundary was something it added on its own. The second states both, and can be read start to finish without an engineering background.

---

## Credit

I found out Claude Code had output styles from Gary Chen's video [你以為 Claude 降智，其實是你少設了這個](https://www.youtube.com/watch?v=E8Bx9OlpmdM) (Traditional Chinese). The rules here are written to my own needs. If you want to write your own, the video covers it more thoroughly than this page does.

---

## License

MIT
