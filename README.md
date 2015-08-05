A Pragmatic Guide to Testing
============================

The goal of this guide is to provide a few simple, easy to follow guidelines on how to write tests. Currently this document specifically covers testing of JavaScript projects, but most aspects should apply to other languages as well.

This guide is intended for readers that had *some* exposure to JavaScript before. It features some rather opinionated standpoints and seasoned developers *might want to take it with a grain of salt*.

Additions and other perspectives are welcome and everyone is encouraged to contribute to this guide.

Table of Contents
-----------------

- Introduction
- Testing Stacks
- Unit tests, integration tests, BDD, TDD
- Writing our first test case
- Structuring Tests
- Golden Rules of Testings
- Further Reading

Introduction
------------

Testing is essential for building reliable, maintainable modules. On the other hand, testing isn't "fun" and might take longer than implementing the business logic itself, which can be annoying at times.

Even worse, testing the "wrong" things can actually slow down development processes and make large-scale refactors and code polishing harder.

The goal therefore shouldn't be to have "100 % test coverage" or other hard numbers. You should aim for testing functionality on a level "that makes sense". Testing internal implementations might make applications more correct, but doesn't necessarily make code quality better on a longer term (who would want to rewrite all those tests after all?).

This guide aims to establish some conventions on how to write "useful" tests in a highly-dyncamic language that allows you to do all kinds of crazy stuff.

Testing Stacks
--------------

There is a multitude of different testing frameworks being used for writing unit tests. The following table lists a couple of them. It doesn't claim to be complete, but gives a rough overview of the available options.

