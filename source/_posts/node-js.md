---
title: Node.js
date: 2016-12-08 20:35:41
tags: node.js
---

## Event queue to solve a number of SQL with same result executed
If you want to use the cache after you execute a SQL, you can just execute it, then the OS will use the cache for you to improve the performance.
But sometimes, there are a lot of client send a lot of SQL request with same result in first (e.g. specific activity start at 10:00am), the result of the SQL haven't store to cache yet, so a number of request send to the OS, maybe your program will die. So it's essential to make requests that after the first not execute, and just use the cache by the first SQL.
``` JavaScript
let proxy = new events.EventEmitter()
let status = 'ready'
let select = callback => {
  proxy.once('selected', callback)
  if (status === 'ready') {
    status = 'pending'
    db.select('sql', result => {
      proxy.emit('selected', result)
      status = 'ready'
    })
  }
}
```


## asynchronous
Asynchronous is a good stuff, you can execute I/O operate without wait it. It's a good way to improve the CPU usage rate.
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
It's easy to do that right? But mention the callback, you write 3 level callback then do the action. Think about it, if you have to do 4 or 5 I/O operate, and you do action after all of theses finished. so you may write '}) }) }) }) })', seems painful to maintain the program.

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
