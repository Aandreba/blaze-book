# Simple Context

The `SimpleContext` type is the most basic implementation of a context. It contains a single command queue, so no complicated logic is required for it's use, since `next_queue` will always return the same queue.