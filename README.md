# PP12: GUI Programming with X11 and GTK+

## Goal

In this exercise you will:

1. Work directly with the X11 (Xlib) API to create a window and draw primitive graphics.
2. Install GTK+ and build a simple GTK3 application, then extend it with an additional feature.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. When time is up, stop immediately and record where you paused.

---

## Workflow

1. **Fork** this repository on GitHub.
2. **Clone** your fork locally.
3. Create a `solutions/` directory at the project root:

   ```bash
   mkdir solutions
   ```
4. Add each task’s source file(s) under `solutions/`.
5. **Commit** and **push** your solutions.
6. **Submit** your GitHub repo link for review.

---

## Prerequisites

* X11 development libraries (`libx11-dev`).
* GTK+ 3 development libraries (`libgtk-3-dev`).
* GNU C compiler (`gcc`).

---

## Tasks

### Task 1: Bare X11 Window & Drawing

**Objective:** Use Xlib directly to open a window and draw a rectangle.

1. Create `solutions/x11_draw.c` with the following skeleton:

   ```c
   #include <X11/Xlib.h>
   #include <stdlib.h>
   #include <stdio.h>

   int main(void) {
       Display *dpy = XOpenDisplay(NULL);
       if (!dpy) {
           fprintf(stderr, "Cannot open display\n");
           return EXIT_FAILURE;
       }
       int screen = DefaultScreen(dpy);
       Window win = XCreateSimpleWindow(
           dpy, RootWindow(dpy, screen),
           10, 10, 400, 300, 1,
           BlackPixel(dpy, screen),
           WhitePixel(dpy, screen)
       );
       XSelectInput(dpy, win, ExposureMask | KeyPressMask);
       XMapWindow(dpy, win);

       GC gc = XCreateGC(dpy, win, 0, NULL);
       XSetForeground(dpy, gc, BlackPixel(dpy, screen));

       for(;;) {
           XEvent e;
           XNextEvent(dpy, &e);
           if (e.type == Expose) {
               // Draw a rectangle
               XDrawRectangle(dpy, win, gc, 50, 50, 200, 100);
           }
           if (e.type == KeyPress)
               break;
       }

       XFreeGC(dpy, gc);
       XDestroyWindow(dpy, win);
       XCloseDisplay(dpy);
       return EXIT_SUCCESS;
   }
   ```
2. Compile and run:

   ```bash
   gcc -o solutions/x11_draw solutions/x11_draw.c -lX11
   ./solutions/x11_draw
   ```

#### Reflection Questions

1. **What steps are required to open an X11 window and receive events?**

<summuray>
Step 1-Connect to the X server
Display *dpy = XOpenDisplay(NULL);

This connects the program to the X server.

Step 2-Get the default screen
int screen = DefaultScreen(dpy);

Step 3-Create a window
Window win = XCreateSimpleWindow(...);

You define its size, position, colors, and borders here.

Step 4-Select input events you want to handle
XSelectInput(dpy, win, ExposureMask | KeyPressMask);

You must tell X11 which events your program wants to receive (e.g., Expose, KeyPress).

Step 5-Map (show) the window
XMapWindow(dpy, win);

Step 6-Create a graphics context (GC) for drawing
GC gc = XCreateGC(dpy, win, 0, NULL);

Step 7-Event loop: wait for and handle events
for(;;) {
    XEvent e;
    XNextEvent(dpy, &e);
    }
</summuray>

2. **How does the `Expose` event trigger your drawing code?**
 
<summuray>
An Expose event is sent when part of the window needs to be redrawn, e.g.:
    The window is first shown
    The window was covered and is now uncovered
    The user resized the window
In the code:
if (e.type == Expose) {
    XDrawRectangle(dpy, win, gc, 50, 50, 200, 100);
}
    This means when the window needs to be redrawn, your program draws a rectangle again.
    If you didn’t handle Expose, the rectangle might disappear when the window is resized or uncovered.
So the Expose event triggers your drawing code by acting as a signal to redraw the contents of the window.
</summuray>

---

### Task 2: GTK+ 3 Application & Extension

**Objective:** Install GTK+ 3, build a basic GTK application, then add a new interactive feature.

1. Install GTK+ 3:

   ```bash
   sudo apt update && sudo apt install libgtk-3-dev
   ```
2. Create `solutions/gtk_app.c` with this skeleton:

   ```c
   #include <gtk/gtk.h>

   static void on_button_clicked(GtkButton *button, gpointer user_data) {
       GtkLabel *label = GTK_LABEL(user_data);
       gtk_label_set_text(label, "Button clicked!");
   }

   int main(int argc, char **argv) {
       gtk_init(&argc, &argv);

       GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
       gtk_window_set_title(GTK_WINDOW(window), "GTK+ Demo");
       gtk_window_set_default_size(GTK_WINDOW(window), 300, 200);
       g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

       GtkWidget *vbox = gtk_box_new(GTK_ORIENTATION_VERTICAL, 5);
       gtk_container_add(GTK_CONTAINER(window), vbox);

       GtkWidget *label = gtk_label_new("Hello, GTK+!");
       gtk_box_pack_start(GTK_BOX(vbox), label, TRUE, TRUE, 0);

       GtkWidget *button = gtk_button_new_with_label("Click me");
       g_signal_connect(button, "clicked",
                        G_CALLBACK(on_button_clicked), label);
       gtk_box_pack_start(GTK_BOX(vbox), button, TRUE, TRUE, 0);

       gtk_widget_show_all(window);
       gtk_main();
       return 0;
   }
   ```
3. Compile and run:

   ```bash
   gcc -o solutions/gtk_app solutions/gtk_app.c $(pkg-config --cflags --libs gtk+-3.0)
   ./solutions/gtk_app
   ```
4. **Extension Feature:** Modify the program to add a **text entry** (`GtkEntry`) above the button, and on button click update the label to show the entry’s current text instead of the fixed string.

   * Hint: use `gtk_entry_get_text()`.

#### Reflection Questions

1. **How does GTK’s signal-and-callback mechanism differ from X11’s event loop?**
   
<summuray>
GTK+ (signal-and-callback):
    High-level abstraction built on top of X11
    Uses a signal system (e.g., "clicked", "destroy") that you connect to callback functions
    Internally handles events and dispatches them to the appropriate widget/function
    Code is cleaner and more modular 
 X11 (event loop):
    Low-level manual handling of events
    You write your own event loop and check event types directly:
XEvent e;
XNextEvent(display, &e);
if (e.type == Expose) { ... }
You must manually handle drawing, resizing, key presses, etc.
</summuray>


2. **Why use `pkg-config` when compiling GTK applications?**
   
<summuray>
pkg-config is a build helper tool that:
    Automatically provides the correct compiler flags for GTK (or any library)
    Includes:
        -I paths for header files
        -L paths for libraries
        -l flags to link the right .so files
Without it:
    You’d have to manually find and type a long list of flags
    It would break on different systems or GTK versions
Conclusion: pkg-config ensures portable, maintainable, error-free compilation of GTK apps.
</summuray>

---

**Remember:** Stop after **90 minutes** and record where you stopped.
