# Embedded Mastery: Build Guide

This guide is for adding a new page or rewriting an existing one **without re-reading the whole
codebase**. It covers every convention, rule and check the current 41 pages follow.

> **Quick start for an AI assistant:** read this file, then read **one** recent page as a style
> reference (`15-rtos-internals.html` or `05-memory-and-startup.html`). Use the "Prompt template"
> section at the bottom. You do not need to open any other page.

---

## 1. What this project is

A set of **self-contained, single-file HTML study dashboards** for embedded-systems interviews
at top product, semiconductor and automotive companies. The target is someone with around two
years' experience. Each page teaches one topic from beginner to advanced and includes simulators,
debugging scenarios, an interview Q&A bank and a cheat sheet.

- `index.html` is the hub. It links every page, grouped by tier, with a search box and a
  "mastered" tick per topic, saved in `localStorage`.
- Each page is one file, `NN-slug.html`, using vanilla HTML, CSS and JS only. It loads no CDNs,
  fonts or images. Pages are 250–550 KB each.
- The default reference part is the **STM32F407 (Cortex-M4)**. Pages name another part
  (STM32L4/U5, H7, MP157, nRF52840, etc.) whenever a topic needs it.

---

## 2. File and numbering rules

- Name files `NN-slug.html`, with a two-digit number and a lowercase hyphenated slug, for example
  `15-rtos-internals.html`.
- **Numbers follow the tier order**, Tier 1 first. If you add a page in the middle, renumber
  everything after it (see §8).
- Put the same number in the page's **kicker**, formatted `// NN — TOPIC NAME` (an em dash, upper
  case), and repeat it on every tab's header, e.g. `// 15 — RTOS INTERNALS · CHEAT SHEET`.
- Cross-references to other pages are written as plain text, "see page 17", and must use the
  current number.
- Every page links back to `index.html` with an `← All topics` link at the top of the sidebar. The
  chip logo also links there.

### Current map (tier → numbers)

| Tier | Numbers | Pages |
|---|---|---|
| 1 Foundations | 01–04 | C for embedded · Bit manipulation · Number representation · Digital basics & GPIO |
| 2 Core MCU | 05–09 | Memory & startup · Interrupts · Timers/clocks/watchdog · DMA · ADC/DAC |
| 3 Communication | 10–14 | UART/SPI/I²C · CAN/CAN-FD · USB · Wireless & IoT · Automotive networks |
| 4 Systems & RTOS | 15–21 | RTOS internals · Concurrency & atomics · Bootloader & FOTA · Low power · Storage & filesystems · Multicore · Zephyr |
| 5 Advanced | 22–30 | CPU architecture · ARM architecture · Embedded Linux · Debugging & faults · Functional safety · HW–SW interface · Security · **29 C++ (not built yet)** · OS fundamentals |
| 6 Software craft | 31–36 | DS & algorithms · Firmware architecture · Compiler & toolchain · Testing & CI · **35 Python (not built yet)** · Performance & power profiling |
| 7 Domain | 37–40 | DSP · Control & motor control · Sensors & DAQ · BMS firmware |
| 8 Interview rounds | 41–43 | C coding round · System design round · Behavioural & project deep-dive |

---

## 3. Look and feel (copy, don't reinvent)

Copy the `<style>` block from an existing page and adapt it. The palette tokens on `:root` are:

```css
--bg:#0a0f1d; --bg2:#0d1426; --panel:#0f1729; --panel2:#131d35; --line:#1f2c4a; --line2:#2a3a60;
--text:#d8e2f6; --muted:#8392b6; --dim:#56648a;
--cyan:#00f0ff;   /* flash / code memory, primary accent */
--amber:#ffaa00;  /* RAM / warnings */
--green:#00ff66;  /* execution flow / "correct" / active */
--violet:#b48cff; /* peripherals / system */
--red:#ff4d6d;    /* faults / wrong answers */
--mono: ui-monospace,"JetBrains Mono","Cascadia Code",Consolas,monospace;
--sans: Inter,system-ui,-apple-system,"Segoe UI",Roboto,Arial,sans-serif;
```

