---
layout: post
title: "Haxe, the Ultimate Transpiler"
date: 2016-07-30 16:45
author: Munir
comments: true
categories: [Dev]
tags: [Haxe, Tutorial]
image:
    src: //i.imgur.com/ECiAiDI.png
    width: 150px
---

[Haxe] is an awesome language that can cross-compile to many other languages,
such as Javascript, C++, C#, Python, Java, Lua, PHP, ActionScript 3,
Flash (byte code), and Neko (byte code). For Flash and Neko, Haxe can compile
to byte codes, whereas for the other targets, Haxe acts as a source-to-source
translator/compiler (transpiler).

If you've heard of [TypeScript], then Haxe is pretty similar. It is statically
typed, and it also supports type inference. TypeScript is also a transpiler,
but it only compiles to Javascript. With Haxe, you have the option of
transpiling to [many other languages][targets]. The Haxe compiler is capable of
doing a number of optimizations, such as constant folding, function inlining,
and dead-code elimination. More Haxe language features after the jump.

<!--more-->

{% include partials/toc.html %}

## Features Overview

Haxe has [a number of cool features][features] that may not be available on the
target language itself. Your class methods can be marked as `inline`, allowing
codes to be inserted at call-site. There's static extensions, which works
similarly to the one in C#. [Pattern matching][pattern] allows you to match
against complex structures and [Algebraic Data Types][enum]. But my favorite is
[macros], allowing you to do [metaprogramming].

Macros allow you to extend the language further, by writing codes that execute
at compile-time, which generates other expressions before being compiled to
the target language. This allows you to define your own syntax, for example,
a custom for loop that [unrolls itself][unroll] when compiled. Instead of
manually unrolling a loop, which affects readability, a macro could allow you
to optimize while still having readable codes.

## Getting Started

### Installing Haxe

To install Haxe, head over to <https://haxe.org/download/> and download it
for your operating system. If you're on Windows, Haxe is also available on
[Chocolatey] and can be installed with:

```batch
choco install haxe
```

### Getting an IDE

On Windows, I'd recommend using [FlashDevelop]. There's also [HaxeDevelop] which
is a [custom FlashDevelop distribution for Haxe][MeetHaxeDevelop]. You can also
use [Atom] with the [language-haxe] package. Here's a [list of other Haxe IDEs][ide]
and code editors compatible with Haxe.

### Try it on Your Browser

Want to just try out some of the language features online, but setting it up
locally in your computer is a bit too much? You can try it online at
<http://try.haxe.org/>.


## Writing a Haxe Program

### Hello World in Haxe

Open up your favorite editor. If you're using FlashDevelop/HaxeDevelop (which
I'll refer to as HaxeDevelop from here on), create a new Haxe project.

Create a HelloWorld.hx:

```haxe
class HelloWorld
{
    public static function main():Void
    {
        trace("Hello World!");
    }
}
```

On HaxeDevelop, simply click the "Play" button and it'll compile and run your
program. On any other text editors, you may have to build from the command-line.
Open the command prompt or terminal.

To build to neko target, and run it:

```batch
haxe -main HelloWorld -neko app.n
neko app
```

To build to Javascript and run it with node:

```batch
haxe -main HelloWorld -js app.js
node app.js
```

You can also link it to your HTML page, and if you open your browser's console,
you should see the "Hello World" message. This is what the contents of app.js
looks like:

```javascript
(function (console) { "use strict";
var HelloWorld = function() { };
HelloWorld.main = function() {
	console.log("Hello World!");
};
HelloWorld.main();
})(typeof console != "undefined" ? console : {log:function(){}});
```

For certain targets like C++, C# and Java, you also need to have a its compiler
installed. You also specify the output folder instead of an output file:

```batch
haxe -main HelloWorld -cpp bin/cpp
cd bin/cpp
HelloWorld.exe
```

Here's a [full list of compiler command-line switches][compiler].

```batch
haxe -main HelloWorld -js bin/js/app.js
haxe -main HelloWorld -as3 bin/as3
haxe -main HelloWorld -swf bin/swf/app.swf
haxe -main HelloWorld -neko bin/neko/app.n
haxe -main HelloWorld -php bin/php
haxe -main HelloWorld -cpp bin/cpp
haxe -main HelloWorld -cs bin/cs
haxe -main HelloWorld -java bin/java
haxe -main HelloWorld -python bin/py/app.py
```

### Using Libraries in Haxe

[Haxelib] is the library manager that comes with Haxe. You can use it to download
user-submitted libraries, or upload your own. Here's an example on how to install
the `moon-core` library:

```batch
haxelib install moon-core
```

On HaxeDevelop, you need to add the library reference to your project settings.
To do so, at the menu, click on `Project > Properties... > Compiler Options`,
and add `moon-core` to the libraries.

Then, to use the Range class in the HelloWorld example:

```haxe
import moon.core.Range;

class HelloWorld
{
    public static function main():Void
    {
        for (i in Range.from(3, 7, 2)) {
            trace('$i: Hello World!');
        }
    }
}
```

On HaxeDevelop, you can simply click the play button, and it'll run. To manually
build it, you need to add the `-lib moon-core` to your build arguments like so:

