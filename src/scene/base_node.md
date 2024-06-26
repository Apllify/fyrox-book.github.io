# Base node

Base node is a scene node that stores hierarchical information (a handle to the parent node and a set of handles
to children nodes), local and global transform, name, tag, lifetime, etc. It has self-describing name - it
is used as a base node for every other scene node (via composition).

It has no graphical information, so it is invisible all the time, but it is useful as a "container" for children
nodes.

## How to create

Use the `PivotBuilder` to create an instance of the Pivot node (remember `Base` node itself is used only to build other
node types):

```rust,no_run
# extern crate fyrox;
# use fyrox::scene::{base::BaseBuilder, pivot::PivotBuilder, Scene};
# fn build_node(scene: &mut Scene) {
let handle = PivotBuilder::new(BaseBuilder::new()).build(&mut scene.graph);
# }
```

## Building a complex hierarchy

To build a complex hierarchy of some nodes, use `.with_children()` method of the `BaseBuilder`, it allows you
to build a hierarchy of any complexity:

```rust,no_run
# extern crate fyrox;
# use fyrox::scene::{base::BaseBuilder, pivot::PivotBuilder, camera::CameraBuilder, Scene};
#
# fn build_node(scene: &mut Scene) {
let handle = PivotBuilder::new(BaseBuilder::new()
    .with_children(&[
        CameraBuilder::new(BaseBuilder::new()).build(&mut scene.graph),
        PivotBuilder::new(BaseBuilder::new()
            .with_children(&[PivotBuilder::new(BaseBuilder::new()).build(&mut scene.graph)]))
            .build(&mut scene.graph),
    ]))
    .build(&mut scene.graph);
# }
```

Note that when we're building a `Camera` instance, we're passing a new instance of `BaseBuilder` to it, this
instance can also be used to set some properties and a set of children nodes.

The "fluent syntax" is not mandatory to use, the above code snipped could be rewritten like this:

```rust,no_run
# extern crate fyrox;
# use fyrox::scene::{base::BaseBuilder, pivot::PivotBuilder, camera::CameraBuilder, Scene};
# 
# fn build_node(scene: &mut Scene) {
let camera = CameraBuilder::new(BaseBuilder::new()).build(&mut scene.graph);

let child_base = PivotBuilder::new(BaseBuilder::new()).build(&mut scene.graph);

let base = PivotBuilder::new(BaseBuilder::new()
    .with_children(&[child_base]))
    .build(&mut scene.graph);

let handle = PivotBuilder::new(BaseBuilder::new()
    .with_children(&[camera, base]))
    .build(&mut scene.graph);
# }
```

However, it looks less informative, because it loses the hierarchical view and it is harder to tell the relations
between objects.

## Transform

Base node has a local transform that allows you to translate/scale/rotate/etc. your node as you want to. For example,
to move a node at specific location you could use this:

```rust,no_run
# extern crate fyrox;
# use fyrox::{
#    core::{algebra::Vector3, pool::Handle},
#    scene::{node::Node, Scene},
# };
#
# fn translate_node(scene: &mut Scene, node_handle: Handle<Node>) {
scene.graph[node_handle]
    .local_transform_mut()
    .set_position(Vector3::new(1.0, 0.0, 2.0));
# }
```

You could also chain multiple `set_x` calls, like so:

```rust,no_run
# extern crate fyrox;
# use fyrox::{
#    core::{algebra::Vector3, pool::Handle},
#    scene::{node::Node, Scene},
# };
#
# fn transform_node(scene: &mut Scene, node_handle: Handle<Node>) {
scene.graph[node_handle]
    .local_transform_mut()
    .set_position(Vector3::new(1.0, 0.0, 2.0))
    .set_scale(Vector3::new(2.0, 2.0, 2.0))
    .set_rotation_offset(Vector3::new(1.0, 1.0, 0.0));
# }
```

See more info about transformations [here](./transform.md).

## Visibility

`Base` node stores all info about local visibility and global visibility (with parent's chain visibility included).
Changing node's visibility could be useful if you want to improve performance by hiding distant objects (however it 
strongly advised to use level-of-detail for this) or to hide some objects in your scene. There are three main methods
to set or fetch visibility:

- `set_visibility` - sets local visibility for a node.
- `visibility` - returns current local visibility of a node.
- `global_visibility` - returns combined visibility of a node. It includes visibility of every parent node in the 
hierarchy, so if you have a parent node with some children nodes and set parent's visibility to `false`, global visibility
of children nodes will be `false` too, even if local visibility is `true`. This is useful technique for hiding complex
objects with lots of children nodes.

## Enabling/disabling scene nodes

A scene node could be enabled or disabled. Disabled nodes are excluded from a game loop and has almost zero CPU consumption
(their global transform/visibility/enabled state is still updated due to limitations of the engine). Disabling a node
could be useful if you need to completely freeze some hierarchy and do keep it in this state until it is enabled again.
It could be useful to disable parts of a scene with which a player cannot interact to improve performance. Keep in mind,
that enabled state is hierarchical like visibility. When you're disabling a parent node with some children nodes, the
children nodes will be disabled too.