The theme is a dark "cyberpunk terminal" look over a faint 32 px grid background with a scanline
overlay. Reuse these classes as they are: `.panel`, `.ptitle`, `.tag` (`t-flash`/`t-ram`/
`t-mmio`/`t-exec`/`t-red`), `.btn` / `.btn.primary` / `.btn.sm`, `pre.code`, `.note`.

### Layout skeleton

```html
<div class="app">                      <!-- grid: 268px sidebar + main -->
  <aside class="sidebar">
    <a href="index.html" class="back">← All topics</a>
    <div class="brand">…kicker / title…</div>
    <nav class="tabs" role="tablist">
      <button class="tab active" data-tab="learn" role="tab">…</button>
      <button class="tab" data-tab="theory" role="tab">…</button>
      …
    </nav>
    <div class="side-stats">…progress bars…</div>
  </aside>
  <main>
    <section class="view active" id="view-learn">…</section>
    <section class="view" id="view-theory">…</section>
    …
  </main>
</div>
```

- Tab switching toggles `.active` on `.tab[data-tab=X]` and `#view-X`. Also listen for
  `hashchange` so links like `#scenarios` open the right tab.
- **Tabs must use class `tab` or `role="tab"`**, because `check.py` finds them that way.
- On phones (≤860 px) the sidebar becomes a horizontal, scrollable top bar.

---

## 4. Required tabs (in this order)

1. **Learn: Big Picture** (the default tab). A guided beginner tutorial read top to bottom:
   - what problem the topic solves;
   - the hardware or physical picture, as an SVG diagram;
   - key terms defined one by one;
   - **one end-to-end worked example with real numbers** and a diagram;
   - how the pieces connect;
   - a "Say it like this" summary.

   Give each section an "✓ I understand this" button that feeds the progress widget.
2. **Concepts.** 20–35 collapsible cards tagged `BEGINNER` / `INTERMEDIATE` / `ADVANCED`, with a
   level filter and expand/collapse-all. Each card follows the teaching structure in §5.
3. **Simulator(s) or lab(s).** The centrepiece. It must be a real, correct simulation of the
   mechanism, not decoration. It needs:
   - step, animate and reset controls with live readouts;
   - a "What just happened" narration panel that explains each step in 2–4 plain sentences;
   - a "How to use this lab" box with 3–5 guided experiments ("Set X to Y, press Step, notice Z…");
   - presets.

   One or two extra tools (calculators, decoders, configurators) are welcome.
4. **Scenarios.** 8–12 debugging or design scenarios. Each has a field report, an evidence block
   (register dump, trace, code or log), a question and 4 options, each option with a full-paragraph
   explanation. After that come ROOT CAUSE (a numbered mechanism), FIX (numbered steps) and
   FOLLOW-UP. **Only the first pick is scored**, and the correct option's position must vary across
   scenarios.
5. **Interview Q&A.** 40 or more questions with difficulty filters, a "whiteboard/coding" filter
   and search. Tag each as big-tech or automotive/semiconductor where it helps. Every answer has
   four parts: **reasoning → worked example → pitfall → "Say it like this"**. Whiteboard questions
   include full, compilable C.
6. **Cheat Sheet.** Scannable tables, including a **"Formulas explained"** table (formula | what
   each symbol means | worked example), X-vs-Y comparisons, key numbers, a "never do this" list and
   a 60-second revision summary.

The **progress widget** in the sidebar tracks per-tab achievements: Learn sections read, concepts
opened, lab experiments done, scenarios cracked and Q&A revealed. If you use `localStorage`, wrap
it in `try/catch`.

---

## 5. Teaching depth (the most important rule)

The reader's feedback that drove the rewrite was: "the content seems a little abstract… I need it
in depth and in detail, because I was not able to understand." Terse expert bullet points are not
acceptable. Write like a patient senior engineer teaching a junior one-on-one.

**Every concept card uses these numbered sub-headings:**

