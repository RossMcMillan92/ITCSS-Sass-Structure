# STV-Sass
Based on Harry Roberts' concept ITCSS. For a complete overview, watch [this talk](https://www.youtube.com/watch?v=1OKZOV-iLj4) and [read the slides](https://speakerdeck.com/dafed/managing-css-projects-with-itcss). This git serves as a working example, however it's not perfect.

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
Reusable/extendable objects should be placed here, generally layout structures. **No cosmetic styling** should go in here.

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


