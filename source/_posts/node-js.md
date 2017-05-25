---
title: Node.js
date: 2016-12-08 20:35:41
tags: [Node.js, MongoDB, Performance]
---

## Perform query SQL for server during variety of requests with same result at specific time
Browser auto cache the result of AJAX for performance, but sometimes we get a special activity which a lot of user will access server for same data at specific time, if we don't handle it, the server may die. So we can use event queue to handle the problem.
``` JavaScript
let proxy = new events.EventEmitter()
let status = 'ready'
let select = callback => {
  proxy.once('selected', callback)  // bind event for every request
  if (status === 'ready') {
    status = 'pending'
    db.select('sql', result => {      // if SQL query finished,
      proxy.emit('selected', result)  // then trigger event
      status = 'ready'                // and recover status
    })
  }
}
```


## asynchronous
Asynchronous is a good object, you can run I/O operate without wait it. It's a good way to improve the CPU usage rate.
Sometimes, we have multiple I/O operates, and these are associated, we want to do some action after all of the I/O operates finished. What can we do? You may do like this below.
``` JavaScript
fs.readFile(path, 'utf8', (err, template) => {
  db.query(sql, (err, data) => {
    l10n.get((err, resources) => {
      // action
    })
  })
})
```
It's easy to do that right? But mention the callback, you run the action in the 3 level callbacks. Think about it, if you have to do 4 or 5 I/O operate, and you do action after all of theses finished. so you may write '}) }) }) }) })', seems painful to maintain the program.

Anyway, we don't do above, that is so stupid. We can make these I/O operate in first level without put an operate in to another callback. Read below.

``` JavaScript
let after = (times, callback) => {
  let count = 0
  let result = {}
  return (key, value) => {
    result[key] = value
    count++
    if (count === times) {
      callback(results)
    }
  }
}


let emitter = new events.Emitter()
let done = after(times, render)

emitter.on('done', done)
emitter.on('done', other)

fs.readFile(template_path, 'utf8', (err, template) => {
  emitter.emit('done', 'template', tempalte)
})
db.query(sql, (err, data) => {
  emitter.emit('done', 'data', data)
})
l10n.get((err, resources) => {
  emitter.emit('done', 'resources', resources)
})
```
> similar with `async` in waterfall

## order asynchronous
https://wuyuchang.github.io/2016/12/09/Principle-of-Promise-Deferred/

## smooth
``` JavaScript
let smooth = method => {
  return () => {
    let deferred = new Deferred()
    let args = Array.prototype.slice.call(arguments, 0) // transform the arguments to an array
    args.push(deferred.callback()) // push an parameter, [file, encode, deferred.callback()]
    method.apply(null, args) // call method, fs.readFile(file, encode, deferred.callback())
    return deferred.promise
  }
}
```
