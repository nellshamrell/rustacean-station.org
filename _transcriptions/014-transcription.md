_Jon Gjengset_: Hello, Ben. How are you doing?

_Ben Striegel_: Hello, Jon. I'm doing good. Welcome back. It's been a while. I mean, the world's a bit different. People who see this in the future might not realize it, but there is a global pandemic ongoing. And I'm recording this from my home. If you hear a jingling, tinkling bell in the background, that is my cat leaping up onto the desk. So, how are you doing, Jon? How are you faring in these uncertain times?

Jon: You know what? It's actually not too bad. It's actually not too bad. You know, it's a different life to be living, but it means that I get to spend more time on various cool side projects. Maybe that's a good thing, although I guess arguably I should be, like, working and finishing my thesis, but you know.

Ben: At one point Rust itself was a side project. So who knows what things might, you know, rise from the ashes?

Jon: That's very true.

Ben: And fortunately, Rust development has always been highly distributed and remote heavy, and so it continues apace.

Jon: It's true, and one other upside of it being a while since last time we recorded is that we now get to record for not just one, but two Rust releases. Yeah, 1.42 and 1.43.

Ben: Let's jump into it. I'm not sure how long it will take me but we'll have a little interlude in between to give our readers time to stretch their legs. But there's plenty of meat to chew on in this episode. So let's jump right in. 1.42.

Jon: So, 1.42 was a pretty cool release, actually. I thought there were a bunch of changes in there that I've wanted for a while and that I think will really have an impact for people. And one of the first ones of these is that you finally get useful line numbers in `Option` and `Result` panic messages.

Ben: This is kind of, it's it's somewhat of a revelation. It's one of those long standing issues that people were like-- it's kind of you accept it as a way things are, like, obviously, if you panic, you're never going to get like a good line number in the error message. It's always going to point to the standard library. And I think people who have been using Rust for a long time might be kind of, habituated to even ignore the output of the panic message. But you have to unlearn that because now they're actually useful, which is-- it's amazing. It's a revelation.

Jon: Yeah, I know, right? Or like, I always have `RUST_BACKTRACE` set to 1 in my environment. And so I always get the full backtrace. And I've sort of trained myself to look about halfway down the backtrace before seeing anything useful. And now that it actually provides something useful near the top, I'm very excited for this change.

Ben: Yeah. Do you want to talk more about how the compiler achieves this and whether or not users can use this in their own code?

Jon: Yeah. So this actually required some kind of subtle changes internally in Rust, because what is needed here is to sort of-- `unwrap` needs to know where it was called from. And so the way this has been implemented internally is this new annotation called `track_caller`. You can't actually use the annotation yourself on stable Rust, although you can on nightly behind a feature flag. And the basic idea here is that every `unwrap` turns, sort of, into a closure. So behind the scenes Rust makes sure that every call to `unwrap` basically remembers where that call happened. And this has some implications for compilation as well as for optimization. And I recommend you take a look at the tracking issue and the underlying RFC if you're interested. I think one thing that's worth pointing out here is `track_caller` is what's currently used to enable this. But `track_caller` itself has not been stabilized because we're still trying to figure out some of the details there. But at least now we know that `unwrap` will give you the useful information, regardless of whether the underlying mechanism changes before it's stabilized.

Ben: So currently, the standard library uses this mechanism internally. And at some point in the future, hopefully users can use it as well, in case they want to make their own kind of abstractions of the same kind.

Jon: Yeah, there's another cool change in 1.42 which is this this notion of subslice patterns and and I hear that you maybe had a hand in this, Ben.

Ben: I don't know about hand. So just like the previous point, this one has been in the works for a long time. And the reason that this one took a while, as opposed to the previous one, which took a long time because of thinking about optimization concerns, code size concerns. This one was just syntax, which is kind of the bane of every programming language designer's existence, where at some point you need to kind of just pick something.

Let's back up, though. So a subslice pattern. If you're used to, you know, working with patterns in Rust or slices, you know that you can match on a slice of things and then put kind of like, you know, if you have a slice of numbers like "1, 2, 3," you can, like, match on that and say, hey, look, you know, if this slice is, you know, the empty slice, just like an open bracket, close bracket, do this. If it contains one element, you can, like, bind the first element to this, and then do two, if it contains two elements, you could do this, and give it a default pattern, for, "I don't know how many are in the slice", which you have to, because there's-- slices don't have any kind of compile-time information about their size.

In this case, now, you can do something different. So if you're-- I think JavaScript has had this for a while now, where you can kind of, like, there's this "dot dot" operator, the "rest" operator, they call it, or I guess it's `...` in JavaScript. But you can say, hey, like, now in a pattern in Rust, you can say, hey, I want to match on this slice, and again, it's Rust. So slices don't have any length information, so we don't know how many are in here, but I want to just take out the first two items here, say, and then ignore the rest.  And you can use the `..` operator, which is generally used in Rust for ignoring a lot of things in patterns.

Jon: So I was going to say, like, so some might argue that this is, like, extending the syntax, right?  Like you're adding more complex syntax to Rust. Do you feel like that's true about this feature?

Ben: No, actually, I think this is a case of-- I judge language features based on whether or not they make the documentation larger or smaller. And in this case, I think this is a change that makes documentation smaller, because generally, if you have a feature and then you have to say, "but you can't use this over here," and then you remove the need for that clause in the documents, you have made your language smaller.

And so I'm not sure if you've ever seen, like, a struct where you want to, like, you know, use, like, you know, I want to say like, hey, I have this struct, or I have this thing, and I put, you know-- let me back up.  So in Rust, if you want to, in a pattern, ignore exactly one thing, like one field in a struct, or like, one item in an array, you use the underscore. And they also have the `..` pattern, which says, you know, ignore all the fields in this, or all the items in this array. And so it just kind of like lets you, you know, avoid writing a bunch of underscores-- underscore comma, underscore comma, etc. And so it's kind of like, `..` is "ignore everything inside of this." And so now, in these slice patterns, you can have, like, a few different things. You can say, hey, I want to, like-- the example given in the blog post for 1.42 is you have, like, a hello world. You know, a slice that says "hello, comma, world, comma." And then you might have, like, you know, some other things on the end and then you can say, I want to match on the slice, it says "hello, comma, world, comma, dot dot", And then, if the slice has anything else after hello world, it will still match that arm of the pattern. And obviously there are some restrictions here where you can't have more than one `..` in a single pattern. Because you have to know, like, understand, where should I, you know, look for-- What parts should I ignore? What parts should I think are concrete? And so, you can have `..` at the end, in the middle, at the beginning, anywhere. But only one of them.

Jon: So what is the subslice part of this feature?

