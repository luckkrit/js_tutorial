---
layout: section
---

## Basic Timeout and Interval

---

### Timeout

- Set timeout

```js {monaco-run} {autorun:false}
setTimeout(() => console.log('ทำงานหลัง 2 วิ'), 2000);
```

- Clear timeout

```js {monaco-run} {autorun:false}
let timerId = setTimeout(() => console.log('ทำงานหลัง 2 วิ'), 2000);
for(let i=0;i<10;i++){
  console.log(i)
}
clearTimeout(timerId);
```

---


<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="timer_interval/timeout.html"
/>

--- 

### Interval

- Set interval

```js {monaco-run} {autorun:false}
setInterval(() => console.log('ทำงานทุก 1 วิ'), 1000);
```

- Clear interval

```js {monaco-run} {autorun:false}
let timerId = setInterval(() => console.log('ทำงานทุก 1 วิ'), 1000);
for(let i=0;i<10;i++){
  console.log(i)
}
clearTimeout(timerId);
```
---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="timer_interval/interval.html"
/>

--- 

### Event Emitter

```js
class EventEmitter {
  constructor() {
    this.events = {}; // { eventName: [callback1, callback2, ...] }
  }

  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
  }

  off(eventName, callback) {
    if (!this.events[eventName]) return;
    this.events[eventName] = this.events[eventName].filter(fn => fn !== callback);
  }

  emit(eventName, data) {
    if (!this.events[eventName]) return;
    this.events[eventName].forEach(fn => fn(data));
  }
}
```

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="timer_interval/countdowntimer.html"
/>