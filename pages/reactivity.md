---
layout: section
---

## Reactivity

---

### Publisher / Subscriber

- Pub/Sub แบบ manual (ไม่มี Proxy) ต้องเรียก notifyAll() เองทุกครั้งที่ค่าจริงเปลี่ยน 
- จุดอ่อน: ลืมเรียกได้ง่าย

```js
let listeners = [];
function subscribe(fn) { listeners.push(fn); }
function notifyAll() { listeners.forEach(fn => fn()); }

let price = 100;
subscribe(() => console.log('ราคาเปลี่ยนเป็น:', price));

price = 200;
notifyAll(); // ต้องเรียกเอง
```

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/pub_sub.html"
/>

---

### Proxy

- Proxy ครอบ object ไว้ แล้วดักจับทุกครั้งที่มีการอ่าน (get) หรือเขียน (set) ค่า

```js
const state = new Proxy({ price: 100 }, {
  get(obj, key) {
    // ทุกครั้งที่มีคน "อ่าน" state.price จะโดนดักตรงนี้ก่อนเสมอ
    console.log('มีคนอ่าน:', key);
    return obj[key];
  },
  set(obj, key, value) {
    // ทุกครั้งที่มีคน "เขียน" state.price = ... จะโดนดักตรงนี้ก่อนเสมอ
    console.log('มีคนเขียน:', key, '=', value);
    obj[key] = value;
    return true;
  }
});
```

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/proxy.html"
/>

---

### Proxy notify all updates

- set trap เรียก notifyAll() ให้เองทุกครั้ง ไม่ต้องเรียกเองอีกต่อไป

```js
const state = new Proxy({ price: 100 }, {
  set(obj, key, value) {
    obj[key] = value;
    notifyAll(); // Proxy เรียกให้เองอัตโนมัติ
    return true;
  }
});
```

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/proxy2.html"
/>

---

### Proxy notify by Key

- แยก listener ตาม property (เหมือน EventEmitter)
- ปัญหาของขั้นก่อน: แจ้งทุกคนเสมอไม่สนว่าใครสนใจ key ไหน — ขั้นนี้แยกให้ตรงเป้า

```js
let listenersByKey = {};
function subscribeTo(key, fn) {
  (listenersByKey[key] ??= []).push(fn);
}
function notifyKey(key) {
  (listenersByKey[key] || []).forEach(fn => fn());
}
```

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/proxy3.html"
/>

---

## Track/Trigger

