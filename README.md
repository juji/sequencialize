
# sequencialize

Run async functions in sequence. Queue async functions.

How to make parallel-running async function run sequencially?
This resolve a problem when something like `findAndUpdate` creates two entries when run almost parallel.
So, just chain them.

If you are using message broker to queue potentially-breaking parallel function calls, you won't need this.

## Usage
```js
const Sequencer = require('sequencialize')
const sequencer = new Sequencer()

await sequencer.run(func:Function, [namespace:String])
```

It will return the value returned from the original function. So:
```js
const n = 3
const func = async n => n

for(let i = n;i;i--){
  sequencer.run(() => func(i)).then(v => console.log(v))
}

// 3
// 2
// 1

```

It will run functions in sequence:
```js
const wait = t => new Promise(r => setTimeout(r,t))
const n = 3
const func = n => {
  return wait(1000+ Math.random() * 5000)
    .then(() => console.log(n))
}

for(let i = n;i;i--){
  sequencer.run(() => func(i))
}

// 3
// 2
// 1

```

you can use namespace. each namespace is it's own sequence:
```js

const wait = t => new Promise(r => setTimeout(r,t))
const n = 3
const func = (namespace, n) => {
  return wait(1000+ Math.random() * 5000)
    .then(() => console.log(`${namespace}: ${n}`))
}

for(let i = n;i;i--){
  sequencer.run(() => func('asdf', i), 'asdf')
  sequencer.run(() => func('ASDF', i), 'ASDF')
}

// each function in a namespace runs in sequence
// asdf: 3
// ASDF: 3
// asdf: 2
// asdf: 1
// ASDF: 2
// ASDF: 1

```

## Use Case
```

// running almost in parallel
db.upsert({
  where: { user_id },
  data
})

```


Cheers,
[juji](https://jujiyangasli.com)
