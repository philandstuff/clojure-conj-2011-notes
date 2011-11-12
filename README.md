# Philip Potter's clojure conj 2011 notes

# day 1

## stuart sierra's initial talk
  - who: people at a loose end, people wanting to see potential
  - why: 

## @IORayne's talk on clojail

- what: sandbox for evaluating clojure code without allowing
      dangerous side-effects
- why
  - entertaining delivery
  - interesting problem

## @chrishouser's cljs talk

- deep dive into cljs, shows power and capability of platform

## phil bagwell -- striving to make things simple and fast

- how do you get clojure into production? great ideas
- persistent data structures with good properties for append and index
    
## ambrosebs clojure.logic

## stuart halloway

- entertaining as usual
- promised to define "power" objectively, don't feel it quite delivered 

# day 2

## Neal Ford, TW

- how do things become popular?

### Top down

- tech radar:
  - needs a target audience
  - developers?
  - CTOs?
- "tests" for technology
  - "can I hire cheap devs?" -- actually means "I'm afraid, I want the status quo over a riskier proposition"
  - "if we pick a weird tech and the project fails, we get fired."
- how do we alleviate these fears?
  - create new clojuristas
  - sponsor SICP user groups? (not sure)
  - target unis
  - sell CTOs on "different", not on LISP

### Botom up

- tools to massively increase productivity (appeal to the delphi/rails
  view)
- ideas: web frameworks, RESTful integration, ORM, rules engines?,
  something else...
- "breaking and entering" -
  - force the enterprise to accept you. Linux/Rails.
  - really hard. by the time you're able to do this, you've probably
    already won
- "cat burglary"
  - camouflage your tech, and piggyback it in
  - Cucumber -- cucumber is nice, and as a side effect, it pulls in
    ruby
  - gradle pulls in groovy
  - "make clojure.jar as commonplace as antlr.jar"
  - mollifies CTOs because they only see it after it's become
    prevalent
    
### propagandize

- stay on message
- knitted castle vs lego castle
- stay relentlessly positive (example of JRuby)
- if you have an advantage, prove it!
- know your enemies
  - NOT:
    - scala
    - agile
  - IS:
    - status quo
- Scala will be the next big thing on the JVM because it fetishizes
  complexity exactly the way that Java develepors are used to
  
- SELL one step at a time:
  - alternate language to Java
  - functional
  - dynamic
  - Clojure
- celebrate Scala's successes!

- integrate from Clojure towards other things
  - eg encapsulate Akka in clojure

### questions

- ignore the enterprise? just sell to the business?
- what about lisp's baggage? how do you sell that?
  - answer: the people you need to sell to don't know about lisp's
    baggage
- RH's question: CTO wants cheap labour, how to you change the view to
  want fewer, quality devs?
  - answer basically agreed with question premise without answering

## David Nolen, Predicate Dispatch

- what is it?
  - core.match
- generalizes:
  - multimethods
  - pattern matching
- obstacles
  - Rich Hickey doesn't like pattern matching
  - it's hard
- is predicate dispatch simple? (particularly, vs open dispatch)

## Kevin Lynagh, extending JS with CLJS

- handout available on Keming labs website
- imperial -> metric as js -> cljs
- javascript has a lot of stupid
  - globals
  - namespaces
  - see handout for crazy example of hoop-jumping to get round this
- coffeescript won't save you
- myth: only geniuses use functional languages
- interop

### CLJS is better at extending JS than JS is

- method chaining style in jQuery isn't extensible -- can't put a user
  method into middle of chain
  - CLJS allows a user fn in middle of a (-> ...) invocation

## Christophe Grand (@cgrand) from linear to incremental

- why do i need to recompile a whole file when i change one var?

### Key idea: incremental parsing

- defnintion:
  - recompute the whole parsetree at a fraction of the cost (log n)
    after an edit
- overloaded term:
  - restartable
  - lazy restartable (Yi, haskell editor)

### how do we do this?