Ben: Basically, in other languages, so in, like, a functional language, you may be used to a pattern where you have, like, you want to have a list of things and you want to get, like, just the head, or just the tail of some kind of list and then, like, recursively go over it and say, hey, stop, the base case for my recursive list is when this is empty. In the meantime, I want to, like, just grab the head, do a thing, pass the tail to the next, you know, recursive invocation of this function, and so on.

And the `..` pattern I was describing just now lets you ignore parts of a list, not actually name them. And so, kind of, like, throw them away, like you would expect from seeing an underscore in a pattern or in a "let". And so if you want to name a pattern in Rust, this was kind of the crux of the syntax issue which held up this RFC for a long time, which is, what kind of syntax we want to actually have for this. If somebody actually wants to name the rest of the stuff that's being matched over.  And so there are various proposals, and then this is where I came in, where just kind of being, like, hey, actually, we already have syntax for this in the language.

And so, I think maybe in the previous podcast we mentioned how in patterns there are these things called "at" patterns. And so it's just the @ symbol on your keyboard, and they let you name a pattern that is also being matched in some other way. And so, last year, I think 1.41 perhaps, we're gradually making @-patterns more powerful.  It's kind of an artifact of Rust 1.0, where because of some last-minute unsoundness, we had to, kind of, limit @-patterns in ways that we knew were a bit too restrictive, but there just wasn't time to actually go through and make it work properly.  Thanks to some heroic efforts over the past year or so, people have finally managed to actually make @-patterns sound again and operate the way they should operate. And so an @-pattern kind of, let's you say, hey, like, I want to match on this pattern but also bind the entire thing -- not just the inside, but the entire thing -- to some name.

Jon: Well, specifically, bind it to this name if it matches.

Ben: So, for example, if I have, like, an array of `1, 2, 3`, of numbers, and I want to match on this, I can have, as a pattern, as an arm of a match, I can say, like, hey, like, yeah, I could say, you know, `[1, 2, 3]`, match on this, or I could say, `[1, _, 3]`, match on that. And I could also say, like, you know, `[a, _, c]`, in that way, and in this case is totally semantically equivalent. If you said `[a, b, c]`, I can also say `[a, b@_, c]` (???), which just says, hey, I want to match this one element, throw it away, but also bind it. It's semantically useless, but it's one of those things where it's, like, yeah, you could do it, you definitely shouldn't, but you could, and we just leverage this fact. This is, kind of, syntax that already, like, exists and should work. And so the syntax for this subslice pattern, finally getting to it, is just like `[a, b@.., c]`, where the `b@..` just says, hey, we're using @-patterns to match the rest of this thing.

Jon: Yeah, I feel like it's it's really neat, right? In some sense it ends up being a very unsurprising syntax, I think.

Ben: Yeah, it's just-- we're composing elements of the syntax that already exist in other places. Again, it is, in my mind, it is a change that makes the language smaller, because, you know, it's-- instead of having to say, "hey, you can't use this here," it's kind of just, like, everything works as you expect.  And then the question is, like, well, like the-- kind of, like, my proposal for this, back in the RFC a long time ago was kind of, hey, like, this doesn't mean that we can't, sometime in the future, have syntax dedicated for, like, sub-slicing, in a way. And I think there's there's even an argument that we should. There are languages that-- so generally in Rust, like, a pattern is the opposite of some kind of like construction. So you're, like, destructing and constructing are two sides of the same coin. And so, for example, in a pattern, if you use an ampersand, to, like, match a reference, that means, like, the opposite of using an ampersand to actually make a reference. So it says, hey, I want to like, you know, dereference this thing because I'm matching whatever is being referenced, and then give me whatever was being referenced by this ampersand. So if you have, like, syntax and patters, you want to, kind of, have this two-way thing.

And the question is, well, is it useful to be able to, like, have a slice and then, like, kind of, like, explode it outward and, yeah, actually, it is sometimes. And so other languages like Python have this, kind of like "unpacking," in a way. I think that's that's the asterisk operator and Python does this, where if I have, like, you know, a list in Python. So `[1, 2, 3]`, or, say I want to-- Let's say I have a list where it's `[2, 3, 4]`, and then I have-- I want to say, make a new list. And so I might say, you know, `a = [1, *b, 5]`, when b is my previous list. And then, you know, a is now `[1, 2, 3, 4, 5]`. And so I can unpack arrays into other arrays, which is just the opposite of what's happening here. So whereas in this case, we are kind of saying, hey, like, match this internal sub-array, we could then imagine in other contexts we want to explode an array and unpack an array. And so, right now there's no syntax for that in Rust. If we did have, we'd probably want to make it general enough that it would work for both pattern contexts and normal contexts.

Jon: Right. And ideally, I guess you would want it to mirror the syntax that's now been introduced for matching them.

Ben: Yeah, and so-- but again, this is not new syntax. All the syntax here is just like old syntax used in ways that are totally composable. The, kind of like, the little thorny thing is that we do somewhat already have, like, in non-pattern contexts and normal expression contexts, a way of doing this. And if you look at-- if you ever use the functional struct updating syntax, this is kind of similar. Where in a struct if you have, like, one instance of a struct, and you want to make a different instance, and you want to reuse-- overwrite one struct with a different struct's fields, you can say, hey, you know, I can have struct a, brace, list some fields, and then do comma, and then, dot dot, the name of the other struct instance. And then it will-- any fields that you didn't name yourself, it will overwrite your new struct's fields, with the fields from the old struct.

Jon: I think this is sort of how people try to emulate something like named function arguments, right?

Ben: Yeah. And it's often used in combination with the default trait to kind of get, like, default documents, too.

Jon: Yeah.

Ben: So there's some question here. If you want to have dedicated syntax for this, you should probably, you know, like, resemble that slightly or is that like, this should be replaced with this and so on. Syntax questions remain difficult to discuss in public, in the open. People always have an opinion. And finding a resolution that satisfies everyone is a difficult job.

Jon: I don't know. I'm pretty happy with the syntax.

Ben: I think it works, yeah.

Jon: So that clearly settles it.

Ben: I'm just happy that we can do it now and that we haven't let syntax...

Jon: Yeah, I think that's true.

Ben: ... block progress any longer where it's like, it's nice to be able to do it. And it's nice that we didn't have to make new syntax. We're just we're reusing all the syntax already means what you wanted to do.

Jon: That's true.

Ben: So, learn to use @-patterns. They're really cool.

Jon: Yeah, they are really handy.

Jon: Speaking of patterns, the next thing that comes up here is something that I'm amazed hasn't existed already. Which is the `matches!` macro.

The `matches!` macro is pretty cool. It just takes two arguments where the first is an expression of any type. And then the second argument is a pattern similar to what you would write in any match arm. And it just evaluates to true, if the given expression matches that pattern and to false otherwise. So it really just desugars to a match, or match of the expression where one arm is the pattern and maps to true and the other arm is like underscore, like anything else, that maps to false.

