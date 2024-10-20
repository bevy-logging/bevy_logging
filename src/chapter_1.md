# Println! and eprintln

## Println!
The first tool which you may reach for when you experience unexpected behaviour is to use `println!`
to print values at key points of your program where your issue might lie.

However 
- Typing or more likely copying and pasting `println!`'s everywhere gets quite tedious.
- Each `println!` should in most cases have some marking string that lets you know which one triggered,
so you can track down the issue and fix it.
- At some point these `println!`'s need to removed or they will block up the logging.
