---
title: OpenGL ES FBO
tags:
---


### FBO开发流程
#### 第一步、创建FBO

##### 创建纹理、开辟GPU显存空间

```kotlin
        val fboTexId = IntArray(1)

        // 纹理：作为 color attachment
        glGenTextures(1, fboTexId, 0)
        glBindTexture(GL_TEXTURE_2D, fboTexId[0])
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE)
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE)

        // 分配纹理存储（关键：null 数据，只要开辟空间）
        glTexImage2D(
            GL_TEXTURE_2D,
            0,
            GL_RGBA,
            fboWidth,
            fboHeight,
            0,
            GL_RGBA,
            GL_UNSIGNED_BYTE,
            null
        )


```

##### 创建FBO并激活

```kotlin
    val fboId = IntArray(1)
    // FBO
    glGenFramebuffers(1, fboId, 0)
    glBindFramebuffer(GL_FRAMEBUFFER, fboId[0])

```

##### 把纹理挂到0号颜色附着点上

```kotlin

   glFramebufferTexture2D(
            GL_FRAMEBUFFER,
            GL_COLOR_ATTACHMENT0,
            GL_TEXTURE_2D,
            fboTexId[0],
            0
        )
```

##### 检查状态
驱动会检查这套组合能不能作为一个合法的渲染目标
```kotlin
    val status = glCheckFramebufferStatus(GL_FRAMEBUFFER)
        if (status != GL_FRAMEBUFFER_COMPLETE) {
            throw RuntimeException("FBO incomplete, status = 0x${Integer.toHexString(status)}")
        }
```

##### 恢复状态
把绑定恢复到默认 framebuffer/解绑纹理，避免后续代码误操作到这个 FBO/纹理。
```kotlin
    glBindFramebuffer(GL_FRAMEBUFFER, 0)
    glBindTexture(GL_TEXTURE_2D, 0)
```

#### 第二步、开始绘制
##### 渲染到FBO

```kotlin
        // ---------- Pass 1: 渲染到 FBO ----------
        glBindFramebuffer(GL_FRAMEBUFFER, fboId[0])
        glViewport(0, 0, fboW, fboH)
        glClear(GL_COLOR_BUFFER_BIT)

        glUseProgram(triProgram)
        glEnableVertexAttribArray(triPosLoc)
        glVertexAttribPointer(triPosLoc, 2, GL_FLOAT, false, 2 * 4, triVertices)
        glDrawArrays(GL_TRIANGLES, 0, 3)
        glDisableVertexAttribArray(triPosLoc)

        glBindFramebuffer(GL_FRAMEBUFFER, 0) // 回到默认帧缓冲
```


##### 把FBO的纹理采样到屏幕
```kotlin
        // ---------- Pass 2: 把 FBO 纹理画到屏幕 ----------
        glViewport(0, 0, viewW, viewH)
        glClear(GL_COLOR_BUFFER_BIT)

        glUseProgram(quadProgram)

        glActiveTexture(GL_TEXTURE0)//激活0号纹理单元
        glBindTexture(GL_TEXTURE_2D, fboTexId[0])//把fboTexId[0]绑定到0号纹理单元
        glUniform1i(quadSamplerLoc, 0)

        glEnableVertexAttribArray(quadPosLoc)
        glVertexAttribPointer(quadPosLoc, 2, GL_FLOAT, false, 2 * 4, quadPos)

        glEnableVertexAttribArray(quadUvLoc)
        glVertexAttribPointer(quadUvLoc, 2, GL_FLOAT, false, 2 * 4, quadUv)

        glDrawArrays(GL_TRIANGLE_STRIP, 0, 4)

        glDisableVertexAttribArray(quadPosLoc)
        glDisableVertexAttribArray(quadUvLoc)
        glBindTexture(GL_TEXTURE_2D, 0)
```