16:49

And this is really handy for, if you just want to turn some complicated structure, you just want to check whether something is true of it or not. Previously, you had to write an if-let or a match and sort of spread it over four lines. Whereas now, you can just use this new `matches!` macro.

And it comes in particularly handy in tests where, very often in a test there are a bunch of fields you don't care about, like you have some error type, for example, where you want to check whether the kind of an `io::Error` is something that is one of these three `io::Error` kinds. And previously that was really annoying to check in a test. And now you can just write, assert-matches-your error and then a pattern like it would in a match. And it just works.

I think the one thing I feel like is missing here is, I want an `assert_matches` macro that will debug-print the expression if it didn't match, but I'm sure someone will PR that.

Ben: You could write it yourself, perhaps.

Jon: I actually proposed it in the PR that added `matches!` and the response was, that's a good idea. Want to submit a PR? And then I did not. So it's wide open for someone to jump on it. It should be pretty straightforward. And it would be a really cool addition.

Ben: Now, speaking of additions, this is the opposite: deprecation. And so, a venerable part of the Rust standard library, `Error::description` is deprecated. Do you want more about what this is and what it does?

Jon: Yeah. So this is actually kind of interesting. So this is one of the... This is something that was recognized pretty early on as being a mistake. So the `Error` trait in Rust has a function `description`, and the `description` function has a signature of, reference to self, returns a reference to a str. And initially, this made sense, right? Like, you want to describe this error and you want to give a string back and it shouldn't need to be an owned string, because why would it? Unfortunately, this has the downside that it can-- it basically has to be a static string or some string that's stored inside the error itself. But imagine that, like the description for your error wants to refer to a number that is stored inside the error or some other error type or some other type of a field inside the error than previously. You didn't really have a way to include that in your description, because the description has to return a reference to a string so you can't use, say, format inside of `description`, because that returns a string and you can't then return a reference into it because it the string would be dropped when `description` returns.

And so that is why `description` has been deprecated. And what it's being deprecated in favor of, is to just use the display trait. We already have a trait that is for writing a type out as a string, and it does not have this problem, right? If you implement display inside of that implementation, you can use the write! macro to write something that is formatted and uses `Debug` and `Display`, and you don't necessarily need to allocate a string. But you can still use the sort of string formatting that we want to be able to use in order to write out descriptions of basically any struct or other type.

And so, `description` has existed since version 1 of Rust and in Rust 1.27 this was sort of soft-deprecated in that the `description` method on the `Error` trait had a default implementation. So you no longer needed to add an implementation for it. And then in 1.42, it actually got the official deprecated warning, saying, don't use or implement this; use display instead.

Ben: Yeah, I think it's interesting to review how the Rust error-handling library has-- situation has changed over the past few years where I think the first, like, back in the day there was `error_chain` and then `failure`. And then all of these have kind of been, like superseded by, like, changes to the standard library itself, and have informed those changes. And so they were very useful.

And these days, I think my favorite is `thiserror`, by David Tolnay, and also his `anyhow` crate. Where kind of like they serve two different purposes where one is like, I want to consume, I don't really care, errors don't really care about what I do with them. One kind of, I want to produce errors than what we kind of have, like a nice information, and like, takes more effort in time to set up kind of, more fine-grained like, hey, I want, like this throws this error, those this error And they all work via the standard library's `Error` trait these days, and you can derive `Error` for any kind of, I think in this area you can then derive the `Error` trait for any-- as long as your `Error` enum has `Display` and all of its fields.

And there's-- I even saw a crate that builds on this, where you put doc comments on your error types, and there's a little kind of, like formatting syntax for, hey, I want the error name in here or like, you know, I want the field that the error variant contains here in the string and then takes doc comments, uses those to implement `Display` and then derives `Error` and then uses those `Display` impls to do the right thing essentially, and so it's getting pretty good.

Jon: I think it's really cool to see what's happening in the error space in Rust and the fact that there's sort of still innovation going on there. One of the things that we saw was sort of the deprecation of description, but also the addition of the source. The source method on error, in addition to cause. And source allows you to actually downcast the thing you get back to a concrete error type. And this is part of what anyhow uses to be able to give you sort of recursive errors, even though there's only one error type. There are also some some pretty cool experiments, like I know Jane has been working on this thing called `eyre`, and that one is like taking a slightly different approach to anyhow, and it's also a really cool experiment. And I saw recently, there's also an experiment to get `eyre` to use this new `track_caller` feature where it will track every time you use the question-mark operator-- track the location of where that question mark happened, so that you actually get a full chain of where this error came from, rather than having to manually annotate things with like `.context`, for example, if you're familiar with how this works in failure.

Ben: I want it back up. I haven't heard of this Jane author before. We're saying that someone named Jane is writing a library spelled "e-y-r-e"? That's a hilarious, uh, pun. I'm not sure if you noticed that.

Jon: Yeah, this is Jane Lusby. She does a lot of, like, really cool, Rust sort of experimentation. And one of them is this `eyre` library, "e-y-r-e".

Ben: Jane Eyre is a famous novel.

Jon: Yeah, I know. It is really funny.

Ben: OK, I wasn't sure.

Jon: No, it's really fun.

Ben: So, awesome. Okay, and that, I think there are also a few other smaller library things in 1.42 which, I don't think are very interesting. I'm not sure if you want to talk about any of them.

Jon: No, I think I think these are relatively minor ones this time around. I think the ability to use like `proc_macro` token stream is nice, but it's not super important. One thing I do want to call out, you know me. I love diving into, like, the actual changelogs for Rust and cargo and clippy and stuff. And one thing I observed is that Eric Huss has been doing a lot of work on updating the documentation for cargo, and that's really cool. Like now the cargo docs will actually explain things like features and workspaces in a in a slightly more structured and better way. And I think there are more changes coming down the line there as well.

Ben: Cool. The only other thing I see here is downgrading 32-bit Apple targets from tier one to tier three. And I think, kind of just, possibly an llvm thing where it's like, hey, we don't really have the resources to support this and it's not clear who actually wants it. I'm not really an Apple user, so I can't really tell how new, 32 bit. How recent was the last 32-bit Apple stuff?

Jon: I don't think you can get 32-bit Apple things any more.

Ben: I thought they made the switch to 64 bit back in, like, the mid-2000s. But maybe I'm misremembering.  Maybe that was the Intel switch. I don't know.

Jon: Yeah. No, I think you're entirely right.

Ben: I'm sure someone out there is, and I'm sure that they are, you know, frustrated. But, you know, there is only so much that we can do-- platform support-- and it's still there. It's still there, just not tier one anymore. And you know, there's tier one platforms or whatever the Rust compiler team tests for automatically and gates on so you can't ship any kind of nightly that breaks any of these things.

