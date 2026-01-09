# Promise

## Definition / Concept
A **Promise** is an object representing the eventual completion or failure of an asynchronous operation. It's a proxy for a value that may not be available yet but will be resolved at some point in the future. Promises provide a cleaner alternative to callback-based asynchronous code, avoiding **callback hell** and enabling better error handling through chaining.

- Represents a value that will be available in the future (pending → fulfilled/rejected)
- Has three states: **pending**, **fulfilled**, **rejected**
- Enables method chaining with `.then()`, `.catch()`, `.finally()`
- Solves **callback hell** problem with cleaner, more readable code

## Visual Representation
```
Promise States:
                   ┌─────────────┐
                   │   PENDING   │
                   │  (initial)  │
                   └──────┬──────┘
                          │
           ┌──────────────┴──────────────┐
           ▼                             ▼
    ┌─────────────┐              ┌─────────────┐
    │  FULFILLED  │              │  REJECTED   │
    │  (success)  │              │   (error)   │
    └─────────────┘              └─────────────┘
         │                              │
         ▼                              ▼
    .then(value)                 .catch(error)
         │                              │
         └──────────────┬───────────────┘
                        ▼
                  .finally(cleanup)

Promise Chain:
fetchUser()
  .then(user => fetchPosts(user.id))
  .then(posts => displayPosts(posts))
  .catch(error => handleError(error))
  .finally(() => hideLoader())
```

## Example
```javascript
// Creating a Promise
const myPromise = new Promise((resolve, reject) => {
  const success = true;
  
  setTimeout(() => {
    if (success) {
      resolve('Operation successful!'); // Fulfilled
    } else {
      reject('Operation failed!'); // Rejected
    }
  }, 1000);
});

// Consuming a Promise
myPromise
  .then(result => {
    console.log(result); // 'Operation successful!'
    return 'Next step';
  })
  .then(result => {
    console.log(result); // 'Next step'
  })
  .catch(error => {
    console.error(error); // Handles any error in chain
  })
  .finally(() => {
    console.log('Cleanup'); // Always executes
  });

// Real-world example: Fetching data
function fetchUserData(userId) {
  return new Promise((resolve, reject) => {
    fetch(`/api/users/${userId}`)
      .then(response => {
        if (!response.ok) {
          reject(new Error('User not found'));
        }
        return response.json();
      })
      .then(data => resolve(data))
      .catch(error => reject(error));
  });
}

// Using the Promise
fetchUserData(123)
  .then(user => {
    console.log('User:', user);
    return fetchUserData(user.friendId);
  })
  .then(friend => {
    console.log('Friend:', friend);
  })
  .catch(error => {
    console.error('Error:', error.message);
  });

// Promise.all - wait for all promises
const promise1 = fetch('/api/users');
const promise2 = fetch('/api/posts');
const promise3 = fetch('/api/comments');

Promise.all([promise1, promise2, promise3])
  .then(([users, posts, comments]) => {
    console.log('All data loaded:', users, posts, comments);
  })
  .catch(error => {
    console.error('One of the requests failed:', error);
  });

// Promise.race - first to settle wins
const timeout = new Promise((_, reject) => {
  setTimeout(() => reject(new Error('Timeout')), 5000);
});

const apiCall = fetch('/api/data');

Promise.race([apiCall, timeout])
  .then(data => console.log('Got data:', data))
  .catch(error => console.error('Request timed out or failed'));

// Promise.allSettled - wait for all, get all results
Promise.allSettled([
  Promise.resolve('Success 1'),
  Promise.reject('Error 1'),
  Promise.resolve('Success 2')
])
  .then(results => {
    results.forEach(result => {
      if (result.status === 'fulfilled') {
        console.log('Success:', result.value);
      } else {
        console.log('Failed:', result.reason);
      }
    });
  });

// Promise.any - first fulfilled promise wins
Promise.any([
  fetch('/api/server1/data'),
  fetch('/api/server2/data'),
  fetch('/api/server3/data')
])
  .then(data => console.log('First successful response:', data))
  .catch(error => console.error('All requests failed'));

// Chaining transformations
fetch('/api/users/123')
  .then(response => response.json())
  .then(user => user.name)
  .then(name => name.toUpperCase())
  .then(upperName => console.log(upperName))
  .catch(error => console.error(error));

// Error handling patterns
// ❌ Bad: Unhandled rejection
const badPromise = new Promise((resolve, reject) => {
  reject('Error!');
});
// No .catch() - unhandled rejection warning!

// ✅ Good: Always handle rejections
const goodPromise = new Promise((resolve, reject) => {
  reject('Error!');
}).catch(error => console.error('Handled:', error));

// Converting callbacks to Promises
function readFilePromise(filename) {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, 'utf8', (error, data) => {
      if (error) {
        reject(error);
      } else {
        resolve(data);
      }
    });
  });
}

// Using promisify (Node.js)
const util = require('util');
const fs = require('fs');
const readFilePromise = util.promisify(fs.readFile);

readFilePromise('file.txt', 'utf8')
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

## Usage
- **When to use**: 
  - Asynchronous operations (API calls, file I/O, timers)
  - Replacing callback-based code
  - Coordinating multiple async operations
  - Error handling in async flows
  - Modern JavaScript libraries and frameworks
  
- **Real-world example**:
  ```javascript
  // Authentication flow
  function login(email, password) {
    return authenticate(email, password)
      .then(token => saveToken(token))
      .then(() => fetchUserProfile())
      .then(profile => updateUI(profile))
      .catch(error => {
        if (error.code === 401) {
          showError('Invalid credentials');
        } else {
          showError('Login failed. Please try again.');
        }
      })
      .finally(() => {
        hideLoadingSpinner();
      });
  }
  
  // Parallel data loading
  function loadDashboard() {
    showLoadingSpinner();
    
    Promise.all([
      fetchUserStats(),
      fetchRecentActivity(),
      fetchNotifications()
    ])
      .then(([stats, activity, notifications]) => {
        renderStats(stats);
        renderActivity(activity);
        renderNotifications(notifications);
      })
      .catch(error => {
        showError('Failed to load dashboard');
        console.error(error);
      })
      .finally(() => {
        hideLoadingSpinner();
      });
  }
  
  // Retry logic
  function fetchWithRetry(url, retries = 3) {
    return fetch(url)
      .then(response => {
        if (!response.ok) throw new Error('Request failed');
        return response.json();
      })
      .catch(error => {
        if (retries > 0) {
          console.log(`Retrying... (${retries} attempts left)`);
          return fetchWithRetry(url, retries - 1);
        }
        throw error;
      });
  }
  
  // Sequential operations
  function processQueue(tasks) {
    return tasks.reduce((promiseChain, task) => {
      return promiseChain.then(() => task());
    }, Promise.resolve());
  }
  
  processQueue([
    () => saveData(data1),
    () => updateCache(),
    () => sendNotification()
  ]);
  ```

- **Best practices**:
  - Always handle rejections with `.catch()` to avoid unhandled rejections
  - Use `Promise.all()` for parallel operations
  - Use `Promise.allSettled()` when you need all results regardless of failures
  - Return promises from `.then()` to maintain the chain
  - Use `async/await` for better readability (modern alternative)
  - Don't nest promises - chain them instead
  - Add `.finally()` for cleanup operations

## FAQ / Interview Questions

**Q: What is a Promise and what are its states?**
A: A **Promise** is an object representing the eventual result of an asynchronous operation. It has three states:
1. **Pending**: Initial state, operation not yet completed
2. **Fulfilled**: Operation completed successfully with a value
3. **Rejected**: Operation failed with a reason (error)

Once a promise settles (fulfilled or rejected), it's **immutable** - the state and value/reason cannot change.

**Q: What's the difference between Promise.all() and Promise.race()?**
A: 
- **Promise.all([p1, p2, p3])**:
  - Waits for ALL promises to fulfill
  - Returns array of all results in order
  - Rejects immediately if ANY promise rejects
  - Use case: Load multiple resources that are all needed

- **Promise.race([p1, p2, p3])**:
  - Returns when FIRST promise settles (fulfilled or rejected)
  - Returns single result from fastest promise
  - Use case: Timeout patterns, fastest server wins

```javascript
// Promise.all - all must succeed
Promise.all([p1, p2, p3]) // [result1, result2, result3]

