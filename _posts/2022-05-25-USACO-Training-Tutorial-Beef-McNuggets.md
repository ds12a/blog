---
layout: post
title: USACO Training Tutorial - Beef McNuggets
description: Tutorial for USACO Training - Beef McNuggets
date: 2022-05-25 21:21:00 -0000
tags: Competitive_Programming
---

I've decided that writing editorials for USACO problems isn't too bad. Here is another editorial. This is for 4.1.1 Beef McNuggets from the USACO Training Pages.

## The Problem:

> Farmer Brown's cows are up in arms, having heard that McDonalds is considering the introduction of a new product: Beef McNuggets. The cows are trying to find any possible way to put such a product in a negative light.
> One strategy the cows are pursuing is that of 'inferior packaging'. "Look," say the cows, "if you have Beef McNuggets in boxes of 3, 6, and 10, you can not satisfy a customer who wants 1, 2, 4, 5, 7, 8, 11, 14, or 17 McNuggets. Bad packaging: bad product."
> Help the cows. Given $N$ (the number of packaging options, $1 <= N <= 10$), and a set of N positive integers ($1 <= i <= 256$) that represent the number of nuggets in the various packages, output the largest number of nuggets that can not be purchased by buying nuggets in the given sizes. Print 0 if all possible purchases can be made or if there is no bound to the largest number.
>
> The largest impossible number (if it exists) will be no larger than $2,000,000,000$.
>
> INPUT FORMAT
> - Line $1$:	$N$, the number of packaging options
> - Line $2..N+1$: The number of nuggets in one kind of box
> 
> OUTPUT FORMAT
> - The output file should contain a single line containing a single integer that represents the largest number of nuggets that can not be represented or $0$ if all possible purchases can be made or if there is no bound to the largest number.

## Analysis:

I feel like this problem is planning its own downfall. What's a USACO problem without the cows (alive and well, *not* in the form of a Beef McNugget)?

This problem can easily be solved with dynamic programming by keeping track of which counts of McNuggets can be achieved and adding each package of McNuggets to the possible totals. If $gcd(SIZES)$ does not equal $1$, we have infinite solutions so we output $0$ as requested.

However, if we do this with the ridiculously high upper bound given of $2,000,000,000$, we are likely to receive TLE. The program would have a complexity of $O(2,000,000,000 * 10)$! Can we find a lower upper bound to make this approach feasible?

This problem reminds me of the [Chicken McNugget Theorem](https://artofproblemsolving.com/wiki/index.php/Chicken_McNugget_Theorem), only with the possibility of more than just 2 numbers. It turns out that this theorem offers a lower upper bound! Assume we only have two sizes of packages. By the Chicken McNugget Theorem, we get the upper bound is $256*256-256-256=65024$. If we have 3 sizes, the 3rd size can only lower this bound; we can always let $0$ packages be of the 3rd size. This is true for any extra sizes. Therefore, our new upper bound is $65,024$, which is far more reasonable.

## Implementation:
```cpp
#include <fstream>
#include <numeric>
#include <vector>

bool reachable[65025];

int main()
{
    std::ifstream fin("nuggets.in");
    std::vector<int> packages;
    int n, x, g = -1;

    fin >> n;

    for (int i = 0; i < n; i++)
    {
        fin >> x;

        packages.push_back(x);

        if (g == -1)
            g = x;
        else
            g = std::gcd(g, x);
    }

    std::ofstream fout("nuggets.out");
    
    if (g != 1)
    {
        // GCD is 1, infinite solutions
        fout << "0\n";
        return 0;
    }

    int maxCannotReach = 0;
    reachable[0] = true;

    for (int i = 0; i <= 65024; i++)
    {
        if (reachable[i])
        {
            for (int p : packages)
            {
                reachable[i + p] = true;
            }
        }
        else
        {
            maxCannotReach = i;
        }
    }

    fout << maxCannotReach << '\n';
}
```

And we are done! If there are any questions or clarifications needed, please comment below (if there are any readers).