Jon: That's true.

Ben: But it doesn't mean that other platforms don't work. I believe that there is-- we could link to the (???) in the release notes where showing, hey, like here is the Rust platform tier list. Where, here are what platforms are officially supported, with, like, here's what cargo supports, here's what the standard library's compiled for. And here is where you're on your own.

Jon: Yeah, no, I think this seems like a very reasonable change, where like-- this is like, there are a bunch of platforms that, like the vim code base supports, that no one-- someone might still be running vim on those platforms, but it's not-- it's as a hobby and not something that's worth spending a lot of developer resources and code complexity on, probably.

So I guess that leads us to Rust 1.43, which only came out, like, a week ago, right?

Ben: Yeah. We're actually not too far behind on this one.

Jon: Wow. Look at us go.

Ben: As of time of recording.

Jon: Yes, that's true. You're not wrong.

1.43 is is interesting for a couple of reasons.  We'll touch on this a little bit later, but 1.43 has mostly minor changes, but I still think it's worth going through some of them because some of them point at bigger, interesting efforts in the Rust ecosystem. And the first of these is this notion of item fragments. So, in Rust macros for any given argument to the macro, you can specify the type for that argument and the types here are more like types of syntax rather than Rust types. So you can say, like, this has to be an expression, or this has to be a block, or this has to be a type, or this has to be a path. And one of those types is item, which is something like, a function is an item. A method is an item. A trait is an item.

They're sort of like definitions, although people are going to yell at me if I actually say they're definitions.  And one thing that was kind of weird about item fragments is that it used to be that a-- if you wrote, like, a function, a top level function, it wouldn't be considered an item that was valid in an implementation block -- for somewhat stupid reasons, and that's now been fixed. But I think this gets at a bigger point, which is: there are a bunch of things in Rust in both 1.42 and 1.43, where the changes are to add support for more syntactic things, like a top level function that takes a reference to self, like, that code would never compile. But it is useful for the parsers to still allow that to appear there. There are a bunch of other similar ones, if you look at the Rust changelog from 1.42 and 1.43, and the reason why these changes are useful is sort of twofold. The first is for macros, so Rust macros require that the input in is syntactically valid. It does not require that the input is semantically valid because the macro is going to rewrite it hopefully into something that is. But it does require that the input can be parsed by the Rust parser. And this is one of the major reasons why you want the parser to be pretty liberal in what it accepts, because it allows macro authors more flexibility in how they design the languages for the macros they write. The other reason to expand What the what the parser allows, what is syntactically valid, is that for things like error reporting and for code completion and such, you sort of want the compiler to keep going where it can, because if you fail at the parsing stage, you have relatively little information about what went wrong. All you know is that you expected one of these, like, tokens, but you don't necessarily know much about the context; all you have is the parser context. Whereas if you successfully parse it and then it's later in the process that you discover that, like, this function wasn't allowed to be there, then you can provide more semantically meaningful operations like you can say, we're in an impl block. So I was expecting `fn`, but I was not expecting `Trait` because you can't stick a `Trait` in an `impl` block, I think.

And so it should be syntactically valid because it's only later at the sort of semantic stage that we know that we're in a trait block. And similar for completion-- it might be that even though your program is not currently syntactically valid, maybe because you're still typing, you still want the compiler to parser and produce as much information as they can, so that completion can help you wherever your cursor currently is, based on the information that it can derive about the program.

I think you mentioned Ben that you were a little worried about the parser being more liberal because of sort of stability guarantees. Can you say some more about this?

Ben: Well, I think I'm not saying worried. I think that's too strong. I think the idea is that just to, like, keep in mind that-- so, for example: you might think, hey, like you have to use the example of I can't write a (???) function that has ampersand-self as the first argument because that makes no sense. And no matter what you do to the parser, that's never going to compile by itself. The difference is that, as you mentioned, macros-- a macro will accept this and you might ship a macro then that knows how to parse this and then transform it. And so when you make the syntax pass more lenient, that effectively kind of makes the-- that's a promise that you can't take back, in other words. And so people could now begin shipping code on stable Rust that has a notion of what is valid syntax. And you can't ever break that code because of backwards compatibility. And so this has kind of been going on for a while, he says. I'm not I'm not trying to, like, say, worried. I'm not implying that they're doing a haphazard job of it. It's kind of just one of those things where it's like, well, like, should this be part of the RFC process? I'm sure they don't want it to because, you know, it's just they want to. A lot of what they're doing right here is for IDE support as well. I think rust-analyzer has been a big influence on this current thing wanting to share code with rust analyzer and extract code and kind of make the stack passes simpler. Is the syntax library simpler as well. And so that's a big part of it. And it just kind of--

Jon: I think it's a good point that this does become it becomes a part of the Rust specification in some sense.

Ben: Yeah.

Jon: ... that if you wrote, like, if you wrote your own Rust compiler, it would also need to support this other syntax because otherwise there would be programs that would not work.

Ben: So rather than saying, I'm worried about it. It's not that. It's more like I'm just saying that it is a thing to keep in mind where you can't make-- in languages-- it's not obvious, is the thing. Is that a language where you can ship syntactic-- or, you can ship procedural macros, say, on stable, that have some notion of what it means to parse the language that your parser can't get more restrictive over time, only less restrictive.

Jon: Yeah, that's true.

This next change is also one that's sort of relaxing something that previously wasn't possible. And here you wanted to, if I remember correctly, you wanted to draw a distinction between what Rust does for something like constant expressions versus what some other language languages do.  Can you say some more about that?

Ben: Okay. I don't want to talk too much, because I'm not the expert here. But I know that, for example, I had to correct misconceptions in the release announcement for this, in the comments, where it's like, people are like, hey, like, so they look at a code example. I think so. Maybe they know Go for example. And in Go, constants are untyped, and so they use arbitrary precision arithmetic for whatever they're doing. And so, if you like, add two constants together in Go, they won't ever overflow. Or if they're floating point, they won't like-- there will be infinite precision and that kind of thing and don't actually kind of, like, coalesce into a type or in some kind of like, you know, whatever the type that you would expect until you use them in a program somewhere in a non-constant context.

And so, in Rust, this is not the case. And so you might, like, look at the example given in the blog post, where it's like, hey, does this mean that you're using arbitrary position arithmetic for doing your thing?  No, in Rust, the idea is that every numeric literal does have a type. And so if you type 0.0, that is going to be one of the floating point types. If you type "42" that's going to be one of the numeric-- the integer types. And so as normal, as you would expect in Rust, you can-- the Rust compiler will use the usual type inference algorithms, try and figure out what type that literal is. The unique thing here is that if it can, it will try and fall back for floats to `f64`, and for integers to `i32`.

