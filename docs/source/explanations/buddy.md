# The Buddy

The Buddy is SprintStart's AI assistant for the people a project takes on: the mentor a new joiner (the hire) meets on their first day. It answers from the project's indexed material and from the hire's own state — their tasks, onboarding path and board — and it renders what it opens, such as a task's orientation packet, in the conversation rather than navigating away.

The buddy is not one page. It has two surfaces that share one conversation: an always-on dock in the corner of the app, and the dedicated page at `/buddy`, which every permission group can open (see [ADR-018](adrs/adr-018-frontend-access-control-model.md)).

## The dock and the page

- The **dock** is mounted for every signed-in user (a page in focus mode hides it), in a corner of whatever page is open. It can be dragged to any of the four corners and is remembered there. It opens a small, non-modal window: the page underneath stays visible and interactive, because the buddy is meant to be consulted *about* it.
- Controls on hire surfaces — for example "Ask your buddy about this" under a card — open the dock with the question already in the composer, without leaving the page.
- The **page** (`/buddy`) shows the same conversation in a full view: the thread with its composer, the mode switcher, and a rail beside it. The dock hides itself there, and "Open full page" grows the dock's window into the page in one motion — it is the same conversation before and after.

## One conversation everywhere

The app provides one buddy session and both surfaces read it, so the dock and the page can never hold different copies of what was said: a question asked in the dock is in the page's thread when it opens, and the draft in the composer survives the move between them.

A visit opens with a proactive greeting, streamed as the buddy typing; the page does not block on it, so the composer works while the greeting is still being written. A hire can also keep several conversations: the rail lists them so an earlier one can be reopened, and a new one is started from the page's "new conversation" control or the dock's own copy.

## Hire conversation and team conversations

The mode switcher offers two kinds of conversation:

- **The hire's own conversation** ("Your onboarding") — the default described on the rest of this page.
- **A team conversation per managed project** — for a user who manages at least one project, the switcher lists the projects they *manage* (not every project they can see). Picking one switches the app's global project selection and points the buddy at that project's team, which is who that conversation is for.

The switcher is only rendered for managers, and it appears in the dock's header as well as on the page. If the global selection moves to a project the user does not manage while a team conversation is open, the conversation falls back to the hire thread. Team mode is scoped to its project, so the hire-only extras below — suggestion chips and the PM replies rail — are not offered there.

## Starting points: the suggestion chips

Before the hire has asked anything, a row of chips ("Not sure where to start?") sits above the composer. The list is not written by hand: it comes from the backend, which derives it from the tools actually mounted for this user. Clicking a chip **fills the composer** rather than sending — the words stay the hire's and can be edited first.

## Asking a person: the PM replies rail

When the buddy cannot help, the hire can send their question to their project's PM. "Send this to your PM" hangs off the hire's **question** — that is the thing that still needs answering — and the reply comes back as durable knowledge, so the buddy can answer the next person directly.

The "Sent to your PM" rail beside the conversation collects those questions grouped by what became of them:

- **Your PM answered these** — the answer itself and its date. The buddy knows it now, so it can be asked directly next time.
- **Still with your PM** — the question, and how long it has been waiting; a wait over a day is flagged.
- **Closed without an answer** — shown rather than quietly dropped, with the advice to ask the PM directly.

The rail appears on the page for the hire's own conversation and only once there is something in it; a team conversation neither loads nor shows it.

## Proposals: nothing changes until you confirm

The buddy can prepare changes — a checklist to keep, a board edit, a task's orientation packet, a question for the PM — but it never applies one by itself. Each arrives as a proposal with the detail needed to judge it (which cards, which lines, what it would do) and applies only after **Confirm**; declining leaves everything as it was. A proposal looks and behaves the same in the dock and on the page.

A reply that contains a to-do list additionally offers **"Keep as checklist"**: it creates a board card from the reply's own lines, so what lands on the board is what the hire read. From then on the card is the hire's — the buddy never touches it again.

## For developers

- The feature lives in `sprintstart-frontend/src/features/buddy/`; the page is `src/pages/BuddyPage.tsx`, routed at `/buddy` in `src/router/AppRouter.tsx`. The dock and the page share the session through `BuddyProvider` — a fallback session was deliberately left out, because two copies disagreeing is the bug the setup prevents.
- Conversation turns run through the backend; the AI service's buddy endpoints (agent, compact, open stream) are listed in [sprintstart-ai](../tutorials/getting-started/getting-started-ai.md).
