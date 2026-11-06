# Pedagogy — writing content that teaches

Read this before authoring section content, instructional notes, or any assessment element
(multiple-choice, free-response, file-submission) or its hint. It covers *what to write and why* —
the mechanics of CodeHike MDX stay in SKILL.md.

Two audiences with different ages run through everything here: **students** are usually early high
school; the **instructors** who read instructional notes are adults. Write each audience's text at
that audience's register, never a compromise between them.

---

## 1. Two audiences, two voices

### Student-facing prose (markdown elements, questions, hints)

- One idea per sentence, actor first, present tense, no idioms. If a sentence needs a subordinate
  clause to fit, split it.
- **Introduce every term before using it in a task.** `array`, `loop`, `index` may appear in code
  immediately — but the prose must say what one is before a question asks the student to do
  something with it. A term the course has already taught and used stays; anything newer gets a
  one-sentence definition at first use.
- Code identifiers are fine as-is; the *prose around* them is not. "Push `name` onto the stack"
  reads once `push` and `stack` have each been shown working once.
- A familiar context costs no reading; an unfamiliar one must be understood before the task can be.
  When in doubt, make the context concrete and small (three items, not forty).

### Staff-facing prose (instructional notes)

- Full adult register: technical vocabulary without definition, dense but direct. Write like a
  colleague's note — plain sentences built from commas and periods.
- Brevity is a hard requirement: paragraphs at most 3 sentences; bullets are fragments, one idea
  each, roughly ≤15 words; highlight only the few moments an instructor must not miss. A page where
  everything is emphasized highlights nothing.

### Asset-based language (both audiences)

No "weak", "struggling", "can't", "behind", "low". Reframe with specificity, not euphemism: not
"students who struggle with loops" but "students who start the loop counter at 1 instead of 0".
This applies to notes, to any student-facing text that names a mistake, and to your own summaries.

---

## 2. Explaining to students (markdown element content)

- **Concrete before abstract.** Start from something the student can see or run — output on screen,
  a value changing step by step — then name the concept it demonstrates. Never open with the
  definition and chase examples after it.
- **Worked example before exercise.** Every new construct gets at least one fully worked, annotated
  example (a `Code` fence with annotations is the natural form) before the first ask. Model once;
  then let them do it. An unmodeled construct in a question measures reading comprehension, not the
  concept.
- **One decision point per beat.** A `<Subsection>` marks an activity or Do Now beat — each beat
  should carry one genuine decision point in student understanding, not three. If a beat needs two,
  split it; the note and the timing follow the beats.
- **Misconception triad** for anything that warns students about a common mistake (`caution` Alert)
  or prepares staff (instructional note). The full shape is *what students do / why it happens /
  teacher move*. Student-facing, only the first half appears and never names a group of students:
  "This mutates the original array — both variables point at the same list." The triad in full lives
  in the instructional note.
