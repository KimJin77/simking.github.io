---
layout:     post
title:      "HTY360Player中GLProgram代码学习 & OpenGL ES绘制初步"
date:       2015-05-16
author:     "Sim"
catalog: false
tags:
    - OpenGL
---

# GLProgram

1. 初始化两个数组，其中`attributes`用于存储着色器的输入变量，`uniforms`用于存储着色器所使用的常量数据.

2. 初始化program对象: `GLuint program = glCreateProgram();`

3. 编译Shader

  1) Vertex Shader

  e.g

  ```
  // Vertex Shader的输入
  attribute vec4 position;
  attribute vec2 texcoord;

  // Vertex Shader的输出，将作为Fragment Shader的输入
  varying vec2 v_texcoord;

  // Vertex Shader使用的常量
  uniform mat4 modelViewProjectionMatrix;

  void main() {
    v_texcoord = texcoord;
    gl_Position = modelViewProjectionMatrix * position;
  }
  ```

  Vertex Shader说白了就是一段字符串，在iOS中可以创建为以.vsh结尾的文件，使用`NSBundle`读取该文件，当然也可以作为字符串进行读取。

  2) Fragment Shader

  e.g

  ```
  precision mediump float; // 表明精确度
  varying vec2 v_texcoord; // Vertex Shaderd的输出，在此作为输入
  uniform sampler2D s_texture;

  void main() {
    gl_FragColor = texture2D(s_texture, v_texcoord);
  }
  ```

  Fragment Shader的跟Vertex Shader类似，只不过作为文件时是以.fsh结尾。

  3）

  ```Objc
  - (BOOL)compileShader:(GLuint *)shader type:(GLenum)type string:(NSString *)shaderString {
    GLint status; // 用于标识编译成功与否
    const GLchar *source;

    source = (GLchar *)[shaderString UTF8String]; // 将着色器的代码进行转换
    if (!source) {
      if (type == GL_VERTEX_SHADER) NSLog(@"Failed to load vertex shader");
      else NSLog(@"Failed to load fragment shader");
      return NO;
    }

    *shader = glCreateShader(type); // 创建着色器，其中type的值为GL_VERTEX_SHADER或者是GL_FRAGMENT_SHADER
    glShaderSource(*shader, 1, &source, NULL); // 将vsh或者fsh中的操作代码加载到shader
    glCompileShader(*shader); // 编译shader

    glGetShaderiv(*shader, GL_COMPILE_STATUS, &status); // 检查编译状态

    if (status != GL_TRUE) { // 编译失败：编译失败的话会将相关信息写入info log中
      GLint logLength;
      glGetShaderiv(*shader, GL_INFO_LOG_LENGTH, &logLength);  // 获取info log的长度
      if (logLength > 0) {
        GLchar *log = (GLchar *)malloc(logLength);
        glGetShaderInfoLog(*shader, logLength, &logLength, log); // 写入
        NSLog(@"Shader compile log:\n%s", log);
        free(log);
      }
      glDeleteShader(*shader);
    }

    return status == GL_TRUE;
  }
  ```

4. 将shader对象添加到program

```ObjC
glAttachShader(program, vertexShader);
```

5. 连接program对象

```ObjC
- (BOOL)link {
  GLint status;
  glLinkProgram(program); // 连接program对象

  glGetProgramiv(program, GL_LINK_STATUS, &status); // 获取连接结果。连接出错的话，可以使用glGetProgramiv()方法来获取错误日志。用法与glGetShaderiv()类似
  glGetError();
  if (status == GL_FALSE)
    return NO;

  if (vertShader)
  {
    glDeleteShader(vertShader);
    vertShader = 0;
  }
  if (fragShader)
  {
    glDeleteShader(fragShader);
    fragShader = 0;
  }

  return YES;
}
```

6. 使用Program： `glUseProgram(program);`

# OpenGL ES初步使用

1. 初始化context对象： `EAGLContext *context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];`

2. 绘制的容器有两种选择，第一种是`EAGLLayer`+`UIViewController`，另一种是`GLKView`+`GLKViewController`。第一种的话允许我们在常见的`UIViewController`进行绘制，但是如果画面是不断进行变化的话，比方说使用OpenGL 绘制视频，那么就要自己写定时器方法来进行定时更新。通常的更新频率是1/60.而`GLKViewController`的话，因为实现了`update()`和`GLKViewDelegate`的delegate方法，所以就不需要通过定时器进行定时更新。

