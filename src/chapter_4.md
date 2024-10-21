# The log Plugin

One of Bevy's default plugins is the log plugin.

Lets configure the plugin then explain as we see what happens


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


```
The rest of the code of an example application.

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

The console is absolutly flooded with debug messages from the bevy engine itself along with its dependencies.
Libraries use a similar login scheme to what is about to be shown below the idea is that you can filter out irrelevant.
In order to start filtering we need to learn about levels and the Env filter.

## Log Levels 


Tracing (Which the default Logplugin uses for most Operating Systems) has the below log levels from highest (most detailed) to lowest (most important)

- Trace (Not printed by default)
- Debug (Not printed by default)
- Info (default level that is printed)
- Warn
- Error

Its up to you exactly what these mean exactly but here is how I use them in Bevy.

- Trace: details inside a loop
- Debug start or end of loops
- Info usually what I am working on currently I then set to either debug or trace
- Warn: I never use
- Error: let else fail when I don't expect it to


## Env Filter

An env filter is used to put specific filters per target, span and fields. For now just equate targets with modules. 
Spans will be discussed later.

To start with lets put a global env filter set all to warn this can give you useful messages such as a uinode without an appopriate parent


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

### Adding our own logging
Ok so lets add in our own startup warning message.
We can print a warn message using Tracings macro


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

Great we can put a message there but its not really a warning we can move it to the info level.

We use Tracings `info!` macro

```rust
    info!("Hello Log");

```
now lets run

```

     Running `target/debug/test_spiral`
2024-10-19T22:08:28.501325Z  WARN wgpu_hal::vulkan::instance: InstanceFlags::VALIDATION requested, but unable to find layer: VK_LAYER_KHRONOS_validation    
2024-10-19T22:08:28.511056Z  WARN wgpu_hal::vulkan::instance: GENERAL [Loader Message (0x0)]
        terminator_CreateInstance: Failed to CreateInstance in ICD 4.  Skipping ICD.    
2024-10-19T22:08:28.511070Z  WARN wgpu_hal::vulkan::instance:   objects: (type: INSTANCE, hndl: 0x63e8b387ee10, name: ?)    
2024-10-19T22:08:28.518035Z  WARN wgpu_hal::gles::egl: No config found!    
2024-10-19T22:08:28.518049Z  WARN wgpu_hal::gles::egl: EGL says it can present to the window but not natively  
```

Perhaps unsurprisingly The message did not appear.

### Higher level logging for our application

First lets add the depenencies

`cargo add tracing`

`cargo add tracing-subscriber`

In order to show our applications logging we need to change the env filter to show our applications at info level or above.

The syntax introduced will be to change targets

`cratename::modulename`

I called my example crate test_spiral

so I will `test_spiral` there is no module name.

Env filters are comma seperated so `warn,test_spiral=info` will mean "run warn level for as normal, for module test_spiral info"


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
lets run

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


We are going to make a second module we will keep it in the same file for simplicity but it will work for seperate files

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
`?i` just means use debug of that variable to print, otherwise you can use the macros same way as `println!`
lets first change the log level for our application to be at trace

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
In this case you can see the print_trash module ran before my setup code.
also note that the crate module path was printed `test_spiral::spammy` this will soon be relevant

#### Filter out by module

Clearly we have a problem with spammy, we could just delete the spammy logs like we would have to do with techniques in the previous chapter.
However for the sake of the excercies lets pretend that we sometimes is useful

lets put in a log code so we cannot just change the applications logging levels

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

Like so using the path that was printed `test_spiral::spammy`
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

if a log does not appear that I think should I usually just put in error and copy the path

We can now selectively hide messages when we need to! This exciting idea is what lead me to learning more about the tracing crate.

But what about showing line numbers and extra details, this will be covered in the next chapter
