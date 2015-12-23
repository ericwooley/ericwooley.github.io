---
layout:     post
title:      "Writing clean code - Part 1: Naming Conventions"
date:       2015-12-22 18:04:45
categories: javascript clean code functional
comments:   true
published:  true

---
Looking back at my own github history, I found plenty of code that was utterly disgusting. But at
the time, I thought that it was great. What will I think of the code I write today in 5 years?
Probably that it is disgusting. I decided to take the time to research other peoples opinions on
what makes code clean. Much of this is gleamed from "Clean Code" by Robert C. Martin.

This is the example of an extremely unclear bit of code that outputs a matrix of cereal and milk
types. This example will be used to show how to clean up code throughout the series.


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

So you can follow it, I will insert some comments

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

## A few other things to note:

1) Avoid diluting your names with obvious or useless informations. `breakfastCereals` works just as
    well as `breakfastCerealList` or `varBreakfastCereal`

2) Avoid disinformation (from Clean Code), if there is a datatype called List, and you make a
    variable called `breakfastCerealList` breakfastCerealList had better be a List, or you are going
    to confuse people.
