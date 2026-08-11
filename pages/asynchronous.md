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

## What is the result?

```js {monaco-run} {autorun:false}
console.log("1: start");

setTimeout(() => console.log("2: timeout"), 0);

Promise.resolve().then(() => console.log("3: promise"));

console.log("4: end");
```

- https://www.jsv9000.app/?code=Y29uc29sZS5sb2coIjE6IHN0YXJ0Iik7DQoNCnNldFRpbWVvdXQoKCkgPT4gY29uc29sZS5sb2coIjI6IHRpbWVvdXQiKSwgMCk7DQoNClByb21pc2UucmVzb2x2ZSgpLnRoZW4oKCkgPT4gY29uc29sZS5sb2coIjM6IHByb21pc2UiKSk7DQoNCmNvbnNvbGUubG9nKCI0OiBlbmQiKTs%3D

---

## Reference

- อธิบายทฤษฎี
  - https://33jsconcepts.com/concepts/event-loop
  - https://master.dev/tutorials/lydia-hallie/event-loop/

- มีตัวอย่างการทำงาน
  - https://www.jsv9000.app/
  - https://js-visualizer.gouranga.eu.org/
  - https://www.jsvisualizer.bytefront.dev/
