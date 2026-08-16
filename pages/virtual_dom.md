---
layout: section
---

## Virtual Dom

---

### What is Virtual Dom

- Virtual DOM ไม่ใช่ DOM จริง เป็นแค่ JS object ที่บรรยายว่า DOM ควรมีหน้าตายังไง

```js
// นี่คือ Virtual DOM ของ <ul><li>A</li><li>B</li></ul>
const vdom = {
  tag: 'ul',
  children: [
    { tag: 'li', children: ['A'] },
    { tag: 'li', children: ['B'] },
  ],
};

```

- สร้าง object แบบนี้เร็วมาก (แค่สร้าง JS object ธรรมดา) ต่างจากสร้าง DOM element จริงที่ browser ต้องทำงานหนักกว่า (layout, paint, ฯลฯ)

---

### Diffing

```js
const oldVdom = {
  tag: 'ul',
  children: [
    { tag: 'li', children: ['A'] },
    { tag: 'li', children: ['B'] },
  ],
};

const newVdom = {
  tag: 'ul',
  children: [
    { tag: 'li', children: ['A'] },
    { tag: 'li', children: ['C'] }, // เปลี่ยนแค่ตัวนี้
  ],
};
```

- Algorithm diff จะไล่เทียบ 2 object นี้ทีละ node แล้วสรุปว่า "เปลี่ยนแค่ text ของ li ตัวที่ 2 จาก B เป็น C" จากนั้นค่อยไปแก้ DOM จริงแค่จุดเดียว ไม่แตะ `<li>A</li>` เลย

---

### 3 Parts of Virtual Dom

1. `h()` — สร้าง virtual node (แค่ JS object ธรรมดา `{ tag, props, children }`) ไม่ใช่ DOM จริง สร้างเร็วมากเพราะเป็นแค่ object literal

2. `render()` — แปลง virtual node เป็น DOM จริง ใช้แค่ตอนแรกครั้งเดียว (initial mount)

3. `patch()` — หัวใจจริงๆ ของ Virtual DOM ไล่เทียบ virtual node เก่ากับใหม่ ทีละ node แบบ recursive แล้วแก้ DOM จริงเฉพาะจุดที่ต่างเท่านั้น:

```js

if (oldVnode.tag !== newVnode.tag) {
  // tag ไม่เหมือนกันเลย → สร้างใหม่ทั้งก้อน (เคสแย่สุด)
} else {
  // tag เดิม → เทียบ props/children ทีละตัว แก้เฉพาะที่ต่าง
}
```

---

### Key-Based Index

- ตัวอย่างต่อไปนี้จะแสดงให้เห็นว่าการใช้ Key-Based Index จะมีข้อเสียยังไง
- โดยจะอธิบายการทำงานของ `h()` กับ `render()` ในกรณีอัพเดต Object ใน array และเพิ่ม Object ใน array

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="virtual_dom/virtual_dom.html"
/>

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="virtual_dom/virtual_dom2.html"
/>

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="virtual_dom/virtual_dom3.html"
/>

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="virtual_dom/virtual_dom4.html"
/>