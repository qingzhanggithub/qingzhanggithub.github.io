---
layout: post
comments: true
title:  "Zombie Dice"
date:   2016-02-01
categories: probability 
---
Zombie dice is such a fun game that it makes me want to model it. Lets say  \\(C\\) is 3-dimension vector denoting the colors, the indices 0,1,2 denotes  GREEN, YELLOW and RED,  and $$S$$ is also a 3-dimension vector representing the pattens on the surface indices (0,1,2) = (BRAIN, FOOT, SHOT).

Current Draw
============
We denote the current draw on a matrix  \\(D=C^TS\\), which is nothing fancy but just for marking the draw outcome. For example first dice is a (G, F), we do D[0,1]++. We can easily compute how many dices with specific color by summing the rows (sum(D[i,:])), and patterns by summing the columns sum(D[:,j]).  Therefore we will know colors in the box for next draw, and how many dices we need to draw ( 3- n_foot).

{%highlight python%}
# Current draw 1 green brain, 1 yellow brain and 1 red brain, denoted by
 np.asarray([[1,0,0],[1,0,0], [1,0,0]])
{%endhighlight%}

Next Draw
=========

#### Colors

First we enumerate all possible k-dice draws (k=3-n_foot) without considering what colors are available. For example next draw could be \\(colors_i\\) =[0,1,2] , meaning 0 G, 1 Y and 2 R.  Subsequently we compute the probability of such a draw given the dices we have in the box.

$$Prob(colors_{i}^{'})=Prob(g_i, y_i, r_i) = \frac{nCr(g, g_i) + nCr(y, y_i)+nCr(r, r_i)}{nCr((g+y+r), (g_i+y_i+r_i))}$$

The probability of [Y, R, R] = 0.113.
We add the foot of current draw to the next draw, which is \\(colors_i = colors_{i}^{'} + foot \\). We then denote the 3 dices with a 3*3 probability table T. Each row represents one dice, and each column is the probabilities of being B,F,S with this dice. For example the table T for draw [0,1,2] is

{%highlight bash%}
[0.3333, 0.3333, 0.3333] # yellow dice
[0.1666, 0.3333, 0.5] # red dice
[0.1666, 0.3333, 0.5] # red dice
{%endhighlight%}

#### Surfaces (Patterns)

Now we compute the probabilities of having a certain surfaces given the new draw.  We first enumerate all possible surface combinations, such as [2,0,1]  meaning 2 brains (B) and 1 shot (S) . These are all possible dice assignment  for [2,0,1]
{%highlight bash%}
[0, 0, 2] # assign0 meaning dice0 and 1 are for B and dice2 for S 
[0, 2, 0] # assign1 meaning dice0 and 2 are for B and dice1 for S
[2, 0, 0] # assign2 meaning dice0 for S and dice1 and 2 for B
{%endhighlight%}

So the probability of assign0 is T[0,0] * T[1,0] * T[2,2] = 0.3333 * 0.1666 * 0.5 = 0.0278, and for assign1 and assign 2 are 0.0278 and 0.0093
Then for each combination we enumerate the ways that it occur with the dices we draw, and the probability of having a particular surface \\(suf_i\\) given the draw \\(colors_j\\) is

$$Prob(surf_i | colors_j)= \sum_{k}{Prob(surf_i | assign_k)}$$

$$assign_i$$ is an assignment of the dices to the colors. 
Note that we assume $$ Prob( assign_j | colors_k) = 1/(n\_assignments) $$, which is the same for all the color configuration. Therefore we do not use it here. The probability of this example $$Prob([2,0,1]|[0,1,2])$$ is 0.0648 ( 0.0278+ 0.0278 + 0.0093).

Probabilities for every surface assignment  of  \\(colors\\) [0, 1, 2] are
{%highlight bash%}
[0, 0, 3]   :       0.0833
[0, 1, 2]   :       0.1944
[0, 2, 1]   :       0.1481
[0, 3, 0]   :       0.0370
[1, 0, 2]   :       0.1389
[1, 1, 1]   :       0.2037
[1, 2, 0]   :       0.0741
[2, 0, 1]   :       0.0648
[2, 1, 0]   :       0.0463
[3, 0, 0]   :       0.0093
{%endhighlight%}

Putting it All Together
==========
The probability of getting a particular surface combination from all possible draws (colors) will be

$$Prob(surf_i) = \sum_{j}{Prob(colors_j) * Prob(surf_i | colors_j)}$$

Given the current draw , the probabilities of all possible surfaces combinations of all possible next draw are
{%highlight bash%}
[0, 0, 3]       :       0.0344
[0, 1, 2]       :       0.1114
[0, 2, 1]       :       0.1190
[0, 3, 0]       :       0.0423
[1, 0, 2]       :       0.1196
[1, 1, 1]       :       0.2534
[1, 2, 0]       :       0.1349
[2, 0, 1]       :       0.1339
[2, 1, 0]       :       0.1431
[3, 0, 0]       :       0.0508
{%endhighlight%}

Code
=====
I have implemented the computation [here](https://github.com/qingzhanggithub/coding/blob/master/zombie_dice.py) .


{% if page.comments %} 
<div id="disqus_thread"></div>
<script>
    /**
     *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
     *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
     */
    /*
    var disqus_config = function () {
        this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
        this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() {  // DON'T EDIT BELOW THIS LINE
        var d = document, s = d.createElement('script');
        
        s.src = '//slowlyspeedingcode.disqus.com/embed.js';
        
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
{% endif %}

