---
title: PDF操作
date: 2018-06-11 14:36:07
tags:
categories:
- 插件
+description: "iText"
---

PDF操作去除页脚部位的水印


<!--more-->
## 问题描述

使用某报表插件时，由于免费版导出PDF会出现水印字体，因此考虑将该水印去除后，输出到客户端浏览器。
{% asset_img 20180611152915.jpg 页脚位置出现水印 %}


考虑的解决方案包括：
1、 报表插件导出PDF ->编辑PDF水印->PDF输出到客户端。
2、 报表插件导出图片->编程处理图片水印->将多张图片生成PDF->PDF输出到客户端。
3、 寻找破解版的报表插件直接导出PDF。
4、 自己通过代码直接生成Word，不使用第三方报表插件。

OK上面就是能够想到的解决方法。

第4种方案应该可以实现，但相对来说工作量大一些，放到最后尝试，所以没有使用。
第3种方案简单粗暴，但在实现过程中发现第三方报表插件的版本5.6虽然有破解版，但由于版本过低，导致页面只能在
低版本的IE中才能查看，明显无法用于实际环境。
第2种方案虽然比直接操作PDF复杂一些，但是肯定能实现，在实现的过程中发现，报表插件导出的图片IMG中会存在分页内容，所以暂时放弃。（后期发现可以导出多张图片）
``` c
        String visitGUID=Guid.NewGuid().ToString();
        IGRExportOption ExportOption = Report.PrepareExport(GRExportType.gretIMG);
        IGRE2IMGOption E2IMGOption = ExportOption.AsE2IMGOption;
        E2IMGOption.ImageType = GRExportImageType.greitPNG;
        E2IMGOption.AllInOne = false; //所有页产生在一个图像文件中
        E2IMGOption.FileName = ReportPath + "\\SuperviseExamine\\Result\\" + visitGUID + "\\" + visitGUID + ".png";
        Report.Export();

```
最后留下第1种方案并开始进行尝试实现。

## 收集网上信息

对PDF操作肯定找第三方控件了，iText上场。
### PDF操作插件 iText
iText is a PDF library that allows you to CREATE, ADAPT, INSPECT and MAINTAIN documents in the Portable Document Format (PDF), allowing you to add PDF functionality to your software projects with ease.  We even have documentation to help you get coding.

We have two currently supported versions: iText 5 and iText 7. Both are available under AGPL and Commercial license.
* iText 5 AGPL
* iText 7 community: https://www.nuget.org/packages/itext7/

iText 5 is a one solution library that is complex, but well documented to help you create your solutions.

iText 7 is a complete re-write of iText 5, allowing you to choose your adventure with add-ons, all based on a simple, modular code structure that is easy to use and well documented.

本次使用iText5插件，版本号5.5.13

## 代码

### 最终实现代码

方便找到这篇文章的朋友，直接将实现的代码放出
``` c
 public class ServerUtility
    {
        //直接使用流对象进行数据处理
        public static void ResponseBinaryPDF(HttpContextBase context, IGRBinaryObject ExportResult, string FileName, string ContentType, string OpenMode)
        {
            //通过已有PDF来修改水印
            //using (FileStream fs = new FileStream(outputFilePath, FileMode.Open))
            //{
            //    byte[] bytes = new byte[(int)fs.Length];
            //    fs.Read(bytes, 0, bytes.Length);
            //    fs.Close();
            //}

            byte[] inputPdfStreamData = (byte[])(ExportResult.SaveToVariant());//PDF字节流

            using (MemoryStream outputPdfStream = new MemoryStream())
            {
                //使用PDF字节数组初始化PDFReader
                PdfReader reader = new PdfReader(inputPdfStreamData);
                //按照原PDF的内容创建PdfStamper对象，准备放一张白色图片去掉水印
                using (PdfStamper stamper = new PdfStamper(reader, outputPdfStream))
                {
                    int pageCount = reader.NumberOfPages;
                    //多页循环
                    for (int i = 1; i <= pageCount; i++)
                    {
                        //创建一个用来覆盖水印的白色背景图片，大小要比水印稍大
                        iTextSharp.text.Image image = iTextSharp.text.Image.GetInstance(new Bitmap(400, 30), BaseColor.WHITE);
                        //把图片放到水印的位置（IText使用的单位是pt而不是px，（0,0）坐标是PDF左下角！！）
                        image.SetAbsolutePosition(Convert.ToInt16(70), Convert.ToInt16(37));
                        //添加图片到PDF的当前页
                        stamper.GetOverContent(i).AddImage(image, true);
                    }
                }

                //将MemoryStream输出到客户端浏览器
                string Disposition = "";

                if (OpenMode != null && OpenMode.Length > 0)
                    Disposition = OpenMode + "; ";

                Disposition += ServerUtility.EncodeAttachmentFileName(context.Request.UserAgent, FileName);

                context.Response.ContentType = ContentType;
                context.Response.AppendHeader("Content-Length", ExportResult.DataSize.ToString());
                context.Response.AppendHeader("Content-Disposition", Disposition);
                context.Response.ClearContent();
                object Data = ExportResult.SaveToVariant();
                context.Response.BinaryWrite(outputPdfStream.ToArray());
                context.Response.Flush();
            }

        }
    }    
```

