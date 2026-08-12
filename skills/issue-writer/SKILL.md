---
name: issue-writer
description: Write, update, or query issues in the project's issue tracker. USE WHEN creating/editing a ticket, writing or rewriting a ticket body/description/acceptance criteria, commenting on an issue or recording an answer to an open question, setting issue fields (State/Type/Assignee/etc.), linking issues (depends on/blocks/relates to/subtask of/duplicates), recording a dependency or blocker or merge-order between tickets, hitting a required-field or validation error when creating or updating an issue, discovering valid field values, or troubleshooting the tracker connection. Ticket-writing craft is tracker-independent; the tracker mechanics are in the last section, per-project connection details and API failure modes live in projects/<project>.md alongside this skill, and each project's own rules file holds its workflow states, field values and branch/PR conventions.
---

# Writing and maintaining issues

Most of this skill is about **what a ticket says and how it reads** - which is the same craft
whatever tracker the project uses. The tracker-specific mechanics (tool calls, field behaviour,
connection troubleshooting) are gathered in **Tracker mechanics** at the end.

**Project-specific configuration is not in this skill.** It sits in two places, and you generally
want both:

- **`projects/<project>.md`, alongside this file** - the instance, the endpoint, the project key,
  any standing authorizations, and the API failure modes for that tracker. Start here: it tells you
  how to reach the tracker and what will bite.
- **The project's own rules file, in its repo** (referenced from its `CLAUDE.md` / `AGENTS.md`) -
  the team conventions: workflow states, status transitions, branch naming, PR-title format, and
  allowed field values. That file is the authority on all of it. The project file above keeps out of
  its way, restating a rule only where getting it wrong is expensive enough to be worth the
  duplication - so where the two overlap, the repo's rules file wins.

This skill tells you _how_; those two tell you _the values_. Wherever a step below says "the project
key", "the instance", or "an allowed value", resolve it from them. Where a project has no file
under `projects/`, ask before guessing at connection details.

> **Every rule here is mandatory. The only negotiable items are those tagged `(optional)`.**
> When asked to list the rules, enumerate the checklists below section by section - do not
> summarise from memory.

## You are ghostwriting - every ticket is in the user's voice

**Everything you put in the tracker is the user's own writing.** Titles, bodies, comments and
status updates all post under their account, land in their colleagues' inboxes with their name on
them, and are read as work they did. You are drafting on their behalf, not filing your own report
about their project. This governs every section below.

**This extends past the tracker to anything else you post as them** - review comments and replies on
a pull request, most often. Those carry their name to a colleague exactly as a ticket comment does,
and every rule in this section applies unchanged.

- **Never mention or hint at how the ticket was written.** No "drafted with Claude", no
  "AI-generated", no co-author or tool trailers, no note that a session produced this. The reader
  is a colleague looking at a ticket, and how it was typed is not part of the work.
- **First person is the user, not you.** "I'll follow up on these" is fine - it is them speaking.
  "I have inferred", "I noticed", "I could not determine", "once this is answered I will update the
  body" are not: they are you narrating your own process, which no colleague can act on. Write
  about the work, never about the writing of it.
