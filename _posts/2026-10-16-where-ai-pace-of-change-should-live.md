---
layout: post
title: "Where AI's Pace of Change Should Live"
description: "Friday writing: where in the architecture AI's fast-moving change should land — an AI gateway seam, and a platform that owns LLM reliability."
date: 2026-10-16 09:00:00 +0800
tags: [Architecture, AI platform, Friday writing]
permalink: /ai-pace-of-change/
---

*Sources: InfoQ, "An Evolutionary Architecture Pattern for Managing AI's Pace of Change" (Price, Đurek, Migkiros, Dearham, Jul 2026) and "The Platform Engineering Playbook for Production LLMs" (Aditya Mulik, Oct 2026). Figures below are the authors' claims, not ours.*

## The problem

Models, agent standards and attack techniques will keep changing faster than our release cycles. We cannot slow that down. What we can decide is where in the architecture the change is allowed to land, so the rest of the system stays boring and stable.

Two recent InfoQ articles answer that from different ends. One looks at the seam between our systems and the AI. The other looks at the contract every AI feature should sign with a shared platform. Read together, they give us a test for our own work.

## Idea 1: put the fast-moving parts behind one seam

The first article argues for an AI gateway as the single place where guardrails, model routing, agent identity, action policy and audit live. The reasoning is that agent failures are not schema failures. A well-formed, authenticated request can still be a bad decision, and a token issued before a prompt-injection hijack still validates. So the check has to happen per action, not per login.

Three details worth borrowing:

- Policy is declared as version-controlled config, reviewed in a pull request, and deployed separately from the apps. Security owns the rules, a platform team owns the gateway.
- Observability and audit are different things. Observability catches drift and cost spikes. Audit answers to regulators, so log Request, Decision, Action, and consider storing an input hash instead of the raw prompt.
- The cheap time to build this is before the incident. The authors describe four stages, from one team on one provider to every team making its own choices, then a forcing event (incident, regulator question, cost surprise), then a rushed retrofit.

The authors are also honest about limits: added latency, a central team that can become known for blocking work, and guardrails that are probabilistic, so false positives hurt usability. With only a few teams and models, in-app guardrails may be enough.

## Idea 2: the platform owns reliability, not the app team

The second article treats the LLM stack as shared infrastructure. Its central claim is that hallucination rate is something a platform can control. The author reports a drop from about 15% to 1.5% over six months without changing the foundation model, and credits the validation and retry layer for most of it.

The mechanics are simple and worth copying:

- Every call declares an output schema. A failure is re-prompted with the validation error appended, then counted.
- Failures are classified before they are retried: schema violation, hallucination signal, or infrastructure error. Each class gets a different response, and infrastructure errors back off.
- Retries have a hard cap per request. The author uses three, on the reasoning that a structured-output failure that has not fixed itself by the third attempt rarely fixes itself on the fourth.
- Prompts are versioned with a rollback path that needs no redeploy.
- Tool authorization is default-deny and enforced at each tool server, not only at the gateway, so one orchestrator bug does not expose every tool.
- Token spend is attributed by team at the point of ingress. Retrofitting that later is described as weeks of work.

## What this means for us

Our pre-deploy config validator harness already follows this pattern, and these articles are useful outside evidence for why. Validators never call an LLM, so a model cannot grade its own homework. Errors are split into RECOVERABLE and HARD_BLOCKER, which is the same move as classifying failures before retrying. No-progress detection and a hard budget with a kill switch are the same instinct as the capped retry count. The same logic sits under our AI code review standard: AI-generated code is the author's code, and the check that matters is the one that does not depend on the model's confidence.

The articles also show where we have not gone yet. I would like us to answer these as a team:

1. Where would a gateway-style seam sit for us today, and who would own it?
2. Do we attribute AI and Bedrock spend by team and use case, or will we find out from a single invoice line?
3. Are our prompts versioned with a rollback path, the way we treat config?
4. Do our tools enforce their own authorization, or do they trust the caller?

## One caution

Both articles come from authors describing their own designs. The numbers are self-reported, and the second author's companion repo is a demonstration, not a production reference. Treat them as a checklist of questions, not as proof that this architecture is right for us.

## Next step

Proposal for the week: pick one of the four questions above, spend an hour as a team mapping our current answer, and write down the gap. No new tooling until we know the gap.
