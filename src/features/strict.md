# Strict
When the `strict` feature is enabled, Blaze will not check for OpenCL support for the specified version at runtime, increasing perfomance. 
When disabled, Blaze will dynamically check the OpenCL version at runtime (when needed), and make adjustments to ensure the maximum compatiblity possible.