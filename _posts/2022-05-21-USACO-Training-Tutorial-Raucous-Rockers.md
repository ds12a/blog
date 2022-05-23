---
layout: post
title: USACO Training Tutorial - Raucous Rockers
description: Tutorial for USACO Training - Raucous Rockers
date: 2022-05-21 21:25:00 -0000
tags: Competitive_Programming
---

Since I'm not quite sure yet about what I should post on this blog, I thought I'd start with a USACO Training Tutorial. My most recently solved one is Raucous Rockers. Here it is:

## The Problem:

> You just inherited the rights to N (1 <= N <= 20) previously unreleased songs recorded by the popular group Raucous Rockers. You plan to release a set of M (1 <= M <= 20) compact disks with a selection of these songs. Each disk can hold a maximum of T (1 <= T <= 20) minutes of music, and a song can not overlap from one disk to another.
>
> Since you are a classical music fan and have no way to judge the artistic merits of these songs, you decide on the following criteria for making the selection:
> - The songs on the set of disks must appear in the order of the dates that they were written.
> - The total number of songs included will be maximized.
>
> INPUT FORMAT
> - Line 1:	Three integers: N, T, and M.
> - Line 2:	N integers that are the lengths of the songs ordered by the date they were written.
> 
> OUTPUT FORMAT
> - A single line with an integer that is the number of songs that will fit on M disks.

## Analysis:

Ok. I'm not so sure that I want to include the whole problem into my post next time.

The first thing I noticed was that the numbers given are small. Very rarely do I see input numbers <= 20. This indicates that this problem can *probably* be brute-forced. But I chose to try to solve it with DP, especially because of the word "maximum". This had me thinking: how can I construct a solution from solutions for "smaller problems" (with smaller N, T, and M). We need to keep track how many disks are we using, how much time remains on the last disk we are processing, and which songs we "may" use (1...k). From that, we have a value for the maximum number of songs we may store in `maxSongs[i][j][k]`, where i, j, and k represent the values mentioned earlier respectively. 

We know that `maxSongs[0][j][k]` and `maxSongs[i][j][0]` must equal 0 because there's no way we can store songs with 0 disks or 0 songs. `maxSongs[i][j][k]` should be >= `maxSongs[i][j][k - 1]` (at least the number of songs stored as with 1 less available song) and `maxSongs[i][j][k]` should be >= `maxSongs[i - 1][T][k]` (if we start a new disk, we can store at least what we could store without the extra disk). We can add a song if the remaining time on the last disk >= the length of the song we want to add. Our answer will be `maxSongs[M][T][N]`.

## Implementation:
```cpp
#include <fstream>
#include <vector>

// Num Disks, Time Remaining On Disk, Num Songs Available
int maxSongs[21][21][21];

int main()
{
    std::ifstream fin("rockers.in");
    int n, t, m;
    std::vector<int> lengths{0};
    fin >> n >> t >> m;

    for (int i = 0; i < n; i++)
    {
        int len;
        fin >> len;
        lengths.push_back(len);
    }
    
    // Disks used
    for (int i = 1; i <= m; i++)
    {
        // Time remaining
        for (int j = 0; j <= t; j++)
        {
            // Songs available (first k songs in 'length')
            for (int k = 1; k <= n; k++)
            {
                // Start with calculated value for a "state" with tighter constraints (fewer disks / fewer songs / less time)
                // Take the maximum number of songs from those cases
                maxSongs[i][j][k] = std::max(maxSongs[i][j][k - 1], maxSongs[i - 1][t][k]);
                // Does having less time provide for higher song count?
                maxSongs[i][j][k] = std::max(maxSongs[i][j][k], maxSongs[i][std::max(j - 1, 0)][k]);
                
                // If we can add song k
                if (j >= lengths[k])
                {
                    // Add song k
                    // Remaining time = Available - Time To Use = j - lengths[k]
                    // First k - 1 songs as constraint to prevent repeats of songs (we are processing the kth song right now)
                    maxSongs[i][j][k] = std::max(maxSongs[i][j][k], maxSongs[i][j - lengths[k]][k - 1] + 1);
                }
            }
        }
    }
    
    std::ofstream fout("rockers.out");
    fout << maxSongs[m][t][n] << '\n';
}
```

And we are done! If there are any questions or clarifications needed, please comment below (if there are any readers).

Writing this took far longer than I anticipated. I'm not sure if I want to write another editorial/tutorial/whatever-you-call-it. I'll see if I have time.