- memoization of old compile -- allow reuse
- exploit associativity:
  - if (reduce f [a b c d]) goes to (reduce f [a b c' d]):
    - strict version has to recompile the whole of (f (f init d) c')
    - reordering version based on associativity can just recompile
      (f init c')
- transplantation?    
- implementation: parsley

## Mark McGranaghan, logs as data

- foo as data
  - code, sql, cli options, decision trees, etc

- data is common in clj
  - avoid hiding through encapsulation
  - use simple abstractions instead -- `map` `seq` `fn` -- which data
    types implement
    
- key idea:
  - data
  - via simple abstractions
  - with a general library

### logs

- simple abstractions for logging completely lacking

- general library for logging not possible
  - instead, have complete but overspecific libraries which cannot be
    generalized

- logging as it currently is: strings

### logging as it should be: stream of events

- (emit event) -- general logging api
  - defined in terms of IFn and IPersistentMap

- then you can apply `map` `fn` `seq*` to logs
  - eg:

    (->> events
      (filter :viewing)
      (count-by some-fn)
      (timechart))

- log lines -> events
- strings -> maps
- files -> streams

- ...
- log crunching -> event processing

### consequences

- for:
  - debugging
  - auditing
  - metrics
  - analytics
  - alerting

### usage in heroku

- technology called [pulse](http://github.com/heroku/pulse)

- general processors: `(f event-stream)`
  - decouples from the thing emitting the events
  
- network of emitters, processors, routers, and consumers
  - emitters: app under monitoring
  - processors: distil down important info, discard unimportant
  - consumers: emails, development help, monitoring, alerts

### questions

- what do you do with errors that happen in the event processing?
  - depends on semantics
  - perhaps drop on floor if noncritical
  - could alert
  
- focus on abstract events; what about parsing out legacy text-based
  logs?
  - convert incoming string-based logs into stream-based sequences of
    maps at point of entry to pulse system

- we used this; messages out on udp port; if noone's listening it
  falls on the floor. have you used TCP instead?
  - we use TCP

- aggregator seems like a generic MQ -- have you considered using one?
  - logbus + pulse (err)
  
- how do you distribute code to the various nodes in the logging
  network?

- RH: do you wish you could send dates as something other than
  strings?
  - perennial embarrassing problem

## @cemerick on probabilistic modelling using Bayesian networks

- why bayesian networks?
- lots of modelling options
- no black boxes
  - by contrast with neural networks, where the weights mean nothing
  - can do root cause analysis
- optimizable

- applications & domains:
  - making decisions based on noisy, incomplete data
  - document analysis
    - example: SEC filings
      - variety of ad-hoc formatting, want to extract structured data
        from them
        
        
- the first rule of bayesian club: select features (ie random variables)
  - is it wet? is it raining? is the sprinkler running? what season is
    it?
  - all of these are correlated in some manner

- first step of analysis: predicate calculus
  - there are no solid logical inferences here

- second step: joint probability distribution
  - ie for each combination of values of (wet, raining, sprinkler,
    season), determine probability
  - can use this to determine P(A|B)
  
- third step: P(a|b) = P(b|a)P(a)/P(b)

- joint probability distributions don't scale; there's a combinatorial
  explosion
  
### Bayesian Network

- DAG of random variables showing a model of dependencies
  - ie you know a priori that rain is likely to make the lawn wet, not
    that a wet lawn is likely to make it rain

- allows you to collapse the joint probability distribution
  - at each level in the DAG, only store the probability of that node
    given its parent
    
O (n log_m(p)) (what are n, m, p?)

### implementation: Raposo


## Daniel Solano Gomez (@deepbluelambda), Clojure on android

- Problem: JVM is not Dalvik VM
  - JVM: stack-based, method JIT, Java SE, .class files
  - Dalvik: register-based, trace JIT, not quite Java SE, DEX file

- .jar file concats lots of .class files
  - causes repetition of eg constants
- DEX is smarter, collapses and collects constants
  - results in smaller code size
  
- DEX is packaged in an .apk
- when installed, converted to Optimized DEX (ODEX)
- implication: loading bytecode is heavyweight
- and dynamically loading bytecode not supported
- need AOT compilation or interpretation
- but want:

### dynamic compilation on android

- why?
  - obvious really
  - makes clojure REPL possible

- review of clojure repl process:
  - start with .clj file
  - reader converts to data structures
  - evaluator evaluates structures
  - which then generates bytecode (magic here)
  
- Dalvik process:
  - take a .class file and generate a DEX file
  - package DEX in an .apk
  - use existing android SDK to load .apk as ODEX
- uses new var: `clojure.core/*vm-type*`
  - either `:dalvic-vm` or `:java-vm`

- All Dalvik-related code is loaded reflectively
- some macros may blow the stack during compilation (!)
- can't depend on APIs which don't exist in Android (such as
  `java.beans`)

### benchmarking clojure vs competition

- package size:
  - java tiny
  - scala small
  - clojure 1.3 bigger
  - clojure 1.2 bigger still
  - ruboto biggest (see slides for graph)
- startup time
  - tracks package size trend above. Takes aaaages to start Clojure
- heap size
  - again tracks same trend.

### where does the space pain come from?

Answer in one word: `clojure.core`.

Biggest space hog (more than half): `clojure.core__init` -- contains
docstrings, arglists, other metadata

### where does the time pain come from?

Clojure's runtime class `RT`. Big things:

- `RT.load("clojure/core")`
- `(in-ns 'user)`
- `(refer "clojure/core")`
- Object churn causing GC
  - lots of metadata
  
### ways to potentially improve performance

- remove `user` ns
- remove metadata
- transients during initialization
- serializable `clojure.core`
- source-level tree shaker

### Build tools

- neko

### what about ClojureScript?

- O web apps
- O mobile development frameworks
- X native development
  - constrained by availability of JS engines
- ~ Scripting Layer for Android
  - not ideal "native experience"
  
### Summing up

- Clojure has potential to be a first-class language for android
- dynamic development is a killer feature
- tooling and community support not there yet :(
- Book coming soon: "Decaffeinated Android" (ie without Java)

### Questions

- even without load time, GCs seem to be very frequent, kills UX
  - early androids sucked
  - 2.3 introduced concurrent GC, helps quite a bit
- have you tried regular clojure with performance tweaks? (eg remove
  `user` ns etc)
  - no, not really
  - mostly affects startup time, not such a big deal for regular
    clojure
- clojure on Google App Engine startup time is a Big Deal, so maybe it
  would actually be a win?
  - probably
- right now on android, when it builds app, a lot of work goes on
  beside generating DEX file; there's also resource generation
  etc. What's your build environment and how do you handle this extra
  stuff?
  - modify compile target to do clojure compilation as well as java
    compilation
    
## Rich Hickey, keynote: Areas of Interest for Clojure's Core

- this is *not* a roadmap
  - half-baked
  - no time scheduled to attack these tasks
  - nead more design time
  
### Leaner

- core part of clojure includes built-in runtime for dynamic
  compilation, but this bulks things up  
- want to deliver clojure libraries to non-clojure
- smaller targets, eg android
- move ASM out (currently it's bundled in)
- dev-less builds
  - no metadata, shaking etc
  - debug builds
  
### unification of clojure and clojurescript

- ClojureScript is Clojure on a different host
- shared source libraries
- conditional compilation
  - also needed by Clojure CLR
- unifying on cljs analysis rep for tooling
- BUT: no objective to be able to have drop-in replacement ability of
  clj from different platforms  

### Prepping for Clojure in Clojure

- Move protocols down to 'bottom'
  - not currently a compiler, primitive feature, like `deftype` is
  - value of this seen in ClojureScript
- organizing cljs compiler for multiple targets?
  - cljs seems to be overall a better design
  
### invokedynamic (Java 7)

- what is is:
  - code-gen and class-free call sites
    - class is pretty big and carries lots of baggage, if each fn
      requires a class that gets expensive
  - access to safe point 'magic'
    - safe point is a magic implementation detail used during eg
      bytecode loading (i think)
- for clojure:
  - protocol/keyword call sites
  - fast & dynamic vars
    - vars are dynamic and volatile
    - hard for HotSpot to inline through vars
  - reflection-free interop

### Leveraging Logic

- code -> analysis -> core.logic
  - don't settle for 'type checking'
  - queryable programs
- predicate dispatch (see David Nolen's talk above)
  - don't settle for 'pattern matching'
    - pattern matching is closed and order-dependent
    - can't add new patterns, depend on order of patterns
  - arbitrary dispatch allows for better domain driven design
    - by allowing a closer match of code to product owner's thinking
    - more declarative
  - is core.logic fast enough for this?
    - precompile dispatch trees
    
### Parallelism

- concurrency is here already
- parallelism is different:
  - concurrency is different tasks running concurrently
  - parallelism is one task split into subparts running concurrently
- is fork/join the right model?
  - particularly in presence of I/O?
  - or in balanced (embarassingly parallel) compute problems?
- parallel algorithms, not colls
  - but colls need a good shape (something other than seqs)
  - "We don't attach our functions to our data, we apply our functions
    to our data"

### Transients and beyond

- transients do too much
  - persistent symbiosis & editing
  - *and* policy (ie, only do stuff from one thread)
- and too little
  - users must use linearly

- transient-based model (see slide)
  - don't want transient to be externally visible, need constrained
    visibility

- procs
  - fns that operate on transients (mutability, but testability)
  - like pure function, can't affect world or be affected by it
  - because transients don't leak
  - can be wrapped in value->transient and transient->value to produce
    a pure function
    
- Pods
  - split no-leak policty from data structure
  - policy is now:
    - value in, value out
  - process goes through pod
  - options for access
    - single thread vs mutex
  - work with transients
    - just take policy out
    - persistent in, transient inside, persistent out
  - but also works with other mutability:
    - String in, StringBuilder inside, String out

- pod usage example (see slide)
  - demonstrates using a StringBuilder as a transient in a pod
  - by allowing String to extend Editable protocol and StringBuilder
    to extend Transient

- open questions on pods
  - i'm not that interested in these

- Accumulators
  - pods -- do they do too much?
  - chief use case is building/birthing
  - maybe only need `conj!`/`append`

### Extensible reader

- why?
  - clojure data as serialization (better than JSON)
    - why is it better?
      - more data types (keywords, sets)
      - metadata
    - is it as good as it could be?
    - XML has eXtensibility
      - JSON and clojure don't
    - finite number of types means less self-describing
      - is that integer actually a date?
      - is that string actually a URL?
      - same problem as with arbitrary binary data
      - xml allows this
- but
  - avoid wheel rebuilding in extensions
  - #= reader macro is cheating; needs the clojure evaluator to read
     the data now, is not a generalized data format any more
  
- Premise
  - meet at semantics and print/read
  - plug in desired programmatic type

- How
  - no custom reader macros!
  - Tags on other readables
  - eg `#instant "1984-02-11T12:12:12.12Z"`
  - self-describing
  
- question: how is this different from ordinary metadata?

### Questions

- Have you looked at YAML's tag format?
  - not claiming originality! ripped it off from erlang!
  
- `*print-dup*` spits out classes and `#=` and suchlike, how is this different?
  - it's about semantic meaning of data, not about specific
    implementation
  - `*print-dup*` will distinguish `HashMap` from `ArrayMap` whether
    you like it or not
    
- from example, where does agreement happen about what `#instant
  "foo"` will rehydrate to?
  - no agreement
  - can't dictate data structure
  
- have you looked at openmath?

- querying programs - what does it mean? what would be an example?
  - simple example - "how deeply nested are calls to this function?"
  - program analysis
  
- conditional computation for targetting different platforms - what
  about specifying an interface that implementations must implement?
  - don't like that at all; reifying environments

- pods solve generating a value, but what about a way of iterating
  over a coll using mutability as a performance optimization?
  - haskell does this with stream fusion; good example of type system
    helping
    
- platform power, access to safe points, devless builds; what is
  largest couple of things in way of platform power?
  - we have great platform power
  - need to finesse utilization for efficiency, but have access to all
    features pretty much
    
- you were talking last year about issue of resource management in
  presence of laziness, have you got closer to solving issue?
  - lazy evaluation might eg hold onto resources until evaluation
    reifies and allows resources to release
  - problem is harder than i thought
  - possibly no silver bullet
  
## Day 3

### Nathan Marz, Cascalog

- High level abstraction for Hadoop map-reduce

- Hadoop redux
  - high latency batch processing
  - fault tolerant
  - petabyte scale
  
- number of tools to provide abstraction over hadoop

- cascalog's USP: abstraction + composition

- operates over datasets of tuples

- example:
  - `(?<- (stdout) [?person] (age ?person ?age) (< ?age 30))`
  - `?<-` - define query
  - `(stdout)` - where to emit results
  - `[?person]` - output variables
  - remainder - predicates: constrain the output variables
  - feels a bit like logic programming or sql
  
- predicates:
  - `(* 2 ?x :> ?z)` -- inputs `:>` outputs
  - define a constraint between a set of inputs and a set of outputs
  - `(+ 2 ?x :> 6)` - when `x` is added to 2, you get 6
  
- families of predicates:
  - functions
  - filters - `(< ?age 30)` above is example
  - aggregators
  - generators: sources of tuples - `(age ?person ?age)` above is
    example
    
- can join over different datasets
  - just combine two generator predicates
  - joins are therefore implicit, look like combining any other two
    predicates
    
- [code demo](https://github.com/nathanmarz/cascalog-conj)
  -
    [basic examples first](https://github.com/nathanmarz/cascalog-conj/blob/master/src/clj/cascalog/conj/play.clj)
  -
    [then more involved with a tunisia-based dataset](https://github.com/nathanmarz/cascalog-conj/blob/master/src/clj/cascalog/conj/tunisia.clj)
    - particularly like the use of `bar?-` to execute a query then use
      incanter to create a bar graph of the results

## Daniel Spiewak, "Functional Data Structures in Scala"

- source on [github](https://github.com/djspiewak/extreme-cleverness)

- categorization:
  - sequential
    - preserves insertion order
    - random access not so good
  - associative
    - doesn't preserve insertion order
    - random access

- Immutable
- comparable asymptotic performance to equivalent mutable structures
  - copy-on-write is not good enough
- *structural sharing* is the main tool to achieve performance

- singly linked list:
  - most things are linear time: `append`, `concat` etc
  - small description: cons cell or empty list

  - Scala `List[+A]` is a slist implemented as a sealed trait
  - structural sharing by two cons cells pointing to same tail

- want a function queue
  - linked list is obvious
  - but `first` and `last` are opposing
  - one will be constant, the other linear
  - can we get both?

- banker's queue
  - naive persistent queue
  - two lazy singly-linked list
  - front list (for dequeue)
  - rear list (for enqueue)
  - periodically reverse rear into the front
  - lazy amortization
  
- clojure's `PersistentQueue` is banker's queue, but with a vector for
  one of the lists to avoid the reverse
  
- amortization redux
  - interesting point: laziness actually distributes the work

- 2-3 finger tree: nice queue
  - constant append, prepend, first, last
  - logarithmic: insert
  - linear: concat
  
- scala example
  - cool
  - but really slow (film at 11)
  
- associative examples

- red-black tree (== clojure's `sorted-map`)
  - log: get/insert/update
  - linear: intersect/union

- invariants:
  - every path from root to leaf contains same number of black nodes
  - no red node has red parent
  - so total depth ranges from d/2 to d

- combination sequential/associative

- Bitmapped Vector Trie (hickey/bagwell)
  - constant: append first/last/nth/update
    - actually O(log_{32}n)
  - linear: concat/insert/prepend

  - array with max length 32
    - copy on write (fast for small n)
  - array of arrays, max length 32
    - array of array of arrays, max length 32
  - max depth is 7 (because max int on JVM is 2^32-1)

- locality of reference
  - this is what killed the 2-3 finger tree
  - and what makes the bitmapped vector trie awesome
    - 32 byte arrays fit cache lines
  
  