And that's just because otherwise it would be really inconvenient to write things like small tests or small examples where it's, like, I want to just like, `println(.., 42)`, oh, I can't because I have no idea what the type of `42` is. And so, if you write just `fn main println(.., 42)` it works because Rust is just like, obviously it doesn't matter here what type it is and so we're going to make it `i32`.

And so this fallback only kind of kicks in when there's no semantic difference, because it's not going to try and do anything weird or magical with your types here. It's going to be-- if it can't figure out-- if it performs the fallback and then something else happens, it doesn't like, go back and try again with a different type. It just says, hey, like, I couldn't figure it out. And so in this case, it's just, (???) this new change here. It's just a case of saying, hey, like making it a bit smarter about inferring the type. And so the fallback itself hasn't changed. It's just doing better type inference around what primitives might be.

Jon: Yeah.

Ben: So yeah, that's it.

Jon: I mean, I think it's a good change, and it's interesting because this is-- it's sort of a small change, right? Like this is something where if you run into this, the fix was pretty straightforward. But it's really just sort of fixing a wart in the type system. Or it's not even a wart in the type system, it's just, like, a little paper cut, sort of-- that someone would run into; its easy to fix, but it should just work.

And I think this is indicative of some of the fixes we've seen in latest releases, that are starting to surface in the release notes.

Ben: Yeah.

Jon: And Steve Klabnik wrote a really interesting blog post recently where he had this thesis that Rust is changing less over time, like it's changing less now than it was. And he did a bunch of analysis of the release notes, trying to figure out whether that was true and the blog post is worth a read.

One of the conclusions he came to -- I'm gonna try to be careful not to misquote him here -- one of the conclusions was that over time, we are making-- we're still making many changes. But the changes are, more along the lines of, like, fixes or slight library changes as opposed to, like, syntax changes or introducing new concepts in Rust. And so they're sort of lower complexity changes that-- and so even people's-- I think his thesis was correct in that Rust, the language, is sort of changing less, but that doesn't mean that Rust, the standard library and the ecosystem, is changing less. And we see this a little bit in the--

Ben: In the toolchain too, with cargo and all these things.

Jon: Yeah, exactly. And I think we see this in the changelog too, that some of the changes that are surfacing are smaller, maybe, than what we've had in the past. I just remember when we had the async/await release, right, that the changes in 1.43 are relatively minor. But that's not a bad thing. It just means that we're sort of getting into the groove of trying to fix rather than trying to add lots of new things, even though new things are still being added in the background.

And I think this ties also into the Rust roadmap that was established for this year, which was this focus on, I forget what word they used-- but sort of on stability, that, rather than innovate, let's make this, like, the year of catching up to all of the rushing that we've done. Let's sort of take a breath, tidy up the technical debt in a sense we've built up from adding all these features and just sort of complete the language more. And then we can have another iteration where we add a bunch of cool new stuff later on. But this, I think, is an indicator that that road map is being followed.

Ben: Speaking of tiny changes, the toolchain, I think you want to talk a little bit about this cargo environment variable for tests.

Jon: Oh, yeah. So this change is is kind of funny because it's something that I think for most of us, it just does not matter. It's very rare that in your tests, you need to know the location of a binary of that crate. But for some crates it matters a lot, right? Think of something like bindgen, or clap where these are crates where the entire thing -- well, yeah, clap is a good example, too -- where the whole thing sort of revolves around running a binary. And you really want to test that that binary does the right thing and you want tests for that, that actually run the binary as opposed to-- so that you actually test the glue code as well, right? And in order to do that, you need to be able to run the binary. But previously, figuring out where Rust placed the binary was a huge pain because, like, first of all, you need to figure out, OK, where is the target directory? That alone can be complicated because it's not always a subdirectory called "target". For workspaces, it might be one level up, but the user might also set something like the environment variable CARGO_TARGET_DIR, which just moves the target directory to somewhere completely different on the system, and so you need to remember to check all of these cases.

But even in that, like, you need to know where under "target" the binary actually ends up for this particular package's binary, even if someone has, like, renamed the directory or whatever. And this new change means that you just don't have to do that anymore. Cargo will just tell you, this is where the binary is. And I think this is-- it's worth highlighting here--

Ben: I was gonna say, like I was curious, if you could use "cargo run" from within the test runner, if that would cause problems, because that would be my go-to, for, like, where is the binary? Just use cargo run, and it'll find it for me.

Jon: You can. There are actually crates to do this. And there's even a crate for invoking cargo from inside of tests. There some reasons why you don't really want to do this--

Ben: I can imagine.

Jon: Because, yeah, like, for example, what do you do with the output, right? In some sense, you don't want to run the entire cargo pipeline inside of your test. Instead of-- because you can't then, like, run tests in parallel, for example. It would be really nice if instead, cargo just built your binary and then told you where it was. And then you just invoke that binary instead. And that's what this lets you do.

Ben: And this is not the first environment variable that cargo sets, right?

Jon: Yeah, that's true.  In fact, cargo sets lots of environment variables. In most contexts, it sets things like the location of cargo. But it also sets a bunch of things about metadata for the current package. These are things like the version number, or different parts of the version number, the name of the authors, the name of the package, the link to the home page, the description, that kind of stuff. And it also sets the output directory. So this is, where am I compiling this crate into?

This is something you'll often see if you're using bindgen, where bindgen needs to know-- bindgen generates a Rust file right that you didn't then want to include in your other source code.  And the way bindgen does this is bindgen writes it to the output directory. And then if you look at the include macro that the bindgen documentation tells you to write, that includes a file relative to that output directory. Because the output directory, in some sense, is the only place that any part of your build pipeline is allowed to write to. The source, for example, is generally read only because your build process should not be modifying the source files itself.

And if you have, like, cargo build scripts, they get a bunch more, right, so they'd get information about features that have been set, config options, targets, the host triple, all of these other things that you need in the, like, more meta context of, I'm not just in the process of running this through cargo, but I'm actually, like, constructing a Rust program and invoking cargo.

Ben: So moving on to the library changes, I actually do have a thing to talk about that I was actually very involved with, as opposed to previous one where I was just one comment and a long a serious of comments. In this case actually wrote this RFC. And it was for the change mentioned here where you can now use associated constants on floats and integers directly rather than having to import the module. And this is actually kind of a strange holdover from Rust 1.0, and so if you've used Rust, you've probably used traits and you've probably seen associated types, associated functions, and there are also associated constants.

You can have a type and say, I want you to be able to say this constant is this value. I tried to give type names so I can say, like foo::BAR equals 42. And that sort of thing. And, the thing is, though, this feature, while it is natural to think that you would use this feature to say, like, if you wanted to, say, hey what is the bound, what is the upper bound of `i32` or `i64` or `i128`?