- กระบวนการ Track/Trigger มีความคล้ายคลึงกับ pub/sub
- ใช้ (Proxy's get/set trap) + (pub/sub แบบแยกตาม key)
- ไม่ใช้ Object literal ({}) เก็บ key ที่เป็น object เพราะจะถูกแปลงเป็น string
  โดยอัตโนมัติ ("[object Object]") ทำให้ object คนละตัวชนกันเป็น key เดียวกันได้
- ใช้โครงสร้าง 3 ชั้นแทน:
  - WeakMap  → เก็บ target(object) เป็น key (แก้ปัญหาข้างบน + กัน memory leak
               เพราะ WeakMap ไม่ถือ object เป็น strong reference)
  - Map      → เก็บ property key ของแต่ละ target (ชั้นนี้ key เป็น string ปกติ
               ไม่มีปัญหาเรื่อง object-as-key จึงใช้ Map ธรรมดาได้)
  - Set      → เก็บ effect ที่ผูกกับ property นั้น (ใช้ Set กันไม่ให้ effect
               ตัวเดียวกันถูกเก็บซ้ำ ถ้า track() ถูกเรียกซ้ำจาก effect เดิม)
---

### การใช้ Map เก็บ state

- จะเกิด bug ได้ในกรณีที่เก็บ Property ของแต่ละ state เพียงอย่างเดียว

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/track_trigger.html"
/>

---

### การใช้ WeakMap เก็บ state

- จะแก้ปัญหาได้เพราะจะใช้ WeakMap อีกตัวมาเก็บ Object แทน Key
- หลังจากนั้นก็จะมาเก็บ Property ของแต่ละ State ไว้ใน Map
- และใน Map ก็จะเก็บ Handler ไว้ใน Set

```js
const targetMap = new WeakMap(); // object → Map(key → Set)

function track(target, key) {
  if (!activeEffect) return;
  let depsMap = targetMap.get(target);
  if (!depsMap) targetMap.set(target, depsMap = new Map());
  let dep = depsMap.get(key);
  if (!dep) depsMap.set(key, dep = new Set());
  dep.add(activeEffect);
}

function trigger(target, key) {
  const depsMap = targetMap.get(target);
  const dep = depsMap?.get(key);
  dep?.forEach(fn => fn());
}
```

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/track_trigger2.html"
/>

---

## Computed/Watch

- computed() — ค่าที่คำนวณจาก reactive state อื่น แล้ว cache ผลลัพธ์ไว้ รู้เองว่าเมื่อไหร่ต้องคำนวณใหม่ เพราะข้างในใช้ effect() ติดตาม dependency

- watch() — เหมือน effect() แต่ให้ค่าที่ track มาเทียบ "ก่อน-หลัง" ได้ด้วย


```js
  function computed(getter) {
    const result = reactive({ value: undefined });
    effect(() => {
      result.value = getter(); // ทุกครั้งที่ dependency เปลี่ยน getter จะถูกเรียกใหม่อัตโนมัติ
    });
    return result;
  }

  function watch(source, callback) {
    let oldValue;
    effect(() => {
      const newValue = source();
      if (oldValue !== undefined && newValue !== oldValue) {
        callback(newValue, oldValue);
      }
      oldValue = newValue;
    });
  }

```
---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/computed_watch.html"
/>

---

### Signal

- Signal implement ด้วย closure ไม่ใช่ Proxy
  (closure = ฟังก์ชันที่ "จำ" ตัวแปรจาก scope ภายนอกได้ แม้ฟังก์ชันที่สร้างมันรันจบไปแล้ว)
- createSignal(value) เก็บ "value" ไว้เป็นตัวแปรธรรมดาผ่าน closure
  (ไม่ใช่ property ของ object แบบที่ reactive() ทำ)
- คืนค่าเป็นคู่ [read, write] — ต้องเรียกเป็นฟังก์ชันเสมอ
  (ต่างจาก reactive() ที่เข้าถึงแบบ property ปกติ เช่น state.count)
- แต่ละ signal มี Set ของตัวเองเก็บ subscriber (effect) ที่ track ไว้
  - read() → ถ้ามี activeEffect อยู่ ให้เพิ่มเข้า Set (เหมือน track ใน Proxy-based)
  - write() → เปลี่ยนค่า แล้ววน Set เรียกทุก effect ที่ subscribe ไว้ (เหมือน trigger)

```js
function createSignal(initialValue) {
  let value = initialValue; // ตัวแปรธรรมดา ไม่ใช่ Object
  // ฟังก์ชันข้างล่างนี้ "จำ" ตัวแปร value ได้ แม้ createSignal จะทำงานจบไปแล้ว
  const read = () => value;
  const write = (v) => { value = v; };
  return [read, write];
}
```

---

### สรุปความแตกต่างระหว่าง Proxy vs. Signal

| | Proxy-based (`reactive()`) | Signal-based (`createSignal()`) |
|---|---|---|
| กลไกเก็บค่า | Proxy ครอบ object ทั้งก้อน | closure เก็บตัวแปรเดี่ยว |
| อ่าน/เขียน | property ปกติ (`state.count`) | เรียกฟังก์ชัน (`count()` / `setCount()`) |
| Set ใช้ทำอะไร | เก็บ effect ต่อ 1 property (อยู่ใน `Map` ใน `WeakMap`) | เก็บ effect ต่อ 1 signal โดยตรง (ไม่มีชั้นซ้อนแบบ `WeakMap`) |

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/closure.html"
/>

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="reactivity/closure2.html"
/>
