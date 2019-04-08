---
title: 'C#去除PNG图片四周多余空白'
date: 2019-04-08 20:13:25
tags:
categories:
- C#基础
---

最近在做关于签字板功能模块，由于签字后生成图片存在多余空白区域，
导致最后在生成的报表中签名大小展示不一致，所以需要将空白区域删除，以此来保证美观。
参考代码地址：
https://blog.csdn.net/rovecat/article/details/30230841

在使用过程中做了一些小修改，此外发现PNG图片在最后生成后存在黑色背景，而签名也只显示轮廓，
所以经过多次尝试后，通过设置新生成的图片的透明度来解决。

```
using System;
using System.Drawing;
using System.Drawing.Imaging;

namespace Learun.Util
{
    public class PictureHelper
    {
        // <summary>  
        /// 剪去图片空余白边  
        /// </summary>  
        /// <param name="bmp">源文件</param>  
        /// <param name="whiteBarRate">保留空白边比例</param>  
        /// <param name="filePath">文件保存地址</param>  
        /// <param name="saveType">保存类型</param>  
        public static void CutImageWhitePart(Bitmap bmp, int whiteBarRate, string filePath, ImageFormat saveType)
        {
            int top = 0, left = 0;
            int right = bmp.Width, bottom = bmp.Height;
            Color white = Color.White;
            //寻找最上面的标线,从左(0)到右，从上(0)到下  
            for (int i = 0; i < bmp.Height; i++)//行  
            {
                bool find = false;
                for (int j = 0; j < bmp.Width; j++)//列  
                {
                    Color c = bmp.GetPixel(j, i);
                    if (IsWhite(c))
                    {
                        top = i;
                        find = true;
                        break;
                    }
                }
                if (find) break;
            }
            //寻找最左边的标线，从上（top位）到下，从左到右  
            for (int i = 0; i < bmp.Width; i++)//列  
            {
                bool find = false;
                for (int j = top; j < bmp.Height; j++)//行  
                {
                    Color c = bmp.GetPixel(i, j);
                    if (IsWhite(c))
                    {
                        left = i;
                        find = true;
                        break;
                    }
                }
                if (find) break; ;
            }
            ////寻找最下边标线，从下到上，从左到右  
            for (int i = bmp.Height - 1; i >= 0; i--)//行  
            {
                bool find = false;
                for (int j = left; j < bmp.Width; j++)//列  
                {
                    Color c = bmp.GetPixel(j, i);
                    if (IsWhite(c))
                    {
                        bottom = i;
                        find = true;
                        break;
                    }
                }
                if (find) break;
            }
            //寻找最右边的标线，从上到下，从右往左  
            for (int i = bmp.Width - 1; i >= 0; i--)//列  
            {
                bool find = false;
                for (int j = 0; j <= bottom; j++)//行  
                {
                    Color c = bmp.GetPixel(i, j);
                    if (IsWhite(c))
                    {
                        right = i;
                        find = true;
                        break;
                    }
                }
                if (find) break;
            }
            int iWidth = right - left;
            int iHeight = bottom - left;
            int blockWidth = Convert.ToInt32(iWidth * whiteBarRate / 100);
            bmp = Cut(bmp, left - blockWidth, top - blockWidth, right - left + 2 * blockWidth, bottom - top + 2 * blockWidth);


            if (bmp != null)
            {
                bmp.Save(filePath, saveType);
            }
        }

        /// <summary>  
        /// 来源于网络的一个函数  
        /// </summary>  
        /// <param name="b"></param>  
        /// <param name="StartX"></param>  
        /// <param name="StartY"></param>  
        /// <param name="iWidth"></param>  
        /// <param name="iHeight"></param>  
        /// <returns></returns>  
        private static Bitmap Cut(Bitmap b, int StartX, int StartY, int iWidth, int iHeight)
        {
            if (b == null)
            {
                return null;
            }
            int w = b.Width;
            int h = b.Height;
            if (StartX >= w || StartY >= h)
            {
                return null;
            }
            if (StartX + iWidth > w)
            {
                iWidth = w - StartX;
            }
            if (StartY + iHeight > h)
            {
                iHeight = h - StartY;
            }
            try
            {
                Bitmap bmpOut = new Bitmap(iWidth, iHeight, PixelFormat.Format24bppRgb);
                bmpOut.MakeTransparent();//注意：PNG此处要设置背景透明，否则最后生成图片底色全部变黑
                Graphics g = Graphics.FromImage(bmpOut);
                g.DrawImage(b, new Rectangle(0, 0, iWidth, iHeight), new Rectangle(StartX, StartY, iWidth, iHeight), GraphicsUnit.Pixel);
                g.Dispose();
                return bmpOut;
            }
            catch
            {
                return null;
            }
        }

        /// <summary>  
        /// 判断白色与否，非纯白色  
        /// </summary>  
        /// <param name="c"></param>  
        /// <returns></returns>  
        private static bool IsWhite(Color c)
        {
            if (c.R < 255 || c.G < 255 || c.B < 255)
                return true;
            else return false;
        }

        public static Bitmap Trim(Bitmap bitmap)
        {
            Bitmap resultBmp = null;

            BitmapData bData = bitmap.LockBits(new Rectangle(0, 0, bitmap.Width, bitmap.Height), ImageLockMode.ReadOnly, PixelFormat.Format24bppRgb);

            unsafe
            {
                int width = bitmap.Width;
                int height = bitmap.Height;

                int newx = 0, newy = 0, newHeight = 0, newWidth = 0;

                bool isbreak = false;

                //得到x坐标
                for (int x = 0; x < width; x++)
                {
                    for (int y = 0; y < height; y++)
                    {
                        byte* color = (byte*)bData.Scan0 + x * 3 + y * bData.Stride;
                        int R = *(color + 2);
                        int G = *(color + 1);
                        int B = *color;

                        if (R != 255 || G != 255 || B != 255)
                        {
                            newx = x;
                            isbreak = true;
                            break;
                        }
                    }

                    if (isbreak)
                    {
                        break;
                    }
                }

                isbreak = false;

                //得到y坐标
                for (int y = 0; y < height; y++)
                {
                    for (int x = 0; x < width; x++)
                    {
                        byte* color = (byte*)bData.Scan0 + x * 3 + y * bData.Stride;
                        int R = *(color + 2);
                        int G = *(color + 1);
                        int B = *color;

                        if (R != 255 || G != 255 || B != 255)
                        {
                            newy = y;
                            isbreak = true;
                            break;
                        }
                    }

                    if (isbreak)
                    {
                        break;
                    }
                }

                isbreak = false;

                int tmpy = 0;

                //得到height
                for (int y = height - 1; y >= 0; y--)
                {
                    for (int x = 0; x < width; x++)
                    {
                        byte* color = (byte*)bData.Scan0 + x * 3 + y * bData.Stride;
                        int R = *(color + 2);
                        int G = *(color + 1);
                        int B = *color;

                        if (R != 255 || G != 255 || B != 255)
                        {
                            tmpy = y;
                            isbreak = true;
                            break;
                        }
                    }

                    if (isbreak)
                    {
                        break;
                    }
                }

                isbreak = false;

                newHeight = tmpy - newy + 1;

                int tmpx = 0;

                //得到width
                //得到x坐标
                for (int x = width - 1; x >= 0; x--)
                {
                    for (int y = 0; y < height; y++)
                    {
                        byte* color = (byte*)bData.Scan0 + x * 3 + y * bData.Stride;
                        int R = *(color + 2);
                        int G = *(color + 1);
                        int B = *color;

                        if (R != 255 || G != 255 || B != 255)
                        {
                            Color newColor = Color.FromArgb(R, G, B);
                            tmpx = x;
                            isbreak = true;
                            break;
                        }
                    }

                    if (isbreak)
                    {
                        break;
                    }
                }

                newWidth = tmpx - newx + 1;

                Rectangle rect = new Rectangle(newx, newy, newWidth, newHeight);

                resultBmp = new Bitmap(newWidth, newHeight);

                resultBmp = bitmap.Clone(rect, bitmap.PixelFormat);
            }

            bitmap.UnlockBits(bData);

            return resultBmp;
        }
    }
}


```