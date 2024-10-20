# Redirceting output to a file

You may already know `stdout` can be redirected to file using the `>` operator.
When you have lots of logging the screen can get fill quickly so redirecting can help you
manage all that information

An example below.

`Cargo run > output.txt`

This gives you two major benefits
- You can search in the file
- you can use a diff program [^note] to compare outputs for different executions


[^note]: Lots of text editors and IDE's have a built in diff editor, Alternitavely I use Delta



