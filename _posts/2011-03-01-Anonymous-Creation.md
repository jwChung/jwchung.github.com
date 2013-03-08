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
*   byte
*   sbyte
*   decimal
*   double
*   float
*   int
*   uint
*   long
*   ulong
*   short
*   ushort

{% highlight C# %}
		public void CreatingAnonymousOfNumericTypes()
		{
		    var fixture = new Fixture();

		    // Default IdGenerator
		    int intResult = fixture.Create<int>();
		    double doubleResult = fixture.Create<double>();

		    Assert.True(0 <= intResult && intResult < 100, "int");
		    Assert.True(0d <= doubleResult && doubleResult < 100d, "double");

		    // Specifying IdGenerator
		    fixture.IdGenerators.Set(typeof(int), new SequentialIdGenerator(0, 3));
		    fixture.IdGenerators.Set(typeof(double), new SequentialIdGenerator(0, 3));
		    Assert.Equal(0, fixture.Create<int>());
		    Assert.Equal(1, fixture.Create<int>());
		    Assert.Equal(2, fixture.Create<int>());
		    Assert.Equal(0, fixture.Create<int>());

		    Assert.Equal(0d, fixture.Create<double>());
		    Assert.Equal(1d, fixture.Create<double>());
		    Assert.Equal(2d, fixture.Create<double>());
		    Assert.Equal(0d, fixture.Create<double>());
		}
{% endhighlight %}


{% highlight python %}
    def yourfunction():
         print "Hello World!"
{% endhighlight %}

#### Character ####
An anonymous object for the character type is determined from a integer value -
0 to 'a', 1 to 'b', 2 'c', '25' to 'z' and '26' to 'a' etc.



	public void CreatingAnonymousOfCharacterType()
	{
		var fixture = new Fixture();

		fixture.Configurations.IdGeneratorProvider = () => new SequentialIdGenerator(0, 3);
		Assert.Equal('a', fixture.CreateAnonymous<char>());
		Assert.Equal('b', fixture.CreateAnonymous<char>());
		Assert.Equal('c', fixture.CreateAnonymous<char>());
		Assert.Equal('a', fixture.CreateAnonymous<char>());

		fixture.Configurations.IdGeneratorProvider = () => new SequentialIdGenerator(24, 27);
		Assert.Equal('y', fixture.CreateAnonymous<char>());
		Assert.Equal('z', fixture.CreateAnonymous<char>());
		Assert.Equal('a', fixture.CreateAnonymous<char>());
		Assert.Equal('y', fixture.CreateAnonymous<char>());
	}

[Built-in types]: http://msdn.microsoft.com/en-us/library/ya5y69ds(v=vs.80).aspx
[Moq]: http://code.google.com/p/moq/

