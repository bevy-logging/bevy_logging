# Design patterns


## Log let else queries
The typical quick way when prototyping queries with only one expected item is to just use `unwrap`

However a slighter better option is to use `let else` statements.

lets start with an example


based on our code [here.](https://github.com/bevy-logging/test_spiral)

```rust

use std::str::FromStr;

use bevy::{log::LogPlugin, prelude::*};
use rand::prelude::*;

fn main() {
    App::new()
        //Disable the defaults
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Startup, spammy::print_trash)
        .add_systems(Update, some_queries.run_if(run_once()))
        .add_systems(Update, query_light.run_if(run_once()))
        .run();
}

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
    // commands.spawn(PointLightBundle {
    //     point_light: PointLight {
    //         intensity: 1500.0,
    //         shadows_enabled: true,
    //         ..default()
    //     },
    //     transform: Transform::from_xyz(4.0, 8.0, 4.0),
    //     ..default()
    // });

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

fn some_queries(transformers: Query<&Transform>) {}

fn query_light(light_query: Query<&PointLight>) {
    let light = light_query.get_single().unwrap();

    info!("found the light");
}

mod spammy {

    pub fn print_trash() {
        for i in 0..10 {
            some_function(i);
        }
    }
    fn some_function(i: u8) {
        if i == 3 {}
    }
}
```


A query has been added for the pointlight but it no longer is created
we can see what happens

```
     Running `target/debug/test_spiral`
2024-10-20T23:47:00.160263Z  INFO bevy_diagnostic::system_information_diagnostics_plugin::internal: SystemInfo { os: "Linux 22.04 KDE neon", kernel: "6.8.0-45-generic", cpu: "AMD Ryzen 7 7800X3D 8-Core Processor", core_count: "8", memory: "30.5 GiB" }
2024-10-20T23:47:00.274629Z  INFO bevy_render::renderer: AdapterInfo { name: "NVIDIA GeForce RTX 3070", vendor: 4318, device: 9348, device_type: DiscreteGpu, driver: "NVIDIA", driver_info: "535.183.01", backend: Vulkan }
2024-10-20T23:47:00.691667Z  INFO bevy_winit::system: Creating new window "App" (Entity { index: 0, generation: 1 })
2024-10-20T23:47:00.692031Z  INFO winit::platform_impl::linux::x11::window: Guessed window scale factor: 1
thread 'Compute Task Pool (3)' panicked at src/main.rs:77:42:
called `Result::unwrap()` on an `Err` value: NoEntities("bevy_ecs::query::state::QueryState<&bevy_pbr::light::point_light::PointLight>")
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
Encountered a panic in system `test_spiral::query_light`!
Encountered a panic in system `bevy_app::main_schedule::Main::run_main`!
```

A crash its good we got the issue pointed out to us straight away but we could change
the query_light function to be as below.

```rust
fn query_light(light_query: Query<&PointLight>) {
    let light = light_query.get_single() else {
        error!("no light found");
        return;
    };

    info!("found the light");
}
```

By stopping crashing but logging the issues we can keep the game going and possibly find
multiple issues in one run.

Plus if the system is not that vital, we don't need to ruin the user's experience.

```
     Running `target/debug/test_spiral`
2024-10-20T23:53:21.775972Z  INFO bevy_diagnostic::system_information_diagnostics_plugin::internal: SystemInfo { os: "Linux 22.04 KDE neon", kernel: "6.8.0-45-generic", cpu: "AMD Ryzen 7 7800X3D 8-Core Processor", core_count: "8", memory: "30.5 GiB" }
2024-10-20T23:53:21.904099Z  INFO bevy_render::renderer: AdapterInfo { name: "NVIDIA GeForce RTX 3070", vendor: 4318, device: 9348, device_type: DiscreteGpu, driver: "NVIDIA", driver_info: "535.183.01", backend: Vulkan }
2024-10-20T23:53:22.316914Z  INFO bevy_winit::system: Creating new window "App" (Entity { index: 0, generation: 1 })
2024-10-20T23:53:22.317300Z  INFO winit::platform_impl::linux::x11::window: Guessed window scale factor: 1
2024-10-20T23:53:23.645988Z  INFO test_spiral: found the light
```

Prefer to use `continue` over return in loops otherwise one bad apple will destroy bunch
