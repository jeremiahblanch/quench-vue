<p align="center">
  <img src="https://s21.postimg.org/7sorzuy5z/logo.png" height="200" alt="Quench Vue logo " />
  <div align="center">
    <a href="https://travis-ci.org/stowball/quench-vue">
      <img src="https://img.shields.io/travis/stowball/quench-vue/master.png?style=flat-square" alt="Travis build status" />
    </a>
    <a href="https://www.npmjs.org/package/quench-vue">
      <img src="https://img.shields.io/npm/v/quench-vue.png?style=flat-square" alt="npm package" />
    </a>
  </div>
</p>

# Quench Vue

**Simple, tiny, client-side hydration of pre-rendered [Vue.js](https://vuejs.org) apps**

Quench Vue allows server-rendered/static markup to be used as the `data` and `template` for a Vue.js app. It's great for when you can't/don't want to use "real" [server-side rendering](https://vuejs.org/v2/guide/ssr.html).

All of Vue's existing features will work as normal when the app is initialised in the browser.

## Demo

A complete demo is available here: https://codepen.io/stowball/pen/awwGBB

## Installation

### npm

```sh
npm install quench-vue --save
```

### Direct `<script>` include

```html
<script src="https://unpkg.com/quench-vue/umd/quench-vue.min.js"></script>
```

*Note: You will need to use [the full build of Vue.js](https://vuejs.org/v2/guide/installation.html#Explanation-of-Different-Builds), which includes the compiler.*

## Usage

There are 2 ways of defining and using `data` for the app:

1. With a stringified JSON object in the app container's `q-data` attribute; and/or
2. With an inline `q-binding` attribute on an element, when `q-convert-bindings` is added to the app container.

Both techniques can be used together or on their own, but the `q-data` is preferred as it's faster, simpler and more versatile.

Let's look at some examples:

### Method 1: Defining the `data` with `[q-data]`

This method allows you to easily specify the `data` for the app, including arrays and objects.

```html
<div id="app" q-data='
  "title": "Hello, world!",
  "year": 2017,
  "tags": [
    "js",
    "library"
  ],
  "author": {
    "firstName": "Matt",
    "lastName": "Stow"
  },
  "skills": [
    {
      "name": "JS",
      "level": 4
    },
    {
      "name": "CSS",
      "level": 5
    }
  ]
'>
…
</div>
```

#### Rendering the data with `[q-binding]` or `[v-text]`

We obviously duplicate the "data" in the markup, and inform Vue which elements are bound to which `data` properties using a `q-binding` or [`v-text`](https://vuejs.org/v2/api/#v-text) attribute whose value points to a property name, such as:

```html
<h1 q-binding="title">Hello, World!</h1>
<p q-binding="year">2017</p>

<ul>
  <li v-for="tag in tags">
    <span q-binding="tag">js</span>
  </li>
  <!-- <q> -->
  <li>
    <span>library</span>
  </li>
  <!-- </q> -->
</ul>

<ul>
  <li v-for="key in author">
    <span q-binding="key">Matt</span>
  </li>
  <!-- <q> -->
  <li>
    <span>Stow</span>
  </li>
  <!-- </q> -->
</ul>

<ul>
  <li v-for="skill in skills">
    <span q-binding="skill.name">JS</span>
    <span q-binding="skill.level">4</span>
  </li>
  <!-- <q> -->
  <li>
    <span>CSS</span>
    <span>5</span>
  </li>
  <!-- </q> -->
</ul>
```

For iterating over lists, we also need to use another syntax, `<!-- <q> --> … <!-- </q> -->`, which [we'll describe later](#hiding-elements-from-the-compiler).

*Note: You only need to output the `v-for` and the `q-binding`/`v-text` attributes on the first iteration of the loop.*

### Method 2: Defining the `data` with inline `[q-binding]` bindings

When `q-convert-bindings` is set on the app's container, we can also use the `[q-binding]` attribute to create a `data` variable that is equal to the value of the element's `.textContent`.

*Note:*
* *Bindings specified in the global `q-data` object take precedence over inline bindings.*
* *Do not nest elements inside a `q-binding` element, or you'll have unexpected results.*

The following examples all perfectly re-create the global `q-data` object from before.

#### Simple bindings

```html
<div id="app" q-convert-bindings>
  <h1 q-binding="title">Hello, World!</h1>
  <p q-binding="year">2017</p>
</div>
```

#### Array and Object bindings

Vue supports iterating over arrays and objects via [the `v-for` directive](https://vuejs.org/v2/guide/list.html) with the syntax `item in items`, where `items` is the source data list and `item` is an **alias** for the array element being iterated on.

To inline bind with Quench, we need to use another special syntax `itemsSource as item`.

##### Array

To replicate the `tags` array from above, we would:

```html
<div id="app" q-convert-bindings>
  <li v-for="tag in tags">
    <span q-binding="tags[0] as tag">js</span>
  </li>
  <!-- <q> -->
  <li>
    <span q-binding="tags[1] as tag">library</span>
  </li>
  <!-- </q> -->
</div>
```

where `itemsSource` is the name of the array (`tags`) plus the index in the array `[0]`/`[1]` which we wish to populate, and `tag` is the `item` alias in the `v-for`.

##### Object

To replicate the `author` object from above, we would:

```html
<div id="app" q-convert-bindings>
  <li v-for="key in author">
    <span q-binding="author.firstName as key">Matt</span>
  </li>
  <!-- <q> -->
  <li>
    <span q-binding="author.lastName as key">Stow</span>
  </li>
  <!-- </q> -->
</div>
```

where `itemsSource` is the name of the object (`author`) plus the relevant object key `.firstName`/`.lastName` which we wish to populate, and `key` is the `item` alias in the `v-for`.

##### Array of Objects

Both of the above techniques can be combined, so to replicate the `skills` array from above, we would:

```html
<div id="app" q-convert-bindings>
  <li v-for="skill in skills">
    <span q-binding="skills[0].name as skill.name">JS</span>
    <span q-binding="skills[0].level as skill.level">4</span>
  </li>
  <!-- <q> -->
  <li>
    <span q-binding="skills[1].name as skill.name">CSS</span>
    <span q-binding="skills[1].level as skill.level">5</span>
  </li>
  <!-- </q> -->
</div>
```

where `itemsSource` is the name of the array and index (`skills[0]`) plus the relevant object key `.name`/`.level` which we wish to populate, and `skill.name`/`skill.level` is the `item` alias in the `v-for` plus the object key.

Hopefully you'll agree that using inline bindings to set the `data` is more complicated than using the `q-data` method, but it can still have its uses.

*Note: When using inline bindings, arrays and objects are limited to a depth of 1 level.*

### Referencing global variables as `data` properties

You can also pass global variables (on `window`) to be used as data properties. Similarly to `[q-data]`, we can pass a stringified JSON object of key/value pairs to a `[q-r-data]` attribute, where *key* is the name of the `data`'s property and *value* the name of the global variable to be used, which can also use dot notation to access properties of an object.

```html
<script>
  var ENV = 'dev';
  var PORT = 3000;
  var obj = {
    foo: 'bar',
    baz: 'qux',
  }
</script>
<div id="app" q-r-data='{
  "env": "ENV",
  "port": "PORT",
  "foo": "obj.foo",
  "baz": "obj.qux"
}'></div>
```

which will produce the following `data`:

```js
{
  env: 'dev',
  port: 3000,
  foo: 'bar',
  baz: 'qux',
}
```

*Note: Bindings specified in the `q-r-data` object take precedence over those in `q-data`.*

### Hiding elements from the compiler

In the previous sections, we introduced the `<!-- <q> --> … <!-- </q> -->` syntax. These are a pair of opening and closing comments that exclude the contents within from being passed to the template compiler.

The most obvious use case (and necessary when using inline bindings) is to strip all but the first element of a `v-for` loop as demonstrated earlier.

Another use case is to replace static markup for a component, such as:

```html
<!-- <q> -->
<div>I will be stripped in the app and "replaced" with the component version below</div>
<!-- </q> -->
<my-component text="I'm not visible until parsed through the compiler"></my-component>
```

*Note: Nesting comments is not supported.*

#### Hiding elements in the pre-rendered HTML

To prevent layout jumping and repositioning when the app's template gets compiled, it can be beneficial to visually (and accessibly) hide elements and content that is inappropriate without JavaScript, such as a `<button>`.

By adding a class on your app container and an appropriate CSS rule, this can be achieved easily:

```html
<div id="app" class="pre-rendered">
  <button class="hide-when-pre-rendered" v-on:click="doSomething">I'm a button that only works with JS</button>
</div>
```

```css
.pre-rendered .hide-when-pre-rendered {
  visibility: hidden;
}
```

When Vue compiles our new template, it strips all of the container's attributes from the DOM, thus the class will no longer match.

### Instantiating the app

Very little needs to change from the way you'd normally instantiate an app.

#### With a module bundler, such as webpack

```js
import Vue from 'vue';
import { createAppData, createAppTemplate } from 'quench-vue';

var appEl = document.getElementById('app');
var data = createAppData(appEl);
var template = createAppTemplate(appEl);

var app = new Vue({
  el: appEl,
  data: data,
  template: template,
});
```

#### For direct `<script>` include

```html
<script src="https://unpkg.com/vue"></script>
<script src="https://unpkg.com/quench-vue/umd/quench-vue.min.js"></script>
```

```js
var appEl = document.getElementById('app');
var data = quenchVue.createAppData(appEl);
var template = quenchVue.createAppTemplate(appEl);

var app = new Vue({
  el: appEl,
  data: data,
  template: template,
});
```

## Benefits

Hopefully you've recognised that you're now able to render fast, SEO-friendly static markup (either from a CMS, static-site generator or component library such as [Fractal](http://fractal.build/)) and have it quickly and easily converted in to a fully dynamic, client-side Vue.js application, without having to set up more complicated server-side rendering processes.

---

Copyright (c) 2017 [Matt Stow](http://mattstow.com)  
Licensed under the MIT license *(see [LICENSE](https://github.com/stowball/quench-vue/blob/master/LICENSE) for details)*
