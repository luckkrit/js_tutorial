---
layout: section
---

## Asyncrhonous

---

## Asynchronous

- Callback
- Promise
- Async/Await

---

## What is the result?

```js {monaco-run} {autorun:false}
console.log('A');
setTimeout(() => console.log('B'), 0);
console.log('C');
```

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="asynchronous/callback.html"
/>

---

## What is the result?

```js
getUser(id, (user) => {
  getPosts(user.id, (posts) => {
    getComments(posts[0].id, (comments) => {
      console.log(comments); // ซ้อนลึกเรื่อยๆ อ่านยาก
    });
  });
});
```

- This is an example of nested asynchronous (Callback Hell) which is hard to read
- Solve by `Promise`

---

## Promise

```js
getUser(id)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => console.log(comments))
  .catch(err => console.error(err));
```
---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="asynchronous/promise.html"
/>

---

## Async/Await syntax

```js
async function loadData() {
  try {
    const user = await getUser(id);
    const posts = await getPosts(user.id);
    const comments = await getComments(posts[0].id);
    console.log(comments);
  } catch (err) {
    console.error(err);
  }
}
```

---

<StackblitzEmbed
  project="web-static-html-4m8kxmfe"
  file="asynchronous/async_await.html"
/>

---

## Event Loop

- JavaScript solves this by delegating long-running tasks to the browser (or Node.js), which handles them in the background. Functions like setTimeout() don’t block:

```js {monaco-run}{autorun:false}
console.log('Start');

// This doesn't block! Browser handles the timer
setTimeout(() => {
  console.log('Timer done');
}, 2000);

console.log('End');
```

- The secret sauce that makes this work? The Event Loop.

---

## Javascript Runtime

![Javascript Runtime](/images/asynchronous/javascript-runtime.png){.w-1/2,.mx-auto}

- https://master.dev/tutorials/lydia-hallie/event-loop/

---

## Javascript Runtime

- Call Stack — จัดการ execution ทีละ context, task ที่รันนานจะบล็อก task อื่นหมด (single-threaded)
- Web APIs — ฟีเจอร์ที่ browser ให้มา (ไม่ใช่ core JS) เช่น fetch, setTimeout, Geolocation — ใช้ offload งานที่ใช้เวลานานออกจาก call stack
- Callback-based APIs (เช่น getCurrentPosition, setTimeout) — callback จะถูกส่งเข้า Task Queue เมื่องานเสร็จ ไม่ใช่รันทันที
- Promise-based APIs (เช่น fetch) — handler จะถูกส่งเข้า Microtask Queue ซึ่งมี priority สูงกว่า Task Queue
- Event Loop — เช็คว่า call stack ว่างหรือยัง ถ้าว่าง จะเคลียร์ microtask queue "จนหมด" ก่อนเสมอ แล้วค่อยหยิบ 1 task จาก task queue มารัน จากนั้น "เริ่มใหม่" คือเช็ค microtask queue อีกรอบ

---

## What is the result?

```js {monaco-run} {autorun:false}
console.log("1: start");

setTimeout(() => console.log("2: timeout"), 0);

Promise.resolve().then(() => console.log("3: promise"));

console.log("4: end");
```

- https://www.jsv9000.app/?code=Y29uc29sZS5sb2coIjE6IHN0YXJ0Iik7DQoNCnNldFRpbWVvdXQoKCkgPT4gY29uc29sZS5sb2coIjI6IHRpbWVvdXQiKSwgMCk7DQoNClByb21pc2UucmVzb2x2ZSgpLnRoZW4oKCkgPT4gY29uc29sZS5sb2coIjM6IHByb21pc2UiKSk7DQoNCmNvbnNvbGUubG9nKCI0OiBlbmQiKTs%3D

