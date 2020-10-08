# install @vue/cli-service

There is something strange in the neighborhood...

## dev-environment

Build the development-container:
```
$ docker build --tag flipflop .
```

Run the development-container:
```
$ docker run -it --rm --user $(id -u):$(id -g) --mount type=bind,src="$(pwd)",dst=/app flipflop
```

Inside the dev-container, we have node (v12.18.4) and npm (v6.14.6).

## setup the repro

Install the package:
```
$ npm install @vue/cli-service
...
+ @vue/cli-service@4.5.7
added 1047 packages from 884 contributors and audited 1050 packages in 50.223s
```

Assumption: Running `npm install` again and again should not change the package-files.

Check assumption:
```
$ npm install
...
audited 1050 packages in 3.723s
```

Looking good.

## repro

As a different dev, we do not have directory `node_modules`:
```
$ rm -rf node_modules
```

We install the packages:
```
$ npm install
```

But now, the assumption that the package-files are not changed does not hold.
Instead, a couple of packages is moved around inside file `package-lock.json`.
After it changes once, subsequent runs of `npm install` do not change the lockfile anymore.

## flipflop

Until we emulate a fresh dev and once more delete directory `node_modules`.

Afterwards, running `npm install` will change the lockfile again.
The change is a "revert" of the last changes.

We can now keep doing this forevermore.
Everytime, we start without directory `node_modules` and run `npm install`,
the lockfile will flipflop between two states.

## Bonus

Diffing the two directories `node_modules` (since the dir is gitignored):
```
$ diff -r flip/ flop/
diff -r flip/big.js/package.json flop/big.js/package.json
25,26c25,26
<     "/loader-utils",
<     "/vue-loader-v16/loader-utils"
---
>     "/@vue/cli-service/loader-utils",
>     "/loader-utils"
diff -r flip/emojis-list/package.json flop/emojis-list/package.json
25,26c25,26
<     "/loader-utils",
<     "/vue-loader-v16/loader-utils"
---
>     "/@vue/cli-service/loader-utils",
>     "/loader-utils"
diff -r flip/hash-sum/package.json flop/hash-sum/package.json
26c26
<     "/vue-loader-v16"
---
>     "/@vue/cli-service/vue-loader-v16"
diff -r flip/minimist/package.json flop/minimist/package.json
25a26
>     "/@vue/cli-service/json5",
27,28c28
<     "/mkdirp",
<     "/vue-loader-v16/json5"
---
>     "/mkdirp"
Only in flop/@vue/cli-service: node_modules
diff -r flip/@vue/cli-service/package.json flop/@vue/cli-service/package.json
13c13,18
<   "_phantomChildren": {},
---
>   "_phantomChildren": {
>     "big.js": "5.2.2",
>     "emojis-list": "3.0.0",
>     "hash-sum": "2.0.0",
>     "minimist": "1.2.5"
>   },
Only in flip/: vue-loader-v16
```

## Final thoughts

But... why?