```ObjC
// 这里有两种使用方法
// 1. 直接使用GLKViewController的view作为绘制容器的话，只需要设置上下文以及绘制格式即可
 GLKView *view = (GLKView *)self.view;
 view.context = self.context;
 view.drawableDepthFormat = GLKViewDrawableDepthFormat24;

// 2. 如果是自己实现GLKView的话，需要指定上下文对象，delegate，并添加到view上面。
// 这种方法有个问题需要注意，自己实现GLKView的话，该view的display()方法只会被调用一次，如果画面需要一直更新的话，最好是在update()方法中手动调用display()进行画面的更新
self.leftView = [[GLKView alloc] initWithFrame:CGRectMake(0, 0, self.view.frame.size.width / 2, self.view.frame.size.height) context:self.context];
self.leftView.delegate = self;
self.leftView.drawableDepthFormat = GLKViewDrawableDepthFormat24;
[self.view addSubview:self.leftView];
```

3. 进行下一步操作之前，设置当前的上下文：`[EAGLContext setCurrentContext:self.context];`

4. 因为我绘制的是球体图片，所以使用下面的代码进行计算

```ObjC
GLfloat *vVertices = NULL; // 顶点数
GLfloat *vTextCoord = NULL; // 纹理坐标
GLushort *indices = NULL; // 顶点索引
int numVertices = 0;

_numIndices = esGenSphere(200, 1.0, &vVertices, NULL, &vTextCoord, &indices, &numVertices);

// 生成球体
// numSlices的指垂直分割的平面，跟经线差不多
int esGenSphere(int numSlices, float radius, float** vertices, float** normals, float **texCoords, uint16_t** indices, int* numVertices_out) {11
  int i, j;
  int numParallels = numSlices / 2;
  int numVertices = (numSlices + 1) * (numParallels + 1);
  int numIndices =  numParallels * numSlices * 6;
  float angleStep = (2.0 * M_PI) / ((float)numSlices);

  if (vertices != NULL) *vertices = malloc(sizeof(float) * 3 * numVertices);
  if (texCoords != NULL) *texCoords = malloc(sizeof(float) * 2 * numVertices);
  if (indices != NULL) *indices = malloc(sizeof(uint16_t) * numIndices);

  for (i = 0; i < numParallels+1; i++) {
    for (j = 0; j < numSlices+1; j++) {
      int vertex = (i * (numSlices + 1) + j) * 3;

      if (vertices) {
        (*vertices)[vertex + 0] = radius * sinf(angleStep * (float)i) * sinf(angleStep * (float)j);
        (*vertices)[vertex + 1] = radius * cosf(angleStep * (float)i);
        (*vertices)[vertex + 2] = radius * sinf(angleStep * (float)i) * cosf(angleStep * (float)j);
      }

      if (texCoords) {
        int texIndex = (i * (numSlices + 1) + j) * 2;
        (*texCoords)[texIndex + 0] = (float)j/(float)numSlices;
        (*texCoords)[texIndex + 1] = 1.0 - ((float)i / (float)(numParallels));
      }
    }
  }

  // Generate the indice
  if (indices != NULL) {
    uint16_t *indexBuf = (*indices);
    for (int i = 0; i < numParallels; i++) {
      for (int j = 0; j < numSlices; j++) {
        *indexBuf++ = i * (numSlices + 1) + j;
        *indexBuf++ = (i + 1) * (numSlices + 1) + j;
        *indexBuf++ = (i + 1) * (numSlices + 1) + (j + 1);

        *indexBuf++ = i * (numSlices + 1) + j;
        *indexBuf++ = (i + 1) * (numSlices + 1) + (j + 1);
        *indexBuf++ = i * (numSlices + 1) + (j + 1);
      }
    }
  }

  if (numVertices_out) *numVertices_out = numVertices;
  return numIndices;
}
```

5. Vertex Buffer和Texture Buffer

```ObjC
// Vertex
  glGenBuffers(1, &_vertexBuffer); // 生成顶点的缓存区。第一个参数表示返回缓存对象名称的数量
  glBindBuffer(GL_ARRAY_BUFFER, _vertexBuffer); // 将vertexBuffer变成array buffer对象
  // GL_STATIC_DRAW -- 缓存对象只会被指定一次，但是会被多次使用
  // GL_DYNAMIC_DRAW -- 会被指定多次，也能多次使用
  // GL_STREAM_DRAW -- 指定一次，但是使用次数较少
  glBufferData(GL_ARRAY_BUFFER, numVertices*3*sizeof(GLfloat), vVertices, GL_STATIC_DRAW); // 创建并初始化数据
  glEnableVertexAttribArray(GLKVertexAttribPosition);
  glVertexAttribPointer(GLKVertexAttribPosition, 3, GL_FLOAT, GL_FALSE, sizeof(GLfloat)*3, NULL);

  // Texture
  glGenBuffers(1, &_textureBuffer);
  glBindBuffer(GL_ARRAY_BUFFER, _textureBuffer);
  glBufferData(GL_ARRAY_BUFFER, numVertices*2*sizeof(GLfloat), vTextCoord, GL_DYNAMIC_DRAW);
  glEnableVertexAttribArray([_glProgram attributeIndex:@"texcoord"]);
  glVertexAttribPointer([_glProgram attributeIndex:@"texcoord"], 2, GL_FLOAT, GL_FALSE, sizeof(GLfloat)*2, NULL);

  // Indice
  glGenBuffers(1, &_indicesBuffer);

  glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, _indicesBuffer);
  glBufferData(GL_ELEMENT_ARRAY_BUFFER, _numIndices*sizeof(GLushort), indices, GL_STATIC_DRAW);

  [_glProgram use];
  glUniform1i([_glProgram uniformIndex:@"s_texture"], 0); // 加载uniform
```