// Promise.race - first to finish wins
Promise.race([apiCall, timeout]) // result from fastest
```

**Q: How do you handle errors in Promise chains?**
A: Multiple approaches:
1. **Single .catch() at end** - catches all errors in chain
2. **Multiple .catch()** - handle specific errors at different points
3. **Return from .catch()** - recover from error and continue chain

```javascript
// Pattern 1: Single catch for all errors
promise1()
  .then(result => promise2(result))
  .then(result => promise3(result))
  .catch(error => console.error(error)); // Catches any error

// Pattern 2: Specific error handling
promise1()
  .then(result => promise2(result))
  .catch(error => {
    // Handle promise2 errors
    return fallbackValue; // Recover and continue
  })
  .then(result => promise3(result))
  .catch(error => {
    // Handle promise3 errors
  });

// Pattern 3: Finally for cleanup
promise()
  .then(result => process(result))
  .catch(error => handleError(error))
  .finally(() => cleanup()); // Always runs
```

**Q: What's the difference between Promise.all() and Promise.allSettled()?**
A:
- **Promise.all()**: 
  - Rejects immediately if any promise rejects
  - Returns array of values only on full success
  - Use when all operations must succeed

- **Promise.allSettled()**:
  - Waits for all promises regardless of outcome
  - Returns array of objects: `{status: 'fulfilled'/'rejected', value/reason}`
  - Use when you need all results even if some fail

```javascript
// Promise.all - fails fast
Promise.all([p1, p2, p3])
  .then(results => console.log(results)) // Only if all succeed
  .catch(error => console.error(error)); // First error

// Promise.allSettled - always waits
Promise.allSettled([p1, p2, p3])
  .then(results => {
    results.forEach(result => {
      if (result.status === 'fulfilled') {
        console.log('Success:', result.value);
      } else {
        console.log('Failed:', result.reason);
      }
    });
  });
```

**Q: How do you convert callback-based code to Promises?**
A: Wrap the callback function in a Promise constructor:
```javascript
// Callback-based
function fetchData(callback) {
  setTimeout(() => {
    callback(null, 'data');
  }, 1000);
}

fetchData((error, data) => {
  if (error) console.error(error);
  else console.log(data);
});

// Promise-based
function fetchDataPromise() {
  return new Promise((resolve, reject) => {
    fetchData((error, data) => {
      if (error) {
        reject(error);
      } else {
        resolve(data);
      }
    });
  });
}

fetchDataPromise()
  .then(data => console.log(data))
  .catch(error => console.error(error));

// Or use util.promisify in Node.js
const util = require('util');
const fetchDataPromise = util.promisify(fetchData);
```

## References
- [MDN Web Docs - Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [JavaScript.info - Promises](https://javascript.info/promise-basics)
- [MDN Web Docs - Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [Promises/A+ Specification](https://promisesaplus.com/)

---
*See also: [Async/Await](AsyncAwait.md), [Event Loop](EventLoop.md), [Callbacks](../JavaScript/)*
