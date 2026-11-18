# NjectAI for Claude

NjectAI keeps your context profiles (your business, product, audience, writing
voice, files and links) and a long-term memory of your work. This plugin brings
them into Claude. Claude pulls the context a request needs before it writes,
answers in your voice, improves a prompt with your profile, searches what you
said in earlier chats, and saves the notes you choose.

## What's inside

- **The NjectAI connector**, the remote MCP server at
  `https://api.nject.ai/api/mcp`. It is the same server as the NjectAI
  connector in Claude's directory, so you see one set of NjectAI tools.
- **The `njectai` skill**, which teaches Claude when and how to use those
  tools: open a context session, choose or switch a profile, pull saved context,
  search memory, improve a prompt, save a note you pick, and show your profile
  dashboard.

## Before you start

You need a NjectAI account. Pulling context, searching memory, improving
prompts and saving notes need an active NjectAI plan. Create profiles at
[nject.ai](https://nject.ai).

The first time Claude uses a NjectAI tool, Claude asks you to connect: you sign
in on NjectAI's own page and approve access. You can disconnect at any time in
Claude's connector settings.

## What it sends and where

The plugin runs no code on your computer. It adds the skill and one remote
server, `https://api.nject.ai/api/mcp`, run by NjectAI. Claude contacts that
server only when it calls a NjectAI tool, and sends what that tool needs:

- the request it wants context for, or the prompt you asked it to improve;
- your search words, when it searches your memory;
- a note, only when you ask Claude to save it, with just the content you chose;
- your choice of profile, when you switch.

The server answers with your own profile data, memory and improved prompts.
Claude never saves whole conversations through this plugin, and it changes the
memory recording setting only when you ask.

How NjectAI handles your data: [Privacy Policy](https://nject.ai/privacy-policy)
and [Terms of Service](https://nject.ai/terms-of-service).

## Help

Docs: [nject.ai/docs/mcp](https://nject.ai/docs/mcp). Support:
[nject.ai/contact](https://nject.ai/contact).

## License

MIT
