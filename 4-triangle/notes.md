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

## Vertex shader

Vertex shader is responsible for color, position, texture of individual parts vertices).
The vertex shader is one of the shaders programmable by people like us. Modern OpenGL requires at least a vertex shader and a fragment shader to do rendering. 
The first thing to do is to write the vertex shader in GLSL and compile it.
```
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```
Each shader begins with declaration of its version. We also want to explicitly mention the use of core functionality. Next we declare al the input vertex attributes with the `in` keyword. Right now all we care about is the position attribute.
To set the output of the vertex shader we have to assign the position data to the predefined `gl_Position` variable which is a vec4 behind the scenes. At the end of the main function, whatever we set `gl_Position` to will be used as the output of the vertex shader. Since our input is a vec3, we have to cast this to vec4 setting 1.0 as it's w component (w component is not a coordinate, rather it's used for something called `perspective division`).

This vertex shader is possibly the simplest vertex shader we can imagine, there's no processing whatsoever of the input data, it's simply forwarded as the shader's output. In real applications, the input data is usually not yet in normalized device coordinates, so it has to be transformed first.

## Compiling a shader

For now, the source code for the vertex shader can be stored as a const C string at the top of our code file.
```
const char *vertexShaderSource = 
"#version 330 core\n"
"layout (location = 0) in vec3 aPos;\n"
"void main()\n"
"{\n"
    "gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
"}\0";
```
In order for OpenGL to use the shader, it has to be dynamically compiled at run-time from its source code. The first thing we need to do is to create a shader object, again referenced by an ID.

``` Vertex shader object creation, attaching source code and compilation.
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

## Fragment shader

Second and final shader we need for rendering a triangle. Fragment shader is all about calculating the final color output of your pixels.
Colors in computer graphics are represented as an array of 4 values: RGBA. When defining a color in OpenGL or GLSL we set strength of each component between 0.0 and 1.0. Given those 3 color components we can generate over 16 million different colors.

```
#version 330 core
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
}
```

The fragment shader requires only one output variable - a vec4 that defines the final color output. We declare the output variables with the out keyword, here named FragColor.
The process of compiling a fragment shader is similar to the vertex shader, although this time we use the GL_FRAGMENT_SHADER constant as the shader type:

```
unsigned int fragmentShader;
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);
```

After that, both of our shaders are compiled, the only thing left to do is to link both shader objects into a `shader program` that we can use for rendering.

## Shader Program

`Shader program object` is the final linked version of multiple shaders combined. To use the recently compiled shaders, we'd have to link them to a shader program object and activate it when rendering the objects.

```Simple creation of SPO
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
```

Then we need to attach previously compiled shaders to the program object and link them with `glLinkProgram`:

```Attaching and linking previously compiled shaders 
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);
```

We can also check for linking successs, just like we did with compilation of the shaders.
```
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
if (!success)
{
    glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << infoLog << std::endl;
}
```

The result of our attaching and linking, is a program object that we can activate simply by calling `glUseProgram(shaderProgram);`. Every shader and rendering call after `glUseProgram` will now use this program object (and thus the shaders!).
Oh, don't forget to delete the shader objects once they're linked into the program object, we no longer want them.

```
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```

So now the GPU has access to input data(this was done previously with `glBufferData`)
Additionally, we've instructed how the GPU should process the data within the vertex and fragment shader source code. (By linking the shader program object!)
We're almost there, but not yet. OpenGL doesn't yet know how to interpret the vertex data in memory, and how it should connect the data to the vertex shader's attributes.

## Linking Vertex Attributes

Vertex shader allows us to specify any input we want in form of attributes, while this allows for great flexibility, it also means that we have to manually specify which parts of our input data goes to which vertex attribute in the vertex shader. In other words we have to specify how OpenGL should interpret the given input data before rendering.

```
 Vertex 1 | Vertex 2 | Vertex 3 |
 [X][Y][Z]  [X][Y][Z]  [X][Y][Z]
 0  4  8    12 16 20   24 28 32 36
 ---------->
 Position: Stride: 12, Offset: 0
```
- The position data is stored as 32-bit (4 byte) floating point values.
- Each position is composed of 3 of those values.
- There is no space between each set of 3 values, they are `tightly packed` in the array.
- The first value in the data is at the beginning of the buffer.

With this knowledge, we can tell OpenGL how it should interpret the vertex data (per vertex attribute) using glVertexAttribPointer:
```
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
```

The function `glVertexAttribPointer` has quite a few parameters, so let's understand them.

- First parameter specifies which vertex attribute we want to configure. Remember that we specified the `location` of the position vertex attribute with `layout (location = 0)`.
- Next argument specifies the size of the vertex attribute. In this case it's a vec3.
- Third argument specifies the type of the data which is GL_FLOAT, since a vec* in GLSL consists of floating point numbers.
- Next argument specifes if we want the data to be normalized (I don't quite get the use case)
- Fifth parameter is known as the `stride` and tells us the space between consecutive vertex attribues. We could've also specified the stride as 0 to let OpenGL determine the stride. This only works if the values are tightly packed.
- The last parameter is of type void* and thus requires this mystic cast. This is the offset of where the position data begins in the buffer. We'll explore this further on.

## Vertex Array Object

A vertex array object (VAO) can be bound just like a VBO, and any subsequent attribute calls from that point on will be stored inside the VAO. Core OpenGL requires, that we use a VAO so it knows what to do with our vertex inputs. If we fail to bind a VAO, OpenGL will most likely refuse to draw anything.

VAO stores the following:
- Calls to `glEnableVertexAttribArray` or `glDisableVertexAttribArray`.
- Vertex attribute configurations via `glVertexAttribPointer`.
- Vertex buffer objects associated with the vertex attributes by calls to `glVertexAttribPointer`.

The process to generate a VAO looks similar to that of a VBO:

```
unsigned int VAO;
glGenVertexArrays(1, &VAO);
```

To use a VAO you have to bind it using `glBindVertexArray`, bind the corresponding VBOs and attribute pointers, "unbind the VAO for later use"(?). As soon as we want to draw an object, we simply bind the VAO with the preferred settings before drawing the object and that's it.

```
glBindVertexArray(VAO);

glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);

[...]

// in render loop
glUseProgram(shaderProgam);
glBindVertexArray(VAO);
drawTriangle();
```
