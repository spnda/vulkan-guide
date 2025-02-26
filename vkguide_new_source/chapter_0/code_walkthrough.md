---
layout: default
title: Code Walkthrough
parent: "New 0. Initial Setup"
nav_order: 3
---

# Walkthrough
Because we are starting this chapter with an already made code skeleton, we are going to see what it actually does.

The files are all stored in the project/src/ folder

- vk_engine.h/cpp : This will be the main class for the engine, and where most of the code of the tutorial will go
- main.cpp : Entry point for the code. Has nothing but just calls into vk_engine code
- vk_initializers.h/cpp : This will contain helpers to create vulkan structures
- vk_images.h/cpp : This will contain image related vulkan helpers
- vk_pipelines.h/cpp : Will contain abstractions for pipelines. 
- vk_descriptors.h/cpp : Will contain descriptor set abstractions.
- vk_loader.h/cpp : Will contain GLTF loading logic.
- vk_types.h : The entire codebase will include this header. it will provide widely used default structures and includes.

vk_engine will be our main engine class, and the core of the project. vk_loader will be tied into it as it will need to interface it while loading GLTF files.
The other files are for generic vulkan abstraction layers that will get built as the tutorial needs. Those abstraction files have no dependencies other than vulkan, so you can keep them for your own projects.

# Code

^code main chapter-0/main.cpp

We start with something simple, main.cpp. We do nothing here except immediately call into the Vulkan engine methods. 

In the future, this could be a good place to set some configuration parameters brought from the command line arguments at argc/argv or a settings file.

vk_types.h holds this

^code intro shared/vk_types.h

`#pragma once` is a preprocessor directive that tells the compiler to never include this twice into the same file. It's equivalent to include guards, but cleaner.

We include the main header for Vulkan, which is the `<vulkan/vulkan.h>` you can see. That will include all the Vulkan function definitions and types that we will need for everything. We also include the fmt lib core header as we will use it across the codebase and create a VK_CHECK macro that we will use for our error handling on vulkan calls. 
We will be using the `vk_enum_string_helper.h` on the tutorial. This is a Vulkan SDK provided header that lets us get the string for a given vulkan enum. Very useful when logging like in this case.

The tutorial is not going to use the standard std::cout for printing information. We will use {fmt} lib instead. This is a very high quality library for formatting strings and printing them. Cpp 20 std::format is based on this library, but we can use the library to get a much wider feature set and better support. In here, we use `fmt::println` to output an error to the console in the case the vulkan gives an error. 

vk_initializers.h is prewritten. It contains initializers for most of the vulkan info structs and other similar ones. They abstract those structs slightly, and every time we use one of them, its code and abstraction will be explained.

We include the vk_types header, which brings Vulkan itself (we will need it), and we declare a namespace for the functions we will add here later.

Finally, we get into vk_engine.h, the main class

^code intro chapter-0/vk_engine.h 

As with vk_initializers, we include vk_types. We already need a Vulkan type in VkExtent2D.
The Vulkan engine will be the core of everything we will be doing. We will be centralizing almost everything the engine does into this class, this way we can simplify the architecture for the project.

We have a flag to know if the engine is initialized, a frame number integer (very useful!), and the size of the window we are going to open, in pixels. 

The declaration `struct SDL_Window* window;` is of special interest. Note the `struct` at the beginning. This is called a forward-declaration, and it's what allows us to have the `SDL_Window `pointer in the class, without including SDL on the Vulkan engine header. This variable holds the window that we create for the application.

We are also adding a Get() function as global singleton pattern.

With the headers seen, let's go to the cpp files.

vk_engine.cpp line 1

^code includes chapter-0/vk_engine.cpp

Unlike in the other files, in this one we include a few more things.
We include both `<SDL.h>` and `<SDL_vulkan.h>`. SDL.h holds the main SDL library data  for opening a window and input, while SDL_vulkan.h holds the Vulkan-specific flags and functionality for opening a Vulkan-compatible window and other Vulkan-specific things. We also add some STL headers we will need.

vk_engine.cpp, line 10

^code init chapter-0/vk_engine.cpp

Here we see our first proper code, in the shape of creating a SDL window.
The first thing we do is init the SDL library. The SDL library contains quite a few things, so we have to send a flag of what do we want to use. SDL_INIT_VIDEO tells SDL that we want the main windowing functionality. That also includes basic input events like keys or mouse. 

We also set a global pointer for the vulkan engine singleton reference. We do that instead of a typical singleton because we want to control explicitly when is the class initalized and destroyed. The normal Cpp singleton pattern doesnt give us control over that.

Once SDL has been initialized, we use it to create a window. The window is stored on the `_window` member for later use.

Because SDL is a C library, it does not support constructors and destructors, and things have to get deleted manually. 

If the window is created, it also has to be destroyed.
^code extras chapter-0/vk_engine.cpp

In a similar way that we did `SDL_CreateWindow`, we need to do `SDL_DestroyWindow`. This will destroy the window for the program. We also clear the singleton pointer for the engine from here, now that the engine is fully cleared.

Over time, we will add more logic into this cleanup function.

Our draw function is empty for now, but here is where we will add the rendering code.

^code drawloop chapter-0/vk_engine.cpp

This is our application main loop. We have an endless loop in the `while()`, that is only stopped when SDL receives the SDL_QUIT event.

On every iteration of the inner loop, we do SDL_PollEvent. This will ask SDL for all of the events the OS has sent to the application during the last frame. In here, we can check for things like keyboard events, mouse movement, window moving, minimization, and many others. For now we are only interested on the SDL_QUIT event and window minimize/restore. When we receive the event that makes the window minimized, we set the stop_rendering bool to true to avoid drawing when the window is minimized. Restoring the window will set it back to false which lets it continue drawing.

And finally, every iteration of the main loop we call either `draw();`, or `std::this_thread::sleep_for` if drawing is disabled. This way we save performance as we dont want the application spinning at full speed if the user has it minimized.

We now have seen how to open a window with SDL, and basically not much else.

There is really only one thing that can be added to this at this point, and that is experimenting with the SDL events. 

As an exercise, read the documentation of SDL2 and try to get keypress events, using `fmt::print` to log them.

Now we can move forward to the first chapter, and get a render loop going.

Next: [Initializing Vulkan]({{ site.baseurl }}{% link docs/new_vkguide/chapter_1/vulkan_init_flow.md %})

{% include comments.html term="Vkguide 2 Beta Comments" %}