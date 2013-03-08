---
layout: post
tags : [TestCommon]
---
{% include JB/setup %}

Introduction
------------
Sometime we need to care about some attriutes of an anonymous object to be constructed,
especially when constructing [SUT][].
Using the [Anonymous Creation][] feature of TestCommon is not satisfied to specify some attributes,
so we introduces the Parameterized Anonymous Creation feature.

There are many steps to construct an parameterized anonymous object in TestCommon,
which are implemented in the methods of the `ISpecimenBuilder<T>` interface.

1. The first step is to specify access modifiers of constructors to be used to delimit
when a certain object is constructed using `WithCtorAccessModifiers(AccessModifiers)`.
2. The second step specifies the way the target constructor is selected.
One way is to use a query to select the target constructor(`QueryCtor(Func<IEnumerable<ConstructorInfo>, ConstructorInfo>)`).
The other way is to use the `QueryModestCtor()` or `QueryGreedyCtor()` method.
`QueryModestCtor()` selects a modest contructor and on the contrary `QueryGreedyCtor()` selects a greedy constructor.
All these ways in this step are applied after specifying access modifiers of constructors.
3. The third step is to inject parameters using the `Inject(...)` methods.
The methods does not only pass parameter values to a constructor but also affect to fiter a target constructor.
If a parameter value is specified, constructors which can takes the parameter value have high priority for the target constructor.
If there is not applied parameter value specified, the exception for this is thrown.
It means that all the specified parameter values should be applied to the target constructor.
One important thing is that this step is related to the [Constructor Injection][].
4. This step is for setting a property value through `With(...)`, `Without(...)`, `WithAll(...)` and `WithoutAll(...)` methods.
A property value can be automatic anonymous object or specified with a certain value. This step is related to [Setter Injection][].
5. Sometime, we need to call some methods when constructing a parameterized anonymous object
like the `Initialize()` method or the methods to set attributes instead of a setter of property.
6. The last step is to invoke the target constructor with parameter values. There are the two method in the `ISpecimenBuilder<T>` to constructor an object.
The one is `CreateAnonymous()` and the other is `CreateMockedAnonymous(...)`.
`CreateAnonymous()` returns the result from invoking the target constructor,
but `CreateMockedAnonymous(...)` returns a mocked instance for the target type using [Moq][]
and the parameter values for `CreateMockedAnonymous(...)` can be passed in to set attributes of the mock such as `DefaultValue`, `CallBase` and `MockBehavior`.

Keep in mind that in the above steps, any higher step than a current step can be called, but the lower step cannot -
eg. the step 1 is followed by the step 6 but the step3 cannot be called as next of the step4, the step5 and the step6.

The `Person` class is used as a test type to explain more detail.

```c#
public class Person
{
	public string Name { get; private set; }
	public int? Age { get; set; }

	public Person()
	{
	}

	public Person(string name)
	{
		Name = name;
	}

	private Person(string name, int age)
	{
		Name = "Used Private Constructor";
	}
}
```

Select Constructor
------------------
The `CtorAccessModifiers` and `IsGreedy` properties on the `Fixture.Configurations` property can be used to construct a parameterized anonymous object.
As `CtorAccessModifiers` is `NotPrivate` and `IsGreedy` is false as default, the constructor(`.ctor()`) is used to construct an object of the `Person` type as the following.

```c#
[Fact]
public void SelectingConstructorFromConfitgurations()
{
	var fixture = new Fixture();

	var result = fixture.Build<Person>().CreateAnonymous();

	Assert.Null(result.Name);
}
```

If the `WithCtorAccessModifiers(AccessModifiers)` method is called, it ignores the `Fixture.Configurations.CtorAccessModifiers` property.

```c#
[Fact]
public void ExampleOfWithCtorAccessModifiers()
{
	var fixture = new Fixture();

	var result = fixture.Build<Person>()
		.WithCtorAccessModifiers(AccessModifiers.Private)
		.CreateAnonymous();

	Assert.Equal("Used Private Constructor", result.Name);
}
```

The `QueryGreedyCtor()` method can be called to use a construtor having more parameters
and the `QueryModestCtor()` is the opposite case.
These methods also ignores the `Fixture.Configurations.IsGreedy` property.

```c#
[Fact]
public void ExampleOfQueryGeedyCtor()
{
	var fixture = new Fixture();
	fixture.Configurations.IsGreedy = false;

	var result = fixture.Build<Person>()
		.QueryGreedyCtor()
		.CreateAnonymous();

	Assert.NotNull(result.Name);
	Console.WriteLine(result.Name); // "name: d1e85e42-6316-4e17-941a-b352c3e743c8";
}
```

The `QueryCtor(Func<IEnumerable<ConstructorInfo>, ConstructorInfo>)` method is used to specify the target constructor explicitly.

```c#
[Fact]
public void ExampleOfQueryCtor()
{
	var fixture = new Fixture();

	var result = fixture.Build<Person>()
		.WithCtorAccessModifiers(AccessModifiers.All)
		.QueryCtor(cis => cis.Where(ci => ci.GetParameters().Length == 2).Single())
		.CreateAnonymous();

	Assert.Equal("Used Private Constructor", result.Name);
}
```

Inject Parameter
----------------
The injecting parameter value naturally is to pass a parameter value to a constructor but also affects to select a target constructor.
As the below code snippet, if `Inject(name)` code is not called, the `.ctor()` constructor is used as the target constructor, so the `Name` property is null.
This means that the condtion of the target constructor selection is to select the modestest constructor among not private constructors.
However, as `Inject(name)` is used, the condition is changed to the modestest , not private and having string parameter value.
As the result, the `.ctor(string)` consturctor is used as the target constructor.

