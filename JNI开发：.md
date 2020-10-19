JNI开发：

~~~
jni 错误抓取
/Users/cailei/Library/Android/sdk/ndk/21.0.6113669/ndk-stack -sym /Users/cailei/AndroidStudioProjects/Ndk01/app/build/intermediates/cmake/debug/obj/armeabi-v7a -dump  
~~~

~~~java
滤镜效果实现，操作java层的bitmap 或者jni层的mat对象
Java_com_darren_ndk_day57_BitmapUtils_gary(JNIEnv *env, jclass type, jobject bitmap) {
    AndroidBitmapInfo bitmapInfo;
    int info_res = AndroidBitmap_getInfo(env, bitmap, &bitmapInfo);

    if (info_res != 0) {
        return -1;
    }
    // void 指针？并不知道具体的类型
    void *pixels;
    AndroidBitmap_lockPixels(env, bitmap, &pixels);

    // 有没有问题？写法
    if (bitmapInfo.format == ANDROID_BITMAP_FORMAT_RGBA_8888) {
        for (int i = 0; i < bitmapInfo.width * bitmapInfo.height; ++i) {
            // 怎么操作？
            uint32_t *pixel_p = reinterpret_cast<uint32_t *>(pixels) + i;
            uint32_t pixel = *pixel_p;

            int a = (pixel >> 24) & 0xff;
            int r = (pixel >> 16) & 0xff;
            int g = (pixel >> 8) & 0xff;
            int b = pixel & 0xff;
            // f = 0.213f * r + 0.715f * g + 0.072f * b
            int gery = (int) (0.213f * r + 0.715f * g + 0.072f * b);

            *pixel_p = (a << 24) | (gery << 16) | (gery << 8) | gery;
        }
    } else if (bitmapInfo.format == ANDROID_BITMAP_FORMAT_RGB_565) {
        for (int i = 0; i < bitmapInfo.width * bitmapInfo.height; ++i) {
            // 怎么操作？
            uint16_t *pixel_p = reinterpret_cast<uint16_t *>(pixels) + i;
            uint16_t pixel = *pixel_p;
            // 8888 -> 565
            int r = ((pixel >> 11) & 0x1f) << 3; // 5
            int g = ((pixel >> 5) & 0x3f) << 2; // 6
            int b = (pixel & 0x1f) << 3; // 5
            // f = 0.213f * r + 0.715f * g + 0.072f * b  (32位来讲的  255,255,255)
            int gery = (int) (0.213f * r + 0.715f * g + 0.072f * b); // 8位

            *pixel_p = ((gery >> 3) << 11) | ((gery >> 2) << 5) | (gery >> 3);
        }
    }

    AndroidBitmap_unlockPixels(env, bitmap);
    return 1;
}
~~~

~~~
图像的掩膜操作：让图像轮廓变得更清晰
图像的均值模糊：
图像的高斯模糊：
中值滤波：去除一些点
双边滤波+掩膜操作

~~~

~~~java
gcc 编译四个步骤：
1.预处理：宏定义展开，宏定义替换，展开include 文件 gcc -E -o hello.i hello.c
2.预编译：gcc 检测代码是否有错误 ，将代码转为汇编  gcc -S -o hello.s hello.i
3.汇编：将.s 文件翻译成 机器语言 得到.o 文件 gcc -c -o hello.o hello.s
4.链接：计算逻辑地址，合并数据段，有些函数在另外一个so中
~~~

~~~
配置ndk
~~~

