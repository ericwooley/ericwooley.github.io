---
layout:     post
title:      "Writing clean code - Part 1: Naming Conventions"
date:       2015-12-23 11:12:45
categories: javascript clean code functional
comments:   true
published:  true

---
Looking back at my own github history, I found plenty of code that was utterly disgusting. But at
the time, I thought that it was great. What will I think of the code I write today in 5 years?
Probably that it is disgusting. I decided to take the time to research other peoples opinions on
what makes code clean. This is the first part in a series on what I consider to be clean code, and
it's not written in stone. If you disagree with something, leave me a comment!

Quick shoutout for "Clean Code" by Robert C. Martin.

## An unclear example.

This is the example of an extremely unclear, and dirty bit of code that outputs a matrix of cereal
and milk types. This example will be used to show how to clean up code throughout the series.


```js
function gcm (bf, mt) {
  const mtx = [['Breakfast Cereal Matrix']]
  for (let i = 0; i < bf.length; i++) {
    mtx[0].push(bf[i])
  }

  for (let i = 0; i < mt.length; i++) {
    mtx.push([mt[i]])
  }

  for (let i = 1; i < mtx.length; i++) {
    for (let j = 1; j < mtx[0].length; j++) {
      mtx[j].push(mtx[i][0] + ' - ' + mtx[0][j])
    }
  }
  return mtx
}
const bcl = ['cheerios', 'raison bran', 'sugar puffs', 'diabetes in a box']
const mtl = ['nonfat', '1%', '2%', 'whole']
gcm(bcl, mtl)

```

So you can follow it, I will insert some comments.

```js
/**
 * Generate cereal matrix
 * @param  {Array} bf = breakfast cereals
 * @param  {Array} mt = milk types
 * @return {Array<Array>} matrix of milk and cereals
 */
function gcm (bf, mt) {
  // new matrix
  const mtx = [['Breakfast Cereal Matrix']]
  // Add Cereal as Headers
  for (let i = 0; i < bf.length; i++) {
    mtx[0].push(bf[i])
  }
  // So far we have
  // [
  //  ['Breakfast Cereal Matrix', cheerios', 'raison bran', 'sugar puffs', 'diabetes in a box']
  // ]

  // Add Left column as milk labels
  for (let i = 0; i < mt.length; i++) {
    mtx.push([mt[i]])
  }
  // Now we have
  // [
  //  ['Breakfast Cereal Matrix', cheerios', 'raison bran', 'sugar puffs', 'diabetes in a box'],
  //  ['nonfat'],
  //  ['1%'],
  //  ['2%'],
  //  ['whole']
  // ]

  // Fill in the matrix
  for (let i = 1; i < mtx.length; i++) {
    for (let j = 1; j < mtx[0].length; j++) {
      mtx[j].push(mtx[i][0] + ' - ' + mtx[0][j])
    }
  }
// [
//  [ "Breakfast Cereal Matrix", "cheerios", "raison bran", "sugar puffs", "diabetes in a box"],
//  [ "nonfat", "nonfat - cheerios", "1% - cheerios", "2% - cheerios", "whole - cheerios"],
//  [ "1%", "nonfat - raison bran", "1% - raison bran", "2% - raison bran", "whole - raison bran"],
//  [ "2%", "nonfat - sugar puffs", "1% - sugar puffs", "2% - sugar puffs", "whole - sugar puffs"],
//  [ "whole", "nonfat - diabetes in a box", "1% - diabetes in a box", "2% - diabetes in a box", "whole - diabetes in a box"]
// ]
  return mtx
}
// breakfast cereal list
const bcl = ['cheerios', 'raison bran', 'sugar puffs', 'diabetes in a box']
// milk type list
const mtl = ['nonfat', '1%', '2%', 'whole']
gcm(bcl, mtl)

```

# Naming Conventions

## Make it pronounceable
I am as guilty of using `i` in a for loop as anyone, but short variable names (i, j, a) are a wasted
opportunity to add clarity. In the offending for loop, it's better to name the variable after what
it's looping through. The same should apply to all your variables and method/funciton names.


If it isn't clear to you that writing pronounceable names would make this more clear, then consider
the following rewritten version of our breakfast cereal example. I'm also going to take out the
excessive comments, because our code is now clear enough to document itself.

