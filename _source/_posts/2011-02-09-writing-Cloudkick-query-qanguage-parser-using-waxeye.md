---
layout: post
title: Writing Cloudkick Query Language parser using Waxeye
published: true
tags:
  - programming
  - pypy
  - waxeye
  - parser generators
  - cloudkick
---

## [{{page.title}}][1]

First, some background.

At [Cloudkick][2] we use a special query language that allows users to filter a set of nodes on which the agent checks are applied. It is also used on the dashboard where it helps users find nodes quickly.

This query language is simple and powerful, allowing for the construction of complex queries with parenthesis and negated expressions. Specific details are available on [our wiki][3]. We encourage users to write simple queries as with most of the other things - simple is almost always better.

I helped develop the Python based parser which we currently use and powers the query language. This parser generates tokens from a user's query, using them to construct a new query which is then sent to Solr (among other things, we use Solr to index node documents). Details about Solr query generation are not important for this post, but I have mentioned them so you get the whole picture.

The current parser is not perfect, but for the most of the times, it works.

Recently we have stumbled across [Waxeye][4]. [Waxeye][4] is an easy to use parser generator, which among other languages also supports Python and JavaScript. This is perfect for us because adding auto-complete support for queries on the frontend is on our road-map. Thanks to Waxeye we will not need to develop another parser in JavaScript and will only have to maintain a single grammar file.

Since the parser generator looks easy to use and is pretty cool, I have dedicated myself to write a simple grammar for our query language which is available on [github][7]. Please keep in mind that what is available now is not the final result; it is just a quick thing I wrote while playing with Waxeye.

Since the parser makes a lot of function calls and there is also a lot of looping going on I was curious how it would perform with [PyPy][10] - after all there are a lot of function calls to in-line and some of the loops should probably be JIT-ed as well.

I decided to make a quick and very unscientific benchmark (although, I don't think my benchmarks are any worse than all the "NodeJS is faster than &lt;enter your favorite web server name here&gt;" benchmarks which have started to appear in the past year or so).

The code used to perform benchmarks is located [on github][5], [as is the parser code][6] generated by Waxeye.

PyPy needed on average **1.71 seconds** to complete **100 iterations** and CPython only needed **0.61 seconds**, which means, that in this case, PyPy was almost 3 times slower.

In the second benchmark I decided to ramp up the loop iteration count to **1000**. In this case, PyPy completed on average in **5.81 seconds** and CPython in **5.97 seconds** - here PyPy beat CPython, but only by a fraction.

The raw results can be found [here][8].

I can imagine the difference would be a lot bigger (in favor of PyPy) if I bump the iteration count even further, but I was only interested about a real world example and not about some unrealistic (in our case), high number of iterations.

So for our specific case and code, it looks like that using PyPy would not bring us speed improvements.

**Update**: Alex Gaynor has emailed and asked me which version of PyPy I have used, because he could not replicate the results using the latest nightly build.

He also said that changing the classes to a new style should speed things up, so I have made [another set of benchmarks][11], this time using the latest nightly build (**PyPy 1.5.0-alpha0**) and I have changed all the parser classes so now they inherit from the base object class.

So yes, when using the latest nightly build and new style classes, PyPy beat CPython for about **0.5 seconds**.

It is also nice see to see such a quick response from the PyPy team, so definitely thumbs for them.

**Update 2**: Here is a table with all the results:

<div style="text-align:center;">
<table style="width: 90%;margin-left:5%; margin-right:5%;">
<tr>
  <th># of iterations</th>
  <th>CPython 2.6.6 (new style classes)</th>
  <th>PyPy 1.4.1 (old style classes)</th>
  <th>PyPy 1.5.0-alpha0 (new style classes)</th>
</tr>
<tr>
  <td>100</td>
  <td><strong>0.557 seconds</strong></td>
  <td>1.719 seconds</td>
  <td>1.813 seconds</td>
</tr>
<tr>
  <td>1000</td>
  <td>5.476 seconds</td>
  <td>5.818 seconds</td>
  <td><strong>5.0134  seconds</strong></td>
</tr>
</table>
</div>
<br />

P.S. #1: I forgot to add that I have emailed the author with some questions about Waxeye and he was very kind and responsive and has answered all my questions and even posted some suggestions how to improve our parse - I thank him for that.

P.S #2. If you develop parsers as a hobby or you enjoy developing scalable solutions in Python or JavaScript (NodeJS), you should check out [this page][9].

[1]: {{ page.url }}
[2]: https://www.cloudkick.com
[3]: https://support.cloudkick.com/Monitor_Setup#Identifying_monitor_targets
[4]: http://waxeye.org/
[5]: https://gist.github.com/818383
[6]: https://gist.github.com/818385
[7]: https://github.com/Kami/cloudkick-query-language-parser/blob/master/query_language.waxeye
[8]: https://gist.github.com/818386
[9]: http://jobs.rackspace.com/search?q=%22san+francisco%22
[10]: http://codespeak.net/pypy/dist/pypy/doc/
[11]: https://gist.github.com/818386
