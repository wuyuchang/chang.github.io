---
title: Principle of Promise/Deferred
date: 2016-12-09 10:21:08
tags: Node.js
---

# Super simple realize separate call an async and handle result in two files.
Assume we have a project base on jQuery, we have to separate call AJAX and handle result in another file.

Most of the time, we handle the result with below.
``` JavaScript
$.get('url', data => {
  //handle data
})
```

But, sometimes, we don't want to handle it in the moment. Maybe we just want to handle in another file. So we can do below.
`request.js`
``` JavaScript
$.get('url', data => {
  window.result(data)
})
```
`handle.js`
``` JavaScript
window.result = data => {
  // handle result
}
```

But, we most of the time, we are not just handle the success result. We also handle error, receiving data.
For example.

`request.js`
``` JavaScript
$.ajax({
  url: 'url',
  data: 'data',
  type: 'get',
  success: data => {
    window.deferrd.resolve(data)
  },
  error: err => {
    window.defered.error(err)
  },
  complete: notify => {
    window.deferred.notify('complete')
  }
})
```

`handle.js`
``` JavaScript
window.deferred = {
  //handle success
  resolve: data => {
    console.log(data)
  },
  //handle error
  error: err => {
    console.log(err)
  },
  //handle complete
  notify: msg => {
    console.log(msg)
  }
}
```

# use jQuery promise/deferred
Fortunately, jQuery already realize this, so we just have to do below.
`request.js`
``` JavaScript
window.promise = $.get('url')
```
`handle.js`
``` JavaScript
window.promise.then(data => {
  // handle success
}, err => {
  // handle error
})
```

# Promise/A
For show how promise and deferred work, we create a new and simple promise and deferred object, so that you can easy understand it.
## Promise Object
Here try to extend EventEmitter module to explain how it work.
`Promise.class.js`
``` JavaScript
class Promise extends EventEmitter {
  then(success, error, progress) {
    if (typeof success === 'function') {
      this.once('success', success)
    }
    if (typeof error === 'function') {
      this.once('error', error)
    }
    if (typeof progress === 'progress') {
      // Note, this function will be recall, so here defined the event use 'on'
      this.on('progress', progress)
    }
  }
}
```
Here you can see we defined three events after you pass these callback function into it through call 'then' function.
So we defined the events, and we have to emit it, so that we can execute these callbacks with the result.

Below, we defined the Deferred object.
## Deferred Object
Here we defined the Deferred, these functions just emit promise, so tell them we finished.
`Deferred.class.js`
``` JavaScript
class Deferred {
  this.state = 'unsuccess'
  this.promise = new Promise()

  resolve(obj) {
    this.state = 'success'
    this.promise.emit('success', obj)
  }
  reject(obj) {
    this.state = 'error'
    this.promise.emit('error', obj)
  }
  progress(obj) {
    this.state = 'progress'
    this.promise.emit('progress', obj)
  }
}
```


## use Promise & Deferred
To realize this, we create a new Deferred, and we trigger deferred function (progress/end/error) when we finish specific action to tell Deferred that we finished.
Then we get promise object, and then we can store it, and call it with callback in another file.
So we successful separate call async and get data in two files.
``` JavaScript
let promisify = res => {
  let deferred = new Deferred()
  let result = ''

  res.on('data', chunk => {
    result += chunk
    deferred.progress(result)
  })
  res.on('end', () => {
    deferred.resolve(result)
  })
  res.on('error', () => {
    deferred.reject()
  })

  return deferred.promise
}

global.result = promisify(res)


// in another file, this just an example to pass the result to global, so we can easy get the result, in the production, you better to pass the result in other way.
global.result.then((result) => {
  // success
}, (err) => {
  // error
}, (chunk) => {
  // progress
  console.log(chunk)
})
```

## Deferred.all()
Do some action after a several of asynchronous finished.
`Deferred.class.js`
``` JavaScript
class Deferred {
  // ...

  all(promises) {
    let count = promises.length
    let that = this
    let results = []
    promises.forEach((promise, i) => {
      promise.then(data => {
        count --
        results[i] = data
        if (count === 0) {
          that.resolve(results)
        }
      })
    }, err => {
      that.reject(err)
    })

    return this.promise
  }
}
```

## Order Run asynchronous
Run asynchronous in order
`Deferred.class.js`
``` JavaScript
const Promise = require('./promise.class')

module.exports = class Deferred {
  constructor() {
    this.promise = new Promise()
  }

  resolve(data) {
    let promise = this.promise
    let handler
    while(handler = promise.queue.shift()) {
      if (handler && handler.resolve) {
        let ret = handler.resolve(data)

        // if there has next promise, then we interrupt current loop, and set next promise.queue equals current queue
        if (ret && ret.isPromise) {
          ret.queue = promise.queue
          return
        }
      }
    }
  }

  reject(err) {
    let promise = this.promise
    let handler
    while(handler = promise.queue.shift()) {
      if (handler && handler.reject) {
        let ret = handler.reject(err)

        // if there has next promise, then we interrupt current loop, and set next promise.queue equals current queue
        if (ret && ret.isPromise) {
          ret.queue = promise.queue
          return
        }
      }
    }
  }

  callback(str) {
    let that = this
    return (err, file) => {
      if (err) {
        return that.reject(err)
      }
      that.resolve(file)
    }
  }
}
```

`promise.class.js`
``` JavaScript
module.exports = class Promise {
  constructor() {
    this.queue = []
    this.isPromise = true
  }

  then(resolve, reject) {
    let handler = {}
    if (typeof resolve === 'function') {
      handler.resolve = resolve
    }
    if (typeof reject === 'function') {
      handler.reject = reject
    }

    this.queue.push(handler)
    return this
  }
}
```

`app.js`
``` JavaScript
const Deferred = require('./deferred.class')
const fs = require('fs')

let action1 = (file, encoding) => {
  let deferred = new Deferred()
  fs.readFile(file, encoding, deferred.callback())
  return deferred.promise
}

let action2 = (file, encoding) => {
  let deferred = new Deferred()
  fs.readFile(file, encoding, deferred.callback())
  return deferred.promise
}

action1('./file1.txt', 'utf8').then(file1 => {
  return action2(file1.trim(), 'utf8')
}).then(file2 => {
  console.log(file2)
})
```
