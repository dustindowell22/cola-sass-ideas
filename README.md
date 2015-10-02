# PreSass / ColaSass / Ideas

ColaSass will compile to vanilla Sass, so all of your existing Sass code is compatible.

ColaSass will use PostCSS for the syntax modifications and a vanilla Sass library for the behavioral modifications.

## nth

```scss
// Input ColaSass
@debug nth('string', 2);
@debug nth(s t r i n g, 2);

// Output Sass
@debug cola-nth('string', 2); // t
@debug cola-nth(s t r i n g, 2); // t

// Imported via ColaSass library
@function cola-index($value, $argument) {
  // ...
}
```

## index (str-index)

```scss
// Input ColaSass
@debug index(s t r i n g, 't');
@debug index('string', 't');

// Output Sass
@debug cola-index(s t r i n g, 't'); // 2
@debug cola-index('string', 't'); // 2

// Imported via ColaSass library
@function cola-index($value, $argument) {
  @if type-of($value) == string {
    @return str-index($value, $argument);
  }
  @return index($value, $argument);
}
```

## type-of

```scss
// Input ColaSass
@debug type-of(!important) == flag;
@debug type-of(border) == string;
@debug type-of(solid) == style;
@debug type-of(null) == null;
@debug type-of(em) == length;
@debug type-of(1) == number;

// Output Sass
@debug cola-type-of(!important) == flag; // true
@debug cola-type-of(border) == string; // true
@debug cola-type-of(solid) == style; // true
@debug cola-type-of(null) == null; // true
@debug cola-type-of(em) == length; // true
@debug cola-type-of(1) == number; // true

// Imported via ColaSass library
@function cola-type-of($value) {
  $styles: none hidden dotted dashed solid double groove ridge inset outset;
  $lengths: px em rem vw vh vmin vmax in cm mm pt pc ex ch;

  @if index($styles, $value) {
    @return style;
  }
  @else if index($lengths, $value) {
    @return length;
  }
  @else if $value == null {
    @return null;
  }
  @else if cola-index($value, '!') {
    @return flag;
  }

  @return type-of($value);
}
```

## Lists

Easier syntax for retrieving items from a list.

```scss
// Input ColaSass
@debug $list(6);

// Output Sass
@debug nth($list, 6);
```

Easier syntax plus added functionality of assigning/replacing a value in a list. If an index is specified that's longer than the list, the items in-between are set to null;

```scss
// Input ColaSass
$list: 1 2 3;
$list(6): 12;
@debug $list;

// Output Sass
$list: 1 2 3;
$list: cola-list($list, 6, 12);
@debug $list; // Output: 1 2 3 null null 12

// Imported via ColaSass library
@function cola-list($list, $index, $value) {
  // ...
}
```

## Maps

```scss
// Input ColaSass
@debug $map.key;
@debug $map('key');


// Output Sass
@debug map-get($map, key);
@debug map-get($map, 'key');

@function cola-map($map, $key, $value) {
  // ...
}
```

## Loops

```scss
// Input ColaSass
@each $item, $i in $list {
  // ...
}

// Output Sass
@for $i from 1 through length($list) {
  $item: nth($list, $i);

  // ...
}
```

## is and isnt

```scss
// Input ColaSass
@debug 1 is 2;
@debug 1 isnt 2;

// Output Sass
@debug 1 == 2;
@debug 1 != 2;
```

## Properties

```scss
red($variable)        =>  $variable.red
green($variable)      =>  $variable.green
blue($variable)       =>  $variable.blue

hue($variable)        =>  $variable.hue
saturation($variable) =>  $variable.saturation
lightness($variable)  =>  $variable.lightness

unquote($variable)    =>  $variable.unquote
quote($variable)      =>  $variable.quote

str-length($variable) => $variable.length
length($variable)     => $variable.length

etc...
```

## Lazy Evaluation

```scss
// Input ColaSass
@function resolve($$expression) {
  @return $$expression;
}

@function convert($value) {
  @return resolve($value / 16);
}

// Output Sass
@function resolve($cola-expression, $value) {
  @return call($cola-expression, $value);
}

@function convert($value) {
  @return resolve('cola-resolve-expression', $value);
}

@function cola-resolve-expression($value) {
  @return $value / 16;
}
```

## Lazy Evaluation with comments

```scss
// Input ColaSass
// The expression argument here triggers all calls to this function to have
// this argument extracted into an expression.
@function resolve($$expression) {
  // Any time and expression `$$` is used outside an argument, the expression
  // is evaluated.
  @return $$expression;
}

@function convert($value) {
  // Since the function being called uses an expression for the first argument,
  // `$value / 16` will be extracted into an expression.
  // All variables in the expression are passed as arguments for the call.
  @return resolve($value / 16);
}

// Output Sass
@function resolve($cola-expression, $value) {
  @return call($cola-expression, $value);
}

@function convert($value) {
  @return resolve('cola-resolve-expression', $value);
}

// The expression function is prepended with `cola` to show that it's owned by
// ColaSass and appended with `expression` to show that it's an expression.
@function cola-resolve-expression($value) {
  @return $value / 16;
}
```

## @expand
PostCSS / Gulp Task

Expand converts `@expand .class` to an HTML element rather than concatenating a list of CSS selectors.

Simmilar to Sass's `@extend`.

### Reasoning?
+ CSS is included in the document as one large chunk
+ Extending extends can create very large lists of selectors, which is counter productive for modularity
+ HTML is included as needed and not as one large chunk

### Result?
+ Reduced file-size overall
+ Increased modularity in ColaSass/Sass
+ No messy selector lists are output in the CSS

#### Input
```css
.class1 {
  color: blue;
}

.class2 {
  @expand .class1;
  background-color: orange;
}
```

```html
<div class='.class2'></div>
```

#### Output
```css
.class1 {
  color: blue;
}

.class2 {
  background-color: orange;
}
```

```html
<div class='.class1 .class2'></div>
```
