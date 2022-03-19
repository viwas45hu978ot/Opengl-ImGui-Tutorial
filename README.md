
## Setting up your project

Start of by making a new project.
Then we are gonna make two new folders and call them imgui and GLFW

Then download the latest imGui and GLFW(either 32 or 64 bit depending on your target platform) release it can be found [here](https://github.com/ocornut/imgui/releases) and [here](https://github.com/glfw/glfw/releases)

Then you copy the files to the folder structure as written below
```
.
├── imgui
│   ├── imgui_impl_glfw.cpp
│   ├── imgui_impl_glfw.h
│   ├── imgui_impl_opengl2.cpp
│   ├── imgui_impl_opengl2.h
│   ├── imgui_impl_win32.cpp
│   ├── imgui_impl_win32.h
│   ├── imgui.h
│   ├── imgui.cpp
│   ├── imgui_draw.cpp
│   ├── imgui_internal.h
│   ├── imgui_tables.cpp
│   ├── imgui_widgets.cpp
│   ├── imconfig.h
│   ├── imstb_rectpack.h
│   ├── imstb_textedit.h
│   └── imstb_truetype.h
└── GLFW
    ├── glfw3.h
    ├── glfw3native.h
    └── glfw.lib
```

Now add these files to your project so it looks like this ![Project bin](https://github.com/viwas45hu978ot/Opengl-ImGui-Tutorial/raw/main/Project.png)

Remember to add the imgui and GLFW folders to include and library directories ![Includes](https://github.com/viwas45hu978ot/Opengl-ImGui-Tutorial/raw/main/IncludeLibary.png)

And under linker add opengl32.lib and glfw3.lib ![Linker](https://github.com/viwas45hu978ot/Opengl-ImGui-Tutorial/raw/main/Linker.png)

Now you can go ahead and create your main.cpp file where we have to include ImGui, GLFW and OpenGL
```c++
#include "imgui.h"
#include "imgui_impl_glfw.h"
#include "imgui_impl_opengl2.h"
#include <stdio.h>
#ifdef __APPLE__
#define GL_SILENCE_DEPRECATION
#endif
#include "GLFW/glfw3.h"

#if defined(_MSC_VER) && (_MSC_VER >= 1900) && !defined(IMGUI_DISABLE_WIN32_FUNCTIONS)
#pragma comment(lib, "legacy_stdio_definitions")
#endif

int main(int, char**) {

}
```


## Setting up OpenGL
Now we are gonna setup OpenGL and GLFW to work with ImGui

First off we are gonna setup some basic error logging we will do this right before our main function
```c++
static void glfw_error_callback(int error, const char* description)
{
    fprintf(stderr, "Glfw Error %d: %s\n", error, description);
}

int main(int, char**) {
...
}
```

Then we are gonna setup our window inside our main function
```c++
int main(int, char**) {
    // Setup window
    glfwSetErrorCallback(glfw_error_callback);
    if (!glfwInit())
        return 1;
    GLFWwindow* window = glfwCreateWindow(1280, 720, "ImGui OpenGL", NULL, NULL);
    if (window == NULL)
        return 1;
    glfwMakeContextCurrent(window);
    glfwSwapInterval(1); // This enables vsync so we dont render frames we dont need

}
```

Then we are gonna setup ImGui context so we can use it with our OpenGL window
```c++
int main(int, char**) {
    ...
    // Setup ImGui context
    IMGUI_CHECKVERSION();
    ImGui::CreateContext();
    ImGuiIO& io = ImGui::GetIO(); (void)io;
}
```

Now we are gonna choose the style and backend to use
```c++
int main(int, char**) {
    ...
    // Setup Dear ImGui style
    ImGui::StyleColorsDark(); // you can also use ImGui::StyleColorsClassic();
    

    // Choose backend
    ImGui_ImplGlfw_InitForOpenGL(window, true);
    ImGui_ImplOpenGL2_Init();
}
```

Then we are gonna make some definitions and choose a color for our backbuffer(the color behind our ImGui windows)

```c++
int main(int, char**) {
    ...
        // Definitions
    bool show_main_window = true;
    ImVec4 clear_color = ImVec4(0.45f, 0.55f, 0.60f, 1.00f); // backbuffer color
}
```

Now we will create our main loop and add poll events along with our ImGui frame

```c++
int main(int, char**) {
    ...
    while (!glfwWindowShouldClose(window))
    {
        glfwPollEvents(); // Poll and handle events (inputs, window resize, etc.)

        // Start the Dear ImGui frame
        ImGui_ImplOpenGL2_NewFrame();
        ImGui_ImplGlfw_NewFrame();
        ImGui::NewFrame();


    }
}
```

Then we will add the code that makes ImGui and OpenGL render our frame

```c++
int main(int, char**) {
    ...
    while (!glfwWindowShouldClose(window))
    {
        ...

        // Gui stuff

        // Rendering    
        ImGui::Render();
        int display_w, display_h;
        glfwGetFramebufferSize(window, &display_w, &display_h);
        glViewport(0, 0, display_w, display_h);
        glClearColor(clear_color.x * clear_color.w, clear_color.y * clear_color.w, clear_color.z * clear_color.w, clear_color.w);
        glClear(GL_COLOR_BUFFER_BIT);
        ImGui_ImplOpenGL2_RenderDrawData(ImGui::GetDrawData());
        glfwMakeContextCurrent(window);
        glfwSwapBuffers(window);
    }
}
```

And finally we will add some code to cleanup our window when we exit the program

```c++
int main(int, char**) {
    ...
    while (!glfwWindowShouldClose(window))
    {
        ...
    }
        // Cleanup
    ImGui_ImplOpenGL2_Shutdown();
    ImGui_ImplGlfw_Shutdown();
    ImGui::DestroyContext();

    glfwDestroyWindow(window);
    glfwTerminate();

    return 0;
}
```
Our finished code should look something like this
```c++
#include "imgui.h"
#include "imgui_impl_glfw.h"
#include "imgui_impl_opengl2.h"
#include <stdio.h>
#ifdef __APPLE__
#define GL_SILENCE_DEPRECATION
#endif
#include "GLFW/glfw3.h"

#if defined(_MSC_VER) && (_MSC_VER >= 1900) && !defined(IMGUI_DISABLE_WIN32_FUNCTIONS)
#pragma comment(lib, "legacy_stdio_definitions")
#endif

static void glfw_error_callback(int error, const char* description)
{
    fprintf(stderr, "Glfw Error %d: %s\n", error, description);
}

int main(int, char**) {
    // Setup window
    glfwSetErrorCallback(glfw_error_callback);
    if (!glfwInit())
        return 1;
    GLFWwindow* window = glfwCreateWindow(1280, 720, "ImGui OpenGL", NULL, NULL);
    if (window == NULL)
        return 1;
    glfwMakeContextCurrent(window);
    glfwSwapInterval(1); // This enables vsync so we dont render frames we dont need

    // Setup ImGui context
    IMGUI_CHECKVERSION();
    ImGui::CreateContext();
    ImGuiIO& io = ImGui::GetIO(); (void)io;

    // Setup Dear ImGui style
    ImGui::StyleColorsDark(); // you can also use ImGui::StyleColorsClassic();


    // Choose backend
    ImGui_ImplGlfw_InitForOpenGL(window, true);
    ImGui_ImplOpenGL2_Init();

    // Definitions
    bool show_main_window = true;
    ImVec4 clear_color = ImVec4(0.45f, 0.55f, 0.60f, 1.00f); // backbuffer color

    while (!glfwWindowShouldClose(window))
    {
        glfwPollEvents(); // Poll and handle events (inputs, window resize, etc.)

        // Start the Dear ImGui frame
        ImGui_ImplOpenGL2_NewFrame();
        ImGui_ImplGlfw_NewFrame();
        ImGui::NewFrame();

        // Gui stuff

        // Rendering   
        ImGui::Render();
        int display_w, display_h;
        glfwGetFramebufferSize(window, &display_w, &display_h);
        glViewport(0, 0, display_w, display_h);
        glClearColor(clear_color.x * clear_color.w, clear_color.y * clear_color.w, clear_color.z * clear_color.w, clear_color.w);
        glClear(GL_COLOR_BUFFER_BIT);
        ImGui_ImplOpenGL2_RenderDrawData(ImGui::GetDrawData());
        glfwMakeContextCurrent(window);
        glfwSwapBuffers(window);
    }

    // Cleanup
    ImGui_ImplOpenGL2_Shutdown();
    ImGui_ImplGlfw_Shutdown();
    ImGui::DestroyContext();

    glfwDestroyWindow(window);
    glfwTerminate();

    return 0;
}
```
Now if we run it we will have a blank window ![Blank window](https://github.com/viwas45hu978ot/Opengl-ImGui-Tutorial/raw/main/window.png)
