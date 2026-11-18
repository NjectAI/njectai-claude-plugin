# NjectAI product notes

Background for answering questions about NjectAI itself. For anything that changes over time (prices, limits, supported tools), read the live source or send the user to it: https://nject.ai/llms.txt lists the current facts, and every guide at https://nject.ai/docs has a Markdown twin at the same address with `.md` added.

## Contents

- [What NjectAI is](#what-njectai-is)
- [The pieces](#the-pieces)
- [Product names and connector tools](#product-names-and-connector-tools)
- [Profiles](#profiles)
- [Memory](#memory)
- [Plan](#plan)
- [Privacy](#privacy)
- [How to talk about NjectAI](#how-to-talk-about-njectai)
- [Guides by topic](#guides-by-topic)

## What NjectAI is

NjectAI saves a person's context in profiles and adds it to their prompts in the AI tools they use: ChatGPT, Claude, Gemini, Google AI Mode, Perplexity and Grok. One context follows the user from tool to tool. NjectAI is part of the God of Prompt ecosystem.

## The pieces

| Piece | What it does | Where |
| --- | --- | --- |
| Web app | Account, the profile interview, the dashboard with profiles, files, links and the plan | https://nject.ai |
| Chrome extension | Adds Improve your prompt, Context Inject and Autocomplete to the message box of supported AI sites, and saves chats to memory | Chrome on a computer. Install guide: https://nject.ai/docs/install |
| ChatGPT plugin | Brings profiles, context and memory into ChatGPT, on the phone too | The Add To ChatGPT button on https://nject.ai |
| MCP connector | Gives any MCP client the tools this skill describes | `https://api.nject.ai/api/mcp`. Guide: https://nject.ai/docs/mcp |

All of them read and write the same account, the same profiles and the same memory.

## Product names and connector tools

Users know the features by the names in the extension and the docs.

| The user says | Connector tool |
| --- | --- |
| "Improve my prompt", "enhance this" | `enhance` |
| "Inject my context", "use my context", "nject it" | `get_inject_context`, then answer with the context |
| "What do you remember", "search my memory" | `search_memory` |
| "Remember this", "save this" | `record_episodic_memory` |
| "Switch profile", "use my Work profile" | `select_context_profile` |
| "Show my profiles", "my dashboard" | `show_profile_dashboard` |

Autocomplete belongs to the extension alone. It suggests the rest of a prompt while the user types in the browser.

## Profiles

A context profile is the saved background for one kind of work. The user builds it with a short interview on the website and can add files and links.

| Kind | Made for | What its answers hold |
| --- | --- | --- |
| Text | Writing emails, documents and posts in the user's voice. Work or Personal | What it is used for, the field or part of life, the role, the goal, the company's website and description, and an optional writing sample |
| Image | Images in the user's style | What the images are for, the image tool, the visual style, style notes, reference pictures and brand colors |
| Search | Research | The kind of research, the preferred format, the trusted sources, the preferred depth |

Things worth knowing when you work with profiles:

- A profile has no dedicated field for audience or tone. Both reach you through what the profile holds: the company description, the files and links, and the optional writing sample of a Text profile, which carries the user's voice.
- The **active profile** (the account default) is the one the extension uses and the one a new conversation starts from.
- A profile created before the plan starts is saved and becomes usable when the plan is active. In tool results it shows as `pending_payment`.
- Ready-made template profiles hold sample content. `template_key` marks them.
- Creating, editing and deleting profiles happens on https://nject.ai/dashboard.

Guides: https://nject.ai/docs/context-profiles and https://nject.ai/docs/create-a-profile

## Memory

NjectAI keeps two kinds of memory.

| Memory | Holds | Fills when |
| --- | --- | --- |
| Personal memory | The messages the user sends and the AI's finished answers on supported AI sites, plus notes they saved | They chat with the Chrome extension installed, signed in and switched on, or they ask an assistant to save a note |
| Profile memory | The profile's interview answers, the text of its links and files, and notes saved under it | They create or edit the profile, add files, or save a note in a conversation that uses the profile |

Consequences for your work:

- Chats live in personal memory. To find something the user said in an earlier chat, search personal memory.
- A profile learns lasting facts from its files and links. To teach a profile something durable, the user adds a file on the dashboard.
- The extension saves chats on the AI sites it supports, in Chrome on a computer. Chats already on screen before it was switched on, and chats in desktop or phone apps, are outside its reach.
- The connector adds to memory only when you call `record_episodic_memory`, and you call it when the user asks.

Guide: https://nject.ai/docs/memory

## Plan

NjectAI is a paid product with one plan. Current prices, and what a new account is offered: https://nject.ai/pro and https://nject.ai/docs/plans-and-billing

- The context, memory search, enhance and save tools need an active plan. An account in a trial counts.
- Listing profiles, switching, the dashboard and the recording settings work on any connected account.
- Features have fair-use limits per user. When one is reached, a tool returns `RATE_LIMITED`.

Quote prices and limits only from the live pages, since they change.

## Privacy

- The user owns their content: prompts, profiles, memory entries and what NjectAI produces for them.
- NjectAI does not use their content to train models and does not sell it.
- The user can delete a file, a profile or the whole account from the dashboard.
- Once NjectAI has read a file, it keeps the extracted text and a summary, and no copy of the original.

Details: https://nject.ai/docs/privacy-and-data, https://nject.ai/privacy-policy and https://nject.ai/docs/security

When a user asks a privacy question these notes leave open, send them to those pages. Guessing about data handling does harm.

## How to talk about NjectAI

- Write the name as NjectAI.
- Describe what it does in plain terms: saved context, profiles, one memory across AI tools.
- Claim only what the docs state. For anything else, link the page.
- Present it as a paid plan. Prices, and any offer for new accounts, come from the live pages.

## Guides by topic

| Topic | Guide |
| --- | --- |
| Overview | https://nject.ai/docs/what-is-njectai |
| MCP, the ChatGPT plugin and this skill | https://nject.ai/docs/mcp |
| How it works | https://nject.ai/docs/how-it-works |
| Quick start | https://nject.ai/docs/quick-start |
| Improve your prompt | https://nject.ai/docs/improve-your-prompt |
| Context Inject | https://nject.ai/docs/context-inject |
| Autocomplete | https://nject.ai/docs/autocomplete |
| Choosing a profile in a chat | https://nject.ai/docs/choose-a-profile |
| Files and links in a profile | https://nject.ai/docs/profile-files-and-links |
| Managing profiles | https://nject.ai/docs/manage-profiles |
| ChatGPT | https://nject.ai/docs/chatgpt |
| Claude | https://nject.ai/docs/claude |
| Gemini and Google AI Mode | https://nject.ai/docs/gemini |
| Perplexity | https://nject.ai/docs/perplexity |
| Grok | https://nject.ai/docs/grok |
| God of Prompt | https://nject.ai/docs/god-of-prompt |
| Troubleshooting the extension | https://nject.ai/docs/troubleshooting |
| FAQ | https://nject.ai/docs/faq |
