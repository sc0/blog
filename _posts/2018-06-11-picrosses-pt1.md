---
layout: post
title: Creating a picross game pt. 1 - generating picrosses
date: 2018-06-11 23:25:05 +0200
published: true
category: gamedev puzzles
comments: true
tags: gamedev picross games jimp backend puzzles javascript images
---

Lately I got addicted to solving picrosses all over again. If you’re not familiar, picross (a.k.a.  nonogram  or logic picture) consist of a grid, where each of the rows and columns have own set of numbers. Here’s an example of a picross puzzle:

![Picross example]({{"images/picross-pt1/empty-picross.png" | absolute_url}})


Every number represents an amount of pixels in line that should be placed in this row/column. If multiple numbers provided, then we know that there is at least one empty cell between the lines. We don’t know where the line is located and it’s our task to figure it out. If you keep struggling with getting your head around this concept, there is a lot of implementations of this game online with great tutorials.

When I started to solve them again few weeks back, I couldn’t stop thinking that it’s amazing how clever some of the puzzles are.  I was sure that to come up with one you have to combine hardcore math with a sprinkle of magic, it can’t just be any low res image with counted pixels… or can it?

Then I thought that there is only one way to find out. However, using Google would be just to easy, wouldn’t it? So I decided to try to generate some puzzles by myself, simply counting pixels of low res images.

## Preparing image
First things first - I am no artist, leave alone pixel-artist, but I needed something to generate a puzzle from. I thought that cartoonish images will work best, so I found [an image of an apple](https://www.kisspng.com/png-juice-apple-clip-art-cartoon-apples-205781/) that I thought will work for us here.

![Apple image]({{"images/picross-pt1/apple.png" | absolute_url}})

OK, so what do we need here to convert it to something that suits our needs? It’s a PNG, which is good, as we won’t have to deal with all this messy compression artifacts, but we have to get rid of transparency. I’m using Pixelmator as my primary image tool, so I’ll be using their terminology here.

To remove all the transparency, I filled whole image with white color and „Darker Color” blending setting. Then I used a threshold color transformation to remove all the colors. In order to make only an outline black, not the whole apple, I set threshold to around 25%.

![Apple outline]({{"images/picross-pt1/apple-outline.png" | absolute_url}})

Now I had to downscale the image - there is no way anyone would like to solve a picross with the width of over 500px. So I scaled it down to 25x28px  and thresholded it again, this time with the threshold of 100%.

![Pixelized apple image]({{"images/picross-pt1/apple-pixels-hires.png" | absolute_url}})


## Counting pixels
Now, having our picture ready, it’s time to convert it into our puzzle. In order to do this, we need to count all the pixels in rows and cols and group it in sets. For this task I used JavaScript and a small, handy library called [Jimp](https://github.com/oliver-moran/jimp). 

First, we need to load our image and get info about the image’s pixels. Jimp provides a nice method for this called[ `scan`](https://github.com/oliver-moran/jimp#low-level-manipulation) It iterates over the pixels in given area and lets us execute a callback on every and each of them:

```js
let Jimp = require('jimp');

let rows = [];
let cols = [];

Jimp.read('picross-apple.png', function (err, pic) {
    if (err) throw err;

    pic.scan(0, 0, pic.bitmap.width, pic.bitmap.height, function (x, y, idx) {
        if (rows.length - 1 < y) rows.push([]);
        rows[y].push(+(pic.bitmap.data[idx] === 0));

        if (cols.length - 1 < x) cols.push([]);
        cols[x].push(+(pic.bitmap.data[idx] === 0));

//	....

    });
});
```

In the callback, we recognize if our pixel is black or white by checking the value of red RGBA component (I chose the red value because of the convenience of it being at `idx` index of `bitmap.data` array. For black, all 3 components of RGB are set to 0 and - given that our picture is only black and white - if red is equal to 0 we know it’s black, otherwise it must be white). Then we push a binary information about a pixel - 1 if black, 0 otherwise - to both arrays describing a row and a column it belongs to.

After all of those operations we end up with two arrays of arrays - `rows` and `cols`. Each of them store full information about an image we can now use to calculate the values we need:

```js
reductor = (prev, cur) => {
    if (cur === 0) {
        if (prev[prev.length - 1] != 0) {
            prev.push(0);
        }
    } else {
        prev[prev.length - 1] += 1;
    }
    return prev;
}

removeTrailingZero = (row) => {
    if (row.length > 1 && row[row.length-1] === 0) row.pop(); 
    return row;
}

processPicture = () => {
    rows = rows.map(row => row.reduce(reductor, [0]));
    cols = cols.map(col => col.reduce(reductor, [0]));

    rows = rows.map(removeTrailingZero);
    cols = cols.map(removeTrailingZero);
}
```

Each row and column is [reduced](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) . The reductor function takes two arguments: `prev`, which is an array containing our already counted values and `cur`, being next value to consider in our set. If our `cur` value is 1 we know we have to increment the last value in `prev`. If not, then we know that a previous line has ended, so we end it by pushing 0 to `prev` (if it hasn’t already been pushed). This solution however has one disadvantage - every time the row or column ends with white pixel, the calculated array has a trailing zero we need to get rid of. This is handled by our `removeTrailingZero` function.

# The result 
Now we have absolutely everything we need to construct our logic image!
I’ve created a simple „client” in React which takes the values of our calculated rows and cols arrays and lets us to render it and try to solve it. However if you actually try to solve it, you will get stuck somewhere around this far:

![Picross failed to complete]({{"images/picross-pt1/failed-to-complete.png" | absolute_url}})

Why is that? Did we fail? Well, kind of. Quick Google search can lead you to many articles stating that not only not every nonogram is solvable, but also that solving and proving that  a picross puzzle is solvable is a NP-complete problem. Is your not familiar with all this NP-completeness stuff - it basically means it’s not that simple and that it actually cannot be done for every picross in polynomial time ([here’s some more insight on this topic](https://en.wikipedia.org/wiki/NP-completeness)). So our picross is too hard to complete. 

This issue can be usually fixed by making our picross image thicker, so the ratio of values describing  our rows and cols to width and hight is bigger. I managed to achieve this in our sample image by bluring an image a bit just before thresholding:

![Solved picross]({{"images/picross-pt1/solved.png" | absolute_url}})

This way our image is fully solvable!

Well, but it doesn’t feel that good when you can generate a puzzle, but aren’t really sure if anyone will be able to solve it, does it? Well, if the only way to tell if the puzzle is solvable is to actually try to solve it, isn’t it?

This is why in the next article I’ll show you how I managed to write a solver and how this sweet button in the top left of the grid works! So stay tuned for the next part!