- https://www.jsvisualizer.bytefront.dev/?share=JTdCJTIyY29kZSUyMiUzQSUyMmNvbnNvbGUubG9nKCU1QyUyMjElM0ElMjBzdGFydCU1QyUyMiklM0IlNUNuJTVDbnNldFRpbWVvdXQoKCklMjAlM0QlM0UlMjBjb25zb2xlLmxvZyglNUMlMjIyJTNBJTIwdGltZW91dCU1QyUyMiklMkMlMjAwKSUzQiU1Q24lNUNuUHJvbWlzZS5yZXNvbHZlKCkudGhlbigoKSUyMCUzRCUzRSUyMGNvbnNvbGUubG9nKCU1QyUyMjMlM0ElMjBwcm9taXNlJTVDJTIyKSklM0IlNUNuJTVDbmNvbnNvbGUubG9nKCU1QyUyMjQlM0ElMjBlbmQlNUMlMjIpJTNCJTIyJTJDJTIyYnJlYWtwb2ludHMlMjIlM0ElNUIlNUQlN0Q=
---

## What is the result?

```js {monaco-run}{autorun:false}
// แบบเดิม: callback-based
setTimeout(() => {
  console.log("3 วินาทีผ่านไป");
}, 3000);

// promisify แล้ว
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

// ใช้งานได้ด้วย async/await — อ่านง่ายขึ้นมาก โดยเฉพาะเวลามีหลาย step
async function main() {
  console.log("start");
  await delay(1000);
  console.log("1 วินาทีผ่านไป");
  await delay(1000);
  console.log("2 วินาทีผ่านไป");
}
main();
console.log('นอก main ยังทำงานได้ต่อ เพราะใน main เจอ await')
```

---

## What is the result?

- ตัวอย่างนี้ใช้กับ jsv9000.app ได้ แต่ต้องไม่มีอักษรไทย และใช้ async/await ไม่ได้

```js
console.log("1: start");

function foo() {
  console.log("2: foo start");
  Promise.resolve(null).then(() => {
    console.log("3: foo end");
  });
}

foo();
console.log("4: sync end");
```

https://www.jsv9000.app/?code=Y29uc29sZS5sb2coIjE6IHN0YXJ0Iik7CgpmdW5jdGlvbiBmb28oKSB7CiAgY29uc29sZS5sb2coIjI6IGZvbyBzdGFydCIpOwogIFByb21pc2UucmVzb2x2ZShudWxsKS50aGVuKCgpID0%2BIHsKICAgIGNvbnNvbGUubG9nKCIzOiBmb28gZW5kIik7CiAgfSk7Cn0KCmZvbygpOwpjb25zb2xlLmxvZygiNDogc3luYyBlbmQiKTs%3D

---

## What is the result?

```js

console.log("start");

function loopMicrotask() {
  Promise.resolve().then(() => {
    console.log("microtask");
    loopMicrotask();
  });
}
loopMicrotask();

setTimeout(() => console.log("this will NEVER run"), 0);
```

- ตัวอย่างนี้จะเกิดลูปใน micro task

---

## What is the result?

```js {monaco-run}{autorun:false}
console.log("1");

setTimeout(() => console.log("2"), 0);

function main() {
  console.log("3");
  new Promise((resolve) => setTimeout(resolve, 0)).then(() => {
    console.log("4");
  });
}
main();

console.log("5");

```

---

## What is the result?

```js {monaco-run}{autorun:false}
console.log("1");

setTimeout(() => console.log("2"), 0);

function delayedLog() {
  console.log("3");
  setTimeout(() => {
    console.log("4");
  }, 0);
}
delayedLog();

console.log("5");


```
---

## Reference

- อธิบายทฤษฎี
  - https://33jsconcepts.com/concepts/event-loop
  - https://master.dev/tutorials/lydia-hallie/event-loop/

- มีตัวอย่างการทำงาน
  - https://www.jsv9000.app/
  - https://js-visualizer.gouranga.eu.org/
  - https://www.jsvisualizer.bytefront.dev/