In this guide we will be using a library called [tape](https://github.com/substack/tape) for sample code.

In contrast to more feature-rich testing frameworks, like Jasmine, tap(e) doesn't provide a lot of bells and whistles. It does not include a feature-rich DSL that allows you to easily de-duplicate setup logic, such as `beforeEach` methods. Instead it sacrifices those little convenient "nice-to-have" features for simplicity and modularity, which is perfect for a tutorial or when writing small, reusable modules.

| Name | Philosophy | Author | Used by | Client-side | Server-side | Stars
|-|
| [Mocha](http://mochajs.org/) | feature-rich | [tj](https://github.com/tj) | [express](https://github.com/strongloop/express/tree/master/test) | yes | yes | 7,132
| [Jasmine](http://jasmine.github.io/) | DOM-less | [Pivotal Labs](https://github.com/pivotal) | [Diaspora](https://github.com/diaspora/diaspora), [...](https://github.com/jasmine/jasmine/wiki/Who's-using-jasmine) | yes | yes |  9,229
| [QUnit](https://qunitjs.com/) | Ease of use | [jQuery](https://jquery.com/) |  [jQuery](https://github.com/jquery/jquery/tree/master/test), [jQuery Mobile](https://github.com/jquery/jquery-mobile) | yes | yes* | 3,226
| [tape](https://www.npmjs.com/package/tape) | [TAP](https://testanything.org/), modular | [substack](https://github.com/substack) | [isomorphic modules](https://www.npmjs.com/browse/depended/tape) | yes | yes | 1,059
| [tap](https://www.npmjs.com/package/tap) | [TAP](https://testanything.org/), modular | [isaacs](https://github.com/isaacs) | [npm](https://github.com/npm/npm/tree/master/test/tap), [browserify](https://github.com/substack/node-browserify/tree/master/test) | no | yes | 516

Stars are not a good approximation for judging the quality of open-source projects, but provide some insight into the popularity and overall usage.

Unit tests, integration tests, BDD, TDD
--------------------------------------

In an ideal world, everything should be tested. In fact, a lot of people are of the opinion that you should write your tests before actually implementing your solution. This idea has been around for a long, long time, but now people gave it fancy names, like [TDD](https://en.wikipedia.org/wiki/Test-driven_development) or [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development).

Also some people claim there is a big difference between the two, there really isn't. BDD is basically TDD "done right". When writing your tests before actually writing "real" code, you're typically tempted to write tests that are too specific, in the sense of testing internals instead of behavior. This makes refactoring despite the large number of available tests hard and even changes to the tests themselves might be required. When people realized that, they basically came up with this fancy term called "BDD". The "B" stands for behavior, explicitly emphasizing the lack of **internal** tests. Internal tests are hard to maintain due to their implementation specific nature.

In general, when talking about tests, the following concepts come up over and over again:

### TDD

> Test-driven development

As the name suggests, the tests are an integral part of the development process (they "drive" it). Most of the time this simply means writing tests before implementing business logic.

### BDD

> Behavior-driven development

Truth being told, everyone is confused about TDD vs. BDD. Reason being that there is no big difference (if any).

Here are some excerpts on what difference people think the difference is:

> That's wrong. BDD and TDD have absolutely nothing whatsoever to do with testing. None. Nada. Zilch. Zip. Nix. Not in the slightest.
> BDD is just TDD with different words. If you do TDD right, you are doing BDD. The difference is that – provided you believe at least in the weak form of the Sapir-Whorf Hypothesis – the different words make it easier to do it right.

- http://stackoverflow.com/a/4396952

> BDD is from customers point of view and focuses on excpected behavior of the whole system. TDD is from developpers point of view and focuses on the implementation of one unit/class/feature. It benefits among others from better architecture (Design for testability, less coupling between modules).

- http://stackoverflow.com/a/4396118

> I understand BDD to be more about specification than testing. It is linked to Domain Driven Design (don't you love these *DD acronyms?).

- http://stackoverflow.com/a/2548

**TL;DR** BDD is TDD done right.

### Unit tests

Unit tests cover the functionality of some encapsulated, independent module of your application, e.g. a function or a class.

An example unit test might look something like this:

```js
  test('sum function', function (t) {
    t.equal(typeof sum, 'function', 'sum should be a function')
    t.equal(sum(1, 2), 3)
    t.equal(sum(2, 1), 3)
    t.equal(sum(-1, 1), 0)
    t.end()
  })
```

### Integration tests

Ok, so you tested all your individual classes, modules and functions. Now you're typically glueing your application together. This brings its own challenge with it: Testing how the modules **integrate** with each other. While unit tests cover individual parts of your application, integration tests ensure that the independent modules work correctly "with" each other. This might involve testing in a browser environment.

### Visual Regression Tests

Visual regression tests are a kind of integration test. They ensure that your application doesn't "look" broken. They are useful for ensuring cross-browser compatibility, but are hard to maintain. Automated visual tests aren't *that* wide-spread, but are definitely useful.

Writing our first test case
---------------------------

Since it's oftentimes almost impossible to come up with tests before playing with the problem at hand with some sample code, I usually just write some very simple tests. E.g. testing if a function returns an object.

I start out with an empty test and source file, e.g. `widget_spec.js`:

```js
var test = require('tape')
var Widget = require('../widget')

test('Widget', function (t) {
  t.end()
})
```

When writing server-side tests, I usually now use [`nodemon`](https://www.npmjs.com/package/nodemon), when running tests in a browser environment, I tend to use [`hihat`](https://www.npmjs.com/package/hihat). Both solutions auto-run your test suite, which adds up after some time.

```
  hihat widget_test.js
```

Now we want out widget to have a method called `appendTo`. The `appendTo` method should append the widget to the passed in Element. Writing a test for this is trivial:

```js
test('Widget', function (t) {
  // tape allows you to nest your tests, pretty cool, huh?
  t.test('Widget#appendTo', function (t) {
    var widget = new Widget()

    t.equal(typeof widget.appendTo, 'function', 'widget should have an appendTo function')


    var container = document.createElement('div')
    t.equal(container.children.length, 0, 'container should be empty')

    widget.appendTo(container)
    t.equal(container.children.length, 1, 'widget.appendTo should have added an element to the container')

    // Some more assertions here
    t.end()
  })
})
```

Now while this is typically a good way to start, we can do better. One of the issues with the test above is that we don't know if the test does anything else to the container element. E.g. maybe it adds a red border or does something else to it other than appending itself.

In the above case we could simply mock the container. Oftentimes a mocking framework like [Sinon.JS](http://sinonjs.org/) is quite handy in such cases, but in this case our mock container is very simply and we can write the mock manually:

```js
test('Widget', function (t) {
  t.test('Widget#appendTo', function (t) {
    t.plan(3)

    var widget = new Widget()

    t.equal(typeof widget.appendTo, 'function', 'widget should have an appendTo function')

    var container = {
      appendChild: function (child) {
        t.pass('widget.appendTo should add itself as a child of the passed in container')
        t.equal(child, container.$el, 'widget.appendTo should append container element to container')
      }
    }

    widget.appendTo(container)
  })
})
```

As you can see above, we added a couple of insertions in the container's appendChild function. We check if the child that is being appended is the widget's element (`widget.$el`).

Another change worth being mentioned is that we are now no longer using `t.end`. Since our test is asynchronous, the container might not append itself immediately, but might wait some time. In that case ending the test before appending the widget would be problematic.

In order to avoid such a case, we "plan" beforehand how many assertions we want to make (`t.plan(3)`). Our test will be ended as soon as tape encountered three assertions, which is the case right after the container's `appendChild` method has been invoked.

Structuring Tests
-----------------

### Directory structure

Ok, so now we wrote our first test and we are happy with it, but there is kind of like a administrative questions: Where should we put all those test file?

There are a couple of options:

1. Putting them into the same dir as the file being tested, e.g. as `widget_test.js` of `widget.spec.js`

2. Putting them into a separate `__test__`, `test` or `spec` dir at the root level of your application (and therefore replicating your directory structure) or in each subdirectory

None of those options is superior to the others. Just be consistent and use globally unique file names, e.g. `test/widget_test.js` instead of `test/widget.js`.

### Structuring Test Cases

The guidance on how to structure tests is TAP-specific, but some points might apply to other testing stacks as well.

[tape](https://github.com/substack/tape) allows you to use nested tests. It's usually a good idea to replicate the signature of the module or class you're testing, e.g. let's say you have a `Bomb` class:

```js
class Bomb {
  constructor () {
    this.exploded = false
  }
  explode () {
    if (this.exploded) {
      throw new Error('Bomb already exploded!')
    }
    this.exploded = true
    console.log('BOOOOOMMMMM')
  }
}
```

Your test might look something like this:

```js
var test = require('tape')
var Bomb = require('./Bomb')

test('Bomb', function (t) {
  t.test('constructor', function (t) {
    t.equal(typeof Bomb, 'function')
    t.doesNotThrow(function () {
      new Bomb()
    })
    t.end()
  })

  t.test('explode', function (t) {
    var bomb = new Bomb()
    t.equal(typeof bomb.explode, 'function')
    t.doesNotThrow(function () {
      bomb.explode()
    })

    t.equla(bomb.exploded, true)
    t.throws(function () {
      bomb.explode()
    })
    t.end()
  })
})
```

Usually it's a good idea to start with "easy" tests, e.g. assertions that check if a function exists and returns an object without throwing an error.

Golden Rules of Testings
------------------------

* Never test internals

  Testing implementation instead of behavior/ functionality leads to unmaintainable code and realistically simply leads to redundant implementations. Don't assert the state of private properties, but concentrate on public APIs of possibly internal modules. E.g. it's perfectly fine to test a utility module that you are using internally, but it is not a good idea to concentrate the inner workings of a function.

  E.g. when testing the function `isPrime`, your test case should not be concerned with how the function actually works, instead it should concentrate on relevant test cases.

* One test case per bug

  Whenever you encounter a bug, don't tear apart your implementation right away. Instead add test cases for the relevant sections of your code that emulate the environment the bug has been encountered in.

  As soon as you tracked down and isolated the bug, add a test case for the specific bug at hand. Add a comment to the relevant issue (e.g. via test comments or source comments).

  That way you automatically avoid regressions and are less likely to encounter the same bug again.

* Start at the surface

  When adding test cases, try to start at the surface. Don't test the most complicated corner case right away, instead try to order your assertions and test cases by depth. First test if a class has a method. Then check if the method returns something roughly reasonable. Then check if the returned result is *exactly* the expected object.

  That way you can better isolate possible bugs. It's not useful to know that all tests in your `util` module failed, but it's useful to know that the `isPrime` function actually doesn't exist at all (maybe it has been renamed).

* Make your tests deterministic

  Don't randomize input. Concentrate on a couple of a bunch of relevant corner cases instead. One of the most important characteristic of a good test suite is that it's predictable. Tests should **always** be reproducible, otherwise there is no way you can effectively isolate a bug.

* No global state

  Global state is extremely hard to manage and leads to unpredictable tests. Oftentimes it's hard to write good tests because the code itself is hardly testable. Global state is oftentimes hard to avoid, especially in web applications due to the nature of the DOM itself.

  Nevertheless there are certain design decisions that lead to arguably better and more testable code:

    * Avoiding singletons
    * Avoiding global state
    * Encouraging decoupled modules

* Bad tests are better than no tests at all

  It's important to keep in mind that bad tests are **always** better then no tests at all. Of course this doesn't mean that you should write bad tests...

* Quality over Quantity

  In an ideal world, you should have a large quantity of quality tests. Unfortunately this is hardly the case. When writing good test cases that cover the most complicated corner cases, it's oftentimes tempting to remove those "dumb tests". Don't do it. As long as they aren't actively getting in your way, keep them as long as you don't have a better alternative. Sometimes you need simple test cases to track down simple, but far-reaching bugs.

Further Reading
---------------

* http://www.macwright.org/2014/03/11/tape-is-cool.html
* https://gist.github.com/substack/7480813