Actually, associated constants were not in Rust 1.0. The funny thing is, they were just barely not in Rust 1.0, they were-- It was kind of, like, down to the wire. And they came-- I think they landed-- I think they stabilized 1.1, in fact, but because 1.0, still had to have these constants in them, he kind of stop-gap solution at the time was, hey, forget about it. We'll just, like, because we can shadow type names, and because, you know, integer types are not, like, they're not keywords or anything. They're just normal types. We'll just-- in the standard library, we'll just have a module called `i8`, and `i16`, and `i32`, and `i64`, and `u8` and so on and so on and so on. And we'll just put these constants in those modules. And for the most part, it'll work out.

So, whereas normally if you were reaching for like, hey, I want to get the max integer, you can't just say like, you know, `i32::MAX`, you have to do like, you know, `std::i32::MAX`, or have `use std::i32` up in your program.

Jon: I see. Because `u32` and `f32` there are modules as supposed to the primitive type.

Ben: Yes. Modules, if actually, like, say you have to actually bring them up, you can't actually `use i32`-- you don't want to `use i32` by itself-- `use std::i32` into your code because then you're shadowing the actual type name. That could cause problems; not any bad problems, just like annoyances-- oh, man, actually, this is a bad idea. And so really, it was one of those paper cuts where even experienced programmers would be like, oh, actually, yeah, this does suck, why does it suck so bad?

And I think I was hitting this last year and I was like, man, why does this suck so bad? And just looking into the history of it, and it was kind of like, yeah-- we needed to have these constants available somewhere, and associated constants just weren't stabilized until regrettably, like, six weeks after 1.0 came out. But also, it gets even worse, in fact, because if you looked at the functions, if you look at the documentation for `i32` there is also a function called `max_value`, which gives you the maximum value for `i32` and `i64` and `i16` and all these things and the reason for this is typedefs. And so if you are, like, reading a C library or, like, doing a kind of like C code and you want to figure out hey, like, you know, I have this like this, this typedef thing I have, like, you know, this like, you know this `long`, or this `short`, or this `int` and I want to get the value of the max here, which is actually even more important in those contexts because unlike in Rust, in C you don't actually know what the underlying size might be. But, if you have say, like an int, a C int, you can't just say `use c_int` because that model doesn't exist. Because the hack that we used to actually get this to work relies on you giving the exact type name that you want. And so to get around this, they added in these functions. And so they were just like a `max_value`. It's now you can write `c_int.max_value` to get the maximum value.

So now we have two ways of doing the exact same thing. And, so, they're both kind of the wrong way because there's no need for that to be a function. There's nothing dynamic at all about it, it's just a single value. And so this situation kind of languished. And like all of the comments from back in the day that I compiled were, like, yeah, we'll fix this someday, and then, like four years later, finally, like, I come along and I'm like, yeah, (???) actually doing this and it-- actually the whole point of this is to show that, tying back into Steve's notion of "how often does Rust change", it actually takes a while to get these things through. So I wrote-- I published this RFC for approval, for comments, May 13th, 2019 and it is today April 29th, 2020. So it took about a year for this to go through the RFC process of, like, comment on this, implement it, and then stabilize it.

I also want to shout out to the person who eventually put in all of the effort for implementing this, also at Rust Fest finally pestered me to continue on, because what happened to this RFC was it went on for a while, conversation got stalled, it was nobody's priority as it hadn't been for the previous four years. And finally, I think, Linus Färnstrand, I believe, is his name, finally saw me at Rust Fest and was like, pestered me, like, let's do this, let's do this, let's do this. And he and I worked together to, like, really push the RFC through, ping the lang team, or the libs team, that is, get them to kind of, like, sign off on it and then he did the all the effort of implementing it, which took us a while. Because you have to think about like, compatibility, and, like, now that we have, like, three ways of doing the same thing, as opposed to just two-- How do you document this? How do you point users to the right thing? You want to redefine the other two in terms of this first one in order to reduce kind of like, you know, maintenance burden or understanding-- you know, like, if people are in the code, like, why is this like this? You don't want duplicated code in there, but also because introducing new things you have to stage it appropriately, where you have to have things in nightly before you have them in stable and so on. So it definitely-- it is a process.

And so, like, people-- anyone who says, you know, Rust changes too fast, for a lot of people Rust does not change fast enough. Because for Linus, he was, like, man, raring to go. And in many cases, people just could not move fast enough for him. And so any time that, like you see a change in Rust this understand that these things go through a long process of discussion and approval and implementation and stabilization.

Jon: Do you think that the old versions are gonna end up being deprecated?

Ben: That was one of the big questions of the RFC that we eventually kind of had to-- the way that I worded it finally, is that-- "it is the opinion of the RFC author that these should be deprecated eventually, but we leave the timeline up to future decision."

Jon: Nice.

Ben: And so the idea is that-- I think the final decision was, yes, eventually they should be. It's not a huge deal to deprecate them. And so they might-- I think, ideally in my mind, they would become deprecated in the 2021 edition, if we have one.  I'm not sure if that's a-- (???) that we're doing it at some point edition, But much of next year at some other point.  And then maybe, I don't know, possibly hard-deprecated later on. We can do some things. I mean, it's not-- again, it isn't a big deal.

The whole point is kind of just, for newcomers, especially, you don't want to have three ways of doing the same very simple thing. It just kind of-- it's a bad look, to use the vernacular.

Jon: I mean, I guess you can also do sort of the same thing they did with `Error::description`, right, where you just deprecate it in this edition and point people at the other one. And then you only make it a hard error in the next edition, right? I don't think we need an edition to make it deprecated.

Ben: I think that even in this case, `Error::description` is not actually becoming a hard error, I don't believe, it's just deprecated.

Jon: No, it's just depreciated for now. But I think the intention, presumably, is for the next edition to make it-- a hard error, maybe?

Ben: That's actually-- I think there is-- that's still up in the air. I think it's actually interesting to talk about the whole, like, edition format and the things that we can do. Obviously, like, the point of editions is that if you have Rust 2015 code, a crate that's 2015, a crate that's 2018, a crate that's 2020, any crate can use any other crate freely without having to make any changes. And that's the cool thing.

And so, like, it is honestly, totally imaginable that you could say, hey, like, if I am a 2021 edition crate just like, don't let anyone using this edition use any thing marked as, like, you know, hard error 2021 edition. And so on. And then as an actual step to upgrading your code, you would then-- like, if you opt into the edition, you would then take it out early, or upgrade to a newer--

Jon: Yeah, it would almost be like a lint that becomes deny.

Ben: But I don't think there's ever actually any precedent for doing this, is the thing. I think there's no precedents for doing an API change this way. Maybe for, like, some semantic changes? I think so. For example, another RFC that I wrote for the 2018 edition, which was, "don't let numeric literals overflow their type", which became a, I believe, from a warning to a hard error in the edition. So it is possible, but I don't think it's been used for API changes.

