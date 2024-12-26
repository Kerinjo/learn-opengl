## GLFW

GLFW is a library, written in C, specifically targeting OpenGL. It gives us bare necessities required for rendering goodies to the screen. It allows us to create an __OpenGL context__, define window parameters and handle user input (OpenGL Context being it's state).

## GLAD

GLAD manages function pointers for OpenGL, we want to initialize GLAD before we call any OpenGL function.

## Viewport

Before we start rendering, we have to tell OpenGL the size of the rendering window, so OpenGL knows we want to display the data and coordinates wrt to the window. We can set dimensions via glViewport function: 
`glViewport(0, 0, 800, 600);`
The first two parameters set the location of the lower left corner, third and fourth parameter set the width and height of the rendering window in pixels.

The moment a user resizes the window the viewport should be adjusted as well. We can register a callback function on the window that gets called each time the window is resized. The resize callback has the following prototype:
`void framebuffer_size_callback(GLFWwindow* window, int width, int height);` 
The framebuffer size function takes a GLFWwindow as its first argument and two integers indicating the new window dimensions. Whenever the window changes in size, GLFW calls this function and fills in the proper arguments for you to process.
```
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    glViewport(0, 0, width, height); }
    
```
We do have to dell GLFW we want to call this function on every window resize by registering it:
`glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);`
When the window is first displayed framebuffer_size_callback gets called as well. there are differences with width and height for __retina displays__.

There are many callback functions we can set to register to our own functions. For example, we can make a callback function to process joystick input changes, process error messages etc. We register the callback function after we've created the window and before the render loop is initiated.


## Render loop

We don't want the application to draw a single image and then immediately quit and close the window. We want the application to keep drawing images and handling the user input until the program has ben explicitly told to stop. An example of a very simple render loop:
```
wglfwSwapBuffers(window);
glfwPollEvents();hile(!glfwWindowShouldClose(window))
{
    glfwSwapBufffers(window);
    glfwPollEvents();
}
```
The `glfwWindowShouldClose` function checks if GLFW has been instructed to close, if so it returns true and the render loop stops running.
The `glfwPollEvents` function checks if any events are triggered (keyboard input, mouse movement), updates the window state, and calls the corresponding functions (which we register via callback methods).
The `glfwSwapBuffers` will swap the color buffer (a large 2D buffer that contains color values for each pixel in GLFW window)

To compile use:
`g++ hello-window.cpp glad.c -lGL -lglfw`
