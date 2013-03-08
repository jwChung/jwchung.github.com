---
layout: post
tags : [TestCommon]
---
{% include JB/setup %}

Built-in Types
--------------
This part is for creating anonymous object of the [built-in types][] of .net framework.

#### Numerics ####
Creating an anonymous object for the numeric types as the below is determined
on an object of the `IIdGenerator` type,
which can be provided from the `Fixure.IdGenerators` property.
Each numeric type has own `IIdGenerator`
and the default algorithm for `IIdGenerator` is to return an random integer from inclusive 0 to exclusive 100
and the algorithm can be changed using the `Fixure.IdGenerators.Set` method.

{% highlight ruby linenos=table %}
def foo
  puts 'foo'
end
{% endhighlight %}

[Built-in types]: http://msdn.microsoft.com/en-us/library/ya5y69ds(v=vs.80).aspx
[Moq]: http://code.google.com/p/moq/

