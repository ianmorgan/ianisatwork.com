---
layout: post
tags:   scala functional-programming monad
disqus: false 
---

# Explaining a monad in 10 steps

I thought i'd try and explain a monad, partly for the selfish reason that it would force me 
understand them better myself. Example code is in Scala.

## Step 1 - Categories are Containers 

Monads come from Category Theory. Categories are just containers with a set of common behaviours 
that should be true for all types and values. Of course formally proving they work for anything may be 
difficult, but in the real world we can normally rely upon intuition and common sense to decide what 
is good category. A list is a classic example.

{% highlight scala %}
scala > val names = List("John", "Paul", "Ringo", "George")
names: List[java.lang.String] = List(John, Paul, Ringo, George)

scala> names.size
res0: Int = 4
{% endhighlight %}


## Step 2 - Changing data in our container 

Our containers are values, that is they are immutable and so they can't be changed directly. And so 
to manipulate them we need to create a new container based on the old ones. Converting from one value to another
is a pure function:

<pre>
f: X -> Y 
</pre>

The container needs to provide its clients a way to pass in the function. In Scala this is by 
convention called the <code>map</code> method. 

{% highlight scala %}
scala> names.map(_.toUpperCase)
res1: List[java.lang.String] = List(JOHN, PAUL, RINGO, GEORGE)
{% endhighlight %}


## Step 3 - Composing small changes together  

Pure functions will compose in a repeatable way, so that a more complex function can be built. 

<pre>
f: X -> Y
g: Y -> Z
</pre>

is the same as 

<pre>
h: X -> Z 
</pre>

Which can also be drawn something like:
         
<img  src="http://upload.wikimedia.org/wikipedia/commons/1/1a/MorphismComposition-01.png">

<div class="footnote">
<a href="http://commons.wikimedia.org/wiki/File:Commutative_diagram_for_morphism.svg#mediaviewer/File:Commutative_diagram_for_morphism.svg">Commutative diagram for morphism</a>" by <a href="//commons.wikimedia.org/wiki/User:Cepheus" title="User:Cepheus">User:Cepheus</a> - Own work, based on <a href="//en.wikipedia.org/wiki/Image:MorphismComposition-01.png" class="extiw" title="en:Image:MorphismComposition-01.png">en:Image:MorphismComposition-01.png</a>. Licensed under Public domain via <a href="//commons.wikimedia.org/wiki/">Wikimedia Commons</a>.
</div>

The same is true of the map method. They may be chained together.

{% highlight scala %}
scala> names.map(_.toUpperCase).map(_.reverse)
res2: List[String] = List(NHOJ, LUAP, OGNIR, EGROEG)
{% endhighlight %}


## Step 4 - What happens if my function creates a new category 

So far we have used functions that just return simple types. But what if the function returned something 
more complex?

{% highlight scala %}
scala> val names = List("John L","Paul M") 
names.map(_.split(" "))
res3: List[Array[java.lang.String]] = List(Array(John, L), Array(Paul, M)
{% endhighlight %}

So this returns a list of lists. 

To make it simpler, it can be flattened out into one list

{% highlight scala %}
scala> names.map(_.split(" ")).flatten
res4: List[java.lang.String] = List(John, L, Paul, M)
{% endhighlight %}

This is such a common pattern it can be combined into a single method, <code> flatMap </code>

{% highlight scala %}
scala> names.flatMap(_.split(" "))
res5: List[java.lang.String] = List(John, L, Paul, M)
{% endhighlight %}


## Step 5: That was a monad, so whats special about it?

So, <code>flatMap</code> is fact a monad. The name is a little arbitrary. In Haskell its called bind, which is the name used in 
category theory. The clever thing is thats its joined two containers into. In this example its two lists 
and they are simply concatenated (flattened) together. Strictly speaking only <code>flatMap</code> must be defined as 
<code>map</code> is really just a special case of flatMap with a single value. But for ease of use both are usually provided.

There are two really useful things here.

Firstly, cardinality (does the function return zero, one or multiple values)
 becomes much less important which make designs more flexible. A similarity is joins in SQL, the syntax stays the same 
regardless as to how many joins exist. In fact, the traditional imperative style of a variable for a single value, 
a null for something that doesn't exist and a container that must be manually unpacked for multiple values is often 
unnecessarily cumbersome. 

Secondly many common problems can be modelled as a Category (aka container) with common behaviours. These include (using Scala's names) <code>Option</code> 
(holder for value that may be null) , a <code>Try</code> (holder for a computation that may have resulted in an exception) 
and <code>Future</code> (holder for result of an asynchronous computation)

## Step 6: Using an Option monad 

In Scala (and Java 8) the Option can be used to wrap a result that may be null, for example it could not be found in the database. 
Option has methods to read the value and so on. It also provides <code>map</code> and <code>flatMap</code>. So in a suitably trivial example, 
we might have a function that returns user names from a database:

{% highlight scala %}
scala> def findName (id:Integer) : Option[String] = if (id == 1) Some("John") else None 
res6: findName: (id: Integer)Option[String]
{% endhighlight %}

and another to get an address if we know the name

{% highlight scala %}
scala> def findAddress (name:String) : Option[String] = if (name.equals("John")) Some("Liverpool") else None 
res7: findAddress: (name: String)Option[String]
{% endhighlight %}

These can now be safely chained together without worrying about null pointers
{% highlight scala %}
scala> findName(1).map(findAddress(_))
res8: Option[Option[String]] = Some(Some(Liverpool))
scala> findName(2).map(findAddress(_))
res9: Option[Option[String]] = None
{% endhighlight %}


And <code>flatMap</code> will get rid the Option of Option complexity. The logic here is simple. If the container 
(i.e. the instance the flatMap is called on) is None, then
it must be a None. If not, the function is applied and its result returned.


<div class="highlight"><pre><code class="language-scala" data-lang="scala"><span class="n">scala</span><span class="o">&gt;</span> <span class="n">findName</span><span class="o">(</span><span class="mi">1</span><span class="o">).</span><span class="n bold">flatMap</span><span class="o">(</span><span class="n">findAddress</span><span class="o">(</span><span class="k">_</span><span class="o">))</span>
<span class="n">res8</span><span class="k">:</span> <span class="kt">Option</span><span class="o">[</span><span class="kt">String</span><span class="o">]</span> <span class="k">=</span> <span class="nc">Some</span><span class="o">(</span><span class="nc">Liverpool</span><span class="o">)</span>
<span class="n">scala</span><span class="o">&gt;</span> <span class="n">findName</span><span class="o">(</span><span class="mi">2</span><span class="o">).</span><span class="n bold">flatMap</span><span class="o">(</span><span class="n">findAddress</span><span class="o">(</span><span class="k">_</span><span class="o">))</span>
<span class="n">res9</span><span class="k">:</span> <span class="kt">Option</span><span class="o">[</span><span class="kt">String</span><span class="o">]</span> <span class="k">=</span> <span class="nc">None</span></code></pre></div>


Of course in this little example the effect seems a bit contrived but hopefully the benefit in more complex real world 
scenario is clear. In Scala the <code>Option</code> type has a number of other useful tricks, one of which is to be 
able to think of itself as a collection with either one value or empty. [Here](http://danielwestheide.com/blog/2012/12/19/the-neophytes-guide-to-scala-part-5-the-option-type.html) is 
good guide.


















 


