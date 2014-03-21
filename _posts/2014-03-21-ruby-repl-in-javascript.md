---
layout: post
title: Interactive Ruby in the browser
---

This is a brief explanation of how to get a read-evaluate-print loop (or repl, for short) for a number of languages to work in your browser.

See the result here: [rubyprog](http://rubyprog.nl). 

I used two libraries: [jsrepl](https://github.com/replit/jsrepl) and [jq-console](https://github.com/replit/jq-console). jq-console emulates a console in your browser using jQuery, while jsrepl contains the language compilation engines.

The docs are sparse and it's not obvious how to combine the two libraries. I will try to make that clear here.

Follow the [build instructions](https://github.com/replit/jsrepl#getting-started) for jsrepl. Before running `cake bake`, you can delete all the languages you don't need from `languages.coffee`. After building, include all the files from the `build` folder in your site. 

In the html:

{% highlight html %}
<div id="console"></div>

<script src="jquery.js"></script>
<script src="jqconsole.js"></script>
<script src="jsrepl.js" id="jsrepl-script"></script>
<script src="repl.js"></script>
{% endhighlight %}

The `#console` will become the repl. We don't touch anything except for our own `repl.js` script, in which we will specify handlers for all sorts of events like user input, output, repl results and so on. It looks like this:

```coffee
# repl.coffee
window.jqconsole = $('#console').jqconsole("Click to load the ruby engine.", '> ')

$(".jqconsole").click(->
  unless jsrepl?
    jqconsole.Write("Loading...")
    initializeRepl()
)
```

We initialize a console with a welcome string and a prompt label. Because loading the language engine costs time, we wait until the user clicks the console, then write `"Loading..."` using `jqconsole.Write`, and run `initializeRepl`.

In `initializeRepl`, we must first instatiate the language engine, providing callbacks for input, output, result and error streams.

```coffee
initializeRepl = ->
  inputCallback = (callback) ->
      window.jqconsole.Input (result) ->
        callback(result)

  outputCallback = (string) ->
    jqconsole.Write("#{string}", "repl-output")

  resultCallback = (string) ->
    jqconsole.Write(" => #{string}\n", "repl-result")

  errorCallback = (string) ->
    jqconsole.Write("#{string}", "repl-error")    

  this.jsrepl = new JSREPL({  
    input: inputCallback,
    output: outputCallback
    result: resultCallback
    error: errorCallback
  });

  jsrepl.loadLanguage("ruby", ->
    jqconsole.Write("done.\n")
  )
```

The `inputCallback` takes a callback function that must be called with the user input. jsrepl hacks around the fact that browsers are asynchronous, while Ruby expects the environment to be synchronous. Read about it [here](https://github.com/replit/jsrepl#standard-input-hacks).

Note that `jqconsole.Write` takes a string as optional second argument, which will become the class of the html output. This allows you to style different kind of outputs in different colors, for example.

Now we need to configure the prompt handling of the `jqconsole`:

```coffee
  promptHandler = (input) ->
    jsrepl.eval(input)
    startPrompt()

  startPrompt = ->
    # Start prompt with history enabled
    jqconsole.Prompt(true, promptHandler)

  startPrompt()
```

This is all done in `initializeRepl`. When the user types something in the console, we `eval` the result in the current language and the appropriate handler (result, input, output, error) gets called. Then we prompt the user again.

That's it!

We can throw in a blinking cursor for good measure:

```coffee
blinkCursor = ->
  cursor = $(".jqconsole-cursor")
  if cursor.css("opacity") == "1"
    cursor.css( {"opacity": 0} )
  else  
    cursor.css( {"opacity" : 1} )

setInterval(blinkCursor, 650); 
```

We can even handle multiline input, by adding a `multiLineCallback` to `jqconsole.Prompt`. The prompt handling then looks like this:

```coffee
  promptHandler = (input) ->
    jsrepl.eval(input)
    startPrompt()

  multiLineHandler = (command) ->
    # A very primitive depth balance estimator.
    levels = 0
    parens = 0
    braces = 0
    brackets = 0
    last_line_changes = 0
    for line in command.split '\n'
      last_line_changes = 0
      for token in (line.match(TOKENS) or [])
        if token in BLOCK_OPENERS
          levels++
          last_line_changes++
        else if token is '('
          parens++
          last_line_changes++
        else if token is '{'
          braces++
          last_line_changes++
        else if token is '['
          brackets++
          last_line_changes++
        else if token is 'end'
          levels--
          last_line_changes--
        else if token is ')'
          parens--
          last_line_changes--
        else if token is ']'
          braces--
          last_line_changes--
        else if token is '}'
          brackets--
          last_line_changes--

        if levels < 0 or parens < 0 or braces < 0 or brackets < 0
          return false

    if levels > 0 or parens > 0 or braces > 0 or brackets > 0
      if last_line_changes > 0
        return 1
      else if last_line_changes < 0
        return -1
      else
        return 0
    else
      return false

  startPrompt = ->
    # Start prompt with history enabled
    jqconsole.Prompt(true, promptHandler, multiLineHandler)

  startPrompt()

  BLOCK_OPENERS = [
    "begin"
    "module"
    "def"
    "class"
    "if"
    "unless"
    "case"
    "for"
    "while"
    "until"
    "do"
  ]

  TOKENS = ///
    \s+
   |\d+(?:\.\d*)?
   |"(?:[^"]|\\.)*"
   |'(?:[^']|\\.)*'
   |/(?:[^/]|\\.)*/
   |[-+/*]
   |[<>=]=?
   |:?[a-z@$][\w?!]*
   |[{}()\[\]]
   |[^\w\s]+
  ///ig
```

Whew! It's easier than it looks. When you provide a `multiLineCallback` to `jqconsole.Prompt`, all user input gets evaluated by this function. When it returns `false`, it means the input is ready for evaluation, and it is passed to the `PromptHandler`. When `multiLineCallback` returns an integer, the user input is stored and the user is prompted again with an incremented or decremented level of indentation, depending on the integer returned (0 for same level, 1 for an increment of one, etc.) Now stuff like:

```ruby
def greet(name)
  "Hello, #{name}"
end
```

can be typed into the repl directly.

I nicked the `multiLineCallback` code from the [jsrepl GitHub](https://github.com/replit/jsrepl/blob/master/langs/ruby/jsrepl_ruby.coffee).

Great many thanks to the guys from [repl.it](https://github.com/replit)!