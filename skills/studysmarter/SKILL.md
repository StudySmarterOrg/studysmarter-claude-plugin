---
name: studysmarter
description: Plan and build study plans, flashcards and lessons in the learner's StudySmarter account with the StudySmarter tools. Use when the learner wants to prepare for an exam, turn material into flashcards, plan when to study what, or change their existing StudySmarter cards, including adding an image to a card (cards are text only); they learn in the StudySmarter app.
---

# Building study plans in StudySmarter

The learner plans and builds here; they learn in the StudySmarter app (spaced repetition, Fluid Learn Mode, chapter
tests). A **study plan** is a study set with an outline of chapters and lessons, each lesson holding flashcards and
the slides that teach them, plus the due dates you plan. A study set without an outline is a plain flashcard set.

## Always

- **Start with `get_study_overview`.** It tells you who the learner is (school or university, class level, language,
  Premium) and which study sets they already have, with their exam dates and plans. Use it to ask fewer and sharper
  questions, and to offer an existing set instead of a new one; ask when unsure. Only sets the learner created are
  reachable here. Sets they only contribute to, saved or got shared are not, and sharing a set does not make it
  reachable: never promise access once someone shares theirs. Tell the learner they find such a set in the app.
- **Numbers come from the tools.** Report counts of cards, lessons or sets only as a tool returned them
  (`cards_total`, `total`, the ids a call returned), never estimated or summed from memory or a first page.
- If the learner is unhappy with the quality, suggest a higher thinking mode once more.

## Creating a study plan

1. **Clarify the basics, once.** Whatever the learner has not said yet: the goal (the exam and its type, or what they
   want to be able to do), the topics it covers and where it ends, their starting point (prior knowledge, depth),
   what they find hardest, and the schedule (exam date, time per week, start day). Ask in the conversation, one or
   two questions at a time, each with a suggested default the learner can just confirm. Ask even when the learner
   asks to create the plan right away; skip it only when they gave goal, topics and schedule already, or said to skip
   questions. Never ask twice. Suggest once to use a higher thinking mode, since it gives better plans.
2. **Preview the outline** in the chat, in whatever format reads best, but only show what the learner will find in
   the app: no images, covers or features StudySmarter doesn't have. Create it once the learner agrees.
3. **Create the plan.** `create_study_plan` with name, study goal, exam date, outline and the files the learner shared
   (`files`). Every chapter needs at least one lesson; aim for 3-8 chapters with 2-6 lessons each, ordered the way the
   learner should study. Its `materials` give each file's `material_id`, so the cards built from a file cite it.
   Schedule in the same call: a `due_date` per lesson and a `test_due_date` per chapter. Everything, chapter tests and
   the final review included, is due by the day before the exam at the latest; the exam day itself only when the
   learner wants that, e.g. for an evening exam. Never move the exam date to make a plan fit. The first lesson is due
   today, so there is something to do right away, unless the learner wants to start later. Leave room for review
   before the exam. Dates the learner gives are binding. The study days and time per week they named are a limit:
   when the plan does not fit into them, explain the conflict and agree on a way out (more time, less scope, other
   priorities) before scheduling more. After saving, check the dates in the result. Tell the learner the plan and
   its dates can be changed later.
4. **Fill the chapters, ahead of the learner.** A new outline's lessons are empty, and the app cannot open a chapter
   without cards, so an empty chapter is a dead end the learner can only leave by coming back to you. Fill a chapter
   completely: for every lesson, first `create_flashcards` with its `lesson_id`, then `write_lesson`, whose slides
   teach, cite and check those cards, with a check on every slide and a figure on most. Fill the first chapter right
   away. Then ask the learner whether to fill the remaining chapters now, which takes a while, or as they go on, so
   later chapters can adapt to their progress.
5. **Check.** `get_study_plan` shows `is_filled` per chapter, and per lesson `cards_total` and `is_written`.
   Fill what is still empty in the chapters agreed on before you hand over.
6. **Hand over.** Tell the learner the plan is in the StudySmarter app, ready to learn, and show its `app_link`,
   which opens the plan in the app. `get_study_overview` gives the link of every set.

## Replanning

- Whenever the learner comes back, check their progress and whether they are on track with the plan.
  `get_study_plan` shows per lesson the `due_date`, completion, `cards_practiced`, `cards_with_mistakes`,
  `cards_memorized` and `cards_mastered`, and per chapter the `test_due_date`, every entry of `test_attempts` and the
  `cards_to_review`. Memorized takes several right answers in spaced repetition, so few memorized cards right after a
  first session are normal, not a problem. Then call `get_lesson` for the lessons with mistakes or weak tests: per
  card its `progress` gives the `wrong_answers` and `unsure_answers`, the `recent_answers` with any misconception, and
  `next_review_at`. Based on that, suggest adapting the plan or filling the next empty chapter (`is_filled`), and act
  once the learner agrees.
