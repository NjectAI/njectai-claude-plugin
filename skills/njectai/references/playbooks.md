# NjectAI playbooks

Worked examples for the subtler requests. `SKILL.md` holds the rules: these show them applied. In the calls below, `S` stands for the `session_id` from `start_context_session` and `V` for your latest `selection_version`. Profile names are examples.

## Contents

1. [Write something in the user's voice](#1-write-something-in-the-users-voice)
2. [Research with a Search profile](#2-research-with-a-search-profile)
3. [Image prompts with an Image profile](#3-image-prompts-with-an-image-profile)
4. [The selected profile does not fit the task](#4-the-selected-profile-does-not-fit-the-task)
5. [Draw on everything NjectAI knows](#5-draw-on-everything-njectai-knows)
6. [Recall something from earlier chats](#6-recall-something-from-earlier-chats)
7. [Improve a prompt, then run it](#7-improve-a-prompt-then-run-it)
8. [Save notes](#8-save-notes)
9. [Recording questions](#9-recording-questions)
10. [One profile per client](#10-one-profile-per-client)
11. [Coding agents and long tasks](#11-coding-agents-and-long-tasks)

## 1. Write something in the user's voice

"Write a LinkedIn post about our launch." This is the first NjectAI request of the conversation.

1. `start_context_session` with `{ "client_request_id": (unique to this conversation) }`. Keep `S` and `V`.
2. `list_context_profiles` with `{ "session_id": S }`. The selected profile is "Brand Voice", a Text profile: it fits.
3. `get_inject_context` with `{ "input": "LinkedIn post announcing our product launch", "session_id": S, "expected_version": V }`.
4. Write the post:
   - Open with one line: "Using your Brand Voice profile."
   - Take facts about the business, product and audience from the context.
   - Match the writing sample if the profile has one.
   - Leave a marked slot for facts it lacks, such as `[launch date]`, and list the slots after the draft.

For "make it shorter" or "more casual", edit with the context you already have. For a new topic, fetch again.

## 2. Research with a Search profile

"Research the new EU rules for my industry." "Compare these three vendors."

1. Use the Search profile: selected already, or by the fit rule.
2. `get_inject_context` with the research question.
3. Apply the profile's preferences to your method and your output:
   - **Preferred sources**: search those kinds of sources first, and say when you step outside them.
   - **Preferred format**: a timeline, a table, a brief, as the profile says.
   - **Preferred depth**: how far to go before you stop.
4. If you have a web search tool, keep the user's private details out of the queries unless the question needs them.

## 3. Image prompts with an Image profile

"Make a thumbnail for this video." "Write a Midjourney prompt for our hero image."

1. Use the Image profile: selected already, or by the fit rule.
2. `get_inject_context` with a description of the image wanted.
3. Build the prompt on the profile's visual style, brand colors and style notes, in the form the user's image tool expects. The profile names the tool.
4. If you can generate images yourself, carry the same style details into your own generation request.

## 4. The selected profile does not fit the task

**One clear match.** The session is on "Legal Research" (Search). The user also has "Brand Voice" (Text) and "Thumbnails" (Image), and asks for a sales email.

1. Brand Voice is the only Text profile, so it is the match.
2. `get_inject_context` with `{ "input": (the request), "session_id": S, "expected_version": V, "profile_id": (Brand Voice) }`. The session stays on Legal Research.
3. Write the email, and open with: "Your selected profile is Legal Research, so I used Brand Voice for this email. Want me to switch this chat to it?"
4. On a yes: `select_context_profile` with Brand Voice and `"update_account_default": false`. Keep the new `V`.
5. For the next writing request in this conversation, use Brand Voice again without asking.

**Several candidates.** The user has "Acme Emails", "Birch Emails" and "Cedar Emails" and asks for "a follow-up email to the client". Ask which client before you fetch anything.

**The user switched by hand.** A call returns `SELECTION_CONFLICT` and the session is now on "Thumbnails", while you are writing a post. Adopt the new state and say what you see: "You switched to Thumbnails. For this post I'm staying with Brand Voice." Then call with the new `V` and Brand Voice as a one-call override.

**Skip all of this** when the user named the profile, or when the selected one fits.

## 5. Draw on everything NjectAI knows

"Use everything you know about me to write my speaker bio."

1. `get_inject_context` with `{ "input": "Speaker bio", "session_id": S, "expected_version": V, "context_policy": "profile_plus_personal" }`. The context now includes the account's personal memory beside the profile.
2. A bio is public. Personal memory spans every client and every private chat, so use what belongs in a bio and check anything sensitive with the user before it goes in.

To keep that policy for the rest of the conversation, call `select_context_profile` with the current profile, the same policy and `"update_account_default": false`. Keep the new `V`.

## 6. Recall something from earlier chats

"What did I decide about pricing last month?"

1. `search_memory` with `{ "query": "pricing decision", "limit": 10, "session_id": S, "expected_version": V, "profile_id": null }`. Personal memory holds the chats captured from the user's AI tools.
2. If that misses, try one other phrasing ("price change", the product's name), or the selected profile when a file or a saved note could hold the answer: the same call without `profile_id`.
3. Keep the results whose dates fall in the period the user named. The tool has no date filter.
4. Report what you found with its date: "On 3 October you settled on annual billing with a monthly option." When memories disagree, lead with the newest and mention the earlier one.
5. If nothing turns up after three searches, say so: "I found nothing about that in your NjectAI memory." Mention that memory holds chats from the time the NjectAI extension was installed and switched on.

## 7. Improve a prompt, then run it

"Improve this prompt: write a promo email for our weekend sale."

1. `enhance` with `{ "text": "write a promo email for our weekend sale", "session_id": S, "expected_version": V }`.
2. Show the suggestion in full, in a quote or code block, under a line such as "Here is the improved prompt, built on your Brand Voice profile:".
3. Close with the options: "Use it here, take it to another tool, adjust it, or keep your original?"
4. If they say to run it here, treat the accepted prompt as their request. Ask for the value of any slot in it, such as `[discount terms]`, then write the result. The suggestion already carries the profile's context, so a separate `get_inject_context` call is rarely needed.

If they take it elsewhere or keep the original, you are done.

## 8. Save notes

**A fact.** "Remember that our launch moved to 3 December."

```text
record_episodic_memory
{
  "mode": "explicit",
  "session_id": S,
  "events": [{ "role": "user", "content": "The Project Cedar launch moved to 3 December 2026." }]
}
```

The note keeps the user's meaning and adds the name and the year that keep it clear later. Confirm with the status: "Saved to your NjectAI memory under Brand Voice: 'The Project Cedar launch moved to 3 December 2026.' It becomes searchable shortly."

**A passage of yours.** "Save that last paragraph." Send the paragraph word for word with `"role": "assistant"`.

**Several notes.** Send them as separate events in one call, up to 20.

**A note that has nothing to do with the selected profile.** The conversation is on a client's profile and the user says "remember that I prefer morning meetings". Leave `session_id` out, so the note goes to personal memory alone, and say so.

**Another profile.** The note goes to the session's selected profile. If the user names a different one, ask whether to switch the conversation to it first, then save.

**"Save this whole conversation."** Propose the handful of points worth keeping as a short list, and save the ones the user confirms, each as a `user` event when it restates what they said or decided. If they still want the full conversation, send the messages in order with their roles, 20 events per call, and tell them how much went in.

**Something sensitive.** If it looks like a password, a key, or a card or ID number, say that memory resurfaces in later prompts across their AI tools, and save only after they confirm.

## 9. Recording questions

**"Is NjectAI recording this chat?"**

`get_memory_capture_settings`, then answer in plain parts:

- Through this connector, nothing from the chat is saved unless they ask you to save a note.
- When you fetch context, search or improve a prompt, the words of that request go to NjectAI to answer it. They stay out of memory.
- The account's automatic recording setting is on or paused, as the tool says. It covers clients that record automatically through the connector.
- The NjectAI Chrome extension is separate. If they are in Chrome with the extension signed in and switched on, it saves the chat on the AI sites it supports.

**"Turn it off." / "Pause recording." / "Turn it back on."**

`set_memory_capture_settings` with `{ "enabled": false }` or `true`. Confirm with the scope: "Automatic recording is now paused for your whole NjectAI account. Notes you ask me to save still go through." Add that chats in the browser are saved by the extension, which has its own Nject switch in its toolbar menu. Someone who wants this chat left out of memory usually needs that switch.

**"Record all my chats from now on."**

Explain that the connector sees only what you send it in a tool call, so other chats are outside its view. This answer needs no tool call, and nothing gets saved. Offer to save specific notes whenever they ask, and point to the Chrome extension for automatic saving on supported sites: https://nject.ai/docs/memory

## 10. One profile per client

Agencies and freelancers often keep a profile for each client.

- Name the client's profile every time you start work for that client.
- When the user moves to another client, switch profiles before you fetch anything, and fetch fresh context after the switch.
- Keep `profile_only` on these conversations. Personal memory spans every client.
- Before you reuse earlier text, check that it was written for the same client.
- For a one-off question about another client, use a one-call override and leave the conversation's profile alone.
- When the request does not say which client, ask.

## 11. Coding agents and long tasks

In an editor or a terminal agent, NjectAI helps with the parts of the work that speak for the user: a README in their voice, release notes for their audience, product copy, a pitch.

- Open the session when such a task appears. Routine code work needs none of it.
- Fetch once per task and pass what is relevant to any helper agents you start. Each of them opening its own session would cost quota and could land on a different profile.
- Keep the user's private context out of source files, commit messages and logs unless the task is to write it there.
- Build the session's `client_request_id` from the host's conversation or thread ID when there is one. After a context reset, `start_context_session` with that value returns the same session. If the value is lost too, start a new session, check which profile it selected, and restore any choice the user made for this conversation.
