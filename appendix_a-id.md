# Lampiran A: Dukungan Fungsi Esensial

Dalam lampiran ini, Anda akan menemukan beberapa implementasi JavaScript dasar dari berbagai fungsi yang dijelaskan dalam buku ini.

Ingatlah bahwa implementasi ini mungkin bukan implementasi tercepat atau paling efisien di luar sana; mereka dibuat _hanya untuk tujuan pendidikan_.

Untuk menemukan fungsi yang lebih siap produksi, lihat [ramda](https://ramdajs.com/), [lodash](https://lodash.com/), atau [folktale](http://folktale.origamitower.com/).

Perhatikan bahwa beberapa fungsi juga mengacu pada struktur aljabar yang didefinisikan dalam [Lampiran B](./appendix_b-id.md)

## always

```js
// always :: a -> b -> a
const always = curry((a, b) => a);
```

## compose

```js
// compose :: ((y -> z), (x -> y),  ..., (a -> b)) -> a -> z
const compose =
  (...fns) =>
  (...args) =>
    fns.reduceRight((res, fn) => [fn.call(null, ...res)], args)[0];
```

## curry

```js
// curry :: ((a, b, ...) -> c) -> a -> b -> ... -> c
function curry(fn) {
  const arity = fn.length;

  return function $curry(...args) {
    if (args.length < arity) {
      return $curry.bind(null, ...args);
    }

    return fn.call(null, ...args);
  };
}
```

## either

```js
// either :: (a -> c) -> (b -> c) -> Either a b -> c
const either = curry((f, g, e) => {
  if (e.isLeft) {
    return f(e.$value);
  }

  return g(e.$value);
});
```

## identity

```js
// identity :: x -> x
const identity = (x) => x;
```

## inspect

```js
// inspect :: a -> String
const inspect = (x) => {
  if (x && typeof x.inspect === "function") {
    return x.inspect();
  }

  function inspectFn(f) {
    return f.name ? f.name : f.toString();
  }

  function inspectTerm(t) {
    switch (typeof t) {
      case "string":
        return `'${t}'`;
      case "object": {
        const ts = Object.keys(t).map((k) => [k, inspect(t[k])]);
        return `{${ts.map((kv) => kv.join(": ")).join(", ")}}`;
      }
      default:
        return String(t);
    }
  }

  function inspectArgs(args) {
    return Array.isArray(args)
      ? `[${args.map(inspect).join(", ")}]`
      : inspectTerm(args);
  }

  return typeof x === "function" ? inspectFn(x) : inspectArgs(x);
};
```

## left

```js
// left :: a -> Either a b
const left = (a) => new Left(a);
```

## liftA2

```js
// liftA2 :: (Applicative f) => (a1 -> a2 -> b) -> f a1 -> f a2 -> f b
const liftA2 = curry((fn, a1, a2) => a1.map(fn).ap(a2));
```

## liftA3

```js
// liftA3 :: (Applicative f) => (a1 -> a2 -> a3 -> b) -> f a1 -> f a2 -> f a3 -> f b
const liftA3 = curry((fn, a1, a2, a3) => a1.map(fn).ap(a2).ap(a3));
```

## maybe

```js
// maybe :: b -> (a -> b) -> Maybe a -> b
const maybe = curry((v, f, m) => {
  if (m.isNothing) {
    return v;
  }

  return f(m.$value);
});
```

## nothing

```js
// nothing :: Maybe a
const nothing = Maybe.of(null);
```

## reject

```js
// reject :: a -> Task a b
const reject = (a) => Task.rejected(a);
```
