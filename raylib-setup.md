# Raylib Setup

Setting up raylib with zig is very easy, since it has a build.zig of it's own already packaged with it.

## Using The Package Manager

If you don't already have a `build.zig.zon` file in the root directory of your project, add one that has this content
```zig
.{
    .name = "My Game",
    .version = "0.1.0",
    .paths = .{"."},
    .dependencies = .{
      .raylib = .{
        .url = "https://github.com/raysan5/raylib/archive/[SHA_HERE].tar.gz",
      },
    },
}
```

Now go to the [Raylib repo](https://github.com/raysan5/raylib) and click on the "commits" button.<br>
![image](https://github.com/freakmangd/zig-tutorials/assets/53349189/3c2c6eb3-99e4-4859-b5bf-3ccff1376f86)

Click on the "Copy the full SHA" button of the top result (the latest commit)<br>
![image](https://github.com/freakmangd/zig-tutorials/assets/53349189/1538507f-81d5-4c28-9b4d-6021cd7c0093)

Then replace the [SHA_HERE] portion of the `url` field with the SHA in your clipboard

Now when you run `zig build` on your project, you will get output that looks like this
```
Fetch Packages [2/1] raylib... .../build.zig.zon:7:20: err
or: dependency is missing hash field
            .url = "https://github.com/raysan5/raylib/archive/39b539c42f61270631c501e481
076902b7f8fd7c.tar.gz",
                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~
note: expected .hash = "122063dab17eb688f223a9e6fc4e6bd40ac12bcac2fe732084d893b2361f87fc
3d79",
```

That just means we have to add that `.hash` field to our `.raylib` dependency object, so now our full `build.zig.zon` looks like this
```zig
.{
    .name = "My Game",
    .version = "0.1.0",
    .paths = .{"."},
    .dependencies = .{
        .raylib = .{
            .url = "https://github.com/raysan5/raylib/archive/39b539c42f61270631c501e481076902b7f8fd7c.tar.gz",
            .hash = "122063dab17eb688f223a9e6fc4e6bd40ac12bcac2fe732084d893b2361f87fc3d79",
        },
    },
}
```

We can now add the dependency in our `build.zig` file
```zig
    // "raylib" here is the name we gave the dependency object in our build.zig.zon file
    const raylib_dep = b.dependency("raylib", .{
        .target = target,
        .optimize = optimize,
    });
    // "raylib" here is the name the library gave it's artifact in *it's* build.zig file
    exe.linkLibrary(raylib_dep.artifact("raylib"));
```

To use raylib, use `@cImport` and `@cInclude`
```zig
const std = @import("std");

const rl = @cImport({
    @cInclude("raylib.h");
});

pub fn main() !void {
    rl.InitWindow(800, 450, "raylib [core] example - basic window");
    defer rl.CloseWindow();

    rl.SetTargetFPS(60);

    while (!rl.WindowShouldClose()) {
        rl.BeginDrawing();
        rl.ClearBackground(rl.RAYWHITE);
        rl.DrawText("Congrats! You created your first window!", 190, 200, 20, rl.LIGHTGRAY);
        rl.EndDrawing();
    }
}
```

When you need to include raylib from more than one file (which will happen eventually with a big enough project) you cant use cImport more than once, so the recommended strategy is to make a `c.zig` file with all of your C imports.
```zig
// pub usingnamespace forwards all of the declarations of whatever container comes after to the current namespace
pub usingnamespace @cImport({
    @cInclude("raylib.h");
});
```
Then our `main.zig` has just a small change:
```zig
const rl = @import("c.zig");
```