- Base the change on that: add review lessons or cards for what had mistakes or tested badly, give a struggling
  chapter more time, move open lessons, and do not move what is done. Tell the learner what the data showed.
- An earlier exam is no permission to study more: keep within the study days and time per week the learner named.
  If the plan no longer fits, explain the conflict and agree on more time, less scope or other priorities before you
  schedule a denser plan.
- `update_study_plan` takes the complete outline in order: existing chapters and lessons by `chapter_id` and
  `lesson_id`, new ones without.
  Every existing chapter and lesson must appear; to drop some, use `remove_from_study_plan` (their cards stay in the
  set). A lesson listed under another chapter moves there.
- Plans generated in the StudySmarter app cannot be changed here; the learner edits those in the app.

## Flashcards

- One fact per card (atomic). The front asks one clear question or names one term; the back answers it, short.
- Cards go into the study set itself. Subtopics the learner made in the app are only shown, as a card's
  `subtopic_path`, so you can tell them where to find it; a card stays in its subtopic when you change it.
- Multiple-choice cards are read-only here: you see their correct answer and `wrong_options`, and can change their
  front, tags and lesson, but their options only in the app.
- Same language as the learner's material or set, unless they ask otherwise.
- Pick a fitting format: question and answer, term and definition, or vocabulary.
- Card texts are Markdown, never HTML: `**bold**`, `*italic*`, `` `code` ``; on the back also paragraphs, lists,
  tables and fenced code blocks, which name their language (```` ```python ````). The front is one line.
- Math on cards is raw LaTeX: inline `\( ... \)`, display `\[ ... \]`, fractions with `\frac{a}{b}`. Never single
  `$` on a card; lesson slides use `$...$` instead (see Lessons).
- Cards are text only: images cannot be added here. When the learner asks to add an image to a card, your first reply
  says so: the card stays unchanged, and they can add the image to the card in the StudySmarter app. Do not generate
  an image instead of answering. You may offer to create one separately afterwards, but never present it as added to
  the card.
- Texts the learner formatted in the app (colours, styled tables, images, formula boxes) read as plain text and can only be
  edited in the app; `update_flashcards` skips them with that reason, and their tags and lesson still change.
- Do not address the learner in the card text.
- A card built from an uploaded material cites it in `source`: its `material_id` and the `page` the fact is on.
  The app shows it with the card. Only materials of the card's study set count.
- At most 100 cards per call. Use tags sparingly, for themes that cut across lessons.
- Prefer `update_flashcards` over deleting and recreating, so the learner's progress on a card is kept.
- Cards added to a lesson that is already written (`is_written`) are not taught on its slides yet. Rewrite that
  lesson with `write_lesson` so every card is taught and cited, or offer to when the learner only asked for cards.

## Lessons

A lesson is a short paged lesson the learner reads one slide at a time, with a tutor beside it: a hook, then teaching
slides, each with a check and most with a figure under the prose. Write it with `write_lesson` once its cards exist, in
the cards' language: each slide's `content` is the text, its `figure` the figure. `get_lesson` shows the cards and
what is written so far.

**Text (`content`):**

- **Hook:** one concrete question, puzzle or situation from the learner's world, at most 30 words, that the lesson
  then answers.
- **Slides:** 4 for up to 5 cards, 5-6 for up to 9, 6-7 for more. Each teaches ONE idea: a `### ` heading naming it,
  then 60-90 words of prose in short sentences or a short list, with a real example; a worked example with real
  numbers where the cards carry quantities, formulas, dates or thresholds. Join cards on one slide only when they
  explain the same idea; a heading that needs "and" is two slides. Parallel items (capitals, organelles, dates) are
  each named and explained, two or three to a slide. Build each slide on the one before; never repeat one. Bold the
  key term once.
- **Reading level:** take it from `learner` in `get_study_overview` (class level, school or university). Pupils up to
  class 10 get 50-70 words, sentences of at most 15 words, every term explained in everyday words and at most one new
  term per slide (in German "du"); pupils from class 11, trainees and adults sentences of at most 20 words; university
  students the field's terms, sentences of at most 25 words. Only the wording changes; every card is still taught.
- **Citations:** cite every claim from a card with `[@flashcard_id]` at the end of its sentence; every card is cited on
  some slide. Never in code or backticks. Add no facts the cards do not support, and do not mention flashcards or ids
  in the text.
- **Checks:** write one on every slide, as the app's own lessons do. The lesson keeps 1 for up to 3 cards, 2 for up
  to 6, 3 for more, spread out and never on two slides in a row; a lesson with fewer valid ones is refused. Each tests
  the idea of its slide, testing ONE idea with the 1-2 cards in
  `flashcard_ids`. Everything needed to answer is on the slides; never ask for a calculation the cards do not teach.
  Fit the type to the idea and vary the types across the lesson:
  - `apply`, for the rest: 3-4 options, exactly one correct. For a rule or mechanism, a short new case the learner
    applies it to; for a plain fact, the question asked directly with close, plausible alternatives, never wrapped in
    an invented scenario. Never quote a card as an option. Wrong options are plausible confusions from the material,
    written for this check; never generic filler, and never the same ones reused across checks.
  - `sort`, where the idea is a distinction between two or three concepts: 2-3 groups, 4-6 new examples in all, each
    in exactly one; without that many clear examples, use `apply`.
  - `order`, only where the idea itself is a real sequence: its 3-6 steps in the correct order, 2-8 words each.
  - `own_words`, at most once per lesson (a second is refused) and only where the cards call for a synthesis rather than a task: the
    explanation is the model answer, one sentence.
  - Every explanation says in 1-2 sentences why.
  - Each check states its `type_reason` first: why that type fits exactly this slide. A lesson whose 3 or more checks
    are all of one type is refused.
- Per slide also 2-3 `follow_up_questions` a curious learner might ask next, and `needs_example` when the tutor
  should add one.
- Math in LaTeX: `$...$` inline, `$$...$$` for display, fractions with `\frac`; money stays plain text. No raw HTML;
  show markup in backticks and multi-line code in a fenced block naming its language.
- No figures and no Markdown tables in the text: a table on a slide is a `table` figure, in the slide's `figure`.

**Figures (`figure`):**

- One figure per slide, shown under its prose. It shows the SHAPE of the slide's idea so it is grasped at a
  glance, adds a shape the prose does not have and never repeats the prose as boxes. Most slides get one, and a lesson
  with figures on fewer than half of its slides is refused; leave the figure empty only where the idea has no shape or
  the figure would just restate the slide.
- Each figure states its `figure_type_reason` first: why its type fits exactly this slide, by the choosing order
  below. Ideas differ, so do their shapes: a lesson whose 3 or more figures are all of one type is refused.
- Only facts stated on that slide. The title names what the figure shows, not the slide heading again.
- Choose the type in this order: numbers to plot are an `interactive-chart` when they are a y = f(x) with a
  parameter to tune (also a quantity that grows or decays in a known way), else a `chart`; a flow that branches or
  merges is a `diagram`; otherwise the most specific structured type. `compare` is the last resort, only where the
  slide contrasts exactly two things point by point; three or more parallel items are a `table` or a `hierarchy`.
- Each figure is a fenced block opened with ```` ```visual ````, holding ONE JSON object with `"version": 1`, a `type`
  and a `title`. All text inside is plain (no LaTeX, no citations), except a formula's `code` and symbol labels,
  whose backslashes are doubled in JSON. The `figure` field's description lists every type's fields; the common ones:
  - `steps`, `timeline`, `cycle`, `hierarchy`, `layers`, `scale`: `items` = `[{"label", "detail"}]`, with `top`
    (what a cycle produces, the whole of a hierarchy, a pole) and `bottom` where they fit.
  - `compare` / `before_after`: `columns` = exactly two `{"heading", "bullets"}`, `bottom` = the takeaway.
  - `table` (3+ columns of facts): `columns` = `[{"heading"}]`, `rows` = arrays with one cell per column.
  - `definition`: `top` = the term, `bottom` = the definition. `callout`: `top` = one thought in full.
  - `formula`: `code` = LaTeX without delimiters, `items` = symbols and their meaning.
  - `chart` and `interactive-chart`: the plot under `chart`, see the field description.
  - Example: `{"version": 1, "type": "timeline", "title": "How the cell theory grew", "items": [{"label": "1665",
    "detail": "Hooke names cells"}, {"label": "1839", "detail": "Schwann: all organisms are made of cells"}]}`.

A lesson that breaks these rules is refused with the reasons; fix them and call again. `write_lesson` gives
`check_count` against `checks_required` and the `figure_count`.

## Material

Files the learner shared in ChatGPT go into `create_study_plan`; one shared after the plan exists goes in with
`upload_material`. Either way it is stored in the study set, where the learner opens it in the app; Word, PowerPoint
and similar files are converted to PDF. Storing does not read the file for you: write the cards and lessons from the file yourself, each card citing its page in
`source`. In Claude, files cannot be handed over; work from text the learner pastes.

## Limits

On the free plan, the learner can create a limited number of study sets, as in the app. When a tool says the study
set limit is reached, tell the learner that StudySmarter Premium in the app removes it, or that they can add to an
existing set or delete one.
