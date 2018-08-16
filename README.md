# Sass Structure
Based on Harry Roberts' concept ITCSS. For a complete overview, watch [this talk](https://www.youtube.com/watch?v=1OKZOV-iLj4) and [read these slides](https://speakerdeck.com/dafed/managing-css-projects-with-itcss). The files in this git serve as a working example, however they aren't perfect and haven't been adapted to work with the current STV codebase. 

Along with a basic introduction to ITCSS, this readme will also contain a few general best practices/methods for working with Sass.

*A lot of things discussed here will be common sense to some, but it's important to write down just to make sure we're all on the same page. There are also some opinionated points which may need to be discussed and ironed out by the team before the system is considered.*

## Contents
[Overview](#overview)

[1. Settings](#1-settings)
  - Variable names
  - Variable units
  
[2. Tools](#2-tools)

[3. Generic](#3-generic)

[4. Base](#4-base)

[5. Objects](#5-objects)

[6. Components](#6-components)

[7. Utilities](#7-utilities)

[Avoiding @extend](#avoiding-extend)

[Use of classes](#use-of-classes)
 

## Overview
ITCSS is a certain way of structuring Sass files to minimise rewriting/undoing code, and to maximise scalability. IT stands for Inverted Triangle, which is the basis of the code structure. In our master Sass file, rules which broadly affect elements on the page are imported at the top. As specificity grows, the further down the file the rules will be imported. 

![Specificity triangle](http://i.imgur.com/okdOFdK.png)

At the top, far-reaching rules (unclassed selectors) e.g.
``` sass
body { font-family: Helvetica, Arial, sans-serif }
```

Further down the list, less specific rules will come into place (classed selectors) e.g.
``` sass
.component { background-color: red; padding: 10px; }
```

And at the bottom of the list, very specific rules are placed (classed selectors, !important can be used) e.g.
``` sass
.bg-alpha { background-color: green !important }
```

By keeping this top-down structure, rewriting and undoing css will be minimised as the order which the rules are placed will dictate specificity. Use of !important should be unnecessary, other than perhaps in helper files at the _very bottom_ of the list.

The full Sass structure looks like the following:

1. **Settings**   	- Variables, config.
1. **Tools** 		    - Mixins, functions
1. **Generic** 	  	- Normalize, reset, * {}
1. **Base** 		    - Unclassed HTML rules
1. **Objects** 	  	- Cosemetic-free design patterns
1. **Components** 	- Chunks of UI
1. **Utilities**   		- Helpers and overrides

## 1. Settings
Site-wide variables such as margin sizes, color schemes, font families/sizes etc. should go in here. There should be minimal 'Magic Numbers' throughout the code. **Note:** These settings are for site-wide variables. Component specific settings should go at the top of the component's file.

Example Structure:
``` 
1-settings
  _breakpoints.scss
  _fonts.scss
  _palette.scss
```

#### Variable names
When declared in the `settings` layer, variable names should have descriptive names in kebab case:

```scss
$base-spacing-unit: 20px;
$palette-alpha: red;
```

When declared in components, it is helpful to manually scope these variables to the component. We can do this by using a BEM-like naming scheme:

```scss
// components/modal.scss
$modal__max-width: 300px;
$modal__background-color: $palette-alpha;
```
By doing this, we circumvent (to an extent) the global nature of css. **Never** Use a component variable outside of it's own component file. If you need shared variables between components, put the variable in the `settings` layer

#### Bad
``` scss
// components/button-primary.scss
$button-primary__bg-color: blue;

// components/modal.scss
.modal__cancel-button {
  background-color: $button-primary__bg-color;
}
```

Now these two components rely on each other existing, meaning they are no longer orthogonal. Import order now matters, and if we remove `button-primary.scss` then `modal.scss` will break.

Instead move the common variables to the `settings` layer:

#### Good
``` scss
// settings/palette.scss
$primary-button-bg-color: blue;

// components/button-primary.scss
$button-primary__bg-color: $primary-button-bg-color;

// components/modal.scss
.modal__cancel-button {
  background-color: $primary-button-bg-color;
}
```

The order of the component imports no longer matters, and if we remove `button-primary.scss` then `modal.scss` will no longer break.


## 2. Tools
Self explanatory; keep any mixins/functions in here. High up in the list as these tools will be used throughout the rest of the Sass codebase.

Example Structure:
``` 
2-tools
  _media-queries.scss
  _units.scss
```

## 3. Generic
Generally 'set and forget' type rules will go in here, e.g. normalize, clearfix, *{}

Example Structure:
``` 
3-generic
  _clearfix.scss
  _generic.scss
  _normalize.scss
```

## 4. Base
Site-wide rules should be set here. A general rule here is there should only be tag selectors, no class selectors (although exceptions can be made.)

Example Structure:
``` 
4-base
  _fonts.scss
  _global.scss
  _headings.scss
```

Example rules: 
``` sass
// 4-base/_global.scss
html{
	font-family: $base-font-family;
	font-size: $base-font-size;
	line-height: $base-line-height;

	min-height: 100%;
}

body{
    background-color: $palette--body-bg;
    -webkit-font-smoothing: antialiased;
}

// 4-base/_headings.scss
h1{
    font-size: $h1-size;
}
h2{
    font-size: $h2-size;
}
h3{
    font-size: $h3-size;
}

h1, h2, h3, h4, h5, h6 {
	line-height: $base-line-spacing--header;
}
```

## 5. Objects
Reusable objects should be placed here, generally layout structures. **No cosmetic styling** should go in here.

Example Structure:
``` 
5-objects
  _grids.scss
  _media.scss
  _slider.scss
```

Example rules: 
``` sass
// 5-objects/_media.scss
.media {
    display:block;
}
    .media__img {
        float:left;
        margin-right:rem($base-spacing-unit);
    }
    
    .media__img--half-spaced {
        margin-right:rem($base-spacing-unit / 2);
    }
    
    .media__body {
        overflow:hidden;
    }
```
Since we will be reusing certain components/widgets across many sites with different themes, declaring any cosmetic or unnecessary styles here will inevitably be overwritten, causing waste. To avoid this, use this section to make minimal structures that generally won't change and can be reused across multiple widgets/components/sections. These can be further styled to suit the specific component later in a new file in the components section. (see [my example](#example) below.) 

In theory, once an object is made here, it should rarely need edited again.


## 6. Components
The majority of the site's styling will go in here. This section may grow quite large depending on the complexity of the site. Keep this import list alphabetical: If import order starts to matter, then something in the structure has went wrong. 

Example Structure:
``` 
6-components
  _breadcrumbs.scss
  _banner.scss
  _header.scss
  _puff.scss
  templates
    _contact-page.scss
```

**Note:** Components may depend on objects to complete the style, but two components should *rarely* rely on each other as they should be kept as completely seperate entities. If you find that you can extend one component to create a new but similar one, try combining the components into a single one and using modifier flags ('.component**--alt-style**') to differentiate.

##### Bad
``` sass
// 6-components/_header-navbar.scss
.header-navbar {
  height: 60px;
  padding: 10px;
  background-color: red;
  // ...
}

// 6-components/_footer-navbar.scss
.footer-navbar {
  @extend .header-navbar;
  background-color: green;
}
```

##### Good
``` sass
// 6-components/_navbar.scss
.navbar {
  height: 60px;
  padding: 10px;
  // ...
}

.navbar--header {
  background-color: red
}

.navbar--footer {
  background-color: green;
}
```

##### Bad
``` sass
// 6-components/button.scss
.button {
	background-color: red;
}

// 6-components/sidebar.scss
.sidebar {
  button {
  	background-color: green;
  }
}

// index.html
<div class="sidebar">
  <button class="button">Press Me</button>
</div>
```

##### Good
``` sass
// 6-components/button.scss
.button {
	background-color: red;
}

button--secondary {
	background-color: green;
}

// index.html
<div class="sidebar">
  <button class="button button--secondary">Press Me</button>
</div>
```



## 7. Utilities
Rules that are added at the very end, generally used for helper classes. These should mostly be classes with a single rule. The rules should always have `!important` so they **always** overwrite previous class rules.

Example Structure:
``` 
// 7-utilities
  _helper.scss
  _palette.scss
```

Example rules: 
``` sass
// 7-utilities/_palette.scss
.bg-alpha {
  background-color: $palette--primary;
}

.txt-center {
  text-align: center;
}
```
Since this file is at the very bottom of the list, adding the '.bg-alpha' class to an element will trump any previously set background-color. While we should strive to keep specificity as low as possible, i.e. single class selectors via BEM, we will always have cases where nested classes are needed. To allow utilities to work on these, we need to add !important to the utilities' rules.

#### Avoiding @extend
[This article by Oliver Jash](http://oliverjash.me/2012/09/07/methods-for-modifying-objects-in-oocss.html) explains clearly the cons of using @extend instead of defering this functionality to the HTML. I'll highlight a few reasons not to use @extend.

My example above could have been achieved by entering only the 'breadcrumb' class in the markup and then extending the 'list-hor' class within Sass. e.g.

``` sass
.breadcrumb {
  @extend .list-hor;
  background-color: #000;
  //  ...
}
```

There are multiple reasons to avoid this:

1. By extending .list-hor, the .breadcrumb class is being hoisted up next to .list-hor...
  ``` sass
  // compiled css
  .list-hor, .breadcrumbs {/*...*/}
  ```
  We now have unrelated rules scattered across the compiled css file which has broken the IT structure, as we now have a component class mixed in with an object class. While this won't be an issue 90% of the time, a complex piece of code may cause unexpected issues elsewhere.
  
2. Extending a class with nested rules can create lots of unnecessary code. ![poorly compiled css](https://pbs.twimg.com/media/B8mlqv_CUAAi7Qg.png:large) Nesting in general should be avoided as much as possible, but if it is necessary, **never** extend it to another class. 

##### Use mixins instead of @extend
[Harry Roberts talks in depth about this here](http://csswizardry.com/2014/11/when-to-use-extend-when-to-use-a-mixin/#when-to-use-a-mixin) so I won't go into much detail. The take-home points are:
- Less specificity issues like I mentioned above.
- Gzip catches and compresses the 'wasted' code.
- While the compiled code isn't DRY, the source code is, and that's what matters.

#### Use of classes
Classes should be used to style every element where possible. Sass makes it very easy to nest rules within each other, causing unnecessary specificity. Avoid this unless you have no control over the markup.

##### Bad
``` sass
// 5-objects/_blocklist.scss
.block-list{
    padding: 0;
    margin: 0;
    list-style: none;
    
    li{
      display: inline-block;
      padding: 10px;
    }
}
```
The above code gives the 'li' element a high specificity. This means when I come to reuse that code for a component, I'll have to create more high spcificity rules, or use !important.

``` sass
// index.html
<ul class="nav block-list">
  <li class="nav__item">Item</li>
</ul>

// 6-components/_nav.scss
.nav {
  // ...
}
.nav__item {
	padding: 20px; // won't work
	padding: 20px !important; // will work but causes even higher specificity
}
```

Trying to overwrite the padding won't work because '.blocklist li' has a higher specificity than '.nav__item', therefore !important is needed.

##### Good
``` sass
// index.html
<ul class="nav block-list">
  <li class="nav__item block-list__item">Item</li>
</ul>

// 5-objects/_blocklist.scss
.block-list {
  padding: 0;
  margin: 0;
  list-style: none;
}
.block-list__item {
	display: inline-block;
	padding: 10px;
}
    
// 6-components/_nav.scss
.nav {
  // ...
}
.nav__item{
	padding: 20px; // Works!
}
```

The padding can now be changed because '.block-list__item' and '.nav__item' have the same specificity (i.e. one class). '.nav__item' will overwrite because it's included in the `components` layer further down in the compiled css.

If you don't have access to the markup and therefore must nest selectors to target elements, try not to be too broad with your selectors, like the following:

``` sass
.header{
  a {
    color: red;
  }
}
```
The rule above will affect *all links in the header*. Can you guarantee they should all be red? If not, you will now have to write more code to change the other links back possibly causing unnecessary waste and even more high specificity rules.