Jon: Yeah, I mean, it's a good question, right? This ties back to "what are editions?" And "what are we allowed to do?" Certainly, one thing I guess that would be necessary is that, it would still-- if you compile against a dependency that uses an old edition, it must still be allowed to use description. And so the deprecation-- it would really be, like, the deprecation becomes a warning-- becomes an error rather than a warning in the new edition. But that doesn't mean that description itself goes away.

Ben: No, it never goes away. And again, because it goes with never goes away, you can't actually ever reuse that, like, so that word is forever used for that purpose. And the only thing you can do is either say, hey, in the new addition, you can't use this. You can't ever make it do something different, say, because you don't ever want to have two symbols with different functions in the same program.

Jon: Yeah. And I mean that we saw this with with `Error::cause` and `Error::source`, right? Where arguably cause should just be source. But you can't do that. That wouldn't be-- that's not a pipeline that we can do without, like, a severe breaking change.

Ben: Wouldn't be compatible.

Jon: There are actually a few other things that I want to mention here. What one of them is in the library changes, which is there's a new primitive module in Rust. And the primitive module is really just a module that re-exports all of the Rust primitive types like `i32` and `i64` and `f32` and all these things. And you might wonder, well, why? Why re-export these? And this ties back to something we've talked about a couple of times on this podcast, which is, if you're writing a macro, like, you can't rely on the user's environment. If you do something, like, just use the `i32` type, who knows whether that is the `i32` type that you expected to be using? The user might have some, like, crate level rewrite or something that just means that that is two completely different type.

And the new primitive module is a way for you to say I want to use `i32` from the standard library. No matter whether there are other `i32`s that exist in this crate. And you really-- it's--

Ben: I do kind of do like this.

Jon: Yeah. I mean, it seems like a sort of weird change, but if you think about it for a second, it actually makes a lot of sense. It's a way for macros to opt out of shadowing, for Rust primitive types.

Ben: Yeah, well also, I don't even use macros. I mean, I don't even write macros, really. But the reason I like it is kind of because, like, in Rust, you have a prelude, which is kind of things that become in scope automatically and so, like, Option, None, Some, Result: these things you don't need to actually import them to use them. And so it may be, it's easy to overlook the fact that, like, all the numeric, the primitive types aren't actually in the prelude, they're kind of just like magically there floating around. And so this kind of makes the primitives a bit less magical, like you can say, hey, these do live here, more or less. I think the way that the (???) put it is that actually might like, sort of actually making the compiler say, hey, these live here, adding new lang items for every single one. It kind of just uses, like, trickery to say, hey, like we're-- can we just export these types as they are.

Jon: Yeah, I think that's right.

Ben: But as far as the users are concerned, it's the same thing.

Jon: So, yeah, it's funny you mentioned the prelude because that's another thing they're discussing for the next edition is, what things do we add to the prelude? And there are arguably a couple of things that people want to add things like maybe future, maybe `TryFrom` and `TryInto`. I've seen questions about `IntoIterator` and `FromIterator`. I think `IntoIterator` maybe already is in the prelude but `FromIterator` is not, so there's an issue you can look up, which is like "things that are being considered for the prelude for the next edition." And it's a pretty interesting read.

Ben: Yeah, we'll find that link that for the show notes. And I think now is the time, if you actually care for these things, to get involved, because the idea is probably by later on this year to have an idea of what's going to change in the next edition and then spent 2021 actually doing it.

Jon: Yeah.

Ben: As opposed to the previous edition, which was kind of, like, just at the last minute, "what are we going to do, what are we going to do, what are we going to do?" And, like, freaking folks out. We have some talks from-- we are a podcast-- have hosted some talks from people who were involved with that saying, hey, that just definitely burned everyone out and did not go over-- even though it went over well, like for users, it definitely had some-- its toll on the developers. Yeah, we want to avoid that this time. If you want to get involved with 2021 edition changes, you want to speak up now and get involved.

Jon: Or forever hold your peace.

Ben: Or I guess until the next edition comes around. I wouldn't say forever. For three years, hold your peace.

Jon: There are a couple of other, like, smaller things I want to bring up, sort of on the tail end here. One is that String now implements `AsMut<str>`. This was a really funny change to me because, like, Strings should obviously work as mutable reference to strings. But the fact that it wasn't added until 1.43 shows just how little people use mutable references to strings. For good reason, right? Like, a mutable reference to a string is a really weird thing because-- because it's UTF-8 encoded, it's very rare that you can mutate anything in place anyway. And so I'm glad this impl was added. It's sort of there for completeness. But it was just funny to observe that no one has needed it until now.

Ben: I think it's kind of like, goes to show too-- I think in Steve's blog post kind of he talks about, hey, most of what is added is library types, standard library types, or standard library functions and types and not all kinds of stuff, traits. But at the same time, Rust has a strange reputation as having a small standard library. And so there's a kind of like discussion to be had about, like some standard libraries are broad, some are very deep, some are broad and deep and-- or, you know, could be broad or shallow, and I think Rust has a very kind of like narrow but deep standard library, is the thing, where it's like there aren't really that many "out there" modules that give you like, you know, there's no web server. There's no regex, say. But like the things it does give you, it gives you an extreme amount of, like, control over, where, it's like all the combinators, all the little reference types that you can do, all the transformations, the Into, the From impls. It's very extensive, and it's an interesting contrast to, say, with like, Python, which is both a very broad and very deep, sort of thing. Language or standard library.

Jon: I think as with every version, I also want to touch briefly on, like, the ecosystem beyond the Rust language itself. The first of these is in cargo.  Cargo has some neat new features in this version that might not be something people notice straight away. The first of these is that in your system wide cargo configuration, you can now use the cargo profile flag that was introduced. I think in 1.41, I want to say. So in 1.41, just to recap, you got the ability to say in your `Cargo.toml`, that like, when you compile this crate, always compile this dependency in release mode. This is handy if, like, you depend on, say, a compression library or something, where even if you're running tests, you need that dependency in release mode because otherwise your tests are going to be too slow. And what has landed now in 1.43 is that you can now set those kind of profile overrides in your global cargo configuration.

This is really neat, right? This means that I can say something like, always compile `ahash` or `fxhash` in release mode, no matter what crate I'm compiling on my system, because I know that the performance of it will just always matter.

And I know you've been-- were surprised that there was such a thing as a system wide cargo config in the first place. And I think this is worth pointing out too. So, in your in your configuration directory for cargo, which from memory is, like, `/home/.cargo/config.toml`, there are a bunch of different configuration options you can set. So one of them is that you can add aliases. So this is where, like, `cargo t` is defined to `test`, but you could add other aliases too, to define additional cargo aliases. You can do things like set what should the default metadata for crates be when I run "cargo new"?  What, like, network settings should cargo use when it fetches stuff? And there are a bunch of other things you can stick in there. And then, of course, one big new one now is you can set these global profile overrides. And that's really cool.