### 实现代码过程

1、 网络上找到了简单实用iText操作PDF的代码，分为三块内容，第一块是创建空白PDF，第二块是创建水印，第三块去除水印。
```c
    static void temp1()
        {
            string workingFolder = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
            string startFile = Path.Combine(workingFolder, "StartFile.pdf");
            string watermarkedFile = Path.Combine(workingFolder, "Watermarked.pdf");
            string unwatermarkedFile = Path.Combine(workingFolder, "Un-watermarked.pdf");
            string watermarkText = "This is a test";

            //watermarkedFile= @"E:\a\1.pdf";
            //SECTION 1
            //Create a 5 page PDF, nothing special here
            using (FileStream fs = new FileStream(startFile, FileMode.Create, FileAccess.Write, FileShare.None))
            {
                using (Document doc = new Document(PageSize.LETTER))
                {
                    using (PdfWriter witier = PdfWriter.GetInstance(doc, fs))
                    {
                        doc.Open();

                        for (int i = 1; i <= 5; i++)
                        {
                            doc.NewPage();
                            doc.Add(new Paragraph(String.Format("This is page {0}", i)));
                        }

                        doc.Close();
                    }
                }
            }

            //SECTION 2
            //Create our watermark on a separate layer.The only different here is that we are adding the watermark to a PdfLayer which is an OCG or Optional Content Group
            PdfReader reader1 = new PdfReader(startFile);
            using (FileStream fs = new FileStream(watermarkedFile, FileMode.Create, FileAccess.Write, FileShare.None))
            {
                using (PdfStamper stamper = new PdfStamper(reader1, fs))
                {
                    int pageCount1 = reader1.NumberOfPages;
                    //Create a new layer
                    PdfLayer layer = new PdfLayer("WatermarkLayer", stamper.Writer);
                    for (int i = 1; i <= pageCount1; i++)
                    {
                        iTextSharp.text.Rectangle rect = reader1.GetPageSize(i);
                        //Get the ContentByte object
                        PdfContentByte cb = stamper.GetUnderContent(i);
                        //Tell the CB that the next commands should be "bound" to this new layer
                        cb.BeginLayer(layer);
                        cb.SetFontAndSize(BaseFont.CreateFont(BaseFont.HELVETICA, BaseFont.CP1252, BaseFont.NOT_EMBEDDED), 50);
                        PdfGState gState = new PdfGState();
                        gState.FillOpacity = 0.25f;
                        cb.SetGState(gState);
                        cb.SetColorFill(BaseColor.BLACK);
                        cb.BeginText();
                        cb.ShowTextAligned(PdfContentByte.ALIGN_CENTER, watermarkText, rect.Width / 2, rect.Height / 2, 45f);
                        cb.EndText();
                        //"Close" the layer
                        cb.EndLayer();
                    }
                }
            }

            //SECTION 3
            //Remove the layer created above
            //First we bind a reader to the watermarked file, then strip out a bunch of things, and finally use a simple stamper to write out the edited reader
            PdfReader reader2 = new PdfReader(watermarkedFile);

            //NOTE, This will destroy all layers in the document, only use if you don't have additional layers
            //Remove the OCG group completely from the document.
            //reader2.Catalog.Remove(PdfName.OCPROPERTIES);

            //Clean up the reader, optional
            reader2.RemoveUnusedObjects();

            //Placeholder variables
            PRStream stream;
            String content;
            PdfDictionary page;
            PdfArray contentarray;

            //Get the page count
            int pageCount2 = reader2.NumberOfPages;
            //Loop through each page
            for (int i = 1; i <= pageCount2; i++)
            {
                //Get the page
                page = reader2.GetPageN(i);
                //Get the raw content
                contentarray = page.GetAsArray(PdfName.CONTENTS);
                if (contentarray != null)
                {
                    //Loop through content
                    for (int j = 0; j < contentarray.Size; j++)
                    {
                        //Get the raw byte stream
                        stream = (PRStream)contentarray.GetAsStream(j);
                        //Convert to a string. NOTE, you might need a different encoding here
                        content = System.Text.Encoding.ASCII.GetString(PdfReader.GetStreamBytes(stream));
                        //Look for the OCG token in the stream as well as our watermarked text
                        if (content.IndexOf("/OC") >= 0 && content.IndexOf(watermarkText) >= 0)
                        {
                            //Remove it by giving it zero length and zero data
                            stream.Put(PdfName.LENGTH, new PdfNumber(0));
                            stream.SetData(new byte[0]);
                        }
                    }
                }
            }

            //Write the content out
            using (FileStream fs = new FileStream(unwatermarkedFile, FileMode.Create, FileAccess.Write, FileShare.None))
            {
                using (PdfStamper stamper = new PdfStamper(reader2, fs))
                {

                }
            }
            //this.Close();
        }
```

