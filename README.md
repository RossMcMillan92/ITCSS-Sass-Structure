# STV Sass
Based on Harry Roberts' concept ITCSS. For a complete overview, watch [this talk](https://www.youtube.com/watch?v=1OKZOV-iLj4) and [read the slides](https://speakerdeck.com/dafed/managing-css-projects-with-itcss). This git serves as a working example, however it's not perfect and certainly not adapted to work with the current STV codebase.

## Contents
1. [Overview](#overview)
1. [1. Settings](#1-settings)
1. [2. Tools](#2-tools)
1. [3. Generic](#3-generic)
1. [4. Base](#4-base)
1. [5. Objects](#5-objects)
1. [6. Components](#6-components)
1. [7. Trumps](#7-trumps)
1. [Example](#example)
1. [Adapting this structure for STV](#adapting-this-structure-for-stv)

## Overview
ITCSS is a certain way of structuring Sass (or css) files to minimise rewriting/undoing code, and to maximise scalability. IT stands for Inverted Triangle, which is the basis of the code structure. In our master sass file, rules which broadly affect elements on the page are imported at the top. As specificity grows, the further down the file the rules will be imported. 

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
Site-wide variables such as margin sizes, color schemes, font families/sizes etc. should go in here. There should be minimal 'Magic Numbers' throughout the code.

Example Structure:
```
1-settings
  _breakpoints.scss
  _fonts.scss
  _palette.scss
```

**Note:** Avoid explicitly naming variables to avoid refactoring code in the future. e.g.
#### Bad
```
// 1-settings/pallete.scss
$red: #ff2134;
...

// 6-components/puff.scss
.puff{
  font-color: $red;
}
```

#### Good
```
// 1-settings/pallete.scss
$red: #ff2134;
$palette--primary: $red;
...

// 6-components/puff.scss
.puff{
  font-color: $palette--primary;
}
```

Another point to make here is all sizes should be stated in px, but converted to em/rem when used in later components. e.g.
```
// 1-settings/content-structure.scss
$base-spacing-unit: 20px;
...

// 6-components/puff.scss
.puff{
  margin-bottom: rem($base-spacing-unit);
}
.puff--half{
  margin-bottom: rem($base-spacing-unit / 2);
}
```
Using pixels allows us to match designs with great detail and will keep math simple, while converting to em/rem allows us to maintain responsiveness and accessibility.

## 2. Tools
Keep any mixins/functions in here.

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
// _global.scss
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

// _headings.scss
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
Reusable/extendable objects should be placed here, generally layout structures. **No cosmetic styling** should go in here. When looking through a design, try to spot layout patterns being reused (even if they don't look the same aesthetically) and build them here. Modify the look of these seperately later in the components section (see [my example](#example) below). 

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
Since we will be reusing certain components/widgets across many sites with different themes, declaring any cosmetic or unnecessary styles here will inevitably be overwritten, causing waste. To avoid this, use this section to make minimal structures that generally won't change and can be extended across multiple widgets. These can be further styled to suit the specific component later in a new files in the components section.

In theory, once an object is made here, it should rarely need edited again.


## 6. Components
Chunks of UI styling go here. This section may grow quite large depending on the complexity of the site, therefore subfolders may be used to group certain files (**Note:** Caution should be exercised here as components should be modular enough to be reused throughout the site. Don't store something in a 'header' folder if it can also be used in the footer). A 'templates' folder may also be used to seperate page styling from other modules. 

Example Structure:
```
6-components
  _breadcrumbs.scss
  header
    _nav.scss
    _banner.scss
  _puff.scss
  templates
    _contact.scss
```

## 7. Trumps
This is the highest specificity section, generally used for helper classes. It's fine to used !important in here, although if the rest of the Sass has been properly following the ITCSS structure, it should be unnecessary.

Example Structure:
```
7-trumps
  _helper.scss
```

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
			color: nth($palette--primary, 1);
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
  This has just broken the ITCSS structure as we now have a component class mixed in with an object class. While this won't be an issue 95% of the time, a complex piece of code may cause specificity issues in later parts of the code. Extending has no real advantages other than arguably being more semantic. If semantics is an issue, leave an html comment in the markup clearly explaining your use of classes.
  
2. Extending a class with nested rules can and will lots of unnecessary code. ![poorly compiled css](https://pbs.twimg.com/media/B8mlqv_CUAAi7Qg.png:large) Nesting in general should be avoided as much as possible, but if it is necessary, **never** extend it to another class.

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

If you must nest tag selectors within a class, try to minimalise code which you'll have to undo later. e.g.
```
.header{
  a {
    color: red;
  }
}
```
With the above rule, all links within the .header element will be coloured red. Can you guarantee all links in there should be red? Are some of them green? If so, you will now have to write more code to set some of the other links to green, probably adding in more nested rules causing higher specificity.

## Adapting this structure for STV
The example code contained within this git works well for a single site where the whole sass codebase can be kept within the same folder. Since the STV code base will have files spread out over a global folder, widgets folders and the individual sites folder, we need to make some adaptations. I propose heavily using !default variables when creating widgets to maximise reusability and minimise undoing/waste. e.g.

```
// core.stv.tv/public/assets/source/sites/emmerdale/6-components/epsiode-list.scss

$episode-list-item-bg-color: #red;
$episode-list-item-font-size: 18px;

@import '#{$widgets_path}/epsiode-list';

// overwriting style that doesn't have a variable option
.epsiode-list__item{
  float: right;
}

```
The '#{$widgets_path}/epsiode-list.scss' file would look like: 
```
$episode-list-item-bg-color: #blue !default;
$episode-list-item-font-size: 16px !default;
//...

.epsiode-list__item{
  background-color: $episode-list-item-bg-color;
  font-size: rem($episode-list-item-font-size);
  float: left;
  //...
}

```

To clarify, our local sites widget scss file would take on the following structure:
- Settings relevant to the sites theme
- Import the base widget style, which contains !default variables
- If necessary, list additional or overwritten rules for the widget here.

#### Things to note (caveats)
Using global/widget files would mean creating a standard file and almost never editing it again, as it will have a knock-on effect on multiple sites. Any changes to a rule in a global/widget file should be made by using the process above, and not directly to the original file. If the new site project is the first to use a certain widget, create it in the widgets folder so it can be reused, and link to it in the sites file using the process above.
