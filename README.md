# Introduction

[Book website is here](https://bevy-logging.github.io/)

### About
I have recently been making a game using the bevy engine.
Along the way I have been helped by community with many issues I have encountered both within the 
broader Rust Community as well as the Bevy community.


### Structure
As I have spent more time working with Rust, I have moved from using `println!` to using the tracing crate
for logging.
My Objectives are to
- Describe the transition from basic methods to more advanced tools.
- Explain the reasons that prompted this change in practice.




### Motivation
A common slogan with Rust is "if it compiles it usually works", 
however I have found this is not as much the case with Bevy.

While Rust's compile-time checks catch many errors, Bevy's ECS can introduce runtime issues:

- Queries breaking due to component changes
- Systems running in an unexpected order

Advanced logging and tracing can help catch these issues earlier and provide more context for debugging. 

I hope to help the reader fix issues with their games quicker and give something back to the community.

The documentation around Tracing is overwhelming for me and talks a lot about async which I have not used.



### What this book is not
This book will not talk about debugging tools beyond logging.

Please note this document is not an official document of bevy's

Referenced code can be found [here](https://github.com/bevy-logging/test_spiral)




### Completion state

This book is in a state of generally good enough for my game development.
I prefer to stay on track with my game rather then add to this book, 
therefore unless others contribute, updates will appear as they are useful to me. 
