Title: OpenWiki: Giving Your Coding Agent a Memory of Your Repo
Date: 2026-10-04
Category: Developer Tools
Tags: OpenWiki, LangChain, AI Agents, Documentation, FastAPI, CLI
Slug: openwiki-repo-documentation-agent

Documentation is the first thing to go stale in any codebase. Code changes every week, and the docs rarely keep up. OpenWiki, an open source CLI from LangChain, tackles this by writing documentation for your repo and keeping it updated as the code changes. It is built mainly for coding agents to read, not just humans.

## What Is OpenWiki

**Repo wiki**

OpenWiki reads your codebase and generates a linked set of Markdown pages in an openwiki/ folder.

Instead of cramming everything into one giant instruction file, the knowledge lives in a proper wiki.

**Agent connection**

After generating the wiki, OpenWiki adds a reference to it in your AGENTS.md or CLAUDE.md file.

Your coding agent already reads those files, so it can find the wiki when it needs repo context, with no change to your workflow.

**Model choice**

You bring your own model provider and API key. The launch post lists OpenRouter, Fireworks, Baseten, OpenAI and Anthropic.

The setup flow I used also offered Gemini through AI Studio, so the list of options is wider than the announcement suggests.

## Trying It on a Simple FastAPI App

To test it, I used the smallest project I could think of: a FastAPI app with one endpoint that reverses a string. A tiny repo keeps the run cheap and makes the output easy to inspect.

**Install**

Install it globally with npm install -g openwiki.

Then move into the project folder, which should be a git repo so you can review changes later.

**Initialize**

Run openwiki --init from the repo root.

The first-run setup asks for your provider, API key and model, and lets you skip the optional LangSmith tracing.

**Wiki brief**

OpenWiki then shows an editable brief describing what the wiki should understand. The default is a plain "code wiki for this repository."

You can keep it or rewrite it to point the agent at what matters, such as the endpoints and how to run the app.

**Run it**

The last prompt offers Run OpenWiki now or Open chat.

Run now writes the initial openwiki/ directory, while Open chat skips generation and drops you into an interactive session.

## Keeping the Wiki Fresh

A wiki is only useful if it stays current. Running openwiki --update refreshes the existing documentation from repository changes. The project also ships an example GitHub Actions workflow that opens a pull request with documentation updates once a day. You copy it into .github/workflows/openwiki-update.yml in your repo.

## Things to Keep in Mind

**Your code goes to a model provider**

OpenWiki uses an LLM to write the docs, so repository content is sent to whichever provider you configure.

Check that this is allowed before running it on private or company code.

**Cost scales with repo size**

A small app like a string reverser is quick and cheap. A large codebase means many more model calls on your API key.

Pick a model with solid reasoning and agentic ability for better results.

**Your config stays local**

Provider settings and keys are saved on your own machine in ~/.openwiki/.env.

Review the generated files with git diff before committing them.

## Final Thoughts

OpenWiki is a small idea with a clear payoff: give your coding agent a maintained, readable map of your repo so it stops rediscovering the project every session. Try it first on a throwaway project, read the generated wiki critically, and then decide whether it earns a place in your real repositories.
