# Image.jl

This contains some musings on a design/refactoring of [Julia](http://julialang.org/)'s image library. This is at an early stage, and contributors are welcome.

## Aims

The library has the following goals:

- Faithful I/O for all common image file types
- Support for multicolor images, multidimensional images, and image sequences (put together, these are sometimes called "5D images")
- Support for different color spaces (including transparency)
- Support for images that are much too large to hold in memory at once, including tiled images
- Algorithms such as resizing, filtering, spatial transforms/warping (via [Grid.jl](http://github.com/timholy/Grid.jl/), neighborhood processing, etc.

## Design principles

In this draft, perhaps the common unifying idea is to maniuplate images by a set of accessor functions. These functions will allow one to use very simple objects (e.g., `Array`s) or more complex types (including metadata) as images. This strategy is intended to avoid a "get your data into my format" problem that might otherwise arise. The key challenge is representing the "meaning" of any image in a way that is unambiguous, doesn't get in your way, and doesn't cause performance problems.

### An example

Let's consider a sequence of 2D color images over time. If this image sequence is obtained from a color camera, it might correspond to a single disk file representable as an array with dimensions `(3, W, H, T)`, where 3 is the number of colors (RGB camera), `W` is the sensor width, `H` is the sensor height, and `T` is the number of images. Alternatively, if the image sequence is obtained from a fluorescence microscope, it's more likely that a dichroic mirror split the light into "green" and "red" color channels, each sampled by a separate monochrome camera. Consequently, the image is probably represented by two separate disk files (one for each color channel), each of dimensions `(W, H, T)`. Of course, it would be possible to convert these two files into a single file of format `(2, W, H, T)`, and in some cases that might be the right thing to do. But if the image files are hundreds of GB, the user might be interested in being able to manipulate the raw data directly.

Algorithms to process these images might be of two flavors: (1) specialized algorithms for each image representation, or (2) generic algorithms that can access data largely independent of representation. Thanks to multiple dispatch, the first is easy to achieve in Julia. The second requires a collection of accessor functions that "do the right thing" in a variety of cases. It's not obvious how well this can be achieved, but a "schematic" of a generic algorithm might be

```julia
for img in temporal(data)        # iterate (slice) over the different times
    for p in spatial(img)        # additionally slice over space, p is a pixel
        # do some color transformation
    end
end
```

This is possible only if `data` contains enough metadata to convey what's needed about the raw representation. 

For all the advantages of generic algorithms, this author is not very interested in algorithms that look pretty but come with a large performance hit. It will be interesting to see how well one can achieve performance and generality. Julia's forthcoming immutable types seem promising as an additional tool for achieving this goal.

## Status

- A framework for generic I/O, using the magic bytes of the file, is already in place. It's also designed to be readily extensible for custom image formats, so `imread(myfile)` should always work.
- Support for PNG and PPM/PGM/PBM is basically ready (needs refactoring, yay for `try...finally`!)
- Some thoughts on support for direct color & indexed images are in place, and a few prototype accessor functions are available
- Some older but useful code is not incorporated into the new framework, so there's a bit of disjointedness to what you'll see later in the file.