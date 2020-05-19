Hello fellow Rustacean!

Ben and I always harp on how Rustacean Station is a community podcast; that even though we are the ones who host episodes most frequently, we are always looking for other people with something to say in the Rust world. And to our delight, Nell Shamrell Harrington, the lead editor of This Week in Rust, got in touch with us the other day asking if Rustacean Station would be a good place to host a weekly This Week in Rust podcast! We jumped at the chance, and so without further ado, here is the very first episode.

Hello everyone! This is the first episode of the This Week in Rust podcast. I’m Nell Shamrell-Harrington, I’m lead editor of This Week in Rust and an engineer on the Rust team at Mozilla.

Every week, this podcast will highlight a few items from the This Week in Rust newsletter. Eventually, I hope to have guests each week to discuss some key stories and announcements from the newsletter, as well as occasional deep dive episodes into some aspect of Rust and the Rust community.

This Week in Rust is usually published on Tuesdays and emailed out on Wednesdays. I will most likely be recording this podcast on Monday night so a link to it can be included in the newsletter. So if you want a chance for your story to be highlighted, make sure to get your pull requests in early each week. I am linking to the This Week in Rust GitHub repo in the show notes.

Let’s dive into this week’s highlights.

News & Blog Post

Rust turned 5 this week! It’s been five wonderful years of pursuing faster, more reliable, and safer software. Check out the core team’s blog post looking back on how the language has changed and grown in the open and through the work of our diverse community of individuals. The title of the article is “Five Years of Rust” and it’s the very first item on the News and Blog Post list.

The next highlighted article was actually posted last year, but brought to our attention this week. It’s called “The case for using Rust for Automotive Software” by Sojan James. It is a fascinating article that uses the 2019 fire in Notre Dame Cathedral to illustrate some of the differences between creating automotive software with C++ and creating it with Rust. Definitely check it out!

One of the Rust teams I am a member of is the Infrastructure team. We have been working on migrating to using GitHub actions as our Continuous Delivery and Integration system. Mateus Costa published an article highlighting how he creates Rust releases for single and multiple targets with GitHub actions. It’s a great reference both for getting started with building Rust software through GitHub actions as well as a great introduction to GitHub actions itself.

Something that has encouraged me as much of the world remains in lockdown is seeing how many Rust meetups are still being held online, it is awesome that we can still connect with each other virtually. Many of these meetups are also posting their recordings on Youtube and it’s a great way to see content from around the world. This week’s newsletter features a video of the Rust and C++ Cardiff Virtual meetup in the UK. It features four talks on Rust and C++ as well as some lightning talks. If you’ve been missing your local Rust meetup, check out what’s available online, there’s a lot.
Finally, I’ve also personally really been enjoying Rust developers streaming their development through YouTube, Twitch, and other services. If you are a newer Rust developer or an advanced developer who just wants to brush up on the fundamentals, you’ll like the called “Jonathan teaches Jason Rust.” You can follow right along as Jonathan Turner - a prominent Rust community member -  takes Jason Turner - A C++ expert - through the basics of Rust development.

If you have always wanted to contribute to Open Source projects, but don’t know where to start, look in the Call for Participation section for tasks that you can pick up and get started with. This week we feature an issue from the clap project - the command line argument parser for Rust - which has a mentor available to help. We also have a couple of issues from the keikan project - an elegant rendering engine written in Rust. Both of these projects are great opportunities to get involved with Open Source software and the Rust language.

The RFC - or request for comment process - is at the core of how the Rust project operates. This week, the RFC which proposes transitioning to Rust-analyzer as our official Language Server Protocol (or LSP) implementation is in final comment period, which means it is reaching a decision soon. If you have an opinion on this, now is the time to express it in that RFC. The link to it is also in the show notes. 

There is also a new RFC about reading into uninitialized buffers. This one focuses on the Read trait - specifically how when you use it it requires that the buffer passed to its various methods be pre-initialized even though the contents will be immediately overwritten. This RFC proposes an interface to allow implementors and consumers of Read types to robustly and soundly work with uninitialized buffers. If you are curious about this, if you have opinions, thoughts, or questions, definitely head to that RFC and comment.

Finally, there are several upcoming events in the Rust community. This week and next there are a lot of online meetups from across the world that are open to all. This includes meetups in Vancouver BC, Turin IT, Dallas TX, Berlin, and Montreal. There is also a meetup in Durham, NC, called the Triangle Rustaceans. Meetups are a great way to not only learn Rust, but also connect with the fantastic community that has made Rust so successful. And because many of these meetups are now online, you can connect with Rust communities from around the world.

That’s all for this week’s podcast - there are a lot more items in this week’s newsletter, make sure to read it to keep up with all the awesome things going on in the Rust world.

This Week in Rust is edited by myself, Andre Bogus, and Colton Donnelly. This week’s newsletter’s contributors include, using their GitHub usernames, mrcosta, nickwilcox, pksunkar, and zoni! Thank you so much to everyone who contributes to This Week in Rust!

Thank you also to the Rustacean station for hosting this podcast! I will be back next week bringing you more of the wonderful news from the wonder world of Rust. Take care everyone.

Show Notes
[This Week in Rust GitHub Repository](https://github.com/emberian/this-week-in-rust)
[Five Years of Rust](https://blog.rust-lang.org/2020/05/15/five-years-of-rust.html)
[The case for using Rust for Automotive Software](https://medium.com/@sojan.james/the-case-for-using-rust-for-automotive-software-19400779f126)
[Rust releases for single and multiple targets with GitHub Actions](https://mateuscosta.me/rust-releases-with-github-actions)
[Rust and C++ Cardiff Virtual Meetup](https://www.youtube.com/watch?v=s8WMaVU3EBs&feature=youtu.be)
[Jonathan Teaches Jason Rust!](https://www.youtube.com/watch?v=EzQ7YIIo1rY&feature=youtu.be)
[RFC: Transition to rust-analyzer as our official LSP (Language Server Protocol) implementation](https://github.com/rust-lang/rfcs/pull/2912)
[RFC: Reading into uninitialized buffers](https://github.com/rust-lang/rfcs/pull/2930)

This Week in Rust is edited by: [nellshamrell](https://github.com/nellshamrell), [llogiq](https://github.com/llogiq), and [cdmistman](https://github.com/cdmistman).

Contributors to this week’s issue include (by GitHub name) mrcosta, nickwilcox, pksunkar, and zoni. Thank you to all who have contributed to This Week in Rust! Take care, everyone.