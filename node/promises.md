# Promise API

In ES6 we have an alternative mechanism built into the language called a _promise_.

A _promise_ is a _placeholder_ for a future value.

It serves the same function as callbacks but has a nicer syntax and makes it easier to handle errors.

## Creating a Promise

We create an instance of a promise by calling `new` on the `Promise` class, like so:

```js
const promise = new Promise((resolve, reject) => {
  // resolve? reject?
});
```

Inside this inner function we perform our asynchronous processing and then when we are ready we call `resolve()`, like so:

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log("Async Work Complete");
    resolve(); // <-- Resolving
  }, 1000);
});
```

We usually return this promise from a function, like so:

```js
function doAsyncTask() {
  // <-- NOTE: Not passing in a callback

  const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("Async Work Complete");
      resolve(); // <-- Resolving
    }, 1000);
  });
  return promise; // <-- Return the promise
}
```

If there was an error in the async task then we call the `reject()` function like so:

```js
function doAsyncTask() {
  let error = false;
  const promise = new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("Async Work Complete");
      if (error) {
        reject(); // <-- Rejecting
      } else {
        resolve();
      }
    }, 1000);
  });
  return promise;
}
```

## Promise Notifications

We can get notified when a promise `resolves` by attaching a _success_ handler to its `then` function, like so:

```js
doAsyncTask().then(() => console.log("Task Complete!"));
```

`then` can take two arguments, the second argument is a _error_ handler that gets called if the promise is `rejected`, like so:

```js
doAsyncTask().then(
  () => console.log("Task Complete!"),
  () => console.log("Task Errored!")
);
```

Any values we pass to the `resolve` and `reject` functions are passed along to the _error_ and _success_ handlers, like so:

```js
function doAsyncTask() {
  let error = false;
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (error) {
        reject("error"); // pass values
      } else {
        resolve("done"); // pass values
      }
    }, 1000);
  });
}

doAsyncTask().then(
  (val) => console.log(val),
  (err) => console.error(err)
);
```

## Immediate Resolution or Rejection

We can create an immediately _resolved_ Promise by using the `Promise.resolve()` method, like so:

```js
let promise = Promise.resolve("done");
```

And an immediately _rejected_ Promise by using the `Promise.reject()` method, like so:

```js
let promise = Promise.reject("fail");
```

One of the nice things about Promises is that if we add a `then` handler _after_ the promise resolves or rejects the handler _still_ gets called.

```js
let promise = Promise.resolve("done");

let promise = Promise.resolve("done");
promise.then((val) => console.log(val)); // 'done'
```

In the above example, even though the Promise has resolved _before_ we added the success handler, the promise framework still calls the success handler.

## Promise is really async

```js
function doAsyncTask() {
  return Promise.resolve();
}

doAsyncTask().then((_) => console.log(message)); // <-- Unlike callbacks, promises are always async
let message = "Promise Resolved";
```

## Chaining

We can also connect a series of `then` handlers together in a chain, like so:

```js
const prom = Promise.resolve("done");
prom
  .then((val) => {
    console.log(val);
    return "done2"; // <-- !NOTE: We have to return something, otherwise it doesn't get passed
  })
  .then((val) => console.log(val));
// 'done'
// 'done2'
```

- We **have** to return something from each `then`, otherwise it doesn't get passed to the next `then`

```js
const prom = Promise.resolve("done");
prom
  .then((val) => {
    console.log(val);
  })
  .then((val) => console.log(val));
// 'done'
// 'undefined'
```

- This is different to forking a promise chain

```js
const prom = Promise.resolve("done");
prom.then((val) => {
  console.log(val);
  return "done2";
});

prom.then((val) => console.log(val)); // <-- Doesn't get passed the result of the previous then
// 'done'
// 'done'
```

We can also pause execution waiting for another promise to resolve

```js
Promise.resolve("done")
  .then((val) => {
    console.log(val);

    return new Promise((resolve) => {
      setTimeout(() => resolve("done2"), 1000);
    });

    // The next then waits for this promise to resolve before continueing
  })
  .then((val) => console.log(val));
```

## Error Handling

Promises pass an error along the chain till it finds an error handler. So we don't need to define an error handler for each `then` function, we can just add one at the end like so:

```js
Promise.reject("fail")
  .then((val) => console.log(val)) // <-- Note we dont have an error handler here!
  .then(
    (val) => console.log(val),
    (err) => console.error(err)
  );
```

If we _throw_ an exception from our promise function or one of the success handlers, the promise gets rejected and the error handler is called, like so:

```js
new Promise((resolve, reject) => {
  throw "fail";
})
  .then((val) => {
    console.log(val);
  })
  .then(
    (val) => console.log(val),
    (err) => console.error(err)
  );