2、由于水印位置固定且位于页脚处无其他内容，因此创建白色图片覆盖水印未知实现去除功能。
结合上面的代码和网上找到的相关实现进行修改后如下：
```c
 static void temp2()
        {
            string watermarkedFile = @"E:\a\1out.pdf";
            string startFile = @"E:\a\1.pdf";
            File.Delete(watermarkedFile);
            PdfReader reader1 = new PdfReader(startFile);
            using (FileStream fs = new FileStream(watermarkedFile, FileMode.Create, FileAccess.Write, FileShare.None))
            {
                using (PdfStamper stamper = new PdfStamper(reader1, fs))
                {
                    int pageCount1 = reader1.NumberOfPages;
                    //Create a new layer
                    PdfLayer layer = new PdfLayer("WatermarkLayer", stamper.Writer);
                    for (int i = 1; i <= pageCount1; i++)
                    {
                        PdfContentByte canvas = stamper.GetOverContent(i);
                        canvas.SaveState();
                        canvas.SetColorFill(BaseColor.RED);
                        //PdfGState gs = new PdfGState();
                        //gs.FillOpacity = 0.6f;//设置透明度
                        //canvas.SetGState(gs);
                        //PDF的左下角的坐标是（0,0），单位是pt
                        canvas.Rectangle(70, 38, 400, 30);
                        canvas.Fill();
                        canvas.RestoreState();
                    }

                    stamper.Close();
                    reader1.Close();
                }
            }
        }

```


3、结合到实际的系统中(将有水印和无水印的两个PDF保存到本地，再以流方式传给客户端浏览器)