```batch
haxe -lib moon-core -main HelloWorld -js app.js
node app.js
```

[Haxelib documentation][UsingHaxelib] is available online, and you can browse
the libraries at <http://lib.haxe.org>.

### Simplifying Command-Line Arguments

If you're building from the command-line, you may notice that the command-line
arguments can get quite long. Lets move your source files into a sub-folder
called `src`. Now you need to indicate to haxe where your class paths are using
the `-cp` switch:

```batch
haxe -lib moon-core -cp src -main HelloWorld -js app.js
```

Sometimes, you may even want to build to multiple targets:

```batch
haxe -lib moon-core -cp src -main HelloWorld -js app.js
haxe -lib moon-core -cp src -main HelloWorld -neko app.n
haxe -lib moon-core -cp src -main HelloWorld -python app.py
```

You can simplify those by creating a `whatever.hxml` file containing one
command-line argument on each line. And then build with `haxe whatever.hxml`.

Here's an example. Create a `build.hxml`:

```hxml
-lib moon-core
-cp src
-main HelloWorld
-js app.js

# separator indicating another build
--next

-lib moon-core
-cp src
-main HelloWorld
-neko app.n

--next

-lib moon-core
-cp src
-main HelloWorld
-neko app.py
```

And then run `haxe build.hxml` to build for both targets. If you notice, there's
still some repetition, and we can simplify it further by using `--each`.

```hxml
-lib moon-core
-cp src
-main HelloWorld

# everything above is the common stuff
--each
-js app.js
--next
-neko app.n
--next
-python app.py
```

> **Tip: What if your build config gets more complicated?**  
> I wrote [moon-run] to allow you to store multiple build configs as well as
> run configs, in a nested JSON structure. So you can build with a command
> like `hx build webserver chatserver jsclient` where webserver,
> chatserver and jsclient are configs stored in `haxe.json`. Then run
> with `hx run something`.

## Haxe Resources

![It's dangerous to go alone! Take this.](//i.imgur.com/iWbLois.png){:width="500px"}

### References and Frameworks

- Learning Resources
    - [Haxe Manual](https://haxe.org/manual)
    - [Haxe API Documentation](http://api.haxe.org/)
    - [Haxe Code Cookbook](http://code.haxe.org/) - Source code snippets and examples
    - [Haxe.io](https://haxe.io/) - Periodic updates on the Haxe community
    - [Haxecoder](http://www.haxecoder.com/) - Haxe and OpenFl tutorials
- Game Development in Haxe
    - [OpenFl](http://www.openfl.org/) - Free and open source framework with an API that mirrors Flash API
    - [HaxeFlixel](http://haxeflixel.com/) - Open source 2D game engine
    - [HaxePunk](http://haxepunk.com/) - Open source framework, ported from FlashPunk. Uses OpenFl.
    - [Flambe](https://github.com/aduros/flambe) - Rapidly cook up games for HTML5, Flash, Android, and iOS
    - [Kha](http://kha.tech/) - Ultra-portable, high performance, open source multimedia framework
- Books
    - [Haxe Game Development Essentials](https://www.amazon.com/Haxe-Development-Essentials-Jeremy-McCurdy/dp/1785289780/)

### Online Communities

- [Google Group](https://groups.google.com/forum/#!forum/haxelang)
- [Reddit](https://www.reddit.com/r/haxe)
- [Twitter @haxelang](https://twitter.com/haxelang)
- [Twitter @haxe_org](https://twitter.com/haxe_org)
- [Facebook](https://www.facebook.com/haxe.org/)
- [Stack Overflow](http://stackoverflow.com/questions/tagged/haxe)
- [GitHub](https://github.com/HaxeFoundation)
- [GitHub Issues](https://github.com/HaxeFoundation/haxe/issues)
- [Gitter](https://gitter.im/HaxeFoundation/haxe)
- [IRC](http://webchat.freenode.net/?channels=haxe)
- [Blog](https://haxe.org/blog)


[Haxe]: https://haxe.org
[Haxelib]: https://haxe.org/manual/haxelib.html
[UsingHaxelib]: http://lib.haxe.org/documentation/using-haxelib/
[TypeScript]: https://www.typescriptlang.org/
[targets]: https://haxe.org/documentation/introduction/compiler-targets.html
[features]: https://haxe.org/documentation/introduction/language-features.html
[pattern]: https://haxe.org/manual/lf-pattern-matching.html
[enum]: https://haxe.org/manual/types-enum-instance.html
[macros]: https://haxe.org/manual/macro.html
[metaprogramming]: https://en.wikipedia.org/wiki/Metaprogramming
[unroll]: https://en.wikipedia.org/wiki/Loop_unrolling
[Chocolatey]: https://chocolatey.org/
[FlashDevelop]: http://www.flashdevelop.org/
[HaxeDevelop]: http://www.haxedevelop.org/
[MeetHaxeDevelop]: https://haxe.org/blog/meet-haxedevelop
[Atom]: https://atom.io/
[language-haxe]: https://atom.io/packages/language-haxe
[ide]: http://www.haxedevelop.org/other-haxe-editors.html
[compiler]: https://haxe.org/manual/compiler-usage.html
[moon-run]: https://github.com/profound7/moon-run