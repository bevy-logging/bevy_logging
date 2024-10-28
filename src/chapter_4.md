# The log Plugin

One of Bevy's default plugins is the log plugin.

Let's configure the plugin, then explain as we see what happens


```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(LogPlugin {
            filter: "".to_string(),
            level: Level::TRACE,
            ..Default::default()
        }))
        .add_systems(Startup, setup)
        .run();
}

// The rest of the code...
```

Code can also be found [here.](https://github.com/bevy-logging/test_spiral)

```rust
use bevy::{log::LogPlugin, prelude::*};
use rand::prelude::*;
use tracing::Level;

//main snip

fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
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



```


Run the code and see what happens.

```
//snip
2024-10-19T21:46:13.389733Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) Id(1573,1,vk)    
2024-10-19T21:46:13.389739Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389745Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389751Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389757Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389763Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389769Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) Id(6992,1,vk)    
2024-10-19T21:46:13.389774Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) Id(3085,1,vk)    
2024-10-19T21:46:13.389780Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389786Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389792Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389801Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389807Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389813Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) Id(8504,1,vk)    
2024-10-19T21:46:13.389818Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) Id(4597,1,vk)    
2024-10-19T21:46:13.389824Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) Id(690,1,vk)    
2024-10-19T21:46:13.389830Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389836Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389842Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389848Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389854Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389860Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389866Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) Id(6109,1,vk)    
2024-10-19T21:46:13.389872Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) Id(2202,1,vk)    
2024-10-19T21:46:13.389877Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389883Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"    
2024-10-19T21:46:13.389889Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Vertex Buffer"    
2024-10-19T21:46:13.389895Z TRACE wgpu_core::resource: Destroy raw Buffer (dropped) "Mesh Index Buffer"  

```

The console is absolutely flooded with debug messages from the Bevy engine itself, along with its dependencies.
Libraries use a similar logging scheme to what is about to be shown below. The idea is that you can filter out the irrelevant logs.
In order to start filtering, we need to learn about levels and the Env filter.

## Log Levels 


Tracing (The default Log plugin used for most Bevy Supported platforms with the major exceptions of the web and mobile) has the below log levels from highest (most detailed) to lowest (most important)

- `Trace` (Not printed by default)
- `Debug` (Not printed by default)
- `Info` (default level that is printed)
- `Warn`
- `Error`
- `None` (you turned off logging)

It's up to you what these mean exactly, but here is how I use them in Bevy.

- `Trace` details inside a loop
- `Debug` start or end of loops
- `Info` usually what I am working on currently, I then set it to either debug or trace. I also use this for information
I want to display but haven't built the UI for yet
- `Warn` I never use
- `Error` let else fail when I don't expect it to

Lower levels are more important, while higher levels (by convention, not enforcement!) are more verbose.

For example, at the default level `Info` the levels below `Warn` and `Error` will also print.

Extract from the Tracing library

```rust 
use tracing_core::Level;

assert!(Level::TRACE > Level::DEBUG);
assert!(Level::ERROR < Level::WARN);
assert!(Level::INFO <= Level::DEBUG);
assert_eq!(Level::TRACE, Level::TRACE);
```


## Env Filter

An env filter is used to put specific filters per target, span and fields. For now just equate targets with modules.

Spans will be discussed later.

I have not needed fields, until I use them or someone else supplies an example fields are an excercise for the reader.

To start with let's put a global env filter set all to warn, this can give you useful messages such as a UI-node without an appropriate parent node


```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(LogPlugin {
            filter: "warn".to_string(),
            level: Level::TRACE,
            ..Default::default()
        }))
        .add_systems(Startup, setup)
        .run();
}
```

The output in full is

```
     Running `target/debug/test_spiral`
2024-10-19T22:03:13.935127Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:03:13.944920Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:03:13.944940Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x57584e197e60, name: ?)    
2024-10-19T22:03:13.952153Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:03:13.952167Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively    
```

### The level field
The level field in the log plugin is a quick way to set all logs globally.
A major use case is having release and debug modes with different log levels.

### Adding our own logging
Ok, so let's add in our own startup warning message, warning because only they and errors will show.
We can print a `Warn` message using Tracing's supplied macros


```rust
fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    warn!("Hello Log");
    // Camera
    commands.spawn(Camera3dBundle {
        transform: Transform::from_xyz(0.0, 50.0, 100.0).looking_at(Vec3::ZERO, Vec3::Y),
        ..default()
    });

```

The output is
```
     Running `target/debug/test_spiral`
2024-10-19T22:06:51.489315Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:06:51.499177Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:06:51.499190Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x61ce04f0ff20, name: ?)    
2024-10-19T22:06:51.506179Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:06:51.506193Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively    
2024-10-19T22:06:52.007886Z  WARN test_spiral: Hello Log

```

Great. We can put a message there, but it's not really a warning, so we can move it to the info level.

We use Tracing's `info!` macro

```info!("Hello Log")```

Now let's run

```

     Running `target/debug/test_spiral`
2024-10-19T22:08:28.501325Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:08:28.511056Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:08:28.511070Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x63e8b387ee10, name: ?)    
2024-10-19T22:08:28.518035Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:08:28.518049Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively  
```

Perhaps unsurprisingly, The message did not appear as it is at a higher level than `warning`.
Therefore, our settings have declared this to be too verbose.

### Higher level logging for our application


First, let's add the depenencies

`cargo add tracing`

`cargo add tracing-subscriber`

In order to show our applications logging, we need to change the env filter to show our applications at the `info` level or above.

The syntax introduced will be to change targets

`cratename::modulename`

I called my example crate test_spiral

So, the target we supply to the env filter will be `test_spiral`. Since there is no module name, it is left blank.

Env filters are comma separated so `warn,test_spiral=info` will mean "run `warn` level for as normal, for module test_spiral `info`"


```rust
fn main() {
    App::new()
        //Disable the defaults
        .add_plugins(DefaultPlugins.set(LogPlugin {
            filter: "warn,test_spiral=info".to_string(),
            level: Level::TRACE,
            ..Default::default()
        }))
        .add_systems(Startup, setup)
        .run();
}
```
Let's run

```
     Running `target/debug/test_spiral`
2024-10-19T22:12:21.298476Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:12:21.307716Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:12:21.307730Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x57003740c3c0, name: ?)    
2024-10-19T22:12:21.314793Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:12:21.314807Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively    
2024-10-19T22:12:21.812880Z  INFO test_spiral: Hello Log
```
Excellent


### Module


We are going to make a second module we will keep it in the same file for simplicity but it will work for separate files

```rust
use bevy::{log::LogPlugin, prelude::*};
use rand::prelude::*;
use tracing::Level;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins.set(LogPlugin {
            filter: "warn,test_spiral=info".to_string(),
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
    use tracing::{debug, trace};

    pub fn print_trash() {
        debug!("I am going to print trash");
        for i in 0..10 {
            trace!(?i);
        }
    }
}
```
`?i` just means use debug of that variable to print. Otherwise you can use the macros the same way as `println!`

Let's first change the log level for our application to be at `trace`

```rust
filter: "warn,test_spiral=trace".to_string(),
```

now run 

```
     Running `target/debug/test_spiral`
2024-10-19T22:24:05.510403Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:24:05.520203Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:24:05.520217Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x59c6f223c5c0, name: ?)    
2024-10-19T22:24:05.527234Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:24:05.527248Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively    
2024-10-19T22:24:06.016182Z DEBUG test_spiral::spammy: I am going to print trash
2024-10-19T22:24:06.016204Z TRACE test_spiral::spammy: i=0
2024-10-19T22:24:06.016210Z TRACE test_spiral::spammy: i=1
2024-10-19T22:24:06.016206Z  INFO test_spiral: Hello Log
2024-10-19T22:24:06.016215Z TRACE test_spiral::spammy: i=2
2024-10-19T22:24:06.016225Z TRACE test_spiral::spammy: i=3
2024-10-19T22:24:06.016230Z TRACE test_spiral::spammy: i=4
2024-10-19T22:24:06.016234Z TRACE test_spiral::spammy: i=5
2024-10-19T22:24:06.016238Z TRACE test_spiral::spammy: i=6
2024-10-19T22:24:06.016243Z TRACE test_spiral::spammy: i=7
2024-10-19T22:24:06.016247Z TRACE test_spiral::spammy: i=8
2024-10-19T22:24:06.016251Z TRACE test_spiral::spammy: i=9
```
In this case, you can see that the `print_trash` module ran before my setup code.
Also note that the crate module path was printed `test_spiral::spammy`. This will soon be relevant

#### Filter out by module

Clearly we have a problem with spammy. We could just delete the spammy logs like we would have to do with techniques in the previous chapter.
However, for the sake of the excercie, let's pretend that we sometimes is useful.

Let's put in a log code so we cannot just change the application's logging levels

```rust
fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    info!("Hello Log");
    debug!("Don't hide me");
```
How do we just hide spammy's messages?

We do it like so, using the path that was printed `test_spiral::spammy`
```rust
filter: "warn,test_spiral=trace,test_spiral::spammy=info".to_string(),

```

```
     Running `target/debug/test_spiral`
2024-10-19T22:32:16.697718Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:32:16.707200Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:32:16.707214Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x55916143c550, name: ?)    
2024-10-19T22:32:16.714329Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:32:16.714344Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively    
2024-10-19T22:32:17.218967Z  INFO test_spiral: Hello Log
2024-10-19T22:32:17.218988Z DEBUG test_spiral: Don't hide me
```

If a log does not appear as I think it should, I usually just put it as an `error` and copy the path.

We can now selectively hide messages when we need to! This exciting idea is what lead me to learning more about the tracing crate.

But what about showing line numbers and extra details? This will be covered in the next chapter.
