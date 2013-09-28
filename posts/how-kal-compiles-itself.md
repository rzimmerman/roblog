# How Kal Compiles Itself

[Kal](http://rzimmerman.github.io/kal) is written in itself and is self-compiling. I've had a couple requests on Hacker News and Reddit to explain how this is accomplished and how the current npm packages came to exist.

This post should show you how to compile Kal, give a quick intro to how the compiler works, and a brief history of how we got here.

## About Kal and Its Parser

Kal is written completely in Kal. The compiler is a recursive descent parser that creates an abstract syntax tree, then traverses that tree to generate JavaScript. The grammar itself, while not in [Bison/YACC](http://www.gnu.org/software/bison/) format, is easily readable in the [grammar.kal](https://github.com/rzimmerman/kal/blob/master/source/grammar.kal) file. Consider the following snippet:


    class Statement inherits from ASTBase
      method parse()
        me.statement = me.req BlankStatement, TryCatch, ClassDefinition, ReturnStatement,
          WaitForStatement, IfStatement, WhileStatement, ForStatement, ThrowStatement,
          SuperStatement, DeclarationStatement, AssignmentStatement, ExpressionStatement

    class WhileStatement inherits from ASTBase
      method parse()
        me.req_val 'while'
        me.lock()
        me.expr = me.req Expression
        me.block = me.req Block

When `WhileStatment::parse` is called, the first thing that happens is that the parser requires the value `"while"`. If the current token isn't `"while"`, parsing the while statement will fail. If `req_val` fails, it throws an error and the parent (`Statement`) will continue on and try the next option in its list (`IfStatement`).

`lock` indicates that we are now sure this is a `while` statement. If any `req` or `req_val` calls fail after this, it will throw syntax error and abort compilation.

Next, we require an `Expression` after `"while"` (which has its own syntax declaration), then we require a code block.

Once this tree is built, we traverse it again to spit out JavaScript code. I'll probably make another post about how the generator and lexer work, but for now feel free to peruse the source. Onto the main topic...

## How it Compiles (Now)

If you clone the [repository](https://github.com/rzimmerman/kal), you'll find you get a `source` directory that contains several `.kal` files. If you dig deeper, you'll notice there's no `.js` files anywhere. So it compiles itself, but it needs to be already compiled to do so.

![Is such a thing even possible?](/images/even_possible_aliens.gif)

In order to make this work, you need to complete the following steps:

1. Get a Kal "binary". By binary, I mean a precompiled JavaScript package from npm. You can get the latest version with `sudo npm install -g kal`.
2. Compile the repository using the Kal "binary" into the `compiled/` directory. This creates the "stage 1" product.
3. Use the "stage 1" compiled version in `compiled/` to recompile itself. This is called the "stage 2" product.
4. Run the tests with the stage 2 product to make sure nothing broke.

Steps 2-4 can all be accomplished with `npm run-script bootstrap`.

![Graphical Flow](/images/kal-compiler-flow.png)

So where did the binary come from? To create the 0.4.5 binary, I ran these steps with the previous version's binary (0.4.4). But where did that come from?


## How It Got Here

So how'd I compile the first version? Is it [turtles all the way down](http://en.wikipedia.org/wiki/Turtles_all_the_way_down)?

At the bottom of Kal, deep in the history, still present in the code (though in memory only), is [CoffeeScript](http://www.coffeescript.org).

[anarbi](https://news.ycombinator.com/user?id=arnarbi) posted a great [link](http://cm.bell-labs.com/who/ken/trust.html) about how compilers can contain a memory of things that are no longer in the code.

Kal was originally written in CoffeeScript up until about version 0.2.0. If you look [here](https://github.com/rzimmerman/kal/tree/cdab3d17ef3b3578e80f9772124ae67b395d10b2), you can see an old version of the compiler written as a set of `.coffee` files. I chose CoffeeScript because:

1. I was familiar with it
2. It's well supported and full-featured
3. It's syntactically similar to Kal
4. It runs on node.js which made cross-compiling easy


I use CoffeeScript extensively in my own professional work and I love it. Kal really isn't ready for prime time yet, so right now I just use it for personal projects. I point this out because CoffeeScript has its own self-build system which is very well done and it inspired a lot of this work.

My goal with the CoffeeScript version was to get it to a point where enough tests were passing and enough features were supported that I could port things over to Kal. I wanted a bare minimum feature set.

One-by-one, I hand-translated the `.coffee` source files to `.kal`, running tests and fixing bugs along the way. Once the Kal source was mature enough (and bug-free enough), I started switching the `require` links to point to the `.kal` versions. [Here](https://github.com/rzimmerman/kal/tree/e7399c6f4e9b216f15ea098dbe94fde70ccf89be) is the first commit where I've switched a reference. At this point, the compiler was written partly in Kal and partly in CoffeeScript. Luckily node.js allows these to play nicely together.

Once I had all the files translated over and all the tests passing, I removed the CoffeeScript source [here](https://github.com/rzimmerman/kal/tree/c3409413e6b3dc8bd57c81b62b74de2e0c68829f).

Since then, I've used the previous version of the Kal compiler to bootstrap forward when making a new release. Extensive use of tests helps enormously. If you wanted to rebuild Kal without a compiled version, you'd need to go back to the original CoffeeScript source and compile up step-by-step.

## Stability Concerns

GCC uses a similar bootstrap method to compile itself. According to the [GCC install page](http://gcc.gnu.org/install/build.html), the bootstrapping process is as follows:

1. Build tools necessary to build the compiler.
2. Perform a 3-stage bootstrap of the compiler. This includes building three times the target tools for use by the compiler such as binutils (bfd, binutils, gas, gprof, ld, and opcodes) if they have been individually linked or moved into the top level GCC source tree before configuring.
3. Perform a comparison test of the stage2 and stage3 compilers.
4. Build runtime libraries using the stage3 compiler from the previous step.

Kal does not currently bother do a third stage compile/compare. This would entail using the stage 2 output of `npm run-script bootstrap` to build itself again, then doing a diff. The goal is to ensure the compiler is stable and not changing its own function. I will probably add this step in the future, though for now I am content with test cases as assurance that nothing is broken.
