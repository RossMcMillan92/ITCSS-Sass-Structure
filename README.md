# STV Sass
Based on Harry Roberts' concept ITCSS. For a complete overview, watch [this talk](https://www.youtube.com/watch?v=1OKZOV-iLj4) and [read these slides](https://speakerdeck.com/dafed/managing-css-projects-with-itcss). This git serves as a working example, however it's not perfect and hasn't been adapted to work with the current STV codebase. I talk about [adapting this structure](#adapting-this-structure-for-stv) later in this readme. Along with a basic introduction to ITCSS, this readme will also contain a few general best practices for working with Sass/css.

*A lot of things discussed here will be common sense and may sound condescending to some, but it's important to write down just to make sure we're all on the same page. There are also some opinionated points which may need to be discussed and ironed out by the team before the system is considered.*

## Contents
1. [Overview](#overview)
1. [1. Settings](#1-settings)
  - Variable names
  - Variable units
1. [2. Tools](#2-tools)
1. [3. Generic](#3-generic)
1. [4. Base](#4-base)
1. [5. Objects](#5-objects)
1. [6. Components](#6-components)
1. [7. Trumps](#7-trumps)
1. [Example](#example)
1. [Adapting this structure for STV](#adapting-this-structure-for-stv)

## Overview
ITCSS is a certain way of structuring Sass (or css) files to minimise rewriting/undoing code, and to maximise scalability. IT stands for Inverted Triangle, which is the basis of the code structure. In our master Sass file, rules which broadly affect elements on the page are imported at the top. As specificity grows, the further down the file the rules will be imported. 

![Specificity triangle](http://i.imgur.com/okdOFdK.png)

At the top, far-reaching rules (unclassed selectors) e.g.
```
body { font-family: Helvetica, Arial, sans-serif }
```

Further down the list, less specific rules will come into place (classed selectors) e.g.
```
.component { background-color: red }
```

And at the bottom of the list, very specific rules are placed (classed selectors, !important can be used) e.g.
```
.bg--alpha { background-color: green }
```

By keeping this top-down structure, rewriting and undoing css will be minimised as the order which the rules are placed will dictate specificity. Use of !important should be unnecessary, other than perhaps in helper files at the _very bottom_ of the list.

The full Sass structure looks like the following:

1. **Settings**   	- Variables, config.
1. **Tools** 		    - Mixins, functions
1. **Generic** 	  	- Normalize, reset, * {}
1. **Base** 		    - Unclassed HTML rules
1. **Objects** 	  	- Cosemetic-free design patterns
1. **Components** 	- Chunks of UI
1. **Trumps**   		- Helpers and overrides

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
Keep variable names ambiguous to prevent refactoring code in the future.

##### Bad
```
// 1-settings/pallete.scss
$red: #ff2134; // Sites primary theme color
// ...

// 6-components/puff.scss
.puff{
  font-color: $red;
}
```
If we want to change the sites primary theme colour to green, we'll need to change it in the settings file, as well as the components file (and all other files it's used in.)

##### Good
```
// 1-settings/pallete.scss
$red: #ff2134; // Use concrete names to describe variables that won't change
$palette--primary: $red; // Use ambiguous names for variables that may change, and that are used throughout the site
// ...

// 6-components/puff.scss
$puff-font-color: $palette--primary !default;
.puff{
  font-color: $puff-font-color;
}
```
This way may seem more complex than the first, but it means if we want to change the sites primary theme colour to green we'll only need to change it once within the settings file. This gives us a lot more flexibility.

#### Variable units
Another point to make here is all sizes should be stated in px, but later converted to em/rem with a Sass function. e.g.
```
// 1-settings/content-structure.scss
$base-spacing-unit: 20px;
// ...

// 6-components/puff.scss
$puff-margin-bottom: $base-spacing-unit !default;

.puff{
  margin-bottom: rem($puff-margin-bottom);
}
.puff--small{
  margin-bottom: rem($puff-margin-bottom / 2);
}
```
Using pixels allows us to match designs with greater detail and will keep any math as simple as possible. Converting to em/rem when it's finally needed will keep the sites responsive and accessible.

## 2. Tools
Self explanatory; keep any mixins/functions in here. High up in the list as these tools will be used throughout the rest of the Sass.

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
Site-wide rules should be set here. A general rule here is there should only be tag selectors, no class selectors.

Example Structure:
```
4-base
  _fonts.scss
  _global.scss
  _headings.scss
```

Example rules: 
```
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
Reusable objects should be placed here, generally layout structures. **No cosmetic styling** should go in here. When looking through a design, try to spot layout patterns being reused (even if they don't look the same aesthetically) and build them here. Modify the look of these seperately later in the components section (see [my example](#example) below). 

Example Structure:
```
5-objects
  _grids.scss
  _media.scss
  _slider.scss
```

Example rules: 
```
// _media.scss
.media{
    @extend .cf;
    display:block;
}
    .media__img{
        float:left;
        margin-right:rem($base-spacing-unit);

        &.is-half-spaced{
            margin-right:rem($base-spacing-unit / 2);
        }
    }
    
    .media__body{
        overflow:hidden;
    }
```
Since we will be reusing certain components/widgets across many sites with different themes, declaring any cosmetic or unnecessary styles here will inevitably be overwritten, causing waste. To avoid this, use this section to make minimal structures that generally won't change and can be extended across multiple widgets/components/sections. These can be further styled to suit the specific component later in a new file in the components section.

In theory, once an object is made here, it should rarely need edited again.


## 6. Components
Chunks of UI styling go here. This section may grow quite large depending on the complexity of the site, therefore subfolders may be used to group certain files (**Note:** Caution should be exercised here as components should be modular enough to be reused throughout the site. Don't store something in a 'header' folder if it can also be used in the footer). A 'templates' folder may also be used to seperate page styling from other modules. 

Example Structure:
```
6-components
  _breadcrumbs.scss
  header
    _banner.scss
    _header.scss
  _puff.scss
  templates
    _contact-page.scss
```

**Note:** Components may depend on objects to complete the style, but two components should *rarely* rely on each other as they should be kept as completely seperate entities. If you find that you can extend one component to create a new but similar one, consider refactoring the component into an object which can be reused and later embellished in the component. Alternatively, try combining the components into one and using modifier flags ('.component**--alt-style**') to differentiate.

##### Bad
```
// 6-components/header-navbar.scss
.header-navbar {
  height: 60px;
  padding: 10px;
  background-color: red;
  // ...
}

// 6-components/footer-navbar.scss
.footer-navbar {
  @extend .header-navbar;
  background-color: green;
}
```

##### Good
```
// 6-components/navbar.scss
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

##### Better
```
// 5-objects/_navbar.scss
.navbar {
  height: 60px;
  padding: 10px;
  // ...
}

// 6-components/_header-nav.scss
.header-nav {
  background-color: red
}

// 6-components/_footer-nav.scss
.footer-nav {
  background-color: green;
}

// index.html
<div class="header-nav navbar">
  <!-- ... -->
</div>

<div class="footer-nav navbar">
  <!-- ... -->
</div>
```

One thing to note about the 'Good' example is that adding modifiers can quickly build up and make a single component overly complex. If this looks like it could happen, it may be best splitting the component up into multiple components like in the 'Better' example.

Both of the above examples work well because no component depends on another. This is not to say components can't be nested, but a single element shouldn't have two different components classes assigned to it. 

## 7. Trumps
This is the highest specificity section, generally used for helper classes. It's fine to used !important in here, although if the rest of the Sass has been properly following the ITCSS structure, it should be unnecessary.

Example Structure:
```
7-trumps
  _helper.scss
  _palette.scss
```

Example rules: 
```
// _palette.scss
.bg--alpha {
  background-color: $palette--primary;
}

.txt--beta {
  color: $palette--secondary;
}
```
Since this file is at the very bottom of the list, adding the '.bg--alpha' class to an element will 'trump' any previously set background-color. (Unless previous rules have a higher specificity, which is why we should strive to use only single classes with a naming convention like BEM. If higher specificity is necessary and we still need to use a trump class, we can safely add !important to the trump rules as there isn't much scope for problems this late on in the file.)

## Example

```
// index.html
<div class="hor-list breadcrumbs">
  <div class="hor-list__item breadcrumbs__item">Link 1</div>
  <div class="hor-list__item breadcrumbs__item">Link 2</div>
  <div class="hor-list__item breadcrumbs__item">Link 3</div>
</div>

// 5-objects/lists.scss
.list-hor{
    padding: 0;
    margin: 0;
    list-style: none;
}
    .list-hor__item{
        display: inline-block;
    }
    
// 6-components/breadcumbs.scss
.breadcrumb{
	background-color: #000;
	width: 100%;
	padding: rem($base-spacing-unit / 2) rem($base-spacing-unit);
}
	.breadcrumb__item{
		font-size: rem(11px);
		text-transform: uppercase;
		color: #fff;
		margin-right: rem($base-spacing-unit / 2);

		&:after{
			position: absolute;
			content: "|";
			color: $palette--primary;
			margin-left: rem($base-spacing-unit / 2);
		}
	}
```

Note that '.list-hor' has no cosmetic styling (color, font-size etc), while '.breadcrumb' does. This allows us to reuse '.list-hor' without having to unnecessarily *undo* any of it's rules. Also note that the padding on '.list-hor' is being overwritten by the '.breadcrumb' class. Since our component comes after the object in the master Sass file, the '.breadcrumb' rules will overwrite the '.list-hor' rules without any hassle with specificity (i.e. no need for !important).

#### Avoiding @extend
In the above example you'll notice I entered two classes in the html markup, 'list-hor' and 'breadcrumb'. This could have been achieved by entering only the 'breadcrumb' and then extending the 'list-hor' class within Sass. e.g.

```
.breadcrumb {
  @extend .list-hor;
  background-color: #000;
  //  ...
}
```

There are multiple reasons to avoid this:

1. By extending .list-hor, the .breadcrumb class is being hoisted up next to .list-hor...
  ```
  // compiled css
  .list-hor, .breadcrumbs {/*...*/}
  ```
  This has just broken the ITCSS structure as we now have a component class mixed in with an object class. While this won't be an issue 95% of the time, a complex piece of code may cause specificity issues in later parts of the site. Extending has no real advantages other than arguably being more semantic. If semantics is an issue, an html comment can be left in the markup to explain the used classes.
  
2. Extending a class with nested rules can create lots of unnecessary code. ![poorly compiled css](https://pbs.twimg.com/media/B8mlqv_CUAAi7Qg.png:large) Nesting in general should be avoided as much as possible, but if it is necessary, **never** extend it to another class.

#### Use of classes
Classes should be used to style every element where possible. Sass makes it very easy to nest rules within each other, causing unnecessary specificity. Avoid this unless you have no control over the markup.

##### Bad
```
.sidebar-nav{
  // styling
  
  li{
    // styling
  }
}
```
##### Good
```
.sidebar-nav{
  // styling
}
  .sidebar-nav__item{
    // styling
  }
```

If you must nest tag selectors within a class, try not to be too broad with your selectors, like the following:
```
.header{
  a {
    color: red;
  }
}
```
The rule above will affect *all links in the header*. Can you guarantee they should all be red? If not, you will now have to write more code to change the other links back possibly causing higher specificity.

## Adapting this structure for STV
The example code contained within this git works well for a single site where the whole Sass codebase can be kept within the same folder. Since the STV code base will have files spread out over a global folder, widgets folders and the individual sites folder, we need to make some adaptations. I propose heavily using !default variables when creating widgets to maximise reusability and minimise undoing/waste. e.g.

```
// core.stv.tv/public/assets/source/sites/emmerdale/6-components/epsiode-list.scss

$episode-list-item-bg-color: red;
$episode-list-item-font-size: 18px;

@import '#{$widgets_path}/epsiode-list';

// overwriting/extending styles 
.epsiode-list__item{
  float: right;
}

```
The '#{$widgets_path}/epsiode-list.scss' file would look like: 
```
$episode-list-item-bg-color: blue !default;
$episode-list-item-font-size: 16px !default;
// ...

.epsiode-list__item{
  background-color: $episode-list-item-bg-color;
  font-size: rem($episode-list-item-font-size);
  float: left;
  // ...
}

```

To clarify, our local sites widget scss file would take on the following structure:
- Settings relevant to the sites theme
- Import the base widget style, which contains !default variables
- If necessary, list additional or overwritten rules for the widget

**Note:** Using global/widget files would mean creating a standard file and almost never editing it again, as it will have a knock-on effect on multiple sites. Any changes to a rule in a global/widget file for a new site should be made by using the process above, and not directly to the original file. If the new site project is the first to use a certain widget, create it in the widgets folder so it can be reused, and link to it in the sites file like above.
