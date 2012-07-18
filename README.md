Console
====================

## Console

Console provides a consistent cross platform logging and debugging API. The API 
is consistent with that of the WebKit console. Developers can:

* redirect calls to "trace" to the console
* log messages are different levels
* print stack traces
* inspect structured objects in the WebKit console
* log multiple values in a single trace
* make boolean assertions that trigger errors when they fail
* count invocations at positions within their code
* visually group log messages
* use timers to measure program performance
* start and stop profiling where available
* log Haxe objects in way that is easily inspectable in the WebKit console
* insert breakpoints (on supported platforms)

Console currently supports a number compiletime modes:

* On system platforms (neko/php/cpp) console prints to stdout, colorising output
* In JS/Flash running in WebKit browser console prints to WebKit console
* In JS running in non-webkit browser console prints to an element in the DOM
* In Flash running standalone or in a non-WebKit browser console prints to an 
  on screen logging panel.
* When the compiler flag `no_console` is set, the console inlines null 
  expressions, removing any runtime overhead from logging.

## Caveats

When logging to the WebKit console (JavaScript in a WebKit browser) redirected 
traces will not display the correct source position in the console panel. The 
position of the Console.haxeTrace will be shown instead. This is a limitation 
of the Haxe logging API.

Flash targets can talk to the WebKit console over external interface, but 
messages will not show the correct position. Use a ConsoleView if you 
require log positions.

## Example Usage:

To enable the console, call:

	Console.start();

To log a message at a named level:

	trace("log", "Something happened.");
	trace("info", "Something interesting happened.");
	trace("debug", "WHY WON'T YOU WORK!!");
	trace("warn", "I didn't test this code at all...");
	trace("error", "Something bad happened.");

Messages default to LogLevel.log:

	trace("Logtastic");

Log multiple values using trace. In WebKit console each value is inspectable 
(rather than concatenating as a string)

	trace("string", 10, {key:"value"});

You can also call the logging API directly:

	Console.log("better than bad, it's good!);

Errors will print a message, then a stack trace:

	Console.error("Epic fail");

	// outputs

	Error: Epic fail
	@ SomeClass.someMethod:20
	@ SomeOtherClass.someOtherMethod:48

You can also use `Console.trace` to print a stack trace without triggering 
an error:

	Console.trace();

	// outputs

	Stack trace:
	@ SomeClass.someMethod:20
	@ SomeOtherClass.someOtherMethod:48

Assertion will print a message and stack trace, then throw and exception when 
the condition is false:

	Console.assert(foo == false, "foo is not false!!");

	// outputs

	Assertion failed: foo is not false
	@ SomeClass.someMethod:20
	@ SomeOtherClass.someOtherMethod:48

To count the number of times a line of code is invoked:
	
	function someCode()
	{
		Console.count("apples");
	}
	
	someCode();
	someCode();

	// outputs

	apples: 1
	apples: 2

Note that counters are identified by their position, not their label:

	Console.count("apples");
	Console.count("apples");

	// outputs

	apples: 1
	apples: 1

To insert a breakpoint in JavaScript or Flash:

	Console.enterDebugger();

Group log messages together visually:

	Console.group("Group");
	Console.log("grouped log");
	Console.group("Nested group");
	Console.warn("nested warn");
	Console.groupEnd();
	Console.groupEnd();

	// output

	Group
	  grouped log
	  Nested Group
	    nested warn

Timers can be used to quickly monitor runtime performance:
	
	Console.time("munging");
	for (i in 0...4200000000) munge();
	Console.timeEnd("munging");

	// output

	munging: 2410ms

### JavaScript/Webkit only

The WebKit profiler can be started and stopped:

	Console.profile("performance");
	var f = function(x) { return x * x; }
	var x = f(10);
	Console.profileEnd("performance");

And markers added to the WebKit timeline view:

	Console.markTimeline("finished");

### Type conversion

As the WebKit console has user friendly structure inspection, console will try 
to convert native Haxe values into something inspectable where possible:

	// inspectable XML dom
	Console.log(Xml.parse("<this><is><some><xml/><with/><elements/></some></is></this>"));

	// enums and enums with parameters
	Console.log(value1);
	Console.log(value2(33));
	Console.log(value3({oooh:"fancy"}));

	// hashes
	var hash = new Hash<String>();
	hash.set("hashy", "goodness");
	Console.log(hash);

	var intHash = new IntHash<String>();
	intHash.set(10, "int hashy goodness");
	Console.log(intHash);

	// and iterable objects in general
	var list = new List<String>();
	list.add("support for iterables in general");
	list.add("also good");
	Console.log(list);

## Printer

Printing is implemented by a `mconsole.Printer`. Custom printers can be used:

	Console.addPrinter(new MyConsolePrinter());

A default printer is always created. This is a `ConsoleView` for JavaScript and 
Flash, or a FilePrinter for system targets.

To implement a custom printer:
	
	/**
	A ConsolePrinter that raises an alert for each log message.
	*/
	class AlertConsolePrinter implements mconsole.Printer
	{
		public function print(level:mconsole.LogLevel, params:Array<Dynamic>, indent:Int, pos:PosInfos):Void
		{
			js.Lib.alert(Std.string(level) + "@" + pos.className + "." + pos.methodName + ":" + params.join(", "));
		}
	}

## Tests

You can run the test suite using munit:

	haxelib run munit test -js -as3 -neko -debug