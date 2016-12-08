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


## async
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
  emitter.emit('doen', 'template', tempalte)
})
db.query(sql, (err, data) => {
  emitter.emit('done', 'data', data)
})
l10n.get((err, resources) => {
  emitter.emit('done', 'resources', resources)
})
```


``` JavaScript
let after = (times, callback) => {
  let count = 0
  let result = {}
  return (key, value) => {
    result[key] = value
    count ++
    if (count === times) {
      callback(result)
    }
  }
}


let done = after(times, render)

fs.readFile(template_path, 'utf8', (err, template) => {
  done('template', template)
})
db.query(sql, (err, data) => {
  done('data', data)
})
l10n.get((err, resources) => {
  done('resources', resources)
})

```
