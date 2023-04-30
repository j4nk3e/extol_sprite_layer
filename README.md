# `extol_sprite_layer`

`extol_sprite_layer` lets you specify the drawing order for sprites in your [Bevy](https://bevyengine.org/) game using a separate component rather than via the z-coordinate of your transforms.

## Motivation

When making a 2D game in [bevy](https://bevyengine.org/), the z-coordinate is essentially used as a layer index: things with a higher z-coordinate are rendered on top of things with a lower z-coordinate. This works, but it has a few problems:

The z-coordinate isn't relevant for things like distance or normalization. The following code is subtly wrong:

```rs
/// Get a unit vector pointing from the position of `source` to `target`, or zero if they're close.
fn get_normal_direction(query: Query<&GlobalTransform>, source: Entity, target: Entity) -> Vec2 {
  let from_pos = query.get(source).unwrap().translation();
  let to_pos = query.get(target).unwrap().translation();
  (from_pos - to_pos).normalize_or_zero().truncate()
}  
```

The bug is that we normalize *before* truncating, so the z-coordinate still 'counts' for purposes of length. So if the source is at `(0, 0, 0)` and the target is at `(0, 1, 100)`, then we'll return `(0, 0.001)`!
 
If you have entities that are children of other entities, the parent's transform propagates to the child. Sometimes this is what you want, but if it isn't, it leads to things being on the completely wrong layer.

Entities on the same layer are drawn in an effectively arbitrary order that can change between frames. If your game can have entities on the same layer overlap with each other, this means entities will 'flip-flop' back and forth. This is distracting. The usual solution is *y-sorting*, where entities on the same layer are sorted by their y-coordinate, so enemies lower on the screen are drawn on top. Here's an example from my WIP game [tengoku](https://codeberg.org/ext0l/tengoku), which is what led me to develop this crate. In both images, all the enemies (the blue 'soldiers') are on the same layer. In the first one, enemies are drawn roughly in spawn order, which makes the pile appear disorganized and unnatural. The second is y-sorted, resulting in a much cleaner-looking pile. (The red robot in the center, representing the player, is on a higher layer in both cases.)

![z-sorted enemies piled up in a disorderly way](./docs/before.png)
![y-sorted enemies in a much cleaner pile](./docs/after.png)

## How to use

At a high level:

- Define an enum `MyIndex` for your sprite layers that implements the `LayerIndex` trait.
- Add the `SpriteLayerPlugin::<MyIndex>::default()` plugin to your app.
- Insert a `Layer<MyIndex>` component on every sprite, and do *not* set their z-coordinates

For more information, either see the examples or the crate documentation.

## How to contribute

Install [the `just` command runner](https://github.com/casey/just) and run `just setup-repo` to set up hooks (or run the command yourself), then send PRs. I use [Nix](https://nixos.org/) to manage my build environment, but you don't have to. I try to adhere to [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