6. 加载纹理图片(将静态图片变成纹理)

```ObjC
- (void)loadTexture {
  CGImageRef spriteImage = [UIImage imageNamed:@"minack-panorama.jpg"].CGImage; // 加载图片

  // 图片宽高
  size_t width = CGImageGetWidth(spriteImage);
  size_t height = CGImageGetHeight(spriteImage);

  // 分配存储，每个像素是rgba4位，所以乘以4
  GLubyte *spriteData = (GLubyte *)calloc(width*height*4, sizeof(GLubyte));
  // 第四个参数表明每个像素所占的内存为8bit
  // 第五个参数表示每一行的内存的byte
  CGContextRef spriteContext = CGBitmapContextCreate(spriteData, width, height, 8, width*4,
                                                   CGImageGetColorSpace(spriteImage), kCGImageAlphaPremultipliedLast); // 创建新的位图
  CGContextDrawImage(spriteContext, CGRectMake(0, 0, width, height), spriteImage); // 绘制

  CGContextRelease(spriteContext); // 释放上下文
  glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
  GLuint texName;
  glGenTextures(1, &texName);
  glBindTexture(GL_TEXTURE_2D, texName);

  glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, spriteData); // 加载纹理
  // 设置过滤方式
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);

    glActiveTexture(GL_TEXTURE0); // 激活纹理单元
}

/*
YUV转纹理

const NSUInteger frameWidth = frame.width;
    const NSUInteger frameHeight = frame.height;    

    glPixelStorei(GL_UNPACK_ALIGNMENT, 1);

    if (0 == _textures[0])
        glGenTextures(3, _textures);

    const UInt8 *pixels[3] = { yuvFrame.luma.bytes, yuvFrame.chromaB.bytes, yuvFrame.chromaR.bytes };
    const NSUInteger widths[3]  = { frameWidth, frameWidth / 2, frameWidth / 2 };
    const NSUInteger heights[3] = { frameHeight, frameHeight / 2, frameHeight / 2 };

    for (int i = 0; i < 3; ++i) {

        glBindTexture(GL_TEXTURE_2D, _textures[i]);

        glTexImage2D(GL_TEXTURE_2D,
                     0,
                     GL_LUMINANCE,
                     widths[i],
                     heights[i],
                     0,
                     GL_LUMINANCE,
                     GL_UNSIGNED_BYTE,
                     pixels[i]);

        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
        glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
        glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    }
*/

/*
RGB转纹理

glPixelStorei(GL_UNPACK_ALIGNMENT, 1);

    if (0 == _texture)
        glGenTextures(1, &_texture);

    glBindTexture(GL_TEXTURE_2D, _texture);

    glTexImage2D(GL_TEXTURE_2D,
                 0,
                 GL_RGB,
                 frame.width,
                 frame.height,
                 0,
                 GL_RGB,
                 GL_UNSIGNED_BYTE,
                 rgbFrame.rgb.bytes);

    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
*/
```

7. 设置映射矩阵

```ObjC
float aspect = fabs(self.view.bounds.size.width / self.view.bounds.size.height);
  GLKMatrix4 projectionMatrix = GLKMatrix4MakePerspective(GLKMathDegreesToRadians(_overture), aspect, 0.1, 1000.f);
  projectionMatrix = GLKMatrix4Rotate(projectionMatrix, 3.14159265f, 1.0f, 0.0f, 0.0f);

  GLKMatrix4 modelViewMatrix = GLKMatrix4Identity;
  modelViewMatrix = GLKMatrix4Scale(modelViewMatrix, 100.0f, 100.0f, 100.0f);

  _modelViewProjectionMatrix = GLKMatrix4Multiply(projectionMatrix, modelViewMatrix);
```

8. 绘制

```ObjC
- (void)glkView:(GLKView *)view drawInRect:(CGRect)rect {

//  glViewport(0, 0, 100, 100);
  
  glUniformMatrix4fv(uniforms[UNIFORM_MODELVIEWPROJECTION_MATRIX], 1, 0, _modelViewProjectionMatrix.m);

  glClearColor(0.2f, 0.5f, 0.3f, 1.0f);
  glClear(GL_COLOR_BUFFER_BIT);
  glDrawElements(GL_TRIANGLES, _numIndices, GL_UNSIGNED_SHORT, 0);
}
```
