# Most complex AI system I've built: the Incrementors lead-to-close operating system

*(Draft answer for the application form. Written from the connectors, workflows and skills present in this workspace. Edit anything that reads wrong; the numbers in square brackets are placeholders for you to fill in.)*

## What it does, in one paragraph

Incrementors is a digital marketing agency that gets inbound leads from five places at once: the website form, Upwork proposals and invitations, LinkedIn, cold email, and old clients coming back. I built an agent layer that sits on top of all of them. Every new lead is captured into one database, qualified against a knowledge base, replied to in the right channel and voice, booked onto a calendar, followed up on a schedule, and handed to a human the moment money or a meeting is on the table. It runs on Claude (the Claude Agent SDK and Claude Code with MCP connectors), n8n for the deterministic plumbing, and a small internal CRM I built for it. Today it handles [X] inbound leads per week with [Y] human hours, versus [Z] before.

## Architecture

**1. Ingestion (n8n + MCP connectors).**
Sources are normalised into a single lead record:
- Gmail (two mailboxes, `marketing@` and `crm@`) via a Gmail MCP: an hourly sweep pulls the last hour of mail, threads it, and strips signatures and quoted replies.
- Upwork via a custom Upwork MCP: proposals I've sent, client invitations, and the message inbox. Job search and proposal submission also run through it.
- LinkedIn DMs through Claude's Chrome extension (no LinkedIn API), capped at 10 messages a day and restricted to decision-maker titles.
- Website forms and Apollo enrichment for company size, title and tech stack.
- Fireflies for call transcripts, so the lead record includes what was actually said on the discovery call.

**2. One CRM, exposed as tools.**
Instead of paying for a CRM, I built one (Postgres + a small API) and exposed every operation as MCP tools: leads, pipelines and stages, conversations, tasks, meetings, bookings, documents, tickets, follow-up templates, and automations. Two retrieval indexes sit beside it:
- a *knowledge* index (sales playbooks, objection handling, SOPs, uploaded PDFs) that the agent must search before writing any reply, and
- an *examples* index (case studies, sites built, rankings achieved) so the agent cites real proof instead of inventing it.
Both are embedding search (cosine match), not keyword filters, so the agent can ask "client says we're too expensive, what do I say?" and get the right passage with its source.

**3. The supervisor agent (n8n "Supervisor_Sales" workflow).**
A Claude-backed supervisor receives the lead plus its full conversation history and decides one of: qualify, reply, follow up, book, escalate, or ignore. Sub-workflows handle project onboarding once a deal is signed. The supervisor's reply goes through hard rules encoded as skills:
- reply inside the existing thread, never a new email;
- draft only, unless the skill is explicitly allowed to send (the hourly website-lead responder is; the old-client reactivation is not);
- one specific, verifiable detail per message (a finding from an audit, a line from their own brief), never generic praise;
- scam, guest-post, link-seller and agency-pitch patterns are dropped before anything is written.

**4. Skills as the unit of behaviour.**
Each recurring job is a versioned skill with its own triggers, guardrails and output format. The main ones:
- *lead-auto-respond*: hourly sweep, qualify, capture, reply, set next follow-up date, then sweep everything due today and re-bump the date so the loop is idempotent.
- *upwork-lead-capture* and *upwork-ai-followups*: the same loop for Upwork, with connects budgeting.
- *old-client-shortlist → old-client-ai-audit → old-client-outreach*: picks dormant clients, logs into SEMrush through a browser profile, captures real metrics, builds a PDF audit per client in the house format, then drafts a reconnect email that references one finding from the audit.
- *ai-visibility-audit* and *competitor-analysis*: given a lead ID and a domain, scrape the site, check visibility in ChatGPT, Perplexity and Gemini, compare against competitors, and produce a PowerPoint or Word report the sales team can send the same day.
- *book-meeting*: checks Google Calendar and TidyCal availability so the agent never double-books.
A "Skill Manager" MCP stores the skills, global instructions (rules applied to every skill), guardrails, logins, and long-term project context, so a new session loads the same rules every time.

**5. Reporting.**
A weekly KPI skill pulls from the CRM, Google Analytics, Search Console, Upwork and the leads database and produces the same report every Monday. Every message the agent sends or drafts is logged to the conversation manager with a timestamp, so the pipeline is auditable end to end.

## Why it's built this way

- **Deterministic where it can be, agentic where it must be.** n8n does scheduling, retries, dedup and delivery. Claude does judgement: is this a real lead, what should we say, what proof do we cite. Neither is asked to do the other's job.
- **Draft-by-default.** Sending is a per-skill permission, not a global one. The hourly responder can send because the audience is inbound and the reply is a welcome message. Reactivation and cold email stay as drafts because a wrong message there costs a relationship.
- **Retrieval before generation.** The agent is not allowed to answer an objection or cite a result from memory. It searches the playbook and the examples library first. That single rule removed most of the hallucinated claims we saw in early versions.
- **Thread discipline.** Replies always land in the existing thread, from the right mailbox. This kept deliverability and made the CRM history honest.
- **Human in the loop at the money line.** Anything involving a price, a contract or a booking with a new client becomes a task for a person. The agent prepares everything; it doesn't close.

## What went wrong and what I changed

- Early versions replied to short, vague inquiries as spam. The fix was a rule that "short and vague still qualifies", plus explicit negative patterns for the things that don't.
- Guessing email addresses from name patterns caused bounces. Now the order is: conversation history, web search, Apollo, and only then a pattern guess on a verified employer domain.
- Two mailboxes with different voices confused the model. Each skill now declares its sender account and tone up front.

## Stack

Claude Agent SDK and Claude Code (skills, MCP), n8n (Supervisor_Sales and sub-workflows), custom CRM with embedding search (Postgres), Gmail API, Google Calendar, TidyCal, Upwork API, Apollo, Fireflies, Firecrawl for scraping, Google Analytics and Search Console, Chrome extension for LinkedIn and SEMrush, Lovable for client-facing pages.
