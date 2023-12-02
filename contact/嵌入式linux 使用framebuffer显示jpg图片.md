# 嵌入式linux 使用LCD屏幕显示jpg图片

使用设备为100ASK_IMX6ULL_PRO开发板，和它自带的1024x600 32bpp的LCD屏幕，先上效果和代码

![](/home/lxj/CLionProjects/nobody_notebook/images/2023-12-02-21-24-59-IMG_20231202_212408.jpg)

```cpp
#include <errno.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <unistd.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <asm/types.h>
#include <linux/videodev2.h>
#include <sys/mman.h>
#include <string.h>
#include <malloc.h>
#include <linux/fb.h>

#include <jpeglib.h>

#include <jerror.h>

struct fb_dev
{
        //for frame buffer
        int fb;
        void *fb_mem;   //frame buffer mmap
        int fb_width, fb_height, fb_line_len, fb_size;
        int fb_bpp;
} fbdev;

//得到framebuffer的长、宽和位宽，成功则返回0，失败返回－1 
int fb_stat(int fd)
{
        struct fb_fix_screeninfo fb_finfo;
        struct fb_var_screeninfo fb_vinfo;

        if (ioctl(fd, FBIOGET_FSCREENINFO, &fb_finfo))
        {
                perror(__func__);
                return (-1);
        }

        if (ioctl(fd, FBIOGET_VSCREENINFO, &fb_vinfo))
        {
                perror(__func__);
                return (-1);
        }

        fbdev.fb_width = fb_vinfo.xres;
        fbdev.fb_height = fb_vinfo.yres;
        fbdev.fb_bpp = fb_vinfo.bits_per_pixel;
        fbdev.fb_line_len = fb_finfo.line_length;
        fbdev.fb_size = fb_finfo.smem_len;

        return (0);
}

//释放framebuffer的映射
int fb_munmap(void *start, size_t length)
{
        return (munmap(start, length));
}

//显示一个像素点的图像到framebuffer上
int fb_pixel(void *fbmem, int width, int height, int x, int y, unsigned int color)
{
        if ((x > width) || (y > height))
                return (-1);

        unsigned int *dst = ((unsigned int *) fbmem + y * width + x);

        *dst = color;
        return 0;
}

int main(int argc, char **argv)
{
     int fb;
     FILE *infile;
     struct jpeg_decompress_struct cinfo;
     long long x,y;
     unsigned char *buffer;
     char s[15];
     struct jpeg_error_mgr jerr;

      if ((fb = open("/dev/fb0", O_RDWR)) < 0)                        //打开显卡设备
      {
                perror(__func__);
                return (-1);
      }

        //获取framebuffer的状态
        fb_stat(fb);                                                    //获取显卡驱动中的长、宽和显示位宽
        printf("frame buffer: %dx%d,  %dbpp, 0x%xbyte= %d\n",
                fbdev.fb_width, fbdev.fb_height, fbdev.fb_bpp, fbdev.fb_size, fbdev.fb_size);

        //映射framebuffer的地址
        fbdev.fb_mem = mmap (NULL, fbdev.fb_size, PROT_READ|PROT_WRITE,MAP_SHARED,fb,0);
memset(fbdev.fb_mem,0xff,fbdev.fb_size);
        if ((infile = fopen("lcd.jpg", "rb")) == NULL)
        {
                fprintf(stderr, "open %s failed\n", s);
                exit(-1);
        }
        ioctl(fb, FBIOBLANK,0);                        //打开LCD背光

        cinfo.err = jpeg_std_error(&jerr);
        jpeg_create_decompress(&cinfo);

        //导入要解压的Jpeg文件infile
        jpeg_stdio_src(&cinfo, infile);
        cinfo.scale_num = 1;
        cinfo.scale_denom = 2;
        //读取jpeg文件的文件头
        jpeg_read_header(&cinfo, TRUE);

        //开始解压Jpeg文件，解压后将分配给scanline缓冲区，
        jpeg_start_decompress(&cinfo);

        buffer = (unsigned char *) malloc(cinfo.output_width
                                        * cinfo.output_components);
        y = 0;
        while (cinfo.output_scanline < cinfo.output_height)
        {
                jpeg_read_scanlines(&cinfo, &buffer, 1);

                unsigned int color;
                       for (x = 0; x < cinfo.output_width; x++)
                       {
                                color = buffer[x * 3 ]<<16|buffer[x * 3 + 1]<<8|buffer[x * 3 +2];
                    //color = R | G | B，图片为RGB888存储的，屏幕是32bpp直接移位即可
                    //因为这里color需要输入32位的数据，0～7bit是Blue,8～15bit是Green,16～23bit是Red，剩下的式透明度不管它
                    //但是libjpegAPI读出来每一个的buffer里面的数据是8bit，所以要把RGB移位赋值放到32bit合适的地方
                    //红就放在16～23bit位置以此类推
                                fb_pixel(fbdev.fb_mem, fbdev.fb_width, fbdev.fb_height, x, y, color);
                       }


                y++;

        }
        printf("show wife\n");
        //完成Jpeg解码，释放Jpeg文件
        jpeg_finish_decompress(&cinfo);
        jpeg_destroy_decompress(&cinfo);

        //释放帧缓冲区
        free(buffer);

        //关闭Jpeg输入文件
        fclose(infile);

       fb_munmap(fbdev.fb_mem, fbdev.fb_size);                                 //释放framebuffer映射

       close(fb); 
}
```

## 解释重点代码

```cpp
jpeg_read_scanlines(&cinfo, &buffer, 1); 
                unsigned int color;
                   for (x = 0; x < cinfo.output_width; x++)
                   {
                            color = buffer[x * 3 ]<<16|buffer[x * 3 + 1]<<8|buffer[x * 3 +2];
                            fb_pixel(fbdev.fb_mem, fbdev.fb_width, fbdev.fb_height, x, y, color);
                   }
```

因为老婆图片为jpg存储的，每一个像素有24bit,而屏幕是32bpp直接移位赋值到color即可
 因为这里color需要输入32位的数据，0～7bit是Blue,8～15bit是Green,16～23bit是Red，剩下的是透明度不用管它
 但是libjpegAPI读出来每一个的buffer里面的数据是8bit，所以要把RGB移位赋值放到32bit合适的地方，于是红就放在16～23bit位置以此类推

然后一行的从第一个像素点是在buffer以0红 1绿 2蓝去存储的，第二个像素点是3红 4绿 5蓝依次类推去存储的，将它拼成32bit的color值按行从左到右显示这行的每一个多色像素点（color），显示完一行再下一行，一行一行地把整个屏幕显示完。

```cpp
//显示一个像素点的图像到framebuffer上
int fb_pixel(void *fbmem, int width, int height, int x, int y, unsigned int color)
{
        if ((x > width) || (y > height))
                return (-1);

        unsigned int *dst = ((unsigned int *) fbmem + y * width + x);

        *dst = color;
        return 0;
}
```

显示color就是从屏幕的映射地址中计算得到每一个像素点的坐标然后给它赋值color就行了

交叉编译好程序然后把老婆图片命名为lcd.jpg和编译出来的arm可执行程序放在一个文件夹里面在板子上跑程序就能看到全彩的wife在LCD上了！！！