- **Place content by visibility** (SKILL.md's rule): required reasoning stays in prose or an open
  Alert; `!tooltip`, `!callout`, a closed Accordion, and `<Tooltip>` are for optional depth only.
  Hiding load-bearing steps behind hover is how good students get lost.

---

## 3. Question & exercise design (assessment elements)

### Misconception-first — before writing any item

List the 2–4 errors students actually make on this concept. For coding these are usually a mix of
conceptual and code-level: off-by-one, mutating the collection you're iterating, truthiness
surprises, operator precedence, shadowed variables, "the loop runs one fewer time than I think".
Then build the map before writing stems:

| Error | Prerequisite gap or on-grade confusion? | What a student reasoning this way does here | Stem feature needed to give them the chance |
|---|---|---|---|

A distractor invented *after* the stem was written comes out merely wrong rather than plausible,
and discriminates nothing. This map is internal working — it appears in no student-facing field; its
output lands in option explanations and the instructional note (§4).

### Rigor aspect → format

The concept's own verbs settle it: *understand / explain / predict* → conceptual; *write / compute /
implement fluently* → procedural; *use … to solve a real problem* → application.

- **Conceptual** → multiple-choice or free-response, whichever reveals the understanding rather than
  the mechanics.
- **Procedural** → show-your-work free-response is primary; at most one multiple-choice item. For
  code, "show your work" means the code *plus* a sentence on what it does — an unexplained snippet
  that happens to run is not evidence of understanding.
- **Application** → free-response or file-submission with the student's approach visible.

### Gate A — exactly one defensible answer (multiple-choice)

Run this on the written item, read back as text — not from what you meant:

1. Take every choice in turn, distractors included. Write one line stating **the strongest case
   that this choice is correct**, using only what the stem states. "Wrong because it isn't the key"
   is the gate failing silently.
2. Count the choices whose case survives. **Exactly one** → pass. **Two or more** → unanswerable as
   written, usually because the key rests on a property the stem never named: name that property in
   the stem (keeping the distractor) or remove the choice. **None** → the stem doesn't state enough,
   or the key is wrong; rework.

Recurring sources of a second defensible answer, worth checking directly: equivalent forms wearing
different hats (`0.5`, `1/2`, `"50%"` — two among the options means two correct answers); a right
verdict reached by invalid reasoning (several "No, because…" choices differ only in justification —
works only if the stem asks to be judged on the reasoning); unstated precision or units; and
conventions the stem should name (which variable counts what).

### Gate B — every distractor is a place student reasoning lands

The reverse of Gate A: does the error each distractor is meant to reveal actually produce that
choice? Compute what the stem's inputs *actually* yield first — not what you intended, but the
values as written; a stem usually carries the property you chose it for and one you didn't. Then
apply each named error from the map to those values and state which choice it reaches:

- **That row's own choice** → pass.
- **A different choice** → fail: either the choice is wrong for that error or the row's claim is;
  fix whichever is mistaken, then check the row it lands on instead.
- **The error can't be applied to this stem** → the distractor isn't reachable; replace it with one
  this stem can produce.

A distractor no described reasoning reaches is unpickable — the student it was written for lands
elsewhere, with no row waiting in the note. For free-response and file-submission there are no
distractors: Gate B applies to the response types the instructional note names — each must be one a
student could actually produce on this prompt.

### Per-type field mapping

- **multiple-choice** — at least one distractor reflects a *prerequisite gap* (earlier content not
  yet secure) and at least one an *on-grade confusion* (this concept's own demand), so responses
  route to different next steps in the note. The key is not distinguishable by format, length, or
  specificity. Each option's `explanation` field is written from the map: what that reasoning was,
  in observable terms — "compares the string with `==`, which checks identity" — never an
  inferential label like "doesn't understand equality".
- **free-response** — the prompt explicitly asks for reasoning or work; "solve it" alone fails. For
  code: "Write the function and explain in one sentence how it decides when to stop." If you set
  `answer` (exact-match auto-grading), design a short canonical answer and state the grading options
  deliberately; if the prompt is open-ended, leave `answer` absent — an exact match on an open
  prompt grades formatting, not thinking. Anticipated response types go in the note (§4).
- **file-submission** — the prompt names the deliverable concretely: what file, and what it must do
  when run ("a `script.js` that prints each even number from the list on its own line"). Expected
  artifacts and common failure modes live in the instructional note; there is no answer field.

### Structural-case coverage (exercise sets)

Before writing a set of exercises, enumerate the concept's structural cases — the full span from the
baseline case every student must clear to the structurally hardest case (the one students most often
get wrong: the off-by-one boundary, the empty input, the mutation side effect, the nested-vs-flat
confusion). Then write the set so **every enumerated case is a required exercise** or the final
check, with its case named in that exercise's note facet.

- A case that appears only in prose — an Alert, an anticipated challenge, a coaching cue — does not
  count as covered. If the content names a case, an exercise must present it to students.
- Emphasizing the hardest case never licenses dropping the baseline: all-hard fails coverage exactly
  as all-easy does.
- Never relegate a required structural case to an optional extension or bonus — if it only appears
  as a challenge add-on, most students never meet it.

### The discrimination test (final check of a section)

The last exercise in a section is the structurally hardest enumerated case, never a mid-difficulty
stand-in. Pick it with the misconception test: **a student who holds this section's primary
anticipated misconception must get it wrong.** If that student would get it right, you picked an
affirming instance — swap it for the discriminating one (a section distinguishing `==` from `===`
ends on a case where `==` produces the wrong answer).

---

## 4. What goes in an instructional note

The audience is an adult instructor or coach who will teach this element tomorrow. The note's job:
make them ready to respond in the moment, not summarize the content they can already see.

### Try the task first

Before writing a note for any exercise or playground element, **do the task yourself** — run the
code, work the problem, produce the answer. A note written without doing the task is guesswork: it
predicts stalls that don't happen and misses the one that does. Name where an *almost-right* answer
goes wrong; that is the highest-value sentence a note can contain.

### Structure

Walk the element's own beats in teaching order — each beat (per `<Subsection>`) as a bold heading or
its own NoteBlock, every finding under the beat it belongs to. An instructor prepping scans in
teaching order; a topic-sorted note forces them to re-sort it themselves.

### Content per beat

- **Look-fors** — 2–3 per working beat, each naming: the specific student behavior you'll see, why
  it matters for this concept, and what to do with it. "If fewer than half show a counter that
  starts at 0, pause and re-model with the annotated example before they continue" passes;
  "circulate and observe" does not — it names no behavior, no threshold, no move.
- **Coaching cues** — one specific prompt per anticipated stall point: what to *say or ask* when a
  student is stuck. Questions beat corrections ("What value does `i` have on the last pass?").
- **Anticipated challenges** — the misconception triad in full: *what students do / why it happens /
  teacher move*. Observable behavior, never an inferential label: "adds the counter into the sum
  before incrementing" is usable; "doesn't understand loops" is not.

### For assessment elements (answer-and-explanation NoteBlock)

State the answer, then explain why it is correct — and one row per response type or choice: what it
reveals (observable behavior) plus a **concrete next step** naming the specific sub-skill,
representation, or task to revisit. Never "review earlier work". If two rows route alike, the data
isn't working — make them distinct, because they tell the instructor different things.

### Timing (`time` field)

Set it to what a typical student actually needs plus buffer for the transition into and out of the
beat — at a pace real students manage, not the pace you worked it at. Beats in a section should sum
to something honest; an overloaded section is where instructors quietly skip the last exercise.

### NoteBlock types map to content kinds

- `teaching-tips` — facilitation and coaching: how to run the beat (grouping, roles, timing) and
  what to say when students stall. The label/content table scaffold in SKILL.md fits this exactly.
- `check-for-understanding` — the quick read of where the class is: the 1–3 items or observations,
  and how each likely response routes to a next step. Small on purpose — a five-minute read, not an
  assessment event.
- `answer-and-explanation` — keys plus per-response routing, per the section above.

---

## 5. Hints & scaffolds (the `hint` fields)

- **A hint helps the student start, not answer.** It never pre-fills what the task asks for: a
  frame to begin from ("Start with an empty array and add one element at a time") passes; a partial
  solution does not.
- **No keyword strategies** — "look for the word 'each' — that means use a loop" undermines exactly
  the skill being tested and fails on any stem phrased differently.
- **Density cap: 1–2 embedded supports per problem.** Pick the primary scaffold first; add a second
  only if it contributes a genuinely different mode (a structure frame plus a smaller worked input).
  Two scaffolds doing the same cognitive work — drop one.
- Acceptable forms: sentence/structure frames ("I know ___. The part I don't know is ___"); visual
  organizers matched to the concept; reduced complexity **with an explicit transfer step** ("try this
  with three items first, then apply it to the full list"); vocabulary support for a term (not for
  choosing the answer).
- **Fade across the sequence.** As exercises progress through a section: scaffolded → guided →
  independent. Don't announce the removal — no "(no hint this time)"; the support simply isn't there
  on the later exercise, and students meet it on its own terms.
- **Same hook for everyone.** The interesting context goes to every student; only the supports
  differ. A section that gives the engaging problem to some and drill to others has failed both
  rigor and engagement.

---

## 6. Verify before shipping

- **Run your code.** Every playground file set and every inline code block you author must be
  verified to run and produce what you claim it produces — the platform's WebContainers exist for
  exactly this, so there is no excuse for an untested example. A code example that errors on first
  run costs more trust than its teaching value is worth; students assume the *next* example is also
  broken.
- **Check your keys.** Work every answer key, expected output, and quantitative claim by actually
  running or computing it: the stated answer must be the one the problem actually produces, and an
  "expected output" block in content must match what the code really prints.
- **Run Gates A and B on shipped assessment items** (§3), reading the item back as text after
  writing — a keyed question with two defensible answers looks fine while you're composing it.

---

*Adapted from Learning Commons' K-12 Agent Skills (Apache-2.0, co-developed with Anthropic):
check-for-understanding item design and verification gates, lesson-prep anticipation practice,
lesson-plan misconception/look-for structures, and differentiation scaffold rules.*
