# Other Subscribers

So far all that has been presented is the fmt subscriber for text output but Tracing can do more then that.
Note I haven't needed to use multiple subscribers, so far I just change globabl subscriber and leave it at that.

## Bunyan Subscriber

I don't know what bunyan formatter is just someone thought it was mentioning, who am I to disagree?
[bunyan](https://crates.io/crates/tracing-bunyan-formatter)

## Gizmo Subscriber

Use to draw gizmos quickly, I have not yet used but I am likely to in future.
I am intereseted in using Gizmos for debugging if I ever get to them, I will write about them.

My understanding of Gizmo's is

They are graphics drawn in immediate mode as opposed to retained;

Therefore they are not fast but can be use for marking places such as where your lights are, or testing rotations.

(Plans are for 0.16 to change this to retained, this is written just before 0.15)

[Gizmo subscriber](https://docs.rs/crate/bevy_gizmo_log/latest)
