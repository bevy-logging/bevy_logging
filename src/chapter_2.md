# Redirceting output to a file

Soon you may notice `stdout` is getting full and scrolling up you lose track of where you are.
You may be aware that you can move the output to a file using the `>` operator.

Like below

`Cargo run > output.txt`

This gives you two major benefits
- You can search in the file
- you can use a diff program [^note] to compare outputs for different executions


[^note]: Lots of text editors and IDE's have a built in diff editor, Alternitavely I use Delta



