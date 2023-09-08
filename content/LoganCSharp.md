---
title: Logan's C Sharp & Unity Coding Standards
---
*+Bonus Guide to Refactoring*

*Originally written by Logan Bowers* (**[bowers-l.github.io](https://bowers-l.github.io)**)

*Reformatted/edited for Obsidian Markdown by David Tran*

*Last Updated: 09/07/2023*

# Introduction

>[!info]+ What is this document?
>This document is a collection of standards, tricks, techniques, etc. that I have found useful when programming in the context of Unity projects. My aim when programming is to create scalable, maintainable, bug free, and realistic designs, similar to a software engineer in the professional world, so I have formatted the guide with this in mind.
>
>This guide was created in honor of Brandon Shockley, whose original guide can be found here: [Code Style - Google Docs](https://docs.google.com/document/d/1VPMxvzhVjg1bco6NsSNZ46y_ja8B7wJTUkdx2zS7dQ4/edit). Later sections also make reference to concepts taught by Robert C. Martin (Uncle Bobby) in this series: [Clean Code - Uncle Bob - all lessons - YouTube](https://www.youtube.com/playlist?list=PLmmYSbUCWJ4x1GO839azG_BBw8rkh-zOj)

>[!tip]+ Quartz Tips for Navigation and Reading
>In Quartz, feel free to use the outline tab located in the left sidebar to jump through the different sections. I recommend using this doc as a reference rather than trying to read through the whole thing at once.

> [!warning]+ DISCLAIMER
>This is not a guide on how to code or how Unity works. For that, I recommend looking up the several Unity tutorials on YouTube. [(1) Brackeys - YouTube](https://www.youtube.com/@Brackeys) is good to start out for implementing specific mechanics. [(1) Infallible Code - YouTube](https://www.youtube.com/@InfallibleCode) is good for learning design patterns and more scalable practices.

# C# and Unity Specific Standards
---
## Class Member Order

>[!info]+ Suggested Organization of a Class in Unity
>1. **Enums**
>2. **Inner Structs/Classes**
>3. **Constant/readonly fields**
>4. **Variable fields**
>		1. Serialized
>		2. Non-serialized, non-components
>		3. Non-serialized, components
>5. **Events**
>6. **Properties**
>7. **Constructors**
>8. **Unity callback methods** (Awake, Start, Update, OnEnable, etc.)
>9. **Non-Unity Methods**

>[!note] Style Reminder
>Separate these groups by a single space.

>[!info]+ Element Organization within Groups
>**Within each of these groups, order the elements using this convention, from increasing to decreasing priority:**
>1. Static, then non-static
>2. Access type
>		1. Public
>		2. Internal
>		3. Protected
>		4. private
>3. Overrides, then Event-handlers, then non-event handlers
>4. Highest to lowest abstraction level.
>		- Ex: If a private method uses another private method, the former should come before the latter
### Exceptions to Order
- For C# and Unity Events that have custom classes/structs for the arguments, put the class/struct directly before the event declaration.
	- For example:
       ```csharp
		public class OnPoweredArgs
		{
			public bool powered;
		}
		[SerializeField]
		public UnityEvent<OnPoweredArgs> OnPowered;
	   ```
	- If an internal class is long, consider putting it closer to the end of the file. 

## Naming Conventions

>[!info] General Conventions
>In general, follow this: [C# Coding Conventions | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
### Variable Names
   * For Unity serialized (inspector) fields that are private or protected, use camelCase
   * For non-serialized class member fields, use _camelCase (prefix the name with an _)
   * For public fields and properties, use PascalCase
   * Follow the usual conventions for everything else.

## Encapsulation
Fields and Methods of a class should be private by default until you need to make them protected/public. 

>[!danger] Unity Messages
>**This includes Unity messages (Awake, Start, Update, OnEnable, etc.).** A lot of people like to make these public even though they shouldn’t be called outside the class. These should ideally be private or protected (if you are using inheritance).
### Unity Inspector Variables
As you all probably know, you can make a MonoBehavior’s variable accessible in the inspector by making the variable public. However, for encapsulation this is not always ideal.

You can achieve the same thing with a private variable by adding \[SerializeField] before the declaration, like so.
```csharp
[SerializeField]
private AudioClip shotSound = null;
```
I suggest opting for this in favor of public variables, unless the variable actually needs to be public.

>[!warning]+ Default Values
>Also, you should give these variables default values. Otherwise, an annoying warning will pop-up in the logs every time the code compiles (this might no longer happen in the most recent versions).

## Composition over Inheritance

>[!info]+ Issues with Inheritance
>Out of the 3 main pillars of Object Oriented Programming: Encapsulation, Inheritance, and Polymorphism, Inheritance is probably the most hotly contested. This is mainly because having large/complex inheritance trees can get tricky when an inheriting class doesn’t use or doesn’t want to inherit everything from its base class. This is also why the Liskov-Substitution Principle exists in order to encourage programmers to write derived classes that are always substitutable for base classes (A rubber duck is not an actual duck).

>[!tip]+ Inheritance in Unity
>Unity heavily favors the use of Composition over Inheritance through its component system. One of the more annoying results of this happens when you attach a component, and then later change it to another class that is derived. Most of the time, Unity will remove/reset the references that you had set from the base class, forcing you to go into the scene and drag in all of the references that were already set before! Because of this drawback, I recommend keeping inheritance hierarchies small (no more than 3 levels deep, but usually you should only need 2). Write new components whenever possible to add behavior rather than trying to fit everything into a derived class. For example, if you have a bunch of entities in your game (like player’s enemies, NPCs, etc.) that behave in a similar way, but the player specifically needs to have user input to handle some things, you can put the input stuff in a separate component rather than trying to derive another class into the player.

>[!missing] Missing
>… ADD EXAMPLES
>
>*(not my document don't blame me for missing examples -David)*

## Functions and Procedures

>[!danger] What's the difference?
>A **function** has a non-void return type. A **procedure** returns void.
>
>A function that returns a non-void type should **ONLY** serve the purpose of returning that type and **NOTHING ELSE**. 
### Side Effects
This happens when you call a function that modifies the state of some variable or object (such as a field of the object you are in or a field of another object). Since functions only serve the purpose of returning a type, **functions should not have side effects**, that is, they should be immutable with respect to other objects, including the ones passed in as arguments. This is mainly a functional programming concept, but it is extremely useful for making the code easier to understand.

## Prefer Exception Throwing to Returning Bools

>[!warning]+ Issues with Returning Booleans
>Often times, it might be tempting to make a procedure return a boolean indicating if the procedure went through correctly. My main issue with doing this is that it is very easy to hide bugs by returning a boolean and not forcing the caller to deal with the possibility of the procedure failing. It can also be confusing why a method would return a boolean instead of void, and usually requires some kind of comment to explain the reasoning behind it.
>
>An easier way is to take advantage of languages’ exception systems. Throwing exceptions forces the caller to either handle the exception gracefully or expose the error to the debug console. 


# C# Tricks/Techniques
---
>[!note]+ 
>These are some things (little and big; mostly little) that I’ve found useful that might be helpful if you are a beginner or if you are coming from Java.

>[!tip]+ Use String Formatting Instead of Concatenation
>A lot of people might be tempted to do something like the following when trying to write a debug statement to the console:
>```csharp
>Debug.Log("Point is at (" + x + ", " + y + ", " + z + ")");
>```
>While this is usually fine, there is a much easier and more readable way to accomplish the same thing:
>```csharp
>Debug.Log($"Point is at ({x}, {y}, {z})");
>```
>This way, you keep the entire string in one place instead of separated by a bunch of + signs. There also may or may not be a slight increase in performance.

>[!tip]+ Use Properties Instead of Traditional Getters/Setters:
>There is a common pattern in Java of making every field in a class private, and then using public getter and setter methods to expose the field to other classes. This is so common that in some IDEs such as IntelliJ, there are buttons to auto-generate getters and setters for fields in a class. This can often make using these fields from other classes quite verbose. For example, let’s say you had a variable called “price” in a class called “Item” in Java. To add 5 to the variable, you might have to do the following:
>```csharp
>Item item = new Item();
>item.SetCount(item.GetCount() + 5);
>```
>In C#, there is a shortcut way to do this by using Properties, which are essentially special fields that implement their own getters and setters.
>
>An example of a C# property can be seen below. It is common to have one private field used in the class, prefixed with a \_, as well as a public property exposing the field. The public property is conventionally written with PascalCase (first letter capitalized). 
>```csharp
>private int _count;
>public int Count {
>	get 
>	{
>		return _count;
>	}
>	set 
>	{
>	 _count = value;
>	 }
>}
>```
>
>Assuming that this property is also in a class called Item, you can use the property like you would a normal field:
>```csharp
>Item item = new Item();
>item.Count += 5;
>```
>Properties are pretty powerful overall, and you can do pretty much anything in the “get” and “set” blocks as long as they return the right return type. I would recommend starting out with simple use cases however. 
>
>Technically, you don’t even need the backing variable, and you can omit the body of a get or set method to make it behave like a normal variable. For example:
>```csharp
>public int Count {
>	get;
>	set;
>}
>```
>You can also specify access modifiers like you would with any other variable. get and set are public by default unless otherwise specified. 
>```csharp
>public int Count {
>	get;
>	private set;
>}
>```
>
>There is also a shortcut way to do the above using the [=> operator - C# reference | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-operator#expression-body-definition) Expression Body Definition syntax:
>```csharp
>public int Count => _count;
>```
>This will implicitly create a property with a getter returning _count. 
>You can learn more about properties here: [Properties - C# Programming Guide | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties)

>[!tip]+ Dealing with Null References
>I hate null reference exceptions. I really do. Fun fact, the creator of null references, Tony Hoare, regrets making them a thing in languages:
>>[!quote]+
>>“I call it my billion-dollar mistake. At that time, I was designing the first comprehensive type system for references in an object-oriented language. My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn’t resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.”
>>
>>\- Tony Hoare
>
>There are some languages, like [Rust](https://doc.rust-lang.org/book/), that create systems such that null references are impossible. Because of typing and how the compiler works, you HAVE to check if something is not null before using it or else it won’t compile. C#, unfortunately, is not one of those languages, but that doesn’t mean we can’t be smart when dealing with this problem. 
>
>One way I’ve found to deal with null is to take advantage of exception throwing. If a method/function returns null, but this is definitely not supposed to happen, you can throw an exception instead. This is really useful for catching bugs, because you can leave a more specific message in the exception, pinpointing the exact location the bug occurs, rather than trying to decipher why a null reference exception was thrown later on in the stack. You can also wrap code in try-catch blocks if you think a null reference exception could occur. **Whatever you decide to do, creating systems that will pinpoint the exact location of bugs as you write code will save you literal hours of time. **
>>[!missing] Missing
>>… ADD EXAMPLES
>>
>>*(seriously logan? at this point i might just make my own -David)*

# Refactoring
---
>[!info] Introduction to Refactoring
>So far, I have talked about ways in which code can be improved to be more robust and bug-free. This next section will talk about a specific mode of programming that aims to clean codebases and reduce the potential for future bugs, rather than adding new features.
## Prerequisites
   * Knowledge of design patterns 
	   * There is a great book for this, even if it is a little old: 
		   * *Design Patterns: Elements of Reusable Object-Oriented Software*
	   * Most important to know for game dev. are probably Observer, Singleton, Factory, Flyweight, and State
   * SOLID Principles
	   * Single Responsibility Principle
	   * Open-Closed Principle
	   * Liskov Substitution
	   * Interface Segregation
	   * Dependency Inversion *(This one is especially helpful for decoupling/responsibility isolation)*
   * GRASP Principles
	   * I don’t use these as much, but low coupling/high cohesion are pretty important.

## What is Refactoring?
Refactoring is the process of changing the structure of a codebase without adding any new features or changing the functionality of the code. Overall, it encompasses a number of tasks that vary in terms of how much time they take/how risky they are to do.

## Why Refactor/Clean Code

>[!info]+ 3 Categories of Benefits for Refactoring
>The benefits of refactoring generally fall into 3 categories: 
>1. Readability - It will take less time for other people to read and understand your code.
>		- **This is important even if you are developing a solo project.**
>			1. For one, you don’t know if someone else might one day join your project, and they might have to deal with the mess that you created when it was just you.
>			2. Even if you know that absolutely no one else will work on your project, it is at the very least a service to your future self to write code that is human readable.
>2. Maintainability - It is easier to find and fix bugs in the codebase.
>3. Extendability - It is easier to add features as the codebase expands. The risk of breaking another feature or part of the codebase is mitigated significantly. 

>[!info]+ Even More Benefits!
>Refactoring can also yield benefits that go beyond the state of the codebase:
>1. Understanding a new Codebase - Large codebases can be very daunting at the beginning. Trying to add new features or fix bugs in a codebase that you don’t understand can be quite the challenge. While you could spend minutes or even hours reading over several script files, a more productive use of time in the beginning is to look for places in which the codebase can be refactored, and start with that. 
>2. Debugging - Often times, if I’m stuck on a bug or don’t understand how something works, I will start moving around parts of the code to understand how it works better. Sometimes, the bug will fix itself if I am focused on what the code is supposed to do and refactor it to fix that purpose. A lot of the time, however, just interacting with the codebase is enough to spark the intuition required to understand why a bug is happening and fix it on the spot. 
>3. Improved Programming Style - As you refactor more code, you will understand how codebases are ideally shaped. Your productivity will increase as a result since your code will be more clean naturally, and you will be able to spend less time refactoring/debugging and more time adding new features without breaking what already exists. You will also begin to see patterns of bugs and identify problem sources more easily while debugging.

>[!question]+ But won’t this slow down my productivity?
>Refactoring is a service to your future self, as well as the people working with you. While, at the end of the day, you are not making any changes that are visible to other users, you are **exponentially** decreasing the amount of time it takes to add new features and fix bugs in the future. This is essential for long term projects, and every serious software development organization does it to some extent, but it can also reap benefits for personal and open-source projects as well.
>
>If half of all of your time programming went into refactoring, you would only be sacrificing a constant amount of time short term for an exponential increase in productivity long term. O(2^n / 2) is still O(2^n)! You will also enjoy programming a lot more, I promise!

## How?
There’s a good video tutorial on a situation in which refactoring might be a good idea, and how it is done: [(1) SOLID Principles in Unity - YouTube](https://www.youtube.com/watch?v=QDldZWvNK_E)

Refactoring ideally should be an ongoing process. Some tasks should be performed in conjunction with or immediately after writing new code, while other tasks may require a time-boxed period to work on. In general, low risk tasks require less time/testing, while higher risk tasks require more time/testing.

>[!example]+ Some of the Tasks Involved in Refactoring and Their Risk
>* Renaming of tokens
>	* Risk: low when using a proper IDE function that auto-renames.
>	* Benefits: Readability
>* Reordering of elements in order to fit a certain standard.
>	* Risk: low
>	* Benefits: Readability
>* Removal of unused code and outdated comments
>	* Risk: low
>	* Benefits: Readability, Maintainability
>* Removal of boilerplate code (DRY)
>	* Risk: low to medium
>	* Benefits: Readability, Maintainability, Extendability
>* Extraction of Functions
>	* Risk: medium
>	* Benefits: Readability, Maintainability, Extendability
>* Extraction of Classes
>	* Risk: medium
>	* Benefits: Readability, Maintainability, Extendability
>* Changing the encapsulation of fields/methods
>	* Risk: low-high (really depends on use case)
>	* Benefits:  Maintainability, Extendability
>* Isolation of responsibility in classes (SRP)
>	* Risk: medium to high
>	* Benefits: Maintainability, Extendability

# More Specific Rules
---
## Clean Code

>[!abstract]+ Cleanup Rule
>You are allowed to make a mess, as long as you clean it up afterwards.
>>[!success]+ Reasoning
>>Our brains work in messy, disjointed patterns. If you are writing a function or class or module or whatever for the first time, you do not need to follow every coding standard. Much like writing the first draft of a novel or essay, your goal at the start is to get the ideas on paper (or in this case on your screen) and get the code working. Feel free to write a long method with multiple nested loops and conditionals that does everything you want it to, as long as you clean it up afterward.

>[!abstract]+ Boy Scout Rule
>If you check out code, you should push it back a little bit cleaner than when you found it.
>>[!success]+ Reasoning
>>If everyone did this, the codebase would naturally become cleaner as it evolved. Additionally, programmers should never be afraid of the code. If you are afraid to touch the code, then it might as well not work (because what are you going to do when it breaks?)

## Methods
### Single Responsibility
Methods should do a single thing. A method does a single thing when you can not meaningfully extract another method from the method. 

>[!abstract]+ Method Extraction Rule
>Once you are done writing a method, you should extract and extract parts of the method until you cannot meaningfully extract any more.

>[!abstract]+ Use Case 1 - Try/Catch and Other Exception Handling
>A try catch statement should be isolated. There should be no prefix or suffix code, and the code inside the try block should be extracted into another method. This is because exception handling counts as one thing. 

### Method Length
A method should be able to fit on your screen. In general, methods should be about 20 lines max.

### Method Abstraction
The contents of a method should all be at the same level of abstraction. If a high level statement is followed by a collection of low level statements, the low level statements should probably be extracted into another method. 

## Renames
Renames are useful for enhancing readability. Often times, a confusing comment attached to a method, field, or local variable can be removed just by renaming the thing (see [[LoganCSharp#Comments|Comments]]).

>[!warning] What to Rename
>Renaming private tokens is risk-free for the most part, whereas protected tokens might take some thought, but are usually ok as well. **DO NOT rename public tokens** under normal circumstances, because if you do, you risk breaking a lot of references not only in your code but others as well (such as with Unity serialized object references in the inspector).

## Comments
Comments are a use case for when the code fails to explain itself. **You should never write comments for the sake of writing comments.**

**TODOs** - As soon as you push a TODO comment, the TODO becomes a TODON’T. 

**Outdated Comments** - A lot of comments become outdated over time as the codebase changes. If you ever see a comment that looks like it doesn’t make sense at all in context, remove it. 

>[!example]+ Some Alternatives to Writing Comments
>* Refactor the code to be more readable, using the methods described in the above section.
>* For comments that separate a method into steps/explain what a series of statements in a method does, extract these parts of the code into separate functions and use the comment (or a variation) as the name of the function (this technique is very useful for readability AND maintainability, don’t underestimate it).
>* In Unity, for comments that explain a serialized value (those variables that appear in the Unity inspector), use the [Tooltip] Attribute and stick the comment in as the string. This way, it will be visible in the code and be displayed as a tooltip in the inspector. 

## Naming Conventions
Good names are the secret ingredient to writing code that explains itself, as well as reducing the amount of comments necessary in the code (see [[LoganCSharp#Comments|Comments]]).

>[!abstract]+ Variable Name Length Rule
>The smaller the scope, the shorter the variable name.
>>[!info]+ Explanation
>>Variables with larger scope are more difficult to remember what they do, so they should have more descriptive names to aid the reader. 1 letter variable names should only be used for very small scopes (~3 lines), since they are discarded immediately, and the use can be inferred from context.

>[!abstract]+ Function Name Length Rule
>The larger the scope, the shorter the function name.
>>[!info]+ Explanation
>>Functions that have a larger scope (public APIs) are used more often and work in a more abstract sense. Functions with a small scope (private methods) have very specific use cases, so they should have longer names in order to describe this specificity.

### Access Types
Favor access types that are as restrictive as possible, until you need broader access types. 

Methods that are extracted should be default private until you need that method in another context.

# References
---
[Clean Code - Uncle Bob - all lessons - YouTube](https://www.youtube.com/playlist?list=PLmmYSbUCWJ4x1GO839azG_BBw8rkh-zOj)

Brandon’s (RIP) Code Conventions: [Code Style - Google Docs](https://docs.google.com/document/d/1VPMxvzhVjg1bco6NsSNZ46y_ja8B7wJTUkdx2zS7dQ4/edit)

The Rust Book: [The Rust Programming Language - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/stable/book/)

Everything else: I made it the \**** up. *(i sure hope not -David)*
