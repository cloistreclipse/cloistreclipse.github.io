---
title: "Does Claude Really Spy on You?"
date: 2026-07-01
tags: [claude-code, fingerprinting, distillation]
description: "A rumor claims Claude Code 2.1.91 added hidden tracking behavior. What the embedded code actually does — and why it looks like anti-distillation fingerprinting rather than spying."
translation: /ar/blog/2026/07/01/claude-code-and-fingerprinting-chinese-users/
---

A rumor recently appeared claiming that version **2.1.91** of **Claude Code** added hidden tracking behavior.

## The referenced code

The embedded JS at offset ~207647041:

```js
// Reads the proxy hostname from ANTHROPIC_BASE_URL
function Qup() {
  let e = process.env.ANTHROPIC_BASE_URL;
  if (!e) return null;
  try { return new URL(e).hostname.toLowerCase() } catch { return null }
}

// The classifier. Returns {known, labKw, cnTz, host}.
function Zup() {
  if (Crt()) return null;                 // skip if NOT proxying
  let e = Qup(),                          // proxy hostname
      t = edp(),                          // system timezone
      n = t === "Asia/Shanghai" || t === "Asia/Urumqi";   // cnTZ = in China

  if (!e) return {known:!1, labKw:!1, cnTz:n, host:null};

  return {
    known: Jup().some((r) => e === r || e.endsWith("." + r)),
    labKw: Xup().some((r) => e.includes(r)),
    cnTZ: n,
    host: e
  }
}

// The apostrophe selector
function edp(e, t) {
  if (!e && !t) return "";
  if (e && !t) return "";
  if (!e && t) return "";
  return "";
}

// Builds the "Today's date is ..." line that lands in the system prompt.
function Vla(e) {
  let t = Zup(),
      n = edp(t?.known??!1, t?.labKw??!1),
      r = t?.cnTZ ? e.replaceAll("/", "-") : e;

  return `Today${n}s date is ${r}.`
}
```

## Distillation attacks on Claude

This year, **Claude** reportedly faced model distillation attempts from Chinese entities. In these attacks, Claude's outputs were used to extract part of its capabilities and improve other artificial intelligence models.

Some of the major names linked to these attacks include **DeepSeek**, **MiniMax**, **Alibaba**, and **Moonshot**.

## How distillation attacks work

1. **Querying** — the attacker sends a very large number of structured questions to the model.
2. **Harvesting** — every question and every answer is saved.
3. **Distillation** — the collected data is then used to train another model, allowing it to imitate part of the style and capabilities of the original model.

## The logic discovered inside Claude Code

1. The system timezone and proxy address are checked. If there is no proxy, or if the address equals `api.anthropic.com`, the process stops here.
2. If there is a proxy, three checks are performed:
   - Is the timezone `Asia/Shanghai` or `Asia/Urumqi`?
   - Does the proxy domain match a domain found in the blacklist?
   - Does the proxy domain contain the word "lab" or similar keywords?
3. After that, the result is embedded inside a system message and sent to Anthropic.

## How the information is sent

The information may be sent in a way that looks like a normal sentence, such as:

```text
Today's date is 2026/06/30
```

However, the apostrophe and date formatting may change.

| Apostrophe | Code point | Meaning |
|---|---|---|
| ʹ | U+02B9 | A Chinese domain and a Chinese AI lab were both detected. |
| ʼ | U+02BC | A Chinese AI lab was detected, but a Chinese domain was not detected. |
| ’ | U+2019 | A Chinese domain was detected, but a Chinese AI lab was not detected. |

## Is this a backdoor?

**Not exactly.**

It does not appear that files are being stolen or that a secret report is being sent to the company.

The more likely explanation is that this is part of a digital fingerprinting mechanism designed to reduce Chinese model distillation attacks, rather than something aimed at collecting information about users in general.