```js
function generateCerealMatrix (breakfastCereals, milkTypes) {
  const cerealMatrix = [['Breakfast Cereal Matrix']]
  for (let cerealIndex = 0; cerealIndex < breakfastCereals.length; cerealIndex++) {
    cerealMatrix[0].push(breakfastCereals[cerealIndex])
  }

  for (let milkIndex = 0; milkIndex < milkTypes.length; milkIndex++) {
    cerealMatrix.push([milkTypes[milkIndex]])
  }

  for (let cerealIndex = 1; cerealIndex < cerealMatrix[0].length; cerealIndex++) {
    for (let milkIndex = 1; milkIndex < cerealMatrix.length; milkIndex++) {
      // TODO: cerealMatrix[milkIndex]? why are we using the milkIndex as the row key for the cerealMatrix???
      // This line still seems very unclear. Lets clear this up in the next section.
      cerealMatrix[milkIndex].push(cerealMatrix[cerealIndex][0] + ' - ' + cerealMatrix[0][milkIndex])
    }
  }

  return cerealMatrix
}

const breakfastCereals = ['cheerios', 'raison bran', 'sugar puffs', 'diabetes in a box']
const milkTypes = ['nonfat', '1%', '2%', 'whole']

generateCerealMatrix(breakfastCereals, milkTypes)

```

### A few other things to note about variables:

1) Avoid diluting your names with obvious or useless informations. `breakfastCereals` works just as
    well as `breakfastCerealList` or `varBreakfastCereal`

2) Avoid disinformation (from Clean Code), if there is a datatype called List, and you make a
    variable called `breakfastCerealList` breakfastCerealList had better be a List, or you are going
    to confuse people.


## More variables and proxy variables

Lets take a moment to address our TODO.

```js
  //...
  for (let cerealIndex = 1; cerealIndex < cerealMatrix[0].length; cerealIndex++) {
    for (let milkIndex = 1; milkIndex < cerealMatrix.length; milkIndex++) {
      // TODO: cerealMatrix[milkIndex]? why are we using the milkIndex as the row key for the cerealMatrix???
      cerealMatrix[milkIndex].push(cerealMatrix[cerealIndex][0] + ' - ' + cerealMatrix[0][milkIndex])
    }
  }
  //...
```

First things first, this line is too long. Lets break it down.

```js
  //...
  for (let cerealIndex = 1; cerealIndex < cerealMatrix[0].length; cerealIndex++) {
    for (let milkIndex = 1; milkIndex < cerealMatrix.length; milkIndex++) {
      const cerealText = cerealMatrix[cerealIndex][0]
      const milkText = cerealMatrix[0][milkIndex]
      const cellData = milkText + ' - ' + cerealText
      // TODO: cerealMatrix[milkIndex]? why are we using the milkIndex as the row key for the cerealMatrix???
      cerealMatrix[milkIndex].push(cellData)
    }
  }
  //...
```
Breaking each portion of the statement into their own variables creates a lot more clarity. Because
each string is assigned a variable name, even if we aren't following the all this matrix index
stuff, at least we know for sure what the intended result of the statement is.

This line `cerealMatrix[milkIndex].push(cellData)` still seems confusing because we are referencing
our row as the milkIndex. In case you don't see why it works, I will explain. Here is our result:

```js

// Expected result
[
  // Header row
 [ "Breakfast Cereal Matrix", "cheerios", "raison bran", "sugar puffs", "diabetes in a box"],
 // Labels are column[0]
 [ "nonfat", "nonfat - cheerios", "1% - cheerios", "2% - cheerios", "whole - cheerios"],
 [ "1%", "nonfat - raison bran", "1% - raison bran", "2% - raison bran", "whole - raison bran"],
 [ "2%", "nonfat - sugar puffs", "1% - sugar puffs", "2% - sugar puffs", "whole - sugar puffs"],
 [ "whole", "nonfat - diabetes in a box", "1% - diabetes in a box", "2% - diabetes in a box", "whole - diabetes in a box"]
]
```
as you can see, the first column of our matrix is the label for milk, so to get each label, we want
to grab the first element from each array starting at array 1. So our inner loop (milkIndex), is
essentially looping through each row, starting at row 1. Our intended use is to grab the milk label
which is in position 0. The problem is, being cleaver little programmers, we realized that we could
reuse this variable to reference the row we are currently generating cells for.

How can we clean this up? Your first instinct (mine as well) is to make a proxy variable.