// [Error: fail]
```

```js
Promise.resolve("done")
  .then((val) => {
    throw "fail";
  })
  .then(
    (val) => console.log(val),
    (err) => console.error(err)
  );
// [Error: fail]
```

The `catch` function works exactly the same way as the `then` error handler, it's just clearer and more explicitly describes our intent to handle errors.

```js
Promise.resolve("done")
  .then((val) => {
    throw "fail";
  })
  .then((val) => console.log(val))
  .catch((err) => console.error(err));
```

## Finally

In preview

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally

```js
Promise.resolve("done")
  .then((val) => {
    throw new Error("fail");
  })
  .then((val) => console.log(val))
  .catch((err) => console.error(err))
  .finally((_) => console.log("Cleaning Up")); // <-- Comming soon!
```

## Multiple Promises

### Promise.all

```js
const util = require("util");
const fs = require("fs");
const readFile = util.promisify(fs.readFile);

const files = ["./files/demofile.txt", "./files/demofile.other.txt"];

let promises = files.map((name) => readFile(name, "utf8"));
Promise.all(promises)
  .then((values) => {
    // <-- Uses .all
    console.log(values);
  })
  .catch((err) => console.error("Error: ", err));
```

### Promise.race

Resolves or rejects when the first promise in the array resolved or rejects

```js
let car1 = new Promise((resolve) => setTimeout(resolve, 1000, "Car 1."));
let car2 = new Promise((resolve) => setTimeout(resolve, 2000, "Car 2."));
let car3 = new Promise((resolve) => setTimeout(resolve, 3000, "Car 3."));

Promise.race([car1, car2, car3]).then((value) => {
  console.log("Promise Resolved", value);
});
```

<!-- More usefull if requesting a lot of data that all needs to return. -->

```js
let car1 = new Promise((_, reject) =>
  setTimeout(reject, 3000, "Car 1 Crashed.")
);
let car2 = new Promise((resolve) => setTimeout(resolve, 1000, "Car 2."));
let car3 = new Promise((resolve) => setTimeout(resolve, 3000, "Car 3."));

Promise.race([car1, car2, car3])
  .then((value) => {
    console.log("Promise Resolved", value);
  })
  .catch((err) => {
    console.log("Promise Rejected", err);
  });
```

# Question 1 

Create a promise version of the async readFile function

```js
const fs = require("fs");

function readFile(filename, encoding) {
  fs.readFile(filename, encoding, (err, data) => {
    //TODO
  });
}
readFile("./files/demofile.txt", "utf-8")
    .then(...)
});
```

Solution:

Create a promise version of the async readFile

```js
const fs = require("fs");
// const util = require("util");

// const readFile = util.promisify(fs.readFile);

function readFile(filename, encoding) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, encoding, (err, data) => {
      if (err) reject(err);
      resolve(data);
    });
  });
}
readFile("./files/demofile.txt", "utf-8").then(
  (data) => console.log("File Read", data),
  (err) => console.error("Failed To Read File", err)
);
```

# Question 2

Load a file from disk using readFile and then compress it using the async zlib node library, use a promise chain to process this work.

```js
const fs = require("fs");
const zlib = require("zlib");

function zlibPromise(data) {
  zlib.gzip(data, (error, result) => {
    //TODO
  });
}

function readFile(filename, encoding) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, encoding, (err, data) => {
      if (err) reject(err);
      resolve(data);
    });
  });
}

readFile("./files/demofile.txt", "utf-8")
    .then(...) // --> Load it then zip it and then print it to screen
});
```

Solution:

```js
const fs = require("fs");
const zlib = require("zlib");
const util = require("util");

// const readFile = util.promisify(fs.readFile);
// const gzip = util.promisify(zlib.gzip);

function gzip(data) {
  return new Promise((resolve, reject) => {
    zlib.gzip(data, (err, result) => {
      if (err) return reject(err);
      resolve(result);
    });
  });
}

function readFile(filename, encoding) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, encoding, (err, data) => {
      if (err) return reject(err);
      resolve(data);
    });
  });
}

// Starting to look like callback hell?
readFile("./files/demofile.txt", "utf-8").then(
  (data) => {
    gzip(data).then(
      (res) => console.log(res),
      (err) => console.error("Failed To Zip", err)
    );
  },
  (err) => console.error("Failed To Read", err)
);
```

# Question 3

Convert the previous code so that it now chains the promise as well.

Solution:

```js
const fs = require("fs");
const zlib = require("zlib");
const util = require("util");

const readFile = util.promisify(fs.readFile);
const gzip = util.promisify(zlib.gzip);

