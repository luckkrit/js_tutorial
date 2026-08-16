---
layout: section
---

## Reactivity

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
