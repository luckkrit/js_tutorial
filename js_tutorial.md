---
# try also 'default' to start simple
theme: seriph
base: /js_tutorial/
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Javascript Tutorial
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply UnoCSS classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true
---

# Javascript Tutorial

---

<Toc minDepth="1" maxDepth="3" columns="3" />

---
src: ./pages/forms.md
---

---
src: ./pages/events.md
---

---
src: ./pages/timer_interval.md
---

---
src: ./pages/asynchronous.md
---

---
src: ./pages/reactivity.md
---

---
src: ./pages/virtual_dom.md
---