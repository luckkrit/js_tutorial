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
