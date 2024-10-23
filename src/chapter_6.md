# Tracing multiple files

Using our powers of span we can "Tag" our logs to go to seperate files per span, first we need to learn about layers.


## Subscribers
Subscribers carry many layers, I am not entirely sure about the definition of a subscriber but I believe its basically
a consumer of tracing logs that decides how to output it. I have only used the one on the tracing crate but their are other
variations such as tree logging.

lets read the documentation
"A subscriber is responsible for the following:

    Registering new spans as they are created, and providing them with span IDs. Implicitly, this means the subscriber may determine the strategy for determining span equality.
    Recording the attachment of field values and follows-from annotations to spans.
    Filtering spans and events, and determining when those filters must be invalidated.
    Observing spans as they are entered, exited, and closed, and events as they occur.
"

## Layers

I will just start with an example.


```rust
let stdout_layer = tracing_subscriber::fmt::layer()
    .with_line_number(true)
    .with_file(true)
    .without_time()
    .pretty()
    /* Put less important stuff info files */
    .with_filter(EnvFilter::from_str("warn,test_spiral=info").unwrap());
```

Layers do the presentation and filtering of data, in the example above this sends data to the standard output.

```rust
let log_file = File::create("logs/start.log").unwrap();
let start_layer = tracing_subscriber::fmt::layer()
    .with_line_number(true)
    .with_file(true)
    .without_time()
    .with_ansi(false)
    .with_writer(Mutex::new(log_file))
    .with_filter(EnvFilter::from_str("none,test_spiral::[start]").unwrap());
```

This above layer will send the file to a file by output log based on all spans that have start

we can add them as below, just above your main application,

```rust
//code for layers above
tracing_subscriber::registry()
    .with(stdout_layer)
    .with(start_layer)
    .init();

    App::new()
        //Disable the defaults
        .add_plugins(DefaultPlugins.build().disable::<LogPlugin>())
        .add_systems(Startup, setup)
        .add_systems(Startup, spammy::print_trash)
        .add_systems(Update, some_queries.run_if(run_once()))
        .run();

```
