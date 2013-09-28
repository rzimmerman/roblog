# Kal Is No Longer Illiterate

I'm excited to release a new feature to the [Kal](http://rzimmerman.github.io/kal) programming language called "Literate Kal". If you're familiar with [Literate CoffeeScript](http://ashkenas.com/literate-coffeescript/), this is going to look pretty familiar. Actually it's pretty much exactly the same.

I've ported the entire Kal compiler to Literate Kal. If your too lazy or not interested in the rest of this article, at least check out the beauty and clarity that is the [source code](http://rzimmerman.github.io/kal/doc/kal.html). Use the drop down at the top right to switch between files. The comments still have some typos and the formatting is a work in progress, but it's all there.

## What It Looks Like

Literate Kal files are simply [Markdown](http://daringfireball.net/projects/markdown/). When run through the compiler, however, all Markdown and text is converted to comments and any code blocks (indented with four spaces) are treated as actual code. It looks like this:

```kal
My Kal Program
==============

This program waits keeps trying to get [Google](http://www.google.com)'s
website until a server error is returned. This might take years to finish...

We use the "http" module for GET requests.

    http = require 'http'

We keep performing a GET request until the status code is something
other than 200. This code runs asynchronously using a "wait for", so
it will not block.

    statusCode = none
    until statusCode isnt 200
      safe wait for response from http.get 'http://www.google.com'
      statusCode = response.statusCode

If the previous loop finishes, we actually got a non-OK response from
Google. We print a statement to the screen to allow preparation for
the Apocalypse.

    print 'it actually happened!'


```

Note how it's documentation first, filled in with code.

## CoffeeScript

CoffeeScript has had a "Literate" feature for some time now. Jeremy Ashkenas has encouraged use of Literate CoffeeScript by introducing [docco](http://jashkenas.github.io/docco/), a pretty documentation generator that works well on literate languages. For professional work with node.js, I use Literate CoffeeScript extensively (I wouldn't push Kal on my production servers just yet).

This is pretty much a copy of Literate CoffeeScript. I do this out of love of the idea and claim that copying is a "good thing" in open source software.

I should note that I used a slightly modified version of docco to generate the Kal source documentation. I'm trying to get this into the main branch soon and I have a [pull request in with highlight.js](https://github.com/isagalaev/highlight.js/pull/283) to add Kal (feel free to comment your support if you think it's worthwhile). Once that's in I'll do the same for docco.

## Benefits of Literacy

[Literate Programming](http://www-cs-faculty.stanford.edu/~uno/lp.html) is a concept advocated by Donald Knuth that encourages combining a programming language with its documentation. Knuth claims it makes programs more robust, easily maintained, and more fun. I claim it also encourages open-source contributions. I noticed a few things while I ported the source to Literate Kal:

#### It Was Fun

I actually enjoyed writing the comments. It felt like I was telling my story and explaining the troubles that I went through. It's nice when someone listens to you brag, even if it's just your compiler.

#### It Made Me Keep Code Organized

When I was forced to break things into sections with headers and intro paragraphs, I went through and moved functions and variable definitions around so they were all grouped logically.

#### I Found Bugs

I found a couple bugs when forced to write out my rationale for everything.

#### I Found Leftover Crap

I found code that was no longer used, unnecessary, and otherwise yucky. I also managed to update some statements to use new language features to be more clear.

#### More Contributions

It's a small data set, but after converting to Literate Kal, open-source interactions on Github switched in character. Previous requests were mostly issues filed for feature requests and bugs. Less than one day after pushing the new source, I started to get more serious pull requests. Making the code readable and friendly encourages people to read it.

## Try It

So I encourage you all to try it. Download version 0.5.4 of Kal from npm (`sudo npm install -g kal`) or [Github](https://github.com/rzimmerman/kal) and write some of the most readable code of your life.