```c
        /// <summary>
        /// 将报表生成的二进制数据响应给 HTPP 请求客户端
        /// </summary>
        /// <param name="context"> HTPP 请求对象</param>
        /// <param name="ExportResult">报表生成的二进制数据</param>
        /// <param name="FileName">指定下载(或保存)文件时的默认文件名称</param>
        /// <param name="ContentType">响应的ContentType</param>
        /// <param name="OpenMode">指定生成的数据打开模式，可选[inline|attachment]，"inline"表示在网页中内联显示，"attachment"表示以附件文件形式下载。如果不指定，由浏览器自动确定打开方式。</param>
        public static void ResponseBinaryPDF(HttpContextBase context, IGRBinaryObject ExportResult, string FileName, string ContentType, string OpenMode)
        {
            #region 保存到本地pdf

            string newFileName = DirFileHelper.GetRamCode() + ".pdf"; //随机生成新的文件名
            string upLoadPath = DirFileHelper.GetUpLoadPath(); //保存目录相对路径
            string fullUpLoadPath = DirFileHelper.GetMapPath(upLoadPath); //保存目录的物理路径
            string newFilePath = upLoadPath + newFileName; //保存后的路径

            //检查保存的物理路径是否存在，不存在则创建
            if (!Directory.Exists(fullUpLoadPath))
            {
                Directory.CreateDirectory(fullUpLoadPath);
            }

            //保存文件
            ExportResult.SaveToFile(fullUpLoadPath + newFileName);


            #endregion

            #region 读取pdf去除水印


            string inputFilePath = fullUpLoadPath + newFileName;
            string outputFilePath = fullUpLoadPath + newFileName.Replace(".pdf", "_removed.pdf");//去除水印后pdf
            try
            {
                using (Stream inputPdfStream = new FileStream(inputFilePath, FileMode.Open, FileAccess.Read, FileShare.Read))
                using (Stream outputPdfStream = new FileStream(outputFilePath, FileMode.Create, FileAccess.Write, FileShare.ReadWrite))
                {
                    //Opens the unmodified PDF for reading
                    PdfReader reader = new PdfReader(inputPdfStream);
                    //Creates a stamper to put an image on the original pdf
                    PdfStamper stamper = new PdfStamper(reader, outputPdfStream); //{ FormFlattening = true, FreeTextFlattening = true };

                    int pageCount = reader.NumberOfPages;
                    ////Create a new layer
                    //PdfLayer layer = new PdfLayer("WatermarkLayer", stamper.Writer);
                    for (int i = 1; i <= pageCount; i++)
                    {
                        //PdfContentByte canvas = stamper.GetUnderContent(i);
                        //Creates an image that is the size i need to hide the text i'm interested in removing
                        iTextSharp.text.Image image = iTextSharp.text.Image.GetInstance(new Bitmap(400, 30), BaseColor.WHITE);
                        //Sets the position that the image needs to be placed (ie the location of the text to be removed)
                        //txtX.Text = 33,txtY.Text = 708 70, 38, 400, 30
                        image.SetAbsolutePosition(Convert.ToInt16(70), Convert.ToInt16(37));
                        //Adds the image to the output pdf
                        stamper.GetOverContent(i).AddImage(image, true);
                    }


                    //Creates the first copy of the outputted pdf
                    stamper.Close();
                }
            }
            catch (Exception ex)
            {
            }


            #endregion

            #region 返回客户端浏览器PDF
            //以字符流的形式下载文件
            using (FileStream fs = new FileStream(outputFilePath, FileMode.Open))
            {
                byte[] bytes = new byte[(int)fs.Length];
                fs.Read(bytes, 0, bytes.Length);
                fs.Close();
                string Disposition = "";

                if (OpenMode != null && OpenMode.Length > 0)
                    Disposition = OpenMode + "; ";

                Disposition += ServerUtility.EncodeAttachmentFileName(context.Request.UserAgent, FileName);

                context.Response.ContentType = ContentType;
                context.Response.AppendHeader("Content-Length", ExportResult.DataSize.ToString());
                context.Response.AppendHeader("Content-Disposition", Disposition);

                context.Response.ClearContent();
                object Data = ExportResult.SaveToVariant();
                context.Response.BinaryWrite(bytes);

                context.Response.Flush();
            }


            #endregion


        }       

```



## 网上相关资源与信息
1、 [iText官网](https://itextpdf.com/)
2、 [iText的NuGet地址](https://www.nuget.org/packages/iTextSharp/)
3、 [How to convert pdfstamper to byte array](http://www.it1352.com/245699.html)
4、 [PdfStamper获得字节数组](http://stackoverflow.org.cn/front/ask/view?ask_id=236528)
5、 [pt,px,em](http://www.runoob.com/w3cnote/px-pt-em-convert-table.html)
