For some reason I lost the introductory part of the notes on the graphics pipeline. Worth revisiting in the future and re-doing the notes.

## Vertex input

Coordinates in OpenGL are in range between -1.0 and 1.0 on all 3 axes (x, y, z). This range is called `normalized device coordinates`. All coordinates within this range will end up on your screen.

```
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};
```

We'd like to sent this defined vertex data as input to the first process of the graphics pipeline: the vertex shader. To do that we create memory on GPU where we store the vertex data, configure how OpenGL should interpret this memory and specify how to sed the data to the GPU.

We manage this memory via `vertex buffer objects (VBO)` that can store large amounts of vertices on the GPU. Thanks to these, we can send large batches of data all at once without having to send the data one vertex at a time. Sending data to GPU from CPU is quite slow, so whenever we can, we try to send as much data as possible. Once the data is there, the vertex shader has almost instant access to it, making it blazingly fast.

``` generating a buffer
unsigned int VBO;
glGenBuffers(1, &VBO);
```
OpenGL has many types of buffer objects and the buffer type of a vertex object is `GL_ARRAY_BUFFER`. OpenGL allows us to bind to several buffers at once as long as they have a different buffer type.

``` binding the buffer
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```

From that point onwards, any buffer calls on GL_ARRAY_BUFFER target will be used to configure the currently bound buffer, which is VBO.

Then we can call `glBufferData` function that copies the previously defined vertex data into the buffer's memory:

```
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

`glBufferData` is a function specifically for sending user-defined data into the currently bound buffer. 
- First argument is the type of the buffer we're sending data into. 
- Second specifies the size of the data (in bytes) a simple `sizeof` of the vertex data.
- Third is the actual data we want to send
- Fourth specifies how we want the GPU to manage the given data. This can take 3 forms:
    - `GL_STREAM_DRAW`: data set only once and used by GPU at most a few times.
    - `GL_STATIC_DRAW`: data set only once and used by GPU many times.
    - `GL_DYNAMIC_DRAW`: data is changed a lot and used many times.

jkk
