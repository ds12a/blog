---
layout: post
title: An Infuriatingly Minuscule Mistake
description: Try finding it!
date: 2022-05-31 11:15:00 -0000
tags: Programming
---

A while back, I came across a segmentation fault in my program that I tracked to the following code:

```cpp
if (deg < 360)
{
    overlaps[deg] = preOverlap;
}
```

`overlaps` is an array containing $360$ integers. `preOverlap` is an integer. The segmentation fault was occuring on the line `overlaps[deg] = preOverlap;`. How could this be? `deg` is clearly less than $360$ so it shouldn't be trying to access memory that it's not supposed to. I became so desperate that I changed it to the following:


```cpp
if (deg < 360)
{
    assert(deg < 360); // ADDED
    overlaps[deg] = preOverlap;
}
```

As you can tell, I was fairly desperate. But to my astonishment, the `assert` failed! Apperently, `deg` sometimes was equal to or greater than $360$. Let's look at a broader version of my code:

```cpp
preOverlap += prefixDiffs[deg];

// If deg < 360, we have not gone around the wheel yet
// so the preOverlap is our calculated overlap\
if (deg < 360)
{
    assert(deg < 360); // ADDED
    overlaps[deg] = preOverlap;
}
else 
{
    overlaps[deg - 360] += preOverlap;
    if (overlaps[deg - 360] >= 5)
    {
        return true;
    }
}
```

**Try finding the issue before reading on!**
---

The issue is clear! Wait, it isn't? 

It took me a *long* time to figure out the issue. Take a closer look at the second line of comments. What do you notice about how it ends? It ends with a `\`!. This makes it a multi-line comment that includes the `if (deg < 360)` line. As a result, the `if` statement is never checked!

More interestingly, the `assert` statement did not fail on my own computer, yet it failed on the USACO grader (this was for a solution to one of the USACO Training problems). 

That's all to that strange segmentation fault! 
