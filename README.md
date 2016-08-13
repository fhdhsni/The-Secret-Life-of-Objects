# The-Secret-Life-of-Objects

In chapter 6 of Eloquent JavaScript(https://github.com/marijnh/Eloquent-JavaScript) there is an example that was hard for me to understand. So with this repo I hope to make it easier for newbies like myself to understand it.

Before getting started I recommend watching there videos.
[`Map`](https://www.youtube.com/watch?v=bCqtb-Z5YGQ&index=2&list=PL0zVEGEvSaeEd9hlmCXrk5yUyqUag-n84),
[`reduce`](https://www.youtube.com/watch?v=1DMolJ2FrNY&index=4&list=PL0zVEGEvSaeEd9hlmCXrk5yUyqUag-n84) by [mpjme](https://twitter.com/mpjme?lang=en)

And [Object-Oriented JavaScript](https://www.youtube.com/watch?v=PMfcsYzj-9M) by [James Shore](https://twitter.com/jamesshore)


Our goal is making a table like this

```
name         height country      
------------ ------ -------------
Kilimanjaro    5895 Tanzania     
Everest        8848 Nepal        
Mount Fuji     3776 Japan        
Mont Blanc     4808 Italy/France 
Vaalserberg     323 Netherlands  
Denali         6168 United States
Popocatepetl   5465 Mexico       
```
out of this array
```js
var MOUNTAINS = [
  {name: "Kilimanjaro", height: 5895, country: "Tanzania"},
  {name: "Everest", height: 8848, country: "Nepal"},
  {name: "Mount Fuji", height: 3776, country: "Japan"},
  {name: "Mont Blanc", height: 4808, country: "Italy/France"},
  {name: "Vaalserberg", height: 323, country: "Netherlands"},
  {name: "Denali", height: 6168, country: "United States"},
  {name: "Popocatepetl", height: 5465, country: "Mexico"}
];
```
Pretty simple right? not for me. I had a hard time understanding what's going on.

This is the entire program that does this.

``` javascript
  const MOUNTAINS = require("./mountains");

  function rowHeights(rows) {
    return rows.map(function(row) {
      return row.reduce(function(max, cell) {
        return Math.max(max, cell.minHeight());
      }, 0);
    });
  }
  function colWidths(rows) {
    return rows[0].map(function(_, i) {
      return rows.reduce(function(max, row) {
        return Math.max(max, row[i].minWidth());
      }, 0);
    });
  }
  function drawTable(rows) {
    var heights = rowHeights(rows);
    var widths = colWidths(rows);
    function drawLine(blocks, lineNo) {
      return blocks.map(function(block) {
        return block[lineNo];
      }).join(" ");
    }

    function drawRow(row, rowNum) {
      var blocks = row.map(function(cell, colNum) {
        return cell.draw(widths[colNum], heights[rowNum]);
      });
      return blocks[0].map(function(_, lineNo) {
        return drawLine(blocks, lineNo);
      }).join("\n");
    }
    return rows.map(drawRow).join("\n");
  }
  function repeat(string, times) {
    var result = "";
    for (var i = 0; i < times; i++)
      result += string;
    return result;
  }
  function TextCell(text) {
    this.text = text.split("\n");
  }
  TextCell.prototype.minWidth = function() {
    return this.text.reduce(function(width, line) {
      return Math.max(width, line.length);
    }, 0);
  };
  TextCell.prototype.minHeight = function() {
    return this.text.length;
  };
  TextCell.prototype.draw = function(width, height) {
    var result = [];
    for (var i = 0; i < height; i++) {
      var line = this.text[i] || "";
      result.push(line + repeat(" ", width - line.length));
    }
    return result;
  };
  function UnderlinedCell(inner) {
    this.inner = inner;
  }
  UnderlinedCell.prototype.minWidth = function() {
    return this.inner.minWidth();
  };
  UnderlinedCell.prototype.minHeight = function() {
    return this.inner.minHeight() + 1;
  };
  UnderlinedCell.prototype.draw = function(width, height) {
    return this.inner.draw(width, height - 1).concat([repeat("-", width)]);
  };
  function dataTable(data) {
    var keys = Object.keys(data[0]);
    var headers = keys.map(function(name) {
      return new UnderlinedCell(new TextCell(name));
    });
    var body = data.map(function(row) {
      return keys.map(function(name) {
        var value = row[name];
        if (typeof value == "number")
          return new RTextCell(String(value));
        else
          return new TextCell(String(value));
      });
    });
    return [headers].concat(body);
  }
  function RTextCell(text) {
    TextCell.call(this, text);
  }
  RTextCell.prototype = Object.create(TextCell.prototype);
  RTextCell.prototype.draw = function(width, height) {
    var result = [];
    for (var i = 0; i < height; i++) {
      var line = this.text[i] || "";
      result.push(repeat(" ", width - line.length) + line);
    }
    return result;
  };
  console.log(drawTable(dataTable(MOUNTAINS)));
```

 Let's get started

```js
  console.log(drawTable(dataTable(MOUNTAINS)));
```
This is where everything begins. We send our `MOUNTAINS` array to `dataTable` function.

```js
function dataTable(data) {
  var keys = Object.keys(data[0]);
  var headers = keys.map(function(name) {
    return new UnderlinedCell(new TextCell(name));
  });
  var body = data.map(function(row) {
    return keys.map(function(name) {
      return new TextCell(String(row[name]));
    });
  });
  return [headers].concat(body);
}
```

Let's go through it line by line. 
```js
  var keys = Object.keys(data[0]);
    // date[0] -> {name: "Kilimanjaro", height: 5895, country: "Tanzania"},
    // keys -> [ "name", "height", "country"]
```  
It is uses [`Objecet.keys`](https://davidwalsh.name/object-keys) method to get the keys of one of our objects in the `data` array.

``` javascript
    var headers = keys.map(function(name) {
      return new UnderlinedCell(new TextCell(name));
    });
    /* headers ->
     * [
     * UnderlinedCell { inner: TextCell { text: ['name'] } },
     * UnderlinedCell { inner: TextCell { text: ['height'] } },
     * UnderlinedCell { inner: TextCell { text: ['country'] } },
     * ]
     */
``` 
As you can see in our final table we have a header that is nicely underlined. We use `map` method on our `keys` to make an object for each key. So what we get in `header` variable is an array of three object. 
Our objects are constructed with two constructors `TextCell` and `UnderlinedCell`. Let's take a look at how they're defined before continuing with our `dataTable` function.

``` javascript
  function TextCell(text) {
    this.text = text.split("\n");
  }
```
`String.prototype.split` returns an array of strings. so `this.text` is an array, something like `["country"]`

`TextCell` is a constructor therefore it implicitly returns `this`. 

``` javascript
  TextCell.prototype.minWidth = function() {
    return this.text.reduce(function(width, line) {
      return Math.max(width, line.length);
    }, 0);
  };

  TextCell.prototype.minHeight = function() {
    return this.text.length;
  };
  
    TextCell.prototype.draw = function(width, height) {
    var result = [];
    for (var i = 0; i < height; i++) {
      var line = this.text[i] || "";
      result.push(line + repeat(" ", width - line.length));
    }
    return result;
  }
```
Each instance of `TextCell` (which indicates a cell in our final table) inherits these methods. 

`minWidth()` returns a number indicating minimum width of (in characters) its cell. It uses `reduce` on `this.text` in case `this.text` has more than one element (more than one line) which in our example it never does.

`minHeight()` returns a number indicating the minimum height that its cell requires (in lines). In our example it's always 1. number of elements in `this.text` indicates the minimum height that a given cell needs.

We use `draw()` to tell a cell to draw itself. It takes two parameter `width` and `height` and returns an array. something like `[ 'Kilimanjaro ' ]`

`width` is maximum width of a particular column that our cell is going reside in. We need this to add extra spaces in order to make our cells in a column aligned with each other.

`height` is height of a particular row. If a cell should, say takes two line, height will be 2. In our example it's always 1. Remember underlined cells of header uses a slightly different version of `draw()`. We'll see it in a moment.

We go through body of loop `height` times.

It adds those extra spaces with help of `repeat` function. `repeat` function takes a string and a number and it returns the given string concatenated with itself number of times. e.g. `repeat("bat! ", 3)` returns `"bat! bat! bat! "`.

We figure out how many extra spaces we should add with subtracting length of widest cell in our column (`width`) from length of current cell that we are drawing (`line.length`). e.g: `kilimanjaro` with the `length` of 11 belongs to first column. The widest cell in the first column is `"Popocatepetl"` with the `length` of 12. So we only need one space after `kilimanjaro`. `result` will be `[ 'Kilimanjaro ' ]`.

We're not done with constructors just yet. `TextCell` will be used for regular cells. For underlined cell (that is headers) we have another constructor `UnderlinedCell`. Fortunately it inherits most of the goodies from `TextCell`.

``` javascript
  function UnderlinedCell(inner) {
    this.inner = inner;
  }
  UnderlinedCell.prototype.minWidth = function() {
    return this.inner.minWidth();
  };
  UnderlinedCell.prototype.minHeight = function() {
    return this.inner.minHeight() + 1;
    // so minHeight of the header will be 2
  };
  UnderlinedCell.prototype.draw = function(width, height) {
    // this method returns an array. e.g. [ 'name        ', '------------' ]
    return this.inner.draw(width, height - 1).concat([repeat("-", width)]);
  };
```
Something that you shouldn't forget about `UnderlinedCell` is that it takes `TextCell` objects as its argument so `inner` is a `TextCell` object.

`UnderlinedCell.prototype.minHeight` adds 1 to the `height` of cell because of dashes under headers that take one extra line more than regular `TextCell` cells.
```
name         height country      
------------ ------ -------------
```

Also for `draw()` method we should consider that extra line. We send `height - 1` to `TextCell.prototype.draw` (because we don't want to add a bare empty line with second evaluation of its loop) but we `concate` the returned array from `TextCell.prototype.draw` (remember `draw` returns an array) with `[repeat("-", width)]`. Let's imagine we wanna draw `name` cell. It's in the first column. The widest cell in first column is `"popocatepetl"` with the `length` of 12 therefore `width` is 12. And height is 2 because `UnderlinedCell.prototype.minHeight` returns 2. With the help of `[repeat("-", 12)]` we get the appropriate underline which is this `['------------']`.
After concatenation, what actually `UnderlinedCell.prototype.draw` returns for `name` cell is this:

``` javascript
[ 'name        ', '------------' ]
```

Let's get back to our `dataTable` function.

```js
function dataTable(data) {
  var keys = Object.keys(data[0]);
  var headers = keys.map(function(name) {
    return new UnderlinedCell(new TextCell(name));
  });

    var body = data.map(function(row) {
      return keys.map(function(name) {
          // keys = [ "name", "height", "country"]
        var value = row[name];
        if (typeof value == "number")
          return new RTextCell(String(value));
        else
          return new TextCell(String(value));
      });
    });

  return [headers].concat(body);
}
```
It's time to examine how `body` of our table gets created.
`body` is an array with 7 elements each element is an array (containing three `TextCell` objs) that makes one `row` of our table, something like:
``` 
  [
    TextCell { text: [ 'Kilimanjaro' ] },
    TextCell { text: [ '5895' ] },
    TextCell { text: [ 'Tanzania' ] }
  ]
```
For each invocation of `map`, `row` is something like this: `{name: "Kilimanjaro", height: 5895, country: "Tanzania"}`.

For each `row` we `map` through our `keys` to get the `value` for each key. Then we examine the `value` to see if it's a `"number"` or not. We do that becuase we want numbers to be aligned to the right of their cells.
If `value` is a number we pass it to `RTextCell` constructor which is a slight variation of `TextCell` constructor.

``` javascript
  function RTextCell(text) {
    TextCell.call(this, text);
  }
  RTextCell.prototype = Object.create(TextCell.prototype);
  RTextCell.prototype.draw = function(width, height) {
    var result = [];
    for (var i = 0; i < height; i++) {
      var line = this.text[i] || "";
      result.push(repeat(" ", width - line.length) + line);
    }
    return result;
  };
```

It inherits everything from `TextCell` except for `draw`. And as you can see the only difference in `draw` method is this line:

``` javascript
result.push(repeat(" ", width - line.length) + line);
```
It puts those extra spaces before `line` instead of after it. So the result will be sth like `[ '..5895' ]` (I used `.` instead of space becauce github removes extra spaces.) Notice we have two extra space before the number.

Getting back to our `dataTable` function, its last line is :

``` javascript
return [headers].concat(body);
```
By concatenating `[headers]` with `body` we get an array of 8 elements. One element for our `headers` and the rest of them for `body`. Each element which an array of three objects represents a `row` in the finale table.

Phew! We are finally done with `dataTable` function. Remember everything started with this line.

```js
  console.log(drawTable(dataTable(MOUNTAINS)));
```

We sent `MOUNTAINS` to `dataTable` and we saw that it returns an array with 8 elements, something like this:

```
[ [ UnderlinedCell { inner: [Object] },
    UnderlinedCell { inner: [Object] },
    UnderlinedCell { inner: [Object] } ],
  [ TextCell { text: [Object] },
    TextCell { text: [Object] },
    TextCell { text: [Object] } ],
  [ TextCell { text: [Object] },
    TextCell { text: [Object] },
    TextCell { text: [Object] } ],
    ...etc
```
This array, returned by `dataTable` is the argument of `drawTable` function. Let's see how this function looks like.

``` javascript
function drawTable(rows) {
  var heights = rowHeights(rows);
  var widths = colWidths(rows);

  function drawLine(blocks, lineNo) {
    return blocks.map(function(block) {
      return block[lineNo];
    }).join(" ");
  }

  function drawRow(row, rowNum) {
    var blocks = row.map(function(cell, colNum) {
      return cell.draw(widths[colNum], heights[rowNum]);
    });
    return blocks[0].map(function(_, lineNo) {
      return drawLine(blocks, lineNo);
    }).join("\n");
  }

  return rows.map(drawRow).join("\n");
}
```

Let's go through it line by line.

``` javascript
  var heights = rowHeights(rows);
```

This is the `rowHeights` function.

``` javascript
function rowHeights(rows) {
  return rows.map(function(row) {
    return row.reduce(function(max, cell) {
      return Math.max(max, cell.minHeight());
    }, 0);
  });
}
```
In order to find `height` of each row we use `rowHeights` function. It returns `[ 2, 1, 1, 1, 1, 1, 1, 1 ]`,  an array of numbers. Each number represents `height` of one row. `height` of first row is `2` becuase it's the header which has a underline. (Remember `this.inner.minHeight() + 1`?)

It uses `reduce` to go through all the cells of each `row` and asks each cell what is use your `minHeight`? Then it returns max `height` of each row (remember there are three cells in each `row`) to the `map` function and `map` produces an array of heights for all the `rows`.

The second line of `drawTable` function is this

``` javascript
  var widths = colWidths(rows);
```
This is `colWidths` function.

``` javascript
function colWidths(rows) {
  return rows[0].map(function(_, i) {
    return rows.reduce(function(max, row) {
      return Math.max(max, row[i].minWidth());
    }, 0);
  });
}
```
In our table we have three columns. We need to know how wide each column should be in order to make straight columns. we give `width` of each column to `draw` method of our cells to tell them our many extra space they should add to themselves. To find out `width` of each column we use `colWidths` function.

`colWidths` needs to know how many columns we have so it maps through `rows[0]` to use its index parameter `i`. And `i` of course gonna be `0`, `1`, `2` becuase we have three objects/cells in each `row`. Again we use `reduce` to go through all `rows` and check `width` of their `i` cell. At the end we have this `[ 12, 6, 13 ]` an array of three number.

`12` for the first column because `"Popocatepetl".length` is 12.

`6`  for the second column because `"height".length` is 6.

`13` for thrid column becuase `"United States".length` is 13.

Getting back to our `drawTable` function, it returns this

``` javascript
  return rows.map(drawRow).join("\n");
```
It maps through `rows` and passes each `row` to `drawRow` function. Therefore for each invocation of `drawRow` we pass it a `row`. Let's take a look at this function and see how it works.

``` javascript
  function drawRow(row, rowNum) {
    var blocks = row.map(function(cell, colNum) {
      return cell.draw(widths[colNum], heights[rowNum]);
    });
    return blocks[0].map(function(_, lineNo) {
      return drawLine(blocks, lineNo);
    }).join("\n");
```
As you can see it takes a `row` (for each invocation) and the corresponding `rowNum`. 
For each `row`, `drawRow` returns a string. For example for the second `row` which is this

``` javascript
  [
    TextCell { text: [ 'Kilimanjaro' ] },
    TextCell { text: [ '5895' ] },
    TextCell { text: [ 'Tanzania' ] }
  ]
```

It returns `"Kilimanjaro....5895.Tanzania....."` I replaced spaces with `.` to make it visually more clear.

`blocks` is an array of arrays. Each inner array will be a cell in finale table. 
For e.g if `rowNum == 0` (so it's our header) blocks is.

``` javascript
[
    [ 'name        ', '------------' ],
    [ 'height', '------' ],
    [ 'country      ', '-------------' ]
]
```
Each inner array has two element because height of first `row` is 2 (`heights[0] --> 2`).

Or for `rowNum == 1`, `blocks` is `[ [ 'Kilimanjaro.' ], [ '..5895' ], [ 'Tanzania.....' ] ]`. (I replaced spaces with `.`)

`blocks` is made with the help of `map`. We go through all the cells in each `row` and in that we use `draw` method on each cell object to ask them to draw themselves. We pass height of current `row` and `width` of corresponding column for each cell to `draw` method. And `draw` in turn returns an array for each cell, something like `[ 'Kilimanjaro ' ]`;

Now that we know what is `blocks` we can go through second part of `drawRow` function.

``` javascript
    return blocks[0].map(function(_, lineNo) {
      return drawLine(blocks, lineNo);
    }).join("\n");
```
When `rowNum` is `0` (so we are working on header) `blocks[0]` is `[ 'name        ', '------------' ]` so `lineNo` will be `0` and `1` other than that `lineNo` is always `0` because `blocks[0]` will be something like `[ 'Kilimanjaro ' ]` which has only one element.
We pass whole `blocks` array to the `drawLine` function. `drawLine` is a helper function that is local of `drawTable` function.
    
``` javascript
  function drawLine(blocks, lineNo) {
    return blocks.map(function(block) {
      return block[lineNo];
    }).join(" ");
  }
```
All it does is making a string out of an array of arrays. It does this in two step. First it flattens the `blocks`. For example for a `blocks` like:

``` javascript
[ [ 'Kilimanjaro ' ], [ '  5895' ], [ 'Tanzania     ' ] ]
```
It makes

``` javascript
[ 'Kilimanjaro ', '  5895', 'Tanzania     ' ]
```
And then it uses `join(" ")` to make a string by joining all the elements of this array with one space in between.
At the end `drawLine` returns `"Kilimanjaro....5895.Tanzania....."`, again I replaced spaces with `.`.

`lineNo` is always `0` except for header. When `rowNum` is `0` (so we are working on header) `blocks[0]` is `[ 'name........', '------------' ]` so `lineNo` can be `0` and `1`. Therefore for header this function will be invoked twice. Once with `lineNo` of `0` another with `lineNo` of `1`. In turns two separate line will be generated. One for headers (that is `name`, `height` and `country`) and another for those dashes.

Now that we know what `drawLine` does we can continue with `drawRow` function.

``` javascript
    return blocks[0].map(function(_, lineNo) {
      return drawLine(blocks, lineNo);
    }).join("\n");
```
Actually there's not much to continue. `drawLine` returns a string to the `map` and `map` returns an array. This array has one or two elements depend on the `row` that we are working on. If it's the header it has two elements.

``` javascript
[ 'name         height country      ',
  '------------ ------ -------------' ]
```
And for all the other rows it has one element something like this:

``` javascript
[ 'Kilimanjaro    5895 Tanzania     ' ]
```
Notice it's stil an array but what we want is a string. So again we use `join("\n")` but this time with line feed as its separator.
So to make it more clear, this is what we get for header (first `row`):

```
"name         height country      \n------------ ------ -------------"
```
and for the second `row` we get:

```
"Kilimanjaro    5895 Tanzania     "
```
This process wil be repeated for each `row`, remember `drawRow` is in a `map`

``` javascript
  return rows.map(drawRow).join("\n");
```
This is the array that `rows.map(drawRow)` returns

``` javascript
[ 'name         height country      \n------------ ------ -------------',
  'Kilimanjaro    5895 Tanzania     ',
  'Everest        8848 Nepal        ',
  'Mount Fuji     3776 Japan        ',
  'Mont Blanc     4808 Italy/France ',
  'Vaalserberg     323 Netherlands  ',
  'Denali         6168 United States',
  'Popocatepetl   5465 Mexico       ' ]
```
And for the last time we use `join("\n")` to make a string out of this array. And that it, we got our table.
```
name         height country      
------------ ------ -------------
Kilimanjaro    5895 Tanzania     
Everest        8848 Nepal        
Mount Fuji     3776 Japan        
Mont Blanc     4808 Italy/France 
Vaalserberg     323 Netherlands  
Denali         6168 United States
Popocatepetl   5465 Mexico       
```
