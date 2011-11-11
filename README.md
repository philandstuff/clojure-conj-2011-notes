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