- **Never hedge on your own behalf.** An assistant's uncertainty is not a ticket's content. If
  something is unclear, ask the user (see "Put the question to the user before you put it in the
  ticket"); if it has to be chased, it is an open question in a comment. It never becomes "this may
  need confirming" hedging in the body.
- **Never point at the conversation that produced it** - see Voice rule 7. Whoever reads the ticket
  next was not there, and referring to a chat transcript exposes the drafting rather than the work.
- **Match how the user writes.** Follow their terminology and the conventions of the tickets already
  in the project. Where an earlier ticket named a thing a particular way, name it that way too.

The test: **could the user paste this into the tracker and defend every line of it as their own?**
If any part of it only makes sense as an assistant's output, rewrite that part.

## Ticket, then branch, then PR

1. **Ticket first.** No work starts without a ticket - plan it there before writing code or
   opening a PR.
2. **Branch off the main branch**, named `<type>/<TICKET-ID>-short-description`. The allowed
   `<type>` values are a repo convention - see the rules file.
3. **PR into the main branch** when work is complete; then move the ticket(s) to the "in review"
   state.
4. **Never commit directly to the main branch.** Every branch and PR must trace to its ticket(s).

The exact workflow states, the PR-title bracket format, and the status transitions driven by the
delivering commit all live in the rules file.

### Work you find with no ticket behind it

Auditing a project turns up branches and PRs that trace to nothing - a placeholder where the ticket
code should be (`PO2-TBC`, `PO2-NNN`), or no code at all. **Do not raise the ticket for them.** The
author knows what the work is for; you do not, and a ticket written from a diff is the design
document this skill exists to prevent. Ask them, on the PR.

- **Read the PR's status before writing a word of it.** Draft or ready, how long since it last moved,
  whether review has already happened. Status decides what you may reasonably ask for, so establish
  it first rather than treating every PR as the same object. See "Draft PRs" below.
- **Fit the ask to the work in front of you.** A substantive PR that is only missing its reference
  needs a ticket raising and its title corrected - say so directly, and say the change itself looks
  fine if it does, so the ask reads as a formality rather than a rejection. A single-file draft that
  has not moved in weeks probably wants closing instead; ask whether it is still live and offer
  closing as the outcome.
- **Never send the same message to both.** This is how the convention gets ignored: a request that
  obviously does not fit its target teaches the reader that the rule is boilerplate, and they stop
  reading it. One badly-aimed chase costs more than the untracked PR did.
- **Do not ask for changes that cost more than they return.** Renaming a branch invalidates the
  review comments already on it, and the branch is not what carries traceability - the ticket code in
  the title is. Ask for the title, leave the branch.
- **Ask, then stop.** Whether they raise a ticket or close the PR is their call, not something to
  chase twice.

#### Draft PRs

**A draft is the author saying "not ready, don't review this yet".** Commenting as though it were
finished work overrides that, and it is the commonest way a well-meant chase lands badly.

- **Do not apply review-gate process to a draft.** No asking them to move a ticket to the "in review"
  state, no telling them it cannot merge until something is fixed - they have not asked to merge it.
  Traceability bites when a PR is put up for review, not while it is being written.
- **Never pass judgement on a draft's contents.** "The change itself looks fine" is presumptuous on
  work the author has explicitly not finished, and it is worthless to them - they already know it is
  incomplete. Save it for a PR that is actually asking for review.
- **Never submit a formal review on a draft** - no approve, no request-changes. A plain comment at
  most.
- **Draft plus no movement is a different signal from draft alone.** A draft touched this week is
  live work; leave it be, or ask lightly. A draft that has not moved in weeks has usually been
  abandoned, and the useful question is whether it is still wanted at all - offer closing as the
  outcome rather than asking them to do paperwork on something they have walked away from.
- **Where the draft was only ever a test** - a one-file spike, a hosted-domain check - closing it is
  almost certainly the right answer, and asking for a ticket manufactures work nobody will do.

A ready-for-review PR is the opposite case, and there the process ask is fair: it is asking to be
reviewed and merged, so it needs to trace to something first.

## Writing ticket bodies

A ticket is a **statement of intent written for a stakeholder**, not a design document written
for a developer. Write it from the project manager's seat: what needs to change, for whom, why
it matters, and how we will know it is done. **Working out the technical solution is the
developer's job - do not do it for them in the body.**

Test the body before saving it: _could someone who has never opened the codebase read this and
understand what changes for the user and why it is worth doing?_ If not, it is written from the
wrong seat - rewrite it.

### Read it back before saving

That first test catches the wrong voice. A second pass catches the body arguing with itself. Read
the finished body top to bottom as one document - not as the sections you wrote one at a time -
and ask of every line: **does this still make sense given everything else here?**

The usual failure is a requirement that solves a problem the ticket now solves another way. It is
commonest when rewriting: a line inherited from the old body made sense under the old behaviour,
survives the edit untouched, and quietly contradicts a decision added above it.

> A ticket adds "the customer's answers are saved and they resume where they left off", and also
> keeps "warn the customer before they leave the page". Once the answers are saved, the warning
> warns about a loss that no longer happens.

**When two lines collide, do not simply delete one.** Ask which is actually wanted. Often the
redundant line survives in a narrower form, covering the case the other one does not - above, warn
only where progress could not be saved.

Check as well that:

- Every expected result traces back to the purpose, the impact, or a decision - not to habit.
- No decision contradicts another.
- Nothing listed out of scope is also required above.
- The summary still describes what the body ended up saying.

### What every body covers

1. **Purpose** - what should be true once this ships, in plain language.
2. **Why it exists** - what prompted it: a customer complaint, a support burden, a regulatory
   obligation, a broken flow, a commercial goal. Where a ticket came from is context nobody can
   reconstruct later.
3. **Who it affects, and how** - patients, pharmacists, admin staff, the business. Name the
   impact concretely: what they cannot do today, what it costs, how often it bites.
4. **Context** - which part of the service, when it matters, any business constraint the reader
   needs (regulatory, commercial, a deadline, a dependency on another ticket).
5. **Decisions already taken** `(optional)` - where the business has settled a question, record
   it so it is not re-litigated during implementation. These are **policy** decisions - what the
   product must do, who may see what, how long something is kept, which rule wins. A policy
   decision is one a stakeholder could have made without knowing how the system is built; if it
   only makes sense to someone reading the code, it is a technical decision and it is the
   developer's to make. Give the decision and its reason, not its implementation.
6. **Expected result** - the observable outcome, phrased so anyone can check it. Acceptance
   criteria describe **behaviour a person can see**, not code paths.
7. **Before this can start** `(optional)` - anything that has to be settled before a developer
   picks the ticket up and is not itself the work: a value someone must supply, a contract to
   agree with another team, a check someone else has to run. **Name an owner only when it is not
   the developer who picks the ticket up** - a specific person or team the item falls to. Anything
   falling to whoever takes the ticket needs no owner line at all. Two consequences of
   the heading meaning what it says: while anything sits here the ticket is not ready for a
   developer to pick up, and anything that does _not_ block starting does not belong here - if it
   can run alongside the work, it is either an expected result or its own ticket. Distinguish
   these from **open questions**, which go in comments (see "Updating a ticket"): an item here has
   a known answer and needs someone to go and get it; a question does not have one yet.

For a **bug**, describe the symptom as experienced: what someone did, what happened, what should
have happened instead. Diagnosing the cause is the developer's work, not the ticket's.

### Establish what happens today - never assume it

Current behaviour is the one part of a ticket that is **checkable**, and the part a developer
trusts most. Get it wrong and they hunt for a defect that is not there, or build against a system
that does not work the way the ticket claims. So establish it, from one of two places:

- **Read the repo.** The code is the authority on what the system does now. Follow the actual path
  - the branch that runs, the flag that is on, the message the customer is really shown.
- **Ask the person raising it.** They have the symptom, the context and the reason, none of which
  the code can tell you.

Prefer both, and reconcile them. Then:

- **If their account is vague, partial, or does not match the code, go to the repo and fill the
  gaps yourself.** Do not write a hedge into the body, do not write down a guess, and do not leave
  the section thin because the answer was thin. "I was told X" is not a source.
- **Report it in user-visible terms.** Reading the code tells you the mechanism; the ticket states
  what a person sees. Convert one into the other - do not paste the mechanism in.
- **If it genuinely cannot be settled either way**, that is an open question for comments, not a
  vague sentence in the body.

### What does not belong in the body

- **File paths, function / component / table / column names, code snippets, SQL.** Where a
  concrete identifier genuinely saves the next person real time - a container ID, a config key, an
  exact value - it goes in `## Notes`, not in the body. See "`(optional)` Notes" below.
- **A prescribed implementation** - "add X to Y", "extract into a shared package", "store it and
  serve a signed URL". If the body specifies a solution, it has overreached.
- **Architecture or dependency decisions** - which package, which pattern, which library.
- **Enumerated test cases.** State the outcome that must hold; the developer decides how to
  prove it.
- **Anything the ticket's own fields already carry.** Assignee, state, type, priority, due date,
  application, subsystem - the fields are the record. Writing "Owner: assignee", "this is high
  priority" or "due 4 June" into the body duplicates a field, adds nothing on the screen where
  both are visible, and goes stale the moment the field changes.
- **A "Workflow" / dev-process section.** The ticket, branch and PR process is a standing rule and
  must not be restated per ticket.
  **Exception:** a "Workflow" heading describing a workflow of the **product being built** (a
  software feature/behaviour) is fine.

The reliable smell is technical nouns, not word count: a body that runs to pages of filenames,
decisions and test lists is a design doc wearing a ticket's clothes. It should be as long as the
_intent_ requires, never as long as the _solution_ would be.

### Findings that are their own work

Investigating a ticket routinely turns up something adjacent: a pre-existing bug, a security
exposure, an inconsistency worth fixing. **Do not absorb it into the ticket you are writing.**
A body that carries three unrelated fixes hides all three - none is scheduled, prioritised or
findable on its own, and the ticket cannot be called done while any of them is open.

- **Raise it as its own ticket** and link it - "relates to", or "depends on" if this work genuinely
  cannot ship without it.
- **A security exposure always goes in its own ticket.** It carries its own urgency and must not
  queue behind a feature.
- Keep in the body only what the reader needs in order to understand this ticket, at the altitude
  of impact rather than cause - e.g. "changing an address today never reaches fulfilment, so
  parcels can still go to the old one".
- The exception is a defect this work would otherwise ship on top of: if the ticket cannot be
  delivered correctly while the defect stands, state the corrected behaviour as an expected result
  here and say it does not hold today.

The same test applies to a ticket that is already several tickets. If its parts have different
owners, different blockers, or different timings, they are separate tickets - otherwise the parts
that could be done today sit behind the ones that cannot.

### The same ticket, both ways

```md
<!-- Written from the developer's seat - wrong -->

## Problem

`ProductDetails` fired `trackEvent('view_item')` on mount, but the redesigned
`use-buy-box.ts` path only fires `add_to_cart` - `view_item` was never ported.

## Fix

Fire `view_item` from the BuyBox when the active variant resolves, mirroring the
legacy payload shape (`items: [{ title, price, id: sku }]`), once per product view.
```

```md
<!-- Written from the stakeholder's seat - right -->

## Purpose

Product page views should be recorded in our analytics again.

## Why now

Since the product page redesign went live in late July, product views have been
recorded as zero. Marketing lost the browse-abandonment emails, the retargeting
audiences and the catalogue signals that all depend on that data - so we are paying
for ads we can no longer target, and the drop-off between browsing and buying is
currently invisible.

## Expected result

Viewing a product page is counted as a product view, once per view, with the product
the customer actually looked at. Marketing can see product views recovering, and the
browse-abandonment emails start sending again.
```

The second version tells the developer what to achieve and why it matters, and leaves how to
achieve it entirely to them.

### `(optional)` Notes

Only when there is genuinely something a developer would otherwise waste time rediscovering - a
known landmine, an earlier failed attempt, somewhere the same problem is already solved - add a
final `## Notes` section. The heading is just `## Notes`: every note is for the developer, so
saying so adds nothing.

**This is also where a ticket earns its self-containment.** Assume the next person to open it has
no memory of the conversation that produced it - a teammate months later, or a fresh AI session
starting cold. Anything they would need and could not reconstruct goes here: the container or
service ID, the config key, the exact value, what an investigation actually found and how it was
checked. That belongs in `## Notes` precisely because it is not the requirement - the body says
what must become true, the notes say what is already known.

- **A ticket without this section is the normal case.** Add it only when it earns its place.
- Keep it to a few lines of **pointers**. It is a hint, not a spec, and the developer is free to
  ignore it.
- If it grows past a few lines, or starts dictating structure, it has turned into the design
  document this section exists to prevent - cut it back.

**This section is where a cut design document tries to creep back in.** A hint tells the developer
_what is already true_ and lets them draw the conclusion; a spec tells them _what to do_. Test each
line: if the developer could reasonably disagree and take a different route, it is a hint. If
following it is the only way to comply, it is a spec - cut it.

| Hint - a fact they would waste time rediscovering                             | Spec - their decision, taken for them                        |
| ----------------------------------------------------------------------------- | ------------------------------------------------------------ |
| "The rendering machinery is an admin-only dependency today, and a heavy one." | "Extract it into a shared package."                          |
| "Admin can already do this and the shared part is in the right place."        | "Add a sibling action calling the same repository function." |
| "PO2-319 defines the same eligibility window."                                | "Reuse its predicate as a named helper."                     |

### Cross-references

- **Link related tickets by ID** in the body (e.g. "Follows on from ABC-107") for context - in
  addition to the machine-readable issue link, not instead of it.

## Updating a ticket - the body is the truth, comments are the discussion

**The body always states the ticket as it stands now.** When scope, wording, acceptance criteria
or a decision changes, **rewrite the body in place**. Do not append the change as a comment and
leave the body stale. A developer reading only the body must get the whole current requirement -
if they have to reconstruct it from a comment thread, the ticket has failed.

**Comments are reserved for discussion** - open questions, a decision being changed, and the
record of what happened (delivery, promotion, verification). A comment is never the only home for
a requirement.

| Situation                         | Comment | Body                                   |
| --------------------------------- | ------- | -------------------------------------- |
| Clarifying or rewording the scope | no      | rewrite in place                       |
| Adding an acceptance criterion    | no      | rewrite in place                       |
| **A decision is changed**         | **yes** | **yes - both**                         |
| Open question, unanswered         | yes     | no                                     |
| That question, once answered      | yes     | yes - fold the answer in as a decision |
| Delivery / promotion / QA note    | yes     | no                                     |

**A changed decision needs both halves.** Comment what changed, what it was before, and why -
that history is discussion and belongs in the thread. Then update the body so it reads as though
the new decision had always been the decision. The body carries no changelog and no struck-through
former position.

**Open questions live in comments until they are answered.** Never park an unanswered question in
the body, and never resolve one by writing an assumption into the body. Once it is answered, reply
on the thread recording the answer and who gave it, then update the body to carry it as a settled
decision. A ticket with an open question is not ready for a developer to pick up.

### Reviewing a ticket means reading its comments, not just its body

**Whenever you review, rewrite or report on a ticket - one or a whole set of them - read the
comment thread before you touch the body.** The body is the truth, but comments are where the truth
arrives, and a ticket that has been answered in its thread and never folded in is exactly the
failure the previous section describes. Reading only the body reproduces the stale ticket instead of
fixing it.

On every ticket you review, work through its comments looking for:

- **An answer to a question the ticket was waiting on.** Fold it into the body as a settled
  decision, phrased as though it had always been the decision. Then say on the thread that it is now
  in the body, so nobody answers it twice.
- **A decision that has changed** since the body was written. The body must end up describing the
  current decision only - no changelog, no struck-through former position.
- **Information the ticket still requires**, that the thread shows was asked for and never supplied.
  That belongs in "Before this can start", not left in a comment where it reads as resolved.
- **A question still genuinely open.** Leave it in comments, and make sure the body does not imply
  the ticket is ready when it is not - a ticket with an open question is not `Ready for Dev`.
- **Anything the thread records that the fields contradict** - work reported as delivered on a
  ticket still sitting in an earlier state, for instance. Correct the field.

Say what you changed and what you deliberately left, per ticket. **A review that reports "no
changes needed" without having read the comments has not been done.**

### Put the question to the user before you put it in the ticket

**Ask the user first - always.** Most questions that come up while writing a ticket can be
answered on the spot by the person you are talking to, and a question posted to a ticket that the
user could have answered in one line costs a round trip and leaves the ticket parked for no reason.

- **Answered in the conversation, so it was never open.** Write it straight into the body as a
  decision. Post no comment.
- **The user does not know, or it needs someone else, so comment** - addressed to whoever has to go
  and find out.
- **The user does not know and will chase it themselves, so comment without addressing anybody** -
  see "When the questions are the user's own to chase" below.

#### Ask them one at a time, and say which ticket each one is for

**One question per prompt.** A batch of questions in a single message gets one answer covering the
easiest of them, and the rest are silently dropped. Ask, take the answer, then ask the next -
the earlier answer often changes the later question, or removes it.

**Anchor every question to the ticket it changes.** The user is looking at a set of drafts, not at
your reasoning, and "should we reference the migration?" is unanswerable without knowing where the
reference would go. Name the ticket by its summary (or its ID, once it exists) in the question
itself.

- **One ticket affected:** name it. _"On `Sign in with Face ID or fingerprint` - is Supabase's
  passkey support out of beta yet?"_
- **Several tickets affected:** say so, and list them, so the user can see the blast radius of
  their answer before they give it. _"This changes three tickets - the epic, `Identity-verified
  account recovery`, and `Retire password sign-in`."_
- **State what each answer would do to the drafts**, in a clause - which section it lands in, or
  which ticket it adds or removes. An answer whose consequence is invisible gets guessed at.

### Phrasing a question in a comment

A comment asking a question is **a task handed to a person** - the assignee where there is one,
otherwise whoever can actually answer it - to go and find something out. Write it as that person's
next action.

- **Address the person who must answer it.** Name them, say what they need to find out, and where
  or who from. An unaddressed question is nobody's job. The one exception is when that person is
  the user you are talking to - see "When the questions are the user's own to chase" below.
- **Ask the question and stop. One or two lines each.** No background, no rationale, no summary of
  why it matters. The team talks these through at stand-up, so raising the question _is_ the whole
  job of the comment - it starts the conversation rather than replacing it. Several questions make
  a list of single lines, not a document with a section each.
- **Never restate process the team already knows.** No "this is not recorded in the body until it
  is settled", no "this needs answering before work can start", no account of how the ticket will
  be updated afterwards. Everyone reading the thread already works here.
- **Never narrate yourself as the writer of the ticket.** No "I noticed", "I have inferred", "once
  this is answered I will update the body" - see "You are ghostwriting". Narrating your own process
  reads as a machine talking about itself, invites no reply, and tells the reader nothing they can
  act on.

| Narrated, padded, unaddressed - wrong                                                                                                                                             | An ask to a colleague - right                                                                           |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| "I noticed the original body never says why this is wanted. Once answered I will fold it into the body."                                                                          | "@assignee - get the monthly volume of address-change requests from support."                           |
| "This was asked when the ticket was written and never answered, so it stays out of scope for now. Retention depends on it, and neither question is recorded until it is settled." | "@assignee - check with the business whether documents are needed for orders placed before this ships." |

### When the questions are the user's own to chase

The comment is the user speaking. **When the person who has to go and find the answers is the user
you are talking to** - which is the common case, since they are the one who knows who to ask -
**do not address them and do not name them.** "@user - find out X" reads as the ticket ordering its
own author about, and the @-mention notifies them of something they just wrote.

Open with one line saying the questions will be followed up, then list them:

> I'll follow up on these:
>
> - Whether documents are needed for orders placed before this ships.
> - The monthly volume of address-change requests.

First person here is the user, not you - this is the one place it belongs. Everything else in the
section above still holds: one or two lines per question, no background, no process, no account of
how the ticket will be updated once they are answered.

## Voice

Applies to bodies and comments alike. It is **the user's voice throughout** - see "You are
ghostwriting" above, which every rule here sits under.

1. **Turn raw input into crisp, professional business prose.** Notes, transcript fragments and
   chat scrollback get rewritten, not pasted.
2. **Lead with the outcome.** Key takeaway first, then the evidence or metrics behind it, then
   progress, then action items - each with a named owner. This governs ordering _within_ the body
   sections above, and it is the whole shape of a status comment.
3. **Direct, transparent, concise. No buzzwords.** Say what is true, including when it is
   awkward. Cut any sentence that survives its own deletion.
4. **Banned words: "delve", "synergy", "cutting-edge", "testament".**
5. **Name things so someone outside the team can picture them.** Internal shorthand - system,
   service, queue and team names, acronyms, repo names - means nothing to the people a ticket is
   written for, and a reader who cannot picture the thing cannot judge the decision attached to
   it. On first mention, say **what it is and who owns it**, in a clause: not "agree it with the
   consumer owners", but "agree it with the team that runs the dispensing system, which is built
   and run outside this codebase". Apply the same test to anything the architecture makes obvious
   to you and invisible to everyone else - message brokers, queues, caches and background jobs are
   the usual offenders. If the term needs more than a clause to explain, it belongs in `## Notes`
   rather than the body.
6. **Absolute dates only.** "2026-08-08", never "yesterday", "last week" or "next sprint". A ticket
   outlives the week it was written in, and a relative date silently becomes wrong the day after.
7. **Never point at a conversation.** No "as discussed", "see the thread above", no reference to a
   chat transcript or an earlier session. Whoever reads this next was not there. If a fact matters
   it goes in the body or `## Notes`; if a past decision on another ticket matters, cite that
   ticket by ID.

## Recording relationships between tickets

Relationships between tickets are recorded as **issue links**, not left implicit in prose. A body
cross-reference is human context; the **link** is the machine-readable signal the board, filters
and merge-order checks depend on. **Whenever a ticket stands in one of the relationships below to
another, add the link** - at creation, or as soon as the relationship is known. Do not skip it
because the connection is "obvious" from the branch or the body.

Most trackers offer these relationships under these names or close equivalents:

| Link type                           | Use when …                                                                                                                                                                                                             | Not this when …                                                          |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **depends on** / `is required for`  | This ticket is in a **planned prerequisite chain**: it cannot be completed or merged until the other lands. Stacked branches; a feature needing another ticket's migration, API or infrastructure first.               | The tickets are merely related with no ordering (use `relates to`).      |
| **blocked by** / `blocks`           | This ticket is **stalled right now** by a live impediment in the other - an open bug, an undecided question, an external dependency. Use for an in-flight blocker; use `depends on` for a prerequisite known up front. | Nothing is actually stopping progress - do not manufacture a blocker.    |
| **subtask of** / `parent for`       | This ticket is **one slice of a larger ticket or epic** - decomposition of scope into child deliverables.                                                                                                              | The two are peers with an ordering (use `depends on`).                   |
| **relates to**                      | Shared context, shared root cause, or a follow-up worth surfacing, but **neither blocks nor orders the other**.                                                                                                        | There is a real ordering (`depends on`) or decomposition (`subtask of`). |
| **duplicates** / `is duplicated by` | This ticket **restates an existing one**. Link it, then close the duplicate and keep the canonical one.                                                                                                                | The issues are similar but distinct (use `relates to`).                  |

**Add a link whenever any of these conditions hold:**

1. **Ordered delivery / stacked work** - one ticket's PR must merge after another's. Use
   `depends on`, pointing from the dependent to the prerequisite.
2. **Live impediment** - work cannot proceed until another ticket clears a bug, decision or
   external item. Use `blocked by`.
3. **Decomposition** - a large ticket or epic is split into smaller deliverables. Use
   `subtask of`, or set the parent when creating the child.
4. **Association without ordering** - same area, shared root cause, or a follow-up. Use
   `relates to`.
5. **Duplicate** - the same work is already tracked. Use `duplicates`, then close the duplicate.

### Cross-PR merge order

Never leave merge order for a reviewer to infer. When PRs are ordered - stacked work, a fix on an
earlier change, a shared migration - spell it out on **both** sides:

- **In the tracker:** add the `depends on` link (condition 1) so the order shows on the board.
- **Dependent PR body:** add a top line `Depends on #<PR> - merge that first.` (chain: list in
  merge order; GitHub's `Blocked by #<PR>` also works).
- **Stack when the change cannot stand alone:** base the dependent branch on the other PR's
  branch, not the main branch, so the dependency is structural and the diff stays incremental.
- **State independence too.** If two PRs touch the same file but have no ordering requirement, say
  so (e.g. `Independent of #<PR> - different sections, merges in any order`).

> A `WIP-N` marker in a PR title (see the rules file's PR-title format) orders PRs **within one
> ticket**; the links above order work **across** tickets and PRs.

---

# Tracker mechanics

Everything above is tracker-independent. This section is not: **this project uses YouTrack**,
reached through the `youtrack` MCP server (tools exposed as `mcp__youtrack__*`). Swap this section
if the project moves tracker; the writing rules above stay as they are.

## Connect the MCP server (check first if tools are missing)

The server is the instance's HTTP MCP endpoint with bearer-token auth - the URL is in the rules
file. If the tools are unavailable:

1. Approve the MCP server if prompted (scoped servers require consent).
2. Check `/mcp` - a failed connection usually means the token is unset or expired.

**Do not proceed with tracker work until the server is connected.**

## Creating an issue - every time

**Show the ticket to the user before you create it.** Put the summary, the body and the field
values you intend to set into the conversation, let them read and change them there, and create
the issue only once they are happy. Framing, wording and scope are all far cheaper to fix before
the ticket exists, and the first version of a ticket is the one people actually read.

- Show the **body as it will appear**, not a description of it, together with the fields.
- Iterate in the conversation until the user is content with it.
- Then create it complete, in one `create_issue` call.

**Skip the review only when** the user has already approved the content in the conversation, or
asks for the ticket to be filed immediately.

> **Do not use YouTrack drafts for this.** `create_draft_issue` looks like the natural fit and is
> not. Nothing in the MCP server can publish a draft; it is visible only to the account that
> created it; and it has no issue number until a human publishes it in the UI, so it cannot be
> linked or used to name a branch or PR. Reviewing in the conversation costs nothing and leaves no
> half-made tickets behind. (Mechanically drafts do work - `create_draft_issue` takes only
> `project` and `summary`, and `update_issue` then fills in the description and every custom field
> against the returned draft ID - but the review gate belongs in the conversation.)

Run this checklist on every issue you create. The allowed values for each field, and which fields
are required, are in the rules file; the failure modes are below.

1. **Call `get_issue_fields_schema` for the project first.** The schema is the source of truth
   and can change - confirm required fields and values before building the call.
2. **Set the required fields** (per the rules file). Some fields are commonly read-only and get
   silently skipped.
3. **Leave `Assignee` unset.** A new ticket is unassigned so it can be picked up from the backlog
   by whoever takes it - do **not** assign it to the user creating it. Set an assignee only when
   the user **names** one, or when filing **on behalf of a PR author** (assign the author). Pass it
   as an **array** when you do: `{"Assignee": ["<login>"]}`. If a named assignee has no account,
   leave the field unset and say so.
4. **Pick a `Type` that doesn't demand a field you can't set.** Some types hard-require fields
   (e.g. an issue type that requires `Environment`) and fail if that field is required-but-empty
   for your account - see the rules file for the repo-specific mapping.
5. **Add issue links** for any dependency, blocker, parent, or duplicate relationship - at
   creation or as soon as it is known.
6. **Write the body to the rules above** - stakeholder framing, no prescribed implementation. This
   is the part that most often goes wrong; check it before sending the call, not after.
7. **`Priority`** is `(optional)`.

## Creating links

Create links with the `link_issues` tool. Direction reads as a sentence -
`targetIssueId` ‹linkType› `issueToLinkId` - so the **target is the subject** (substitute this
repo's project key for `ABC`):

```jsonc
// ABC-42 depends on ABC-41  (41 must land first):
{
  "targetIssueId": "ABC-42",
  "linkType": "depends on",
  "issueToLinkId": "ABC-41",
}
```

## Field-handling gotchas (the general mechanisms)

These are the YouTrack/MCP behaviours that cost real time. The **specific fields and values** they
apply to for a given repo are in the rules file; the mechanisms are:

- **Invalid enum value, so atomic rejection.** Passing a value that is not an existing member of a
  fixed-enum field fails the **whole** `create_issue` / `update_issue` call - nothing is created.
  Discover valid members via `search_issues` with `customFieldsToReturn: ["<Field>"]`, or from the
  rules file.
- **Read-only field, so silently skipped.** If a field is read-only for your account, the issue
  **is still created**; that field is just not set (it appears in the returned `errors` /
  `failedToUpdateFields`). Do not block on it - set it in the UI later if needed.
- **Required-but-read-only, so pick a type that doesn't need it.** If an issue type hard-requires a
  field that is read-only for you, the call is rejected atomically. Use a type that does not
  require it, or have someone with write access file it in the UI.
- **Conditionally required fields.** A field can be required only for certain values of another
  field, with nothing in the schema to say so - the call simply fails with `<Field> is required`.
  Set it and retry; find a valid value with `search_issues` filtered on the field that triggered
  it. The rules file records the ones already discovered here.
- **`Assignee` must be an array**, even for one person: `["login"]` - a bare string is rejected.
- **Tell the failure modes apart** from the response: an invalid enum means **nothing was
  created**; a read-only field means **the issue exists** but that field was skipped. Read the
  returned `errors` / `failedToUpdateFields`.

## Comments

`add_issue_comment` adds a comment; there is **no tool to edit or delete one**. A comment posted in
error has to be removed by a human in the UI, so get the wording right before sending it.
