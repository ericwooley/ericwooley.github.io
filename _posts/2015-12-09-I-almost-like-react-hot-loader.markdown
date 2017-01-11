---
layout:     post
title:      "I almost like React Hot Loader"
date:       2015-12-09 11:45:45
categories: react react-hot-loader build-systems webpack
comments:   true
published:  false

---
If you haven't seen it yet, checkout [react hot loader](https://github.com/gaearon/react-hot-loader).
It's a great project which allows you to develop in react, and syncs your react code (as you edit
it) with what the browser is running. The result is that your state and props stay the same in the
browser, as you edit your code. This is a really cool concept. I have used it in several projects
now, and I find that I really want to like it, but I just can not justify using it.

____________________________________________________________________________________________________

## Browserify vs Webpack

If you are like me, you like to customize your project setups to use exactly the tech you like, and
I like browserify. React hot loader only works with Webpack. My main issue with webpack is it
encourages you to things like:

```
import 'test.scss'
```

which completely ties you to webpack.

<img src="http://i.imgur.com/x6AnRkL.png?fb" />

If you want css files specific to that component, apply them with javascript.

## Best Practices
React Hot Loader is great for when you want to hack things together, and develop quickly. But if you
have a reasonably sized project, you should be using test driven development, and escape hack mode.

Surprisingly, having setup a whole project with react hot loader, I realized that I didn't really
use it, most of my development is done by setting up a watch task and running my unit tests every
time I change a file.

## React hot loader vs Unit tests
React hot loader seems like a great idea to maintain a simple buggy state, and develop the component
until the state is no longer an issue. Why not use a unit test instead? If you write the unit test
for that state, you get the same benefits, and it will be checked every time you change the module
in the future.

## Dependency hell
As of this writing, react hot loader doesn't support babel 6, so now I have to choose between using
old versions of plugins that work with babel5 or abandon react hot loader...


## What about checking how it looks?
Checking how it looks shouldn't require an incredible amount of editing the component. Sure, the hot
loader might save you a small amount time when you come across this use case, but it is worth it? I
don't think so, I'll stick with unit tests. Especially with plugins like [this](https://github.com/VictorBjelkholm/atom-react-preview) coming out.

____________________________________________________________________________________________________

There is a port of react hot loader available for browserify, but I haven't tried it yet because,
after trying webpack with react hot loader, I discovered I didn't like React hot Loader.
