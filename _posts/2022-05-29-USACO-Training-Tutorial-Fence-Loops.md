---
layout: post
title: USACO Training Tutorial - Fence Loops
description: Tutorial for USACO Training - Fence Loops
date: 2022-05-29 17:32:00 -0000
tags: Competitive_Programming
---

Yet another USACO editorial: Fence Loops. Enjoy!

## The Problem:

> The fences that surround Farmer Brown's collection of pastures have gotten out of control. They are made up of straight segments from 1 through 200 feet long that join together only at their endpoints though sometimes more than two fences join together at a given endpoint. The result is a web of fences enclosing his pastures. Farmer Brown wants to start to straighten things out. In particular, he wants to know which of the pastures has the smallest perimeter.
> 
> Farmer Brown has numbered his fence segments from $1$ to $N$ ($N$ = the total number of segments). He knows the following about each fence segment: 
> - the length of the segment
> - the segments which connect to it at one end
> - the segments which connect to it at the other end.
> Happily, no fence connects to itself.
> Given a list of fence segments that represents a set of surrounded pastures along with their lengths, write a program to compute the smallest perimeter of any pasture. 
>
> INPUT FORMAT
> - Line $1$:	$N$ $(1 <= N <= 100)$
> - Line $2..3*N+1$: $N$ sets of three line records:
>   - The first line of each record contains four integers: s, the segment number $(1 <= s <= N)$; $Ls$, the length of the segment $(1 <= Ls <= 255)$; $N1s (1 <= N1s <= 8)$ the number of items on the subsequent line; and $N2s$ the number of items on the line after that $(1 <= N2s <= 8)$.
>   - The second line of the record contains $N1$ integers, each representing a connected line segment on one end of the fence.
>   - The third line of the record contains $N2$ integers, each representing a connected line segment on the other end of the fence.
> 
> OUTPUT FORMAT
> - The output file should contain a single line with a single integer that represents the shortest surrounded perimeter.

## Analysis:

Represent each segment as a node of a graph and the length of each segment is the weight of the edge to connecting segments. Run Dijkstra's algorithm from each segment to determine the shortest path (longer than 0) from a node to itself. This produces the minimum perimeter.

## Implementation:
```cpp
#include <fstream>
#include <iostream>
#include <queue>
#include <unordered_set>

// Segments the segment connects to are seperated by which end they connect to, {front, end}
std::pair<std::unordered_set<int>, std::unordered_set<int>> connectedTo[101];
int length[101];

int distance[101];
bool processed[101];

int main()
{
    std::ifstream fin("fence6.in");
    int n;

    fin >> n;

    for (int i = 0; i < n; i++)
    {
        int s, l, n1, n2, segment;

        fin >> s >> l >> n1 >> n2;

        length[s] = l;

        for (int j = 0; j < n1; j++)
        {
            fin >> segment;

            connectedTo[s].first.insert(segment);
        }

        for (int j = 0; j < n2; j++)
        {
            fin >> segment;

            connectedTo[s].second.insert(segment);
        }
    }

    int minPerimeter = INT32_MAX;
    
    // For each node, run Dijkstra's algorithm
    for (int i = 1; i <= n; i++)
    {
        // Reset arrays needed for Dijkstra's algorithm
        for (int j = 1; j <= n; j++)
        {
            distance[j] = INT32_MAX / 4;
            processed[j] = false;
        }
        
        // Start
        std::priority_queue<std::pair<std::pair<int, int>, std::pair<int, int>>>
            toProcess; // {-Dist, segment id}, {prev, first}
        toProcess.push({{0, i}, {-1, -1}});

        while (!toProcess.empty())
        {
            int dist = -toProcess.top().first.first;
            int segmentID = toProcess.top().first.second;
            int prev = toProcess.top().second.first;
            int first = toProcess.top().second.second; // First node visited after starting node

            toProcess.pop();

            if (processed[segmentID])
            {
                continue;
            }

            processed[segmentID] = true;
            
            // If the first node is set, we can check to see if the current node connects to the 
            // end of the initial segment that we DID NOT start from
            if (first != -1)
            {
                // Get nodes reachable from the node we started from but NOT from the end of the segment
                // we began from
                std::unordered_set<int> &segmentsBeforeReturning =
                    connectedTo[i].first.find(first) == connectedTo[i].first.end() ? connectedTo[i].first
                                                                                   : connectedTo[i].second;
                
                // If current node is connected to the other end of the segment we started from
                // we have a perimeter!
                if (segmentsBeforeReturning.find(segmentID) != segmentsBeforeReturning.end())
                {
                    minPerimeter = std::min(minPerimeter, dist + length[segmentID]);
                    continue;
                }
            }
            
            // Nodes that we can reach from here (not from the end we came from)
            std::unordered_set<int> &next =
                connectedTo[segmentID].first.find(prev) == connectedTo[segmentID].first.end()
                    ? connectedTo[segmentID].first
                    : connectedTo[segmentID].second;

            for (int connectedSeg : next)
            {
                if (distance[connectedSeg] > dist + length[segmentID])
                {
                    distance[connectedSeg] = dist + length[segmentID];
                    toProcess.push(
                        {{-distance[connectedSeg], connectedSeg}, {segmentID, (first == -1) ? connectedSeg : first}});
                }
            }
        }
    }

    std::ofstream fout("fence6.out");
    fout << minPerimeter << '\n';
}
```

And we are done! If there are any questions or clarifications needed, please comment below (if there are any readers).
