---
layout: post
title: I Replaced My AI Agent's Permission System with Duo Push
categories: Dispatches
excerpt_separator: <!--more-->
---

Every time Claude Code (Anthropic's AI coding agent) wants to run a command, write a file, or call an API, it asks for permission. A little prompt pops up in the terminal: approve, approve for session, or deny. It's functional. It's also the least interesting way to solve the problem.

Tonight I replaced it with Duo Push.

<!--more-->

Now when my AI agent wants to do something (run a bash command, edit a file, send a Webex message) my phone buzzes. The push notification tells me exactly what the agent is trying to do. I tap approve from Duo Mobile, and the agent proceeds. I tap deny, and it's blocked. Same Duo Push that protects your VPN, your cloud apps, your admin panels. Now it's protecting an AI agent's actions.

<video controls playsinline style="width:100%; max-width:720px; border-radius:8px; margin:1.5em 0;">
  <source src="/files/claude-duo-mobile-push.mp4" type="video/mp4">
</video>

Here's how I built it, and why you should try it yourself.

## The Setup

The ingredients:

- **[duo-cli](https://github.com/cmedfisch/duo-cli-python)**: a Python CLI for Duo Security that lets you trigger Duo Push, TOTP, and Universal Prompt flows from the command line. Created by [Colin Medfisch](https://www.linkedin.com/in/cmedfisch/), a Duo engineer who built it specifically for programmatic Duo integrations like this.
- **Claude Code hooks**: a system that lets you run shell commands before or after any tool call. A `PreToolUse` hook runs before the agent acts and can return `allow` or `deny`.
- **A flag file**: because sometimes you just want the normal terminal prompt, and sometimes you want Duo.

The whole thing took about 30 minutes to wire up.

## The Hook

Claude Code's `PreToolUse` hook receives a JSON payload on stdin with the tool name and input. The hook script does three things:

1. Checks if Duo mode is active (a flag file at `/tmp/duo-approvals-active`)
2. Extracts context from the tool call to build a human-readable reason
3. Sends a Duo Push and returns the result

```bash
# Send the Duo Push with contextual reason
RESULT=$(duo-cli auth push canderson \
    --reason "$REASON" \
    --wait 2>&1)
```

The `--reason` flag is the key. Instead of a generic "Claude wants to do something," the push notification shows you what's actually happening:

| Tool Call | What You See on Your Phone |
|-----------|---------------------------|
| `git push origin main` | **Bash:** git push origin main |
| Edit a config file | **Edit:** /path/to/config.json |
| Send a Webex message | **webex_send_message** (roomId=abc123) |
| Fetch a web page | **WebFetch:** https://example.com |

You know exactly what you're approving before you tap.

Read-only operations like file reads and searches skip the push entirely. No point buzzing your phone for a `grep`.

## The Toggle

Not every session needs Duo Push. Some days I'm doing quick exploratory work and the terminal prompt is fine. So the whole thing is opt-in:

I start a session and say **"Let's use Duo for approvals today."** Claude creates the flag file, and every tool call flows through Duo Push for the rest of the session.

If I don't say anything, the normal terminal approval works as usual. A reboot clears the flag. Simple.

## The Demo Moment

The real test wasn't writing a file or running a command. It was this:

I asked Claude to find a specific thread in our team's Webex channel, one where a colleague had just replied, and post a message to it. That meant:

1. **Search for the Webex room**: read-only, no push needed
2. **List messages to find the thread**: read-only, no push
3. **Send a message to the thread**: Duo Push

My phone buzzed. The notification showed `webex_send_message` with the room ID. I tapped approve. The message appeared in the thread.

An AI agent, sending a message on my behalf, gated by Duo Push on my phone. The agent couldn't act until I, verified through my device with biometric unlock, said yes.

## Try It Yourself

Everything you need is open source:

1. **Install duo-cli**: `pip install git+https://github.com/cmedfisch/duo-cli-python.git`
2. **Create a Duo Auth API integration** in your Duo Admin Panel (Applications > Protect an Application > Auth API)
3. **Configure credentials**: `duo-cli configure --api auth`
4. **Write a hook script** that calls `duo-cli auth push <username> --reason "<context>" --wait` and returns JSON with the permission decision
5. **Register the hook** in your Claude Code settings as a `PreToolUse` command hook

The [duo-cli repo](https://github.com/cmedfisch/duo-cli-python), created by [Colin Medfisch](https://www.linkedin.com/in/cmedfisch/), has the full documentation. The tool supports both Auth API (direct push, simple allow/deny) and Universal Prompt (browser-based OIDC flow with full Duo policy enforcement). Pick whichever fits your use case.

## What's Next

This was a Monday night build, not a production deployment. But it works, and it surfaces something interesting: the same identity infrastructure we've spent years building to protect human access works just as well for AI agent access. Duo Push doesn't care if the action is "log into Salesforce" or "let an AI agent send a Webex message." It's the same flow: a request, context about what's being requested, and a human making the call.

If you build something with duo-cli, I want to hear about it. Drop a note in the [repo](https://github.com/cmedfisch/duo-cli-python) or find me on [LinkedIn](https://www.linkedin.com/in/intheclouds/).

---

**Chris Anderson** is a Principal Product Manager at Cisco Duo, where he focuses on identity security, access management, and making sure AI agents ask before they act. He writes at [chrisanderson.cloud](https://chrisanderson.cloud).