```c#
[Fact]
public void ExampleOfInjectParameterValue()
{
	var fixture = new Fixture();

	var person = fixture.Build<Person>().CreateAnonymous();
	Assert.Null(person.Name);

	// inject parameter value

	const string name = "anonymous string value";

	var result = fixture.Build<Person>()
		.Inject(name)
		.CreateAnonymous();

	Assert.Equal(name, result.Name);
}
```

Inject Property
---------------
The injecting property value is to set value to a property using a `With(...)` method as the following example.

```c#
[Fact]
public void ExampleOfInjectingPropertyValue()
{
	var fixture = new Fixture();

	const string name = "anonymous string value";

	var result = fixture.Build<Person>()
		.With(x => x.Name, name)
		.CreateAnonymous();

	Assert.Equal(name, result.Name);
}
```
Using the `Fixture.Configurations.AutoPropertyAccessModifiers` property, anonymous objects are set automatically for the setters satisfied with the `AccessModifiers` value
and then if a `With(...)` / `Without(...)` method is called,
the value specified from the method overrides the previous values set automatically or from `With(...)` / `Without(...)` method.

```c#
[Fact]
public void ExampleOfInjectingPropertyValue2()
{
	var fixture = new Fixture();
	fixture.Configurations.AutoPropertyAccessModifiers = AccessModifiers.All;

	const string name1 = "anonymous string value1";
	const string name2 = "anonymous string value2";

	var result = fixture.Build<Person>()
		.With(x => x.Name, name1)
		.With(x => x.Name, name2)
		.CreateAnonymous();

	Assert.Equal(name2, result.Name);
}
```

Inject Behavior
---------------
We can call a method while constructing a parameterized anonymous object.
It is possible that to do [Setter Injection], we need to call a method intead of a setter.
To show this scenairo, the following method is needed to add to the `Person` class.

```c#
public void InjectName(string name)
{
	Name = name;
}
```

Through calling the `Do(...)` method, The `Name` property of the `Person` class can be set with an anonymous string or a custom string value.

```c#
[Fact]
public void ExampleOfInjectingBehavior()
{
	var fixture = new Fixture();

	const string customName = null;

	var result = fixture.Build<Person>()
		.With(x => x.Name, null)
		.Do(sut => sut.InjectName(customName))
		.Do((Person sut, string anonymousName) => sut.InjectName(anonymousName))
		.CreateAnonymous();

	Assert.NotNull(result.Name);
	Console.WriteLine(result.Name); // "e19e276f-de9a-43d2-a402-be9fb521bb21"
}
```

Construct
---------
Using the `CreateAnonymous()` method, not mocked object is constructed.

```c#
[Fact]
public void ExampleOfConstructingNotMockedType()
{
	var fixture = new Fixture();

	var result = fixture.Build<Person>()
		.CreateAnonymous();

	// The Mock.Get<T>(T mocked) method is to convert a object to a mocked object.
	Assert.Throws<ArgumentException>(() => Mock.Get(result));
}
```

However, the `CreateMockedAnonymous(...)` method returns a mocked object.

```c#
[Fact]
public void ExampleOfConstructingMockedType()
{
	var fixture = new Fixture();

	var result = fixture.Build<Person>()
		.CreateMockedAnonymous();

	Assert.DoesNotThrow(() => Mock.Get(result));
}
```

The mock attributes, such as `DefaultValue`, `CallBase` and `MockBehavior`, are first affected by the `Fixture.Configurations.MockSettings` property.
The default value of the `MockSettings` property is false as `CallBase`, `Mock` as `DefaultValue` and `Loose` as `Behavior`.
These attributes can be ignored with the parameter values passed in the `CreateMockedAnonymous(...)` method.
If the null value as the mock attributes is passed in the `CreateMockedAnonymous(...)` method,
the mock attibutes of the `Fixture.Configurations.MockSettings` property are used to construct a mock object.

```c#
[Fact]
public void ExampleOfConstructingMockedTypeWithMockAttributes()
{
	var fixture = new Fixture();
	
	// the default mock attributes.

	var personWithDefault = fixture.Build<Person>().CreateMockedAnonymous();
	var personMockWithDefault = Mock.Get(personWithDefault);
	Assert.False(personMockWithDefault.CallBase);
	Assert.Equal(DefaultValue.Mock, personMockWithDefault.DefaultValue);
	Assert.Equal(MockBehavior.Loose, personMockWithDefault.Behavior);

	// custom mock attributes.
	var personWithCustom = fixture.Build<Person>().CreateMockedAnonymous(callBase:true, behavior:MockBehavior.Strict);
	var personMockWithCustom = Mock.Get(personWithCustom);
	Assert.True(personMockWithCustom.CallBase);
	Assert.Equal(DefaultValue.Mock, personMockWithCustom.DefaultValue);
	Assert.Equal(MockBehavior.Strict, personMockWithCustom.Behavior);
}
```


[Anonymous Creation]: 1.-Anonymous-Creation
[SUT]: http://xunitpatterns.com/SUT.html
[Constructor Injection]: http://xunitpatterns.com/Dependency%20Injection.html#Constructor%20Injection
[Setter Injection]: http://xunitpatterns.com/Dependency%20Injection.html#Setter%20Injection
[Moq]: http://code.google.com/p/moq/