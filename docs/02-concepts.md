# Concepts in the AXLearn Library

**This doc is still under construction.**


## Input Batch Sharding

When using `SpmdTrainer`, it is common to read and process inputs across all processes and hosts.
For the most common use case where you want each process to have an equal portion of the input batch, this process is mostly transparent to the user.
For more complex use cases, it can be helpful to have a general idea of the what is happening behind the scenes.

When using AXLearn's support for TFDS inputs, the typical way input batch sharding works is:

1. You specify the split for the Tensorflow dataset you want each process to have either
   explicitly using the `read_config` option of `input_data.tfds_dataset()` or
   let it default to splitting evenly per process.
  https://github.com/apple/axlearn/blob/c00c632b99e6a2d87ee7ba94f295b39e0871a577/axlearn/common/input_tf_data.py#L205
  See `input_tf_data.tfds_read_config()` for an example of how to construct a suitable value for
  `read_config` that sets per-process splits.
https://github.com/apple/axlearn/blob/c00c632b99e6a2d87ee7ba94f295b39e0871a577/axlearn/common/input_tf_data.py#L87-L98
2. In each step, each process reads in the data specified by its split, but it is only a local array
   initially.
3. `SpmdTrainer` combines these local arrays into a globally sharded array using
   `utils.host_to_global_device_array()` before passing the global input batch to `_run_step()`.
https://github.com/apple/axlearn/blob/c00c632b99e6a2d87ee7ba94f295b39e0871a577/axlearn/common/trainer.py#L420
https://github.com/apple/axlearn/blob/c00c632b99e6a2d87ee7ba94f295b39e0871a577/axlearn/common/utils.py#L496
