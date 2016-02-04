---
layout: post
title:  "Data Science With Command Lines"
date:   2016-02-01
categories: linux
---
There are useful command lines to do basic data analysis.

A simple tsv file
{%highlight bash%}
cat income.tsv

tom 100 200
tom 1000 100
john 2000 10
mike 5000 3000
jim 2000 200
{%endhighlight%}

Sorting

Sort by second column acending
{%highlight bash%}
sort -nk2 income.tsv

tom 100 200
tom 1000 100
jim 2000 200
john 2000 10
mike 5000 3000
{%endhighlight%}
Descending

{%highlight bash%}
sort -nk2 -r income.tsv

mike 5000 3000
john 2000 10
jim 2000 200
tom 1000 100
tom 100 200
{%endhighlight%}

Frequency

Frequency of 2nd column
{%highlight bash%}
cut -f2 income.tsv | sort | uniq -c

1 100
1 1000
2 2000
1 5000
{%endhighlight%}
The purpose of sorting is to help uniq command, and we do lexical sorting (the default).

You can use awk too

{%highlight bash%}
awk '{counts[$2] = counts[$2] + 1; } END { for (val in counts) print val, counts[val]; }' income.tsv

2000 2
1000 1
100 1
5000 1
{%endhighlight%}

Sum of a column
{%highlight bash%}
awk '{ SUM += $2} END { print SUM }' income.tsv
10100
{%endhighlight%}

Awk is a rather powerful tool for not only text process but also computation.

Percentage

{%highlight bash%}
awk 'NR==FNR{ sum += $2; next;} {print $0"\t"$2/sum }'  income.tsv

tom 100 200 0.00990099
tom 1000 100 0.0990099
john 2000 10 0.19802
mike 5000 3000 0.49505
jim 2000 200 0.19802
{%endhighlight%}

Histogram

So now you can make an histogram!
