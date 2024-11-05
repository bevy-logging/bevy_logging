# Performance


## Disclaimer
I have not empirically measured any performance at all, so this is all a matter of if very obvious impacts.
If I do measure it later I will update it.

## Logging statements runtime ignored

I have not experienced any issues with just having lots of logging that is ignored at runtime.
It is possible to set levels at compile time, I haven't yet got around to doing it (that is why there is no instructions yet).
Tracing from what was said in Jon Gjensets video goes to great pains to minimise performance costs.


## Logging multiple files.

Previous chapters discussed how its possible to have files throughly recording everything you can possibly care about;
However I noticed massive performance gains just by commenting out the layers that print to the file.


### Mitigating the perfomance issues.

TODO: Test if I can somehow make an IO logging thread in bevy work with tracing.
