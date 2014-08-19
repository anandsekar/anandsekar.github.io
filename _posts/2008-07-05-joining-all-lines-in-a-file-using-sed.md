---
layout: post
title: "Joining All Lines in a File Using Sed"
modified:
categories: 
excerpt:
tags: []
image:
  feature:
date: 2008-07-05T17:07:29-07:00
---
Sed (Stream EDitor) is a very powerful stream editor to manipulate text files.
I needed an utility that would join all lines in a text file to a single line with “\n” as a separator so that I could easily cut and paste inline velocity templates into java code. Here is the final sed command !
{% highlight bash %}
sed -e :a -e ‘/$/N; s/\n/\\n/; ta’ [filename]
{% endhighlight %}
Explanation :
{% highlight bash%}
-e - denotes a command to be executed
:a - is a label
/$/N - defines the scope of the match for the current and the (N)ext line
s/\n/\\n/; - replaces all EOL with “\n”
ta; - goto label a if the match is sucessful
{% endhighlight %}
