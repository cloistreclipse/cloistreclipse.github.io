---
title: "هل Claude فعلاً يتجسس عليك؟"
date: 2026-07-01
lang: ar
tags: [claude-code, fingerprinting, distillation]
description: "إشاعة تقول إن الإصدار 2.1.91 من Claude Code أضاف سلوك تتبع مخفياً. ماذا يفعل الكود المضمّن فعلياً — ولماذا يبدو بصمة رقمية مضادة للتقطير لا تجسساً."
permalink: /ar/blog/2026/07/01/claude-code-and-fingerprinting-chinese-users/
translation: /blog/2026/07/01/claude-code-and-fingerprinting-chinese-users/
---

ظهرت إشاعة مؤخراً تقول إن الإصدار **2.1.91** من **Claude Code** أضاف سلوكاً مخفياً للتتبع.

## الكود الذي تمت الإشارة إليه

الكود المضمّن (JS) عند الإزاحة ~207647041:

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

## هجمات التقطير على Claude

شهدت **Claude** هذا العام محاولات تقطير نماذج من جهات صينية، حيث تم استخدام مخرجات **Claude** لاستخلاص جزء من قدراته وتحسين نماذج ذكاء اصطناعي أخرى.

ومن أبرز الأسماء المرتبطة بهذه الهجمات: **DeepSeek** و **MiniMax** و **Alibaba** و **Moonshot**.

## كيفية عمل هجمات التقطير

1. **الاستعلام (Querying)** — يرسل المهاجم عدداً ضخماً من الأسئلة المنظمة إلى النموذج.
2. **الحصاد (Harvesting)** — يتم حفظ كل سؤال وكل إجابة للسؤال.
3. **التقطير (Distillation)** — يتم استخدام هذه البيانات لتدريب نموذج آخر، مما يسمح له بتقليد جزء من أسلوب وقدرات النموذج الأصلي.

## المنطق المكتشف داخل Claude Code

1. يتم فحص المنطقة الزمنية للنظام وعنوان البروكسي. إذا لم يكن هناك بروكسي، أو كان العنوان يساوي `api.anthropic.com`، يتوقف هنا.
2. إذا كان هناك بروكسي، يتم تنفيذ ثلاثة فحوصات:
   - هل المنطقة الزمنية `Asia/Shanghai` أو `Asia/Urumqi`؟
   - هل نطاق البروكسي يطابق نطاقاً موجوداً في القائمة السوداء؟
   - هل نطاق البروكسي يحتوي على كلمة lab أو كلمات مفتاحية مشابهة؟
3. بعد ذلك يتم تضمين النتيجة داخل رسالة نظام وترسل إلى Anthropic.

## طريقة إرسال المعلومات

تُرسل المعلومات بطريقة قد تبدو كجملة عادية مثل:

```text
Today's date is 2026/06/30
```

لكن علامة الاقتباس وتنسيق التاريخ قد يتغيران.

| العلامة | الترميز | المعنى |
|---|---|---|
| ʹ | U+02B9 | تم اكتشاف نطاق صيني ومختبر ذكاء اصطناعي صيني معاً. |
| ʼ | U+02BC | تم اكتشاف مختبر ذكاء اصطناعي صيني، لكن لم يتم اكتشاف نطاق صيني. |
| ’ | U+2019 | تم اكتشاف نطاق صيني، لكن لم يتم اكتشاف مختبر ذكاء اصطناعي صيني. |

## هل هذا يعتبر باباً خلفياً؟

**ليس بالضبط.**

لا يبدو أنه يتم سرقة ملفات أو إرسال تقرير سري للشركة.

من المرجح أنه جزء من آلية بصمة رقمية للتخفيف من هجمات التقطير الصينية، وليس موجهاً لجمع معلومات عن المستخدمين بشكل عام.
