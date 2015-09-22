---
layout: post
title: A simple way to set up GLKView without the Xcode template
date: '2013-01-13T21:34:00+02:00'
tags:
- opengl
- glkit
- cocoa
- xcode
- glkview
tumblr_url: http://somerandomtechblog.tumblr.com/post/40459534493/set-up-opengl-without-xcode
---
The motivation

Xcode’s OpenGL Game template is not quite the cleanest way to start your OpenGL project. It adds a lot of code you don’t initially need and by the time you will need it, you’ll probably structure it in a different manner, which means you’ll have to clean it form the template.

The good news is that, to set up the most basic GLKView to handle your drawing routines you only need 3 lines of code.

The example project for this article can be found here



The solution

So, here are the three lines of code that get you started.

//these are the only three lines you
//need to write in order to init your GLKView
//while there are several options you could set here too,
//you can actually skip them and go straight ahead to
//SGLView (which is a subclass of GLKView by the way) drawRect: method
//and start drawing in it

//we create the context using the latest API
EAGLContext * context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
[EAGLContext setCurrentContext:context];
//we init our GLKView subclass with our newly created context
SGLView * view = [[SGLView alloc] initWithFrame:self.window.frame context:context];


After this, you should be able to override your view’s drawRect: method and draw. Let’s try something that doesn’t need preliminary setup.

- (void)drawRect:(CGRect)rect
{
    //clear the screen
    glClearColor(0.0f, 1.0f, 1.0f, 1.0f);
 }


That’s it, your GLKView is set up, from here the additional settings depend on how you plan to use OpenGL (2D, 3D, on screen, off screen, etc.).

A little extra

To make the matter complete, let’s try to draw a simple 2D square. So, in our GLKView subclass we will use GLKBaseEffect to handle the projection and the drawing colour. For more details, refer to the comments below.

- (id)initWithFrame:(CGRect)frame context:(EAGLContext *)context
{
    self = [super initWithFrame:frame context:context];
    if (self) {


        //the effect makes our life easier
        //we need not worry about the colour array nor the projection
        effect = [[GLKBaseEffect alloc] init];
        effect.useConstantColor = GL_TRUE;

        //nor for the projection
        //which will be orthogonal for the sake of simplicity
        //the parameters are for 4-inch retina, 3.5 retina would have been
        //-1.0f, 1.0f, -1.5f, 1.5f, 1.0f, -1.0f 
        GLKMatrix4 projectionMatrix = 
        GLKMatrix4MakeOrtho(-1.0f, 1.0f, -1.7f, 1.7f, 1.0f, -1.0f);

        effect.transform.projectionMatrix = projectionMatrix;
        //set a color for our drawing
        effect.constantColor = 
        GLKVector4Make(190.0f/255.0f, 182.0f/255.0f, 171.0f/255.0f, 1.0f);




        //generate and bind each of the vertex and index buffer
        //upload data to them
        glGenVertexArraysOES(1, &vertexArray);
        glBindVertexArrayOES(vertexArray);

        glGenBuffers(1, &vertexBuffer);
        glBindBuffer(GL_ARRAY_BUFFER, vertexBuffer);
        glBufferData(GL_ARRAY_BUFFER,
                 kVertexNum*sizeof(GLKVector3),
                 &vertexData,
                 GL_STATIC_DRAW);

        glGenBuffers(1, &indexBuffer);
        glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, indexBuffer);
        glBufferData(GL_ELEMENT_ARRAY_BUFFER,
                 kIndexNum*sizeof(GLbyte),
                 &indexData,
                 GL_STATIC_DRAW);

        glBindBuffer(GL_ARRAY_BUFFER,0);
        glBindVertexArrayOES(0);

    }
    return self;
}


And then the drawing code.

- (void)drawRect:(CGRect)rect
{

    //clear the screen
    glClearColor(1.0f, 1.0f, 1.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);


    glBindVertexArrayOES(vertexArray);

    //draw
    glBindBuffer(GL_ARRAY_BUFFER, vertexBuffer);
    glEnableVertexAttribArray(GLKVertexAttribPosition);
    glVertexAttribPointer(GLKVertexAttribPosition,
                      3,
                      GL_FLOAT,
                      GL_FALSE,
                      sizeof(GLKVector3),
                      (void*)0);
    [effect prepareToDraw];

    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, indexBuffer);
    glDrawElements(GL_TRIANGLE_STRIP, kIndexNum, GL_UNSIGNED_BYTE, (void*)0);


    glBindBuffer(GL_ARRAY_BUFFER,0);
    glBindVertexArrayOES(0);
}


And finally, we have two static const buffers that provide the necessarily drawing data.

static const GLbyte kVertexNum = 4;
static const GLbyte kIndexNum = 6;


//the vertex data is added from the left bottom corner
//towards right, up and finally left
//as the ascii art suggests
static const GLKVector3 vertexData[kVertexNum] =
{
    {-0.5f, -0.5f, 0.0f},//0  3-----2
    {0.5f, -0.5f, 0.0f},//1   |     |
    {0.5f, 0.5f, 0.0f},//2    |     |
    {-0.5f, 0.5f, 0.0f},//3   0-----1
};


//we basically draw 2 triangles
//as described in the ascii art
static const GLbyte indexData[kIndexNum] =
{
    0, 3, 1,
    3, 2, 1

    /*

    3--2
    |\ |
    | \|
    0--1

    */
};
