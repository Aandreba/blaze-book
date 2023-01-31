# Event Callbacks

OpenCL event callbacks are supported from OpenCL 1.1 onwards. In Blaze, when using OpenCL 1.0, every time you pass a new callback to an `Event` (with `on_complete`, for example) that callback will be sent to a diferent thread, which will execute it when appropiate.

Callback handling threads are spawned for every thread from which you send a callback. This means that if, for example, you call `on_complete` on 10 different threads, 10 new threads will be spawned to handle the callbacks spawned on each thread, but if you call `on_complete` two times on one thread and once in a differen thread, only 2 new threads will be spawned.

These new threads will complete execution whenever their recievers are disconnected and they have no more listeners to handle.