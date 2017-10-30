Say we have an application which is compiled with babel, bundled with webpack and styled with SASS.

There are a couple of commands which we can run to compile this app:

 * Create a dist dir & copy any necessary html, images into it
 * Run webpack using the babel loader
 * Run SASS

We can definitely do this in gulp, grunt or npm scripts.  But consider Make.  People may understandably
think that Make is for old fogies or hardcore kernel developers who work with rough languages like C.
It's true Make was designed to build such codebases but really it's nothing more than a way of specifying
dependent tasks to build _anything_.

A Makefile is nothing more than a list of things you'd like to build, what they depend on and how to build it.

A Simple Makefile
-----------------

Here's a simple Makefile which declares the above tasks:

```make
js:
	webpack app/js/main.jsx dist/js/main.js

css:
	sass app/css/main.scss:dist/css/main.css
```

Here we have 2 things that we'd like to build: "js" and "css".  Let's call these targets.
Each thing has an action.  For 'js' it's run webpack; 'css' it's run sass.  We'll call these recipes.
Targets & recipes paired together are called "rules".  One thing that Make requires that we do is
use a TAB character to indent recipes. (this is configurable see later)

From the command line we can run:

 * `make js`; or
 * `make css`

et voila, we have a nice convenient way of running our scripts.

Specifying a Default
--------------------

If you run `make` without any targets it will run the _default goal_.  This will either simply be the first target found that doesn't start with a . or one specified explicitly with `..DEFAULT_GOAL`.

Simplifying the command
-----------------------

Running `make` without any arguments will run the first target found.  It's not very convenient to
only run js, we need to run both.

```make
dist: js css

js: 
	webpack app/js/main.jsx dist/js/main.js

css:
	sass app/css/main.scss:dist/css/main.css
```

We've added a new target at the top, 'dist', which defines js & css as "prerequisites", the final element of a rule.
From the command line we can now simply run `make` to build our app:

 * `make`

We can see our format is:

```make
thing we want built (target): other things that should be done first (prerequisites)
	actions that need to be done in order to build our thing (recipes)
```

You're allowed to have multiple targets, prerequisites & recipes.

Starting the build afresh
-------------------------

```make
dist: js css

js: prep
	webpack app/js/main.jsx dist/js/main.js

css: prep
	sass app/css/main.scss:dist/css/main.css

prep:
	mkdir -p dist/js && mkdir -p dist/css

clean:
	rm -rf dist
```

Let's optimise!
---------------
Don't build things unnecessarily.

```make
js: prep dist/js/main.js

dist/js/main.js: app/js/main.jsx
	webpack app/js/main.jsx dist/js/main.js

css: prep dist/css/main.css

dist/css/main.css: app/css/main.scss
	sass app/css/main.scss:dist/css/main.css
```

Dry our Makefile
----------------

 * $< the first prereq
 * $^ all prereqs
 * $@ the target

```make
js: prep dist/js/main.js

dist/js/main.js: app/js/main.jsx
	webpack $< $@

css: prep dist/css/main.css

dist/css/main.css: app/css/main.scss
	sass $<:$@
```

 * % pattern matching

```make
js: prep dist/js/main.js
css: prep dist/css/main.css

dist/js/%.js: app/js/%.jsx
	webpack $< $@

dist/css/%.css: app/css/%.scss
	sass $<:$@
```