The other new feature that's in cargo, although still I think in cargo nightly is that cargo is getting a new feature resolver. So the current feature resolver, this is sort of if you're in a crate and you say I want this feature, what happens?  Or if you define a dependency with some feature and then some transitive dependency of, you, also depends on the same crate with some different set of features. How does cargo resolve those?  And cargo is getting a new version of that component, that you have to opt into.  And this fixes two-- well, it fixes a couple of different issues. One of the big ones is that currently, the resolver-- imagine that you depend on, say, `tokio`, right. So the `tokio` create has a bunch of different features in it, feature flags that enable things like, Do you want to the multi threaded run time? Do you want support for IO, and whatever. If you write a crate that Only needs, say, the traits that `tokio` defines, but does not need anything else, it doesn't need the runtime. But then in your tests, in your `dev-dependencies`, you do need the runtime, because you need to actually run the tests. So in your dependencies, you list `tokio` with, like, the the "traits" feature or whatever.  But in `dev-dependencies` you list `tokio` with, like, all features turned on. Then, the current resolver is actually going to compile `tokio` only once, regardless of whether you compile it for testing or as a dependency. And it will always include the union of all the features. With this new resolver, it will only compile the features that are needed for the particular thing that you're compiling. And this is really handy if you have a dependency on such a crate. If someone has a dependency on my crate, I don't want them to have to compile, like, the multi-threaded `tokio` runtime if they don't need it, just because I had it in my `dev-dependencies`. This is of course a backwards-incompatible change; this is why you have to opt into it, because it could be that I forgot to list the feature in my dependencies, but it magically worked because it was listed in my `dev-dependencies`. Whereas now that cargo does the right thing, I'm now lacking that feature, my code will no longer compile or run correctly.

Ben: And I'm wondering if that might be the kind of thing that might, even though it's opt-in, it might become automatically added to your `Cargo.toml`, if you have-- in the new edition. And so just like how, if you have a new edition and you type "cargo new", you automatically are using that new edition in your `Cargo.toml`. Maybe at some point that could be automatic

Jon: That would be my guess. This new resolver fixes a bunch of other things too, that I think makes it pretty obvious you would want it to be the default. For example, it means that now features and workspaces are much less of a pain. Previously, if you were in, like, the root of a workspace and you wanted to compile some particular sub-crate with a feature enabled for that sub-crate there just, like, wasn't a way to do that. And with the new resolver you can actually express this. It's just like, it was a problem, and that problem now goes away, with the new resolver, but sadly, the new resolver is not backwards compatible completely. But I think it will actually lead to things like shorter compile times because you will no longer compile quite as many flags and generally, not including a flag removes dependencies and thus reduces compile time.

There's one last thing in the ecosystem that that I think is worth highlighting because it tends to sort of be forgotten, which is, clippy has gotten really good. And if you're not aware of this, try going to the release notes of the last few Rust releases, click clippy and, like, the more detailed changes towards the bottom and look at the changelog and you'll see that every single release adds a heap of new lints, it changes a bunch of the existing lints to get rid of false positives, for example, and some of these are really cool. Some of them actually catch things like correctness issues.  So for example, if you use, say, the atomic types in Rust, then you'll be familiar with the fact that you have to pass in an atomic ordering for every operation and only certain orderings are valid for certain operations. Like, for example, you're not allowed to do a load with, I forget-- you're not allowed to do a load with "release" or a store with "acquire". And previously that would just crash at runtime. The load operation would say "you told me to use this ordering and that's not valid," because ordering is just an enum, and it can't type-check the particular variant. Whereas clippy will actually catch this for you, will say, "you're trying to do a load with the release ordering, and this will crash at runtime. Your program is wrong." and actually give you an an error for that case.  So I really highly recommend, if you're not doing it already, update clippy and run clippy on your programs, because it will catch correctness things, in addition to just un-idiomatic code, which is sort of where clippy started.

Ben: And to get clippy, it doesn't come by default with rustup; I think you need to do "rustup add component clippy", right?

Jon: I forget. This might actually have changed with the the rustup profile stuff that landed a while back. But yeah, if you don't have clippy than just do, I think it's "rustup component add clippy". And then you get it. And you have to add it for each different toolchain, so you want to add it for stable and beta and nightly, if you are using all the those different toolchains. And it's really neat, right? So clippy tries to be somewhat conservative by default, and there's been a push around that recently for getting rid of lints that were maybe overly-annoying and not that helpful. Where clippy actually differentiates between lints that are correctness bugs, lints that are idiomatic, which are on for warnings by default. Then it also has this large category of lints that are pedantic which are turned off by default, like, they're allowed by default. But you can opt into them if you wish.

And this could also be a good way-- just to, if you want to learn more idiomatic Rust, like, turn on pedantic clippy and just see what kind of things it suggests.  Not all of them might be great. Some of them might give false positives, but they might give you a sense for ways in which you could change your code to make it more Rusty. I think that might be all I have for 1.43 and consequently also for 1.42. You have anything else you wanna pitch?

Ben: No, I'm good. So I think we're about ready to wrap this up.

Jon: Nice. Do you-- are there other Rust Fest interviews coming? I'm forgetting whether we have more.

Ben: I have one currently in the works that may or may not be-- it probably will be released before this one does. But then we have one last one and then, uh, who knows? I mean Rust Fest this year, because of The Incident, has gone. I think they're trying for a remote conference around December. So probably be, not the kind of circumstance that will require any kind of interview. But I will be trying to get interviews as I can for people who are doing cool things about the edition this year, I think. And so--

Jon: Yeah, they'll be good. That sounds like a great idea.

Ben: ... interview Niko Matsakis, maybe Felix Klock. So yeah, keep an eye out for those.

Jon: Yeah, I think that would be really cool. And a reminder, of course, to everyone listening that you can also contribute to this podcast.  The goal here is for this to be a community thing. If you have an idea, even just for a single episode, of someone you want to talk to, or some project you want to talk about, then like, reach out and we will happily help you make it happen. We have people who know sound. We have people who can help you getting it published, and that would be awesome. That's how we get more content out there and especially given The Incident, it's really good to, like produce more of this content that people can listen to.  If you saw the recent Rust survey results that came out, people are really looking for sort of more in-depth Rust content that goes beyond just the beginner material. And I think things like interviews with people who are experienced with Rust could be a really good source of that kind of information.

Ben: All right, well, it's been good hearing from you again, Jon. Let's do it again before too long.

Jon: Absolutely, Ben, Likewise. I'll see you soon.