```js
  //...
  for (let cerealIndex = 1; cerealIndex < cerealMatrix[0].length; cerealIndex++) {
    for (let milkIndex = 1; milkIndex < cerealMatrix.length; milkIndex++) {
      // Proxy variable
      const currentRowIndex = milkIndex
      const cerealText = cerealMatrix[cerealIndex][0]
      const milkText = cerealMatrix[0][milkIndex]
      const cellData = milkText + ' - ' + cerealText
      cerealMatrix[currentRowIndex].push(cellData)
    }
  }
  //...
```

Finally, our code is starting to look pretty good. Each individual line is self documenting, and
most programmers could follow whats happening without too many mental gymnastics. However, there is
still room for improvement. The proxy variable works, but it's a symptom that the entire algorithm
could be more clear. Our problem stems from poor choices in our loop index names. Lets clean that
up.

```js
  //...
  for (let columnIndex = 1; columnIndex < cerealMatrix[0].length; columnIndex++) {
    for (let rowIndex = 1; rowIndex < cerealMatrix.length; rowIndex++) {
      const cerealText = cerealMatrix[columnIndex][0]
      const milkText = cerealMatrix[0][rowIndex]
      const cellData = milkText + ' - ' + cerealText
      cerealMatrix[rowIndex].push(cellData)
    }
  }
  //...
```
We had it wrong when we called it `cerealIndex` and `milkIndex` because we were trying to associate
the index with only the data out of the matrix that we wanted. Once we decided the variable could
be used as the row index as well, it really needed to change to reflect what it really is.

## Magic Numbers

The more astute of you may have noticed by now that there is a mistake in the code.

```js
  //...
  // Cereal Text should come from the header row, we are changing the row and grabbing the 0 column
  const cerealText = cerealMatrix[cerealIndex][0]
  // Milk text should come from the 0 column on different row, but its grabbing all of it from row 0
  const milkText = cerealMatrix[0][milkIndex]
  //...
```

We messed up with our row and columns. Effectively the variables were switched. We were just lucky
our test data had the same number of rows and columns. I don't know about you, but my logic is
always flawless, obviously the problem is that our code isn't clean enough to spot this mistake
easily enough.

```js
//...
const headRow = 0
const labelCol = 0
// ...
for (let columnIndex = 1; columnIndex < cerealMatrix[headRow].length; columnIndex++) {
  for (let rowIndex = 1; rowIndex < cerealMatrix.length; rowIndex++) {
    // Why is our column index in the row spot?????
    const cerealText = cerealMatrix[columnIndex][headRow]
    // Again??? Damn it Eric, get it together!
    const milkText = cerealMatrix[labelCol][rowIndex]
    const cellData = milkText + ' - ' + cerealText
    cerealMatrix[rowIndex].push(cellData)
  }
}
//...
```

Having magic numbers makes it easy to miss that they were in the wrong spot. Naming the magic numbers
makes it a little more obvious that we have the labelCol referencing the row number.

Lets step back, clean up the other loops, and have a look at the whole function again.

```js
function generateCerealMatrix (breakfastCereals, milkTypes) {
  const headRow = 0
  const labelCol = 0
  const cerealMatrix = [['Breakfast Cereal Matrix']]
  for (let cerealIndex = 0; cerealIndex < breakfastCereals.length; cerealIndex++) {
    const cerealHeader = breakfastCereals[cerealIndex]
    cerealMatrix[headRow].push(cerealHeader)
  }

  for (let milkIndex = 0; milkIndex < milkTypes.length; milkIndex++) {
    const milkLabel = milkTypes[milkIndex]
    cerealMatrix.push([milkLabel])
  }

  for (let columnIndex = 1; columnIndex < cerealMatrix[headRow].length; columnIndex++) {
    for (let rowIndex = 1; rowIndex < cerealMatrix.length; rowIndex++) {
      const cerealText = cerealMatrix[headRow][columnIndex]
      const milkText = cerealMatrix[rowIndex][labelCol]
      const cellData = milkText + ' - ' + cerealText
      cerealMatrix[rowIndex].push(cellData)
    }
  }

  return cerealMatrix
}

const breakfastCereals = ['cheerios', 'raison bran', 'sugar puffs', 'diabetes in a box']
const milkTypes = ['nonfat', '1%', '2%', 'whole']

generateCerealMatrix(breakfastCereals, milkTypes)
```

The result is so much more readable. This code can be still be cleaned up, but the difference is
night and day.

## Whats in Part 2?

Next up: Breaking the function down into smaller functions. Don't worry, it shouldn't take as long
as this part 1.


Leave me some comments if you think I missed something, or disagree about any part of this. I am
always open to changing my opinion on something!