1. **What is it?** Start with one plain-language sentence on what it is and why it exists. Define
   every acronym the first time it appears (for example, "MSP = Main Stack Pointer, the register
   that…").
2. **How does it work, step by step?** A numbered sequence of what the hardware, compiler or
   software actually does, in order.
3. **Worked example.** Real numbers with the arithmetic shown, such as addresses, register values,
   bit patterns and timings. For example: 98 vectors × 4 B = 392 B, so round up to 512 and VTOR
   must be 0x200-aligned; 0x0800_8000 qualifies and 0x0800_8100 does not.
4. **Code, line by line.** Code followed by a `Code | Meaning` table. Show **wrong vs right** side
   by side wherever there is a pitfall.
5. **What goes wrong if…** The concrete failure, the **symptom you would observe**, and why it
   happens.
6. **Say it like this.** A 2–4 sentence model answer the reader can say out loud in an interview.

Add a small ASCII or SVG diagram wherever it helps: memory layouts, timelines, bit fields, state
machines.

### Reference example of the expected depth

The `.isr_vector` entry in `05-memory-and-startup.html` shows the target level. It covers:

- word 0/1/2 addresses (0x0800_0000/04/08) and their contents;
- why `Reset_Handler` shows up as an odd address (0x0800_0199: bit 0 = Thumb, the real instruction
  is at 0x0800_0198);
- weak-alias code next to a misspelled handler (`USART1_IRQhandler`), and what happens as a result;
- a `KEEP()` line-by-line table with the `--gc-sections` failure;
- the VTOR alignment maths with an aligned and an unaligned example;
- a closing "good interview answer" paragraph.

### Accuracy rules

- Every number must be either computed (and checked with a script where possible) or labelled as
  "typical datasheet value" / "illustrative".
- Say when a fact is vendor-specific and name the vendor and part.
- Keep security content **defensive**: explain threats conceptually and focus on prevention,
  detection and fixing. Do not include step-by-step attack procedures, exploit payloads or bypass
  recipes.

---

## 6. Technical pitfalls already hit (avoid them)

| Problem | Fix |
|---|---|
| Phone layout overflowing sideways because a wide `<pre>` or table stretches a grid column | Use `grid-template-columns: minmax(0,1fr)` (not `1fr`), set `min-width:0` on panels and grid children, and wrap wide tables and diagrams in `overflow-x:auto` |
| `let`/`const` TDZ error at startup ("Cannot access X before initialization") | Never call a function that touches a `let`/`const` before its declaration runs; guard with a ready flag |
| Duplicate element IDs (a new narration panel reused an input's ID, so nothing showed) | Prefix new IDs per feature |
| `alert()` / `confirm()` dialogs | Forbidden. Use an inline "click again to confirm" pattern instead |
| Labels "SCENARIO 010" | Zero-pad with `String(n).padStart(2,'0')` instead of hard-coding the "0" |
| Ciphertext/pixel visualisation hiding patterns | Colour per block, not per byte, and say so on the page |
| Several builders sharing one scratch folder and overwriting each other's parts | Each build uses its own scratch folder (`pNN/`) |
| Build folders left in the published folder | Keep `pNNr/` build folders out of the folder you publish |

---

## 7. Testing (mandatory before calling a page done)

Use `check.py` below (Python + Playwright with Chromium). It loads the page at **1400 px and 390
px**, clicks every tab and up to 150 visible buttons per tab, and reports JS errors,
`console.error` output, any `alert`/`confirm` dialog, and horizontal page overflow. Run it until
it prints `PASS`.

```
python3 check.py NN-slug.html
```

<details><summary>check.py (full source)</summary>

```python
"""Smoke-test a dashboard: JS errors, every tab, every button, overflow at 1400 and 390 px."""
import sys, os
from playwright.sync_api import sync_playwright

path = os.path.abspath(sys.argv[1])
problems = []
with sync_playwright() as p:
    b = p.chromium.launch()
    for w in (1400, 390):
        pg = b.new_page(viewport={"width": w, "height": 900})
        pg.on("pageerror", lambda e, w=w: problems.append(f"[{w}px] JS error: {e}"))
        pg.on("console", lambda m, w=w: problems.append(f"[{w}px] console.error: {m.text}") if m.type == "error" else None)
        pg.on("dialog", lambda d: (problems.append("dialog opened (alert/confirm) - not allowed"), d.dismiss()))
        pg.goto("file://" + path)
        pg.wait_for_timeout(300)
        tabs = pg.query_selector_all(".tab, [role=tab]")
        if len(tabs) < 4:
            problems.append(f"[{w}px] only {len(tabs)} tabs found (need >=4, class 'tab' or role=tab)")
        for ti in range(len(tabs)):
            tabs = pg.query_selector_all(".tab, [role=tab]")
            try:
                tabs[ti].click(timeout=2000)
            except Exception as e:
                problems.append(f"[{w}px] tab {ti} not clickable: {str(e)[:80]}")
                continue
            pg.wait_for_timeout(150)
            pg.evaluate("""() => {
              const vis = el => el.offsetParent !== null;
              const bs = [...document.querySelectorAll('button, [role=button]')].filter(vis)
                .filter(b => !b.classList.contains('tab') && b.getAttribute('role') !== 'tab');
              for (const b of bs.slice(0, 150)) { try { b.click(); } catch (e) {} }
            }""")
            pg.wait_for_timeout(200)
            ov = pg.evaluate("document.documentElement.scrollWidth - window.innerWidth")
            if ov > 2:
                problems.append(f"[{w}px] tab {ti}: horizontal page overflow {ov}px")
        pg.close()
    b.close()
print(f"size={os.path.getsize(path)//1024} KB")
if problems:
    print("FAIL")
    for x in dict.fromkeys(problems):
        print(" -", x)
else:
    print("PASS")
```
</details>

Also, by hand:
- take screenshots of the Learn tab, one expanded concept card and the main simulator, and look at
  them;
- script the simulator presets and compare the outputs with hand calculations;
- for coding answers, compile the C with `gcc -std=c11 -Wall -Wextra` and run it with test inputs.

---

## 8. Adding a page to the index

In `index.html`, find the `const T=[ … ]` array. Each tier is
`{n:"TIER X — NAME", c:"var(--colour)", d:"one-line description", items:[ … ]}`, and each item is:

```js
["NN","slug","Display Title","One-line summary of what the page covers."]
```

- Insert the item in its tier, in number order.
- `const SOON=["29","35"];` lists numbers shown greyed out as "coming later", with no link. Take a
  number out of this list once its page exists.
- If inserting the page shifts the numbering:
  1. renumber the files;
  2. update every kicker `// NN —`;
  3. update every "page NN" cross-reference and every `NN-slug.html` link inside all pages;
  4. update the index items.

  Do this with a script using an old → new mapping, never by hand. Then check that every index
  link and every in-page `href="NN-…html"` resolves, and re-run `check.py` on every page.

---

## 9. Prompt template for building or rewriting a page

> Read `BUILD-GUIDE.md` in this folder and follow it exactly. Use `15-rtos-internals.html` only as
> a style and structure reference.
> Build **`NN-slug.html`** with kicker **`// NN — TOPIC`** for the topic **<topic>**.
> Coverage, beginner → advanced: <list every sub-topic to include>.
> Centrepiece simulator: <describe the mechanism to simulate and its controls/outputs>.
> Second tool: <calculator / decoder / configurator>.
> Scenarios (≥8): <list realistic failures and design questions>.
> Q&A ≥40, including whiteboard C for: <list>.
> Apply §5 TEACHING DEPTH to every card, answer and narration. Run `check.py` until PASS,
> screenshot the Learn tab and the simulator, then add the page to `index.html` per §8.

For a **rewrite** of an existing page, keep every working feature, add the Learn tab, expand every
card to the §5 structure, add lab narration and how-to boxes, add 2 scenarios and 10 Q&A, and add
the "Formulas explained" table. The size should end up 1.5–2.5× the original.

---

## 10. Still to do

- **29 — C++ for Embedded**: RAII, vtables vs CRTP, templates/constexpr, what to avoid
  (exceptions, RTTI, heap), static-init order, ISRs in C++, ETL.
- **35 — Python for Test & Automation**: `struct`, pyserial, python-can/cantools, pytest HIL
  fixtures, log analysis, CRC matching with firmware.

Once built, remove them from `SOON` in `index.html`.