readFile("./files/demofile.txt", "utf-8")
  .then(
    (data) => {
      return gzip(data);
    },
    (err) => {
      console.error("Failed To Read", err);
    }
  )
  .then(
    (data) => {
      console.log(data);
    },
    (err) => {
      console.error("Failed To Zip", err);
    }
  );
```

# Question 4

Convert the previous code so that it now handles errors using the catch handler

Solution:

```js
const fs = require("fs");
const zlib = require("zlib");
const util = require("util");

function gzip(data) {
  return new Promise((resolve, reject) => {
    zlib.gzip(data, (err, result) => {
      if (err) return reject(err);
      resolve(result);
    });
  });
}

function readFile(filename, encoding) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, encoding, (err, data) => {
      if (err) return reject(err);
      resolve(data);
    });
  });
}

readFile("./demofile.txt2", "utf-8")
  .then((data) => {
    return gzip(data);
  })
  .then((data) => {
    console.log(data);
  })
  .catch((err) => {
    console.error("Failed", err);
  });
```

- Again, throw doesn't work as you expect.

```js
const fs = require("fs");
const zlib = require("zlib");
const util = require("util");

function gzip(data) {
  return new Promise((resolve, reject) => {
    zlib.gzip(data, (err, result) => {
      if (err) return reject(err);
      resolve(result);
    });
  });
}

function readFile(filename, encoding) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, encoding, (err, data) => {
      if (err) throw err; // <-- Same as before this doesn't work, it's async
      resolve(data);
    });
  });
}

readFile("./demofile.txt2", "utf-8")
  .then((data) => {
    return gzip(data);
  })
  .then((data) => {
    console.log(data);
  })
  .catch((err) => {
    console.error("Failed", err);
  });
```

# Question 5

Create some code that tries to read from disk a file and times out if it takes longer than 1 seconds, use `Promise.race`

```js
function readFileFake(sleep) {
  return new Promise((resolve) => setTimeout(resolve, sleep));
}

readFileFake(5000); // This resolves a promise after 5 seconds, pretend it's a large file being read from disk
```

Solution:

```js
function readFileFake(sleep) {
  return new Promise((resolve) => setTimeout(resolve, sleep, "read"));
}

function timeout(sleep) {
  return new Promise((_, reject) => setTimeout(reject, sleep, "timeout"));
}

Promise.race([readFileFake(5000), timeout(1000)])
  .then((data) => console.log(data))
  .catch((err) => console.error(err));
```

# Question 6

Create a process flow which publishes a file from a server, then realises the user needs to login, then makes a login request, the whole chain should error out if it takes longer than 1 seconds. Use `catch` to handle errors and timeouts.

```js
function authenticate() {
  console.log("Authenticating");
  return new Promise(resolve => setTimeout(resolve, 2000, { status: 200 }));
}

function publish() {
  console.log("Publishing");
  return new Promise(resolve => setTimeout(resolve, 2000, { status: 403 }));
}

function timeout(sleep) {
  return new Promise((resolve, reject) => setTimeout(reject, sleep, "timeout"));
}

Promise.race( [publish(), timeout(3000)])
  .then(...)
  .then(...)
  .catch(...);
```

Solution:

```js
function authenticate() {
  console.log("Authenticating");
  return new Promise((resolve) => setTimeout(resolve, 2000, { status: 200 }));
}

function publish() {
  console.log("Publishing");
  return new Promise((resolve) => setTimeout(resolve, 2000, { status: 403 }));
}

function timeout(sleep) {
  return new Promise((resolve, reject) => setTimeout(reject, sleep, "timeout"));
}

Promise.race([publish(), timeout(1000)])
  .then((res) => {
    if (res.status === 403) {
      return authenticate();
    }
    return res;
  })
  .then((res) => {
    // Process save responce
    console.log("Published");
  })
  .catch((err) => {
    if (err === "timeout") {
      console.error("Request timed out");
    } else {
      console.error(err);
    }
  });
```

Alternative answer with safePublish returning a publish promise

```js
function authenticate() {
  console.log("Authenticating");
  return new Promise((resolve) => setTimeout(resolve, 2000, { status: 200 }));
}

function publish() {
  console.log("Publishing");
  return new Promise((resolve) => setTimeout(resolve, 2000, { status: 403 }));
}

function timeout(sleep) {
  return new Promise((resolve, reject) => setTimeout(reject, sleep, "timeout"));
}

function safePublish() {
  return publish().then((res) => {
    if (res.status === 403) {
      return authenticate();
    }
    return res;
  });
}

Promise.race([safePublish(), timeout(1000)])
  .then((res) => {
    // Process save responce
    console.log("Published");
  })
  .catch((err) => {
    if (err === "timeout") {
      console.error("Request timed out");
    } else {
      console.error(err);
    }
  });
```
