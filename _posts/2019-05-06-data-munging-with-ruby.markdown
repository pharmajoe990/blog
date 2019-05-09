---
layout: post
title:  "Data munging with Ruby"
date:   2019-05-06 09:04:00 +1100
categories: data development ruby
---

### What?

Data engineering is usually carried out with languages including Python or Scala. These are the defacto tools for good reason, however there is no reason a language like Ruby can be used for the same thing. Recently I have been using Ruby to play around with some JSON data and thought I'd record my thoughts.

### Basics

Ruby has a reputation for performing poorly in comparison to some other platforms, and I guess this is somewhat true; for me the design of Ruby has been to make programming more simple and enjoyable for the developer where performance might be less important. This language design certainly makes working with Key/Value based datasets like JSON quite easy. The translation to Ruby's hash API exposes a rich set of tools via the `hash` class and `enumerable` interface.

### Load that JSON

Here are a few random JSON documents. We will play around with them a bit. Suppose the first file represents the language runtimes installed on my development system, and the second file represents the programs I have written on the same system.

**languages.json**
```json
[
    {
        "language": "ruby",
        "version": "2.6.3",
        "size_mb": 125, 
        "installed": true
    },
    {
        "language": "java",
        "version": "11",
        "size_mb": 245,
        "installed": false
    },
    {
        "language": "python",
        "version": "3.0.0",
        "installed": true
    }
]
```

**programs.json**
```json
{
  "hello-world": {
    "language": "java",
    "supported_versions": [ "11", "9", "4" ]
  },
  "disk-usage-calculator": {
    "language": "python",
    "supported_versions": ["2.6"]
  },
  "conways-game-of-life": {
    "language": "ruby",
    "supported_versions": [ "2.6.3" ]
  }
}
```

For this purpose I will be using IRB, ruby's REPL. This comes with all recent versions. So get yourself a bunch of JSON files and load them in. Some of the output has been removed from below to make it a bit easier to read:

```
$ irb
irb(main):001:0> require 'json'
=> true
irb(main):002:0>languages = JSON.parse(File.read("languages.json"))
irb(main):003:0>programs = JSON.parse(File.read("programs.json"))
```

The output will dump the data structures just read in like below:

```ruby
{
  "hello-world"=>{"language"=>"java", "supported_versions"=>["11", "9", "4"]},
  "disk-usage-calculator"=>{"language"=>"python", "supported_versions"=>"2.6"}, 
  "conways-game-of-life"=>{"language"=>"ruby", "supported_versions"=>["2.6.3"]}
}
```

We can validate this is a hash using the `respond_to?` method. This is a Ruby idiom for determining types at runtime, known as _duck typing_:

```
irb(main):007:0> programs.respond_to? 'values'
=> true
```

### Munging that data

Nice, we have a couple of hashes in memory. Let's play around with them a bit. A Ruby hash is a map or dictionary data structure, which is a key/value structure. The key can be of any type supported by Ruby, as can the value. in our case we have a mixture of `strings`, `numbers` and `booleans` in our datasets.

We can check how many programs we have:
```
irb> programs.count
=> 3
```

And check what languages are supported by a program:
```
irb> programs["hello-world"]["language"]
=> "java"
```

We can iterate through the structure as well. The `each` method iterates each of the top-level entries of the map. It exposes the key/value as 2 lexical arguments of the `each` method. These 2 arguments can be named arbitrarily, with the convention of single letter arguments, however for the sake of this demo I have named them `key` and `value` for obvious reasons. Lets see which languages are required for each program:
```
irb> programs.each { |key,value| puts "#{key}: #{value["language"]}" }
hello-world: java
disk-usage-calculator: python
conways-game-of-life: ruby
```

### Joining datasets

So we have 2 related datasets, what can we determine from mixing them? The multiline function input to IRB line by line precedes the console output:
```ruby
programs.each do |k,v|
  supported = languages.any? { |l| l["language"] == v["language"] && l["installed"] }
  puts "#{k} #{supported ? "can execute" : "can't execute"}"
end
```

Here is the IRB console showing which of our programs we can currently execute, based on the installed runtime versions:
```
irb(main):094:0> programs.each do |k,v|
irb(main):095:1* supported = languages.any? { |l| l["language"] == v["language"] && l["installed"] }
irb(main):096:1> puts "#{k}: #{supported ? "can execute" : "can't execute"}"
irb(main):097:1> end
``` 

And the output:
```
hello-world: can't execute
disk-usage-calculator: can execute
conways-game-of-life: can execute
```

What if we want to create a new dataset, joining the 2 existing ones to tell us what programs we can execute? We would like to end up with a dataset like this:

```json
{
  "runtime-installed": [
  {
    "name": "conways-game-of-life",
    "runtimes": {
      "ruby": ["2.6.3"]
    }
  }
  ],
  "runtime-missing": [
  {
    "name": "hello-world",
    "runtimes": {
      "java": [ "11", "9", "4" ]
    }
  },
  {
    "name": "disk-usage-calculator",
    "runtimes": {
      "python": ["2.6"]
    }
  }
  ]
}
```

```ruby
# Partition the programs by installed language version
first, last = programs.partion do |name,detail|
  languages.any? do |l| 
    l["language"] == detail["language"] and 
    detail["supported_versions"].any? l["version"] and
    l["installed"]
  end
end
# Generate the installed/missing program runtimes from the destructured array above
runtime_missing = first.map do |l| 
  {"name" => l[0],"runtimes" => {l[1]["language"] => l[1]["supported_versions"]} }
end

runtime_installed = last.map do |l| 
  {"name" => l[0],"runtimes" => {l[1]["language"] => l[1]["supported_versions"]} } 
end
# Create the transformed dataset
joined_data = {
  "runtime-installed" => runtime_installed,
  "runtime-missing" => runtime_missing
}
pp joined_data
```
*output*
```
{"runtime-installed"=>
  [{"name"=>"conways-game-of-life", "runtimes"=>{"ruby"=>["2.6.3"]}}],
 "runtime-missing"=>
  [{"name"=>"hello-world", "runtimes"=>{"java"=>["11", "9", "4"]}},
   {"name"=>"disk-usage-calculator", "runtimes"=>{"python"=>["2.6"]}}]}
```

### Wrapping up

Whilst this is a bit of a toy in real-world terms, and perhaps a bit messy, it demonstrates the power of the core Ruby SDK in dealing with data. AFAIK Ruby doesn't have much support for distributed compute compared with Platforms like Apache Spark or Hadoop. These platforms have a proven track record when dealing with extremely large (PB) datasets, however in a lot of use cases a single machine has more than enough resources to suffice (like your laptop). Ruby is quick to install, expressive and could be a quick way to get down to playing with data, if your data doesn't require the resources of hundred or thousand node clusters.