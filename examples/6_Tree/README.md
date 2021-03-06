# TinyEngine

## Tree Growth Model
This was a small idea I had to grow trees, but it didn't really constitute its own repository. Since I built it with TinyEngine in about 6 hours, I decided to include it as an example program here.

This was based off of an observation I had while walking in the forest: When a tree branch splits, the sum of the cross-section of the two branches it splits into is conserved (apparently).

This idea was adapted to give a general, simple growth model for trees. The model can produce different tree types by altering parameters. The system is explained below.

![Example Tree Growth](https://github.com/weigert/TinyEngine/blob/master/examples/6_Tree/screenshots/Redtree.png)

![Example Growth 2](https://github.com/weigert/TinyEngine/blob/master/examples/6_Tree/screenshots/Yellowtree.png)

### How it Works
The tree is segmented into branches. Every branch has a depth on the tree (higher depth, further along the tree).

When a branch reaches a certain size (`splitsize`), it will split into two branches. The two branches are split according to the `splitratio` parameter, which determines how balanced the split is. A branch that has not split yet is considered a leaf. The splitsize is dependent on the depth of the branch, using an exponential decay (decay rate `splitdecay`).

The tree is "fed" at a certain rate `growthrate` from the root every time-step. The feed is passed down the tree. Every branch it touches takes a fraction of the feed to grow itself in girth. The remainder is passed to the child branches according to their split-ratio, and this is repeated until the feed is used up. Leaf branches will also grow in length.

The amount that is consumed by a branch vs. passed by the branch is given by the `passratio`. This can be set to a constant value - but in order to conserve cross-sectional area, the pass ratio needs to be set using the formula:

        pass = 1.0 - (A->area + B->area) / (A->area + B->area + area);

Which implements feed-back control on the cross-sectional areas. This means: if the sum of cross-sectional area of the child branches is smaller than that of the original branch, we will pass more than half of the feed along. Otherwise, the branch will consume more than half. This makes the ratio automatically converge to one.

Note that this formula can be adjusted to conserve any other quantity, e.g. branch radius.

Sometimes nice results can be produced while not conserving the cross-sectional area, particularly for branches of small size. Very nice little bushes can be generated by using a constant pass-ratio instead of a size-conserving, continuously adjusted one.

Branches have a direction property, which is a vector in 3D space. The direction of a new branch is computed as a weighted sum between the parent branch's direction and a normal vector to the parent branch. The weight is given by the split ratio, so that thicker child branches are more likely to grow straight, and small ones are more likely to grow perpendicular. Additionally, a `spread` parameter adds extra weight to the normal vector if desired so that branches tend more outwards or straight.

A random normal vector can be computed by taking a random vector in 3D space and computing the cross product.

The branch growth can be directed away from areas with high foliage density by using the normal vector computed from the cross-product of branch direction and a vector pointing towards the local foliage density maximum. This has been implemented to be computed recursively in the `leafdensity` function. Random noise is added for variation. The `directedness` parameter determines the mix between noise and direction away from the local density maximum. The parameter `local size` determines how deep the tree searches / considers a leaf to be "local". Read the `leafdensity` function for more comments on this.

All of these properties can be controlled directly from the control panel.

#### Parameters

        Growth Rate - Rate of Tree Feeding at Root
        Split Ratio - Value of X when a branch splits (how lopsided is the split)
        Pass Ratio - How much of the feed does a branch consume vs. pass on
                Note: If you control for constant cross-sectional area, this does nothing.
        Branch Spread - How much do branches prefer to grow perpendicular vs. straight
        Split Size - Critical size when a branch stops being a leaf node and splits
        Split Decay - Rate of split size exponential decay based on branch depth

Note that these parameters are currently fixed, but could potentially be sampled from distributions instead (e.g. normal distribution around a mean) to give more realism.

This is merely an approximation of reality, and isn't entirely realistic. Other possible improvements could be:

        - More than single splits
        - Dynamic Properties
          - Dependency on the branch depth
        - Different branch types, with varying properties
          - Also spawning different branch types

### Visualization
The leaves are a particle system. They use the image `leaf.png`, and they always face the camera. You can alter color, opacity, size and spread (you can have multiple leaves around a branch).

The tree is meshed by a bunch of cylinders that correspond to the branches. A few parameters then make it look nice (for instance a taper, the scaling of length and width).

All colors can be edited. Optionally, a wire-mesh can be displayed on top. Leaves and the tree itself can be turned off.

Note that in order to make sure the leaf positions are randomized but don't change every frame, their position offset is hashed. This is done by giving every branch a random ID (which stays the same every tick), which can be used to hash the leaves position offset. This allows for generating a particle cloud that doesn't change all the time and stays fixed in 3D space.

A leaf will only show on a leaf-branch that has some minimum depth in the tree.

#### Reading

The model itself is implemented in `tree.h`. Here you will also find how the mesh for the tree and particle system for the leaves are generated.

To see how the interface and event handler are made, read `model.h`.

Everything is wrapped in `main.cpp`.

Overall the code to generate the trees is very brief.

### Usage

Compile with make

    make all

Control Panel:

    Toggle Pause - P
    Toggle Auto-Rotate - A
    Toggle Control Panel - ESC
    Regrow Tree - R
