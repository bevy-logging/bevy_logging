
# Introduction to Tracing

## About Tracing
[Tracing](https://crates.io/crates/tracing)  `tracing` is a framework for instrumenting Rust programs to collect structured, event-based diagnostic information."

Tracing is made for and often talked about in regards to asynchronous code, which is where a lot of documentation and discussion is addresses.

However you don't need to use async code to benefit from this crate, I expect I will use this crate for every non-trival program I write in rust!

## Why use Tracing
- Instead of deleting logs that you are not actively using, you can stop sending those logs until they are needed.
- You can log to multiple files by category

## Next Steps from what we have learnt

So far we have learned about logging levels and filtering by module our next goal is to learn how to add more context to our logs.

### instrument

```rust

use bevy::{log::LogPlugin, prelude::*};
use rand::prelude::*;
use tracing::Level;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(LogPlugin {
            filter: "warn,test_spiral=trace,test_spiral::spammy=info".to_string(),
            level: Level::TRACE,
            ..Default::default()
        }))
        .add_systems(Startup, setup)
        .add_systems(Startup, spammy::print_trash)
        .run();
}

fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    info!("Hello Log");
    debug!("Don't hide me");
    // Camera
    commands.spawn(Camera3dBundle {
        transform: Transform::from_xyz(0.0, 50.0, 100.0).looking_at(Vec3::ZERO, Vec3::Y),
        ..default()
    });

    // Light
    commands.spawn(PointLightBundle {
        point_light: PointLight {
            intensity: 1500.0,
            shadows_enabled: true,
            ..default()
        },
        transform: Transform::from_xyz(4.0, 8.0, 4.0),
        ..default()
    });

    let mut rng = rand::thread_rng();

    // Create spiral galaxy
    for i in 0..10000 {
        let t = i as f32 * 0.005;
        let r = 20.0 * t.sqrt();
        let theta = t * 2.5;

        let x = r * theta.cos();
        let z = r * theta.sin();
        let y = (rng.gen::<f32>() - 0.5) * 2.0; // Add some vertical spread

        let size = (0.05 + rng.gen::<f32>() * 0.1) * (1.0 - t / 50.0); // Smaller stars towards the edge
        let brightness = 1.0 - t / 50.0; // Dimmer stars towards the edge

        let color = Color::srgb(brightness, brightness * 0.8 + 0.2, brightness * 0.6 + 0.4);
        let emissive = Color::srgb(
            brightness * 5.0,
            (brightness * 0.8 + 0.2) * 5.0,
            (brightness * 0.6 + 0.4) * 5.0,
        );

        commands.spawn(PbrBundle {
            mesh: meshes.add(Mesh::from(Sphere { radius: size })),
            material: materials.add(StandardMaterial {
                base_color: color,
                emissive: emissive.into(), // Make stars glow
                ..default()
            }),
            transform: Transform::from_xyz(x, y, z),
            ..default()
        });
    }
}

mod spammy {
    use tracing::{debug, info, instrument, trace};

    pub fn print_trash() {
        debug!("I am going to print trash");
        for i in 0..10 {
            trace!(?i);
            some_function(i);
        }
    }
    #[instrument]
    fn some_function(i: u8) {
        if i == 3 {
            info!("Fizz");
        }
    }
}
```

Of particular note is instrument

```rust
#[instrument]
    fn some_function(i: u8) {
        if i == 3 {
            info!("Fizz");
        }
    }
```
Lets run
```
     Running `target/debug/test_spiral`
2024-10-19T22:42:47.680435Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:42:47.690495Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:42:47.690509Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x56938d752410, name: ?)    
2024-10-19T22:42:47.697443Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:42:47.697458Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively    
2024-10-19T22:42:48.169391Z  INFO test_spiral: Hello Log
2024-10-19T22:42:48.169411Z DEBUG test_spiral: Don't hide me
2024-10-19T22:42:48.169448Z  INFO some_function{i=3}: test_spiral::spammy: Fizz
```
Notice that some_function is now printing along with its argument some more context

we can skip the arguments using skip_all which can be useful if some arguments don't have debug

`#[instrument(skip_all)]`

I tend to use skip_all for all my systems as queries can be quite spammy
If I am a system to query all transforms

`.add_systems(Update, some_queries.run_if(run_once()))`

```rust
#[instrument]
fn some_queries(transformers: Query<&Transform>) {
    info!("not actually going to run");
}
```
The output gets quite verbose per query

```
     Running `target/debug/test_spiral`
2024-10-19T22:49:03.036115Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:49:03.046122Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:49:03.046136Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x57ca2fa1e580, name: ?)    
2024-10-19T22:49:03.053094Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:49:03.053108Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively    
2024-10-19T22:49:03.537979Z  INFO setup: test_spiral: Hello Log
2024-10-19T22:49:03.538003Z DEBUG setup: test_spiral: Don't hide me
2024-10-19T22:49:03.537998Z  INFO some_function{i=3}: test_spiral::spammy: Fizz
2024-10-19T22:49:04.866039Z  INFO some_queries{transformers=Query 
{ matched_entities: 10002, state: QueryState { world_id: WorldId(0), matched_table_count: 3, matched_archetype_count: 3, .. }, 
last_run: Tick { tick: 1036800056 }, this_run: Tick { tick: 57 }, 
world: World { id: WorldId(0), entity_count: 10003, archetype_count: 8, component_count: 239, resource_count: 190 } }}: 
test_spiral: not actually going to run
```
Note I added the line breaks to fit the margins

So lets add skip all
`#[instrument(skip_all)]`

### Spans 

I think of spans as tags, I don't have a lot of experience with them so this will be brief and incomplete.

My main use case for tags is grouping structually unrelated code that has a common theme such as timing or system type.

The examples for me are
- Startup systems
- Yearly or montly systems in my stratergy games
- Related ui or input handling

You can use as many spans as you want.

Now lets look at the syntax to use a span inside a fuction
you need code that looks like this

```rust
let span = span!(Level::DEBUG, "start");
let _guard = span.enter();
```
Spans only apply when the logging level is set at it or above.
note _guard is there to stop the variable being dropped immediately.
While _guard is valid the span is applied.

There is shorthands for all logging levels such as `trace_span!`

we can use square brackets to filter by span 

```filter: "warn,test_spiral=trace,test_spiral::spammy=info,test_spiral[start]=trace"```

Now for the entire code

```rust
use bevy::{log::LogPlugin, prelude::*};
use rand::prelude::*;
use tracing::{instrument, span, Level};

fn main() {
    App::new()
        .add_plugins(
            DefaultPlugins.set(LogPlugin {
                filter: "warn,test_spiral=debug,test_spiral::spammy=info,test_spiral[start]=trace"
                    .to_string(),
                level: Level::TRACE,
                ..Default::default()
            }),
        )
        .add_systems(Startup, setup)
        .add_systems(Startup, spammy::print_trash)
        .add_systems(Update, some_queries.run_if(run_once()))
        .run();
}

#[instrument(skip_all)]
fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    let span = span!(Level::DEBUG, "start");
    let _guard = span.enter();

    info!("Hello Log");
    debug!("Don't hide me");
    // Camera
    commands.spawn(Camera3dBundle {
        transform: Transform::from_xyz(0.0, 50.0, 100.0).looking_at(Vec3::ZERO, Vec3::Y),
        ..default()
    });

    // Light
    commands.spawn(PointLightBundle {
        point_light: PointLight {
            intensity: 1500.0,
            shadows_enabled: true,
            ..default()
        },
        transform: Transform::from_xyz(4.0, 8.0, 4.0),
        ..default()
    });

    let mut rng = rand::thread_rng();

    // Create spiral galaxy
    for i in 0..10000 {
        let t = i as f32 * 0.005;
        let r = 20.0 * t.sqrt();
        let theta = t * 2.5;

        let x = r * theta.cos();
        let z = r * theta.sin();
        let y = (rng.gen::<f32>() - 0.5) * 2.0; // Add some vertical spread

        let size = (0.05 + rng.gen::<f32>() * 0.1) * (1.0 - t / 50.0); // Smaller stars towards the edge
        let brightness = 1.0 - t / 50.0; // Dimmer stars towards the edge

        let color = Color::srgb(brightness, brightness * 0.8 + 0.2, brightness * 0.6 + 0.4);
        let emissive = Color::srgb(
            brightness * 5.0,
            (brightness * 0.8 + 0.2) * 5.0,
            (brightness * 0.6 + 0.4) * 5.0,
        );

        commands.spawn(PbrBundle {
            mesh: meshes.add(Mesh::from(Sphere { radius: size })),
            material: materials.add(StandardMaterial {
                base_color: color,
                emissive: emissive.into(), // Make stars glow
                ..default()
            }),
            transform: Transform::from_xyz(x, y, z),
            ..default()
        });
    }
}

#[instrument(skip_all)]
fn some_queries(transformers: Query<&Transform>) {
    info!("not actually going to run");
    trace!("shouldn't appear");
}

mod spammy {
    use tracing::{debug, info, instrument, span, trace, Level};

    pub fn print_trash() {
        let span = span!(Level::DEBUG, "start");
        let _guard = span.enter();

        debug!("I am going to print trash");
        for i in 0..10 {
            trace!(?i);
            some_function(i);
        }
    }
    #[instrument]
    fn some_function(i: u8) {
        if i == 3 {
            info!("Fizz");
        }
    }
}

```

The output

```
     Running `target/debug/test_spiral`
2024-10-19T23:11:16.601564Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T23:11:16.611283Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T23:11:16.611297Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x5662f0635080, name: ?)    
2024-10-19T23:11:16.618229Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T23:11:16.618243Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively    
2024-10-19T23:11:17.119600Z DEBUG start: test_spiral::spammy: I am going to print trash
2024-10-19T23:11:17.119605Z  INFO setup:start: test_spiral: Hello Log
2024-10-19T23:11:17.119626Z TRACE start: test_spiral::spammy: i=0
2024-10-19T23:11:17.119633Z DEBUG setup:start: test_spiral: Don't hide me
2024-10-19T23:11:17.119652Z TRACE start: test_spiral::spammy: i=1
2024-10-19T23:11:17.119665Z TRACE start: test_spiral::spammy: i=2
2024-10-19T23:11:17.119678Z TRACE start: test_spiral::spammy: i=3
2024-10-19T23:11:17.119688Z  INFO start:some_function{i=3}: test_spiral::spammy: Fizz
2024-10-19T23:11:17.119698Z TRACE start: test_spiral::spammy: i=4
2024-10-19T23:11:17.119711Z TRACE start: test_spiral::spammy: i=5
2024-10-19T23:11:17.119723Z TRACE start: test_spiral::spammy: i=6
2024-10-19T23:11:17.119736Z TRACE start: test_spiral::spammy: i=7
2024-10-19T23:11:17.119748Z TRACE start: test_spiral::spammy: i=8
2024-10-19T23:11:17.119760Z TRACE start: test_spiral::spammy: i=9
2024-10-19T23:11:18.441500Z  INFO some_queries: test_spiral: not actually going to run
```


## Using Tracing on its own

For the sake of learning we will disable the default plugin and use Tracing itself.
This also means you can use Tracing on other applications using what you have learnt here.
Tracing uses a subscriber made up of layers I don't really understand the details

```rust
use bevy::{log::LogPlugin, prelude::*};
use rand::prelude::*;
use tracing::{instrument, span, Level};

fn main() {
    tracing_subscriber::fmt().init();

    App::new()
        //Disable the defaults
        .add_plugins(DefaultPlugins.build().disable::<LogPlugin>())
        .add_systems(Startup, setup)
        .add_systems(Startup, spammy::print_trash)
        .add_systems(Update, some_queries.run_if(run_once()))
        .run();
}
```

Lets modify Tracing

```rust   
tracing_subscriber::fmt()
  .without_time()
  .with_file(true)
  .with_line_number(true)
  .pretty()
  .with_env_filter(
      EnvFilter::from_str(
          "warn,test_spiral=debug,test_spiral::spammy=info,test_spiral[start]=trace",
      )
      .unwrap(),
  )
  .init();

 ```
I find time spammy with diff files
I mentioned that `dbg!` had line numbers and file names.
Pretty is useful for console remove if you are redirecting to a txt file

Now lets run


```
     Running `target/debug/test_spiral`
   WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation
    at /home/lyndonm/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-hal-0.21.1/src/vulkan/instance.rs:724

   WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.
    at /home/lyndonm/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-hal-0.21.1/src/vulkan/instance.rs:89

   WARN wgpu_hal::vulkan::instance:     objects: (type: INSTANCE, hndl: 0x5fb6876377c0, name: ?)
    at /home/lyndonm/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-hal-0.21.1/src/vulkan/instance.rs:148

   WARN wgpu_hal::gles::egl: No config found!
    at /home/lyndonm/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-hal-0.21.1/src/gles/egl.rs:269

   WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively
    at /home/lyndonm/.cargo/registry/src/index.crates.io-6f17d22bba15001f/wgpu-hal-0.21.1/src/gles/egl.rs:258

  DEBUG test_spiral::spammy: I am going to print trash
    at src/main.rs:107
    in test_spiral::spammy::start

   INFO test_spiral: Hello Log
    at src/main.rs:40
    in test_spiral::start
    in test_spiral::setup

  TRACE test_spiral::spammy: i: 0
    at src/main.rs:109
    in test_spiral::spammy::start

  DEBUG test_spiral: Don't hide me
    at src/main.rs:41
    in test_spiral::start
    in test_spiral::setup

  TRACE test_spiral::spammy: i: 1
    at src/main.rs:109
    in test_spiral::spammy::start

  TRACE test_spiral::spammy: i: 2
    at src/main.rs:109
    in test_spiral::spammy::start

  TRACE test_spiral::spammy: i: 3
    at src/main.rs:109
    in test_spiral::spammy::start

   INFO test_spiral::spammy: Fizz
    at src/main.rs:116
    in test_spiral::spammy::some_function with i: 3
    in test_spiral::spammy::start

  TRACE test_spiral::spammy: i: 4
    at src/main.rs:109
    in test_spiral::spammy::start

  TRACE test_spiral::spammy: i: 5
    at src/main.rs:109
    in test_spiral::spammy::start

  TRACE test_spiral::spammy: i: 6
    at src/main.rs:109
    in test_spiral::spammy::start

  TRACE test_spiral::spammy: i: 7
    at src/main.rs:109
    in test_spiral::spammy::start

  TRACE test_spiral::spammy: i: 8
    at src/main.rs:109
    in test_spiral::spammy::start

  TRACE test_spiral::spammy: i: 9
    at src/main.rs:109
    in test_spiral::spammy::start

   INFO test_spiral: not actually going to run
    at src/main.rs:96
    in test_spiral::some_queries
```

Line numbers and file names. You may be happy to just work from this point.
Next chapters is basically breaking Tracing up into layers
