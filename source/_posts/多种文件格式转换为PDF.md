---
title: 多种文件格式转换为PDF
date: 2021-05-29 19:54:04
tags:
---

[TOC]

<!--more-->

# ImgUtil

```
@Slf4j
public class ImgUtils {
    public static void main(String[] args) {
        img2PDF("d://test2.png","d://test3.pdf");
    }

    /**
     * 图片转PDF
     * @param imgFilePath
     * @param pdfFilePath
     * @return
     */
    public static boolean img2PDF(String imgFilePath, String pdfFilePath){
        File file = new File(imgFilePath);
        if (file.exists()) {
            com.lowagie.text.Document document = new com.lowagie.text.Document();
            FileOutputStream fos = null;
            try {
                fos = new FileOutputStream(pdfFilePath);
                com.lowagie.text.pdf.PdfWriter.getInstance(document, fos);

                // 添加PDF文档的某些信息，比如作者，主题等等
                document.addAuthor("newprint");
                document.addSubject("test pdf.");
                // 设置文档的大小
                document.setPageSize(PageSize.A4);
                // 打开文档
                document.open();
                // 写入一段文字
                // document.add(new Paragraph("JUST TEST ..."));

                // 读取一个图片
                com.lowagie.text.Image image = com.lowagie.text.Image.getInstance(imgFilePath);
                float imageHeight = image.getScaledHeight();
                float imageWidth = image.getScaledWidth();
                int i = 0;
                while (imageHeight > 500 || imageWidth > 500) {
                    image.scalePercent(100 - i);
                    i++;
                    imageHeight = image.getScaledHeight();
                    imageWidth = image.getScaledWidth();
                    System.out.println("imageHeight->" + imageHeight);
                    System.out.println("imageWidth->" + imageWidth);
                }

                image.setAlignment(com.lowagie.text.Image.ALIGN_CENTER);
                // //设置图片的绝对位置
                // image.setAbsolutePosition(0, 0);
                // image.scaleAbsolute(500, 400);

                // 插入一个图片
                document.add(image);
            } catch (com.lowagie.text.DocumentException de) {
                System.out.println(de.getMessage());
            } catch (IOException ioe) {
                System.out.println(ioe.getMessage());
            }
            document.close();
            try {
                fos.flush();
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            return true;
        } else {
            return false;
        }
    }
}
```

# PptxUtils

```java
@Slf4j
public class PptxUtils {

    public static void main(String[] args) throws Exception {
        //toImage("d:test.pptx","d:2.pdf");
        //createPdf("d://2.pdf1.jpg","d://2.pdf1.pdf");
        ImgUtils.img2PDF("d://2.pdf1.jpg","d://2.pdf1.pdf");
    }

    //将pptx转换成pdf
    public static void toImage(String str,String local) throws Exception {
        FileInputStream is = new FileInputStream(str);
        XMLSlideShow ppt = new XMLSlideShow(is);
        is.close();
        Dimension pgsize = ppt.getPageSize();
        System.out.println(pgsize.width + "--" + pgsize.height);
        for (int i = 0; i < ppt.getSlides().size(); i++) {
            try {
                for (XSLFShape shape : ppt.getSlides().get(i).getShapes()) {
                    if (shape instanceof XSLFTextShape) {
                        XSLFTextShape tsh = (XSLFTextShape) shape;
                        for (XSLFTextParagraph p : tsh) {
                            for (XSLFTextRun r : p) {
                                r.setFontFamily("图片转换");
                            }
                        }
                    }
                }
                BufferedImage img = new BufferedImage(pgsize.width, pgsize.height, BufferedImage.TYPE_INT_RGB);
                Graphics2D graphics = img.createGraphics();
                // clear the drawing area
                graphics.setPaint(Color.white);
                graphics.fill(new Rectangle2D.Float(0, 0, pgsize.width, pgsize.height));
                // render
                ppt.getSlides().get(i).draw(graphics);
                // save the output
                String filename = local + (i + 1) + ".jpg";
                System.out.println(filename);
                FileOutputStream out = new FileOutputStream(filename);
                javax.imageio.ImageIO.write(img, "png", out);
                out.close();
            } catch (Exception e) {
                log.info("图片" + i + "pptx文件名");
            }
        }
        System.out.println("success");

    }

    public static boolean createPdf(String imageFolderPath, String pdfPath) throws Exception {
        Document doc = new Document();
        try {
            PdfWriter.getInstance(doc, new FileOutputStream(pdfPath));
            doc.open();
            File file = new File(imageFolderPath);
            File[] files = file.listFiles();
            if (files != null && files.length > 0) {

                for (File file1 : files) {
                    File file2 = new File(imageFolderPath + "/" + file1.getName());

                    if (file2.exists()) {
                        Image img = Image.getInstance(imageFolderPath + "/" + file1.getName());
                        Float height = img.getHeight();
                        Float width = img.getWidth();
                        Integer percent = getPercent(height, width);
                        img.setAlignment(Image.MIDDLE);
                        img.scalePercent(percent);
                        //img.scalePercent(50, 100);
                        doc.add(img);
                    }
                }
            }
            doc.close();
        } catch (IOException e) {
            e.printStackTrace();
            return false;
        } catch (DocumentException e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    public static Integer getPercent(Float h, Float w) {
        Integer p = 0;
        Float p2 = 0.0f;
        p2 = 530 / w * 100;
        System.out.println("--" + p2);
        p = Math.round(p2);
        return p;
    }
}
```

# Excel2PDF

```java
public class TestForExcel2PDF {

    public static void main(String[] args) {

        String sourceFilePath="d:/1.xlsx";
        String desFilePath="d:/1.pdf";

        excel2pdf(sourceFilePath, desFilePath);
    }

    /**
     * 获取license 去除水印
     * @return
     */
    public static boolean getLicense() {
        boolean result = false;
        try {
            InputStream is = TestForExcel2PDF.class.getClassLoader().getResourceAsStream("\\license.xml");
            License aposeLic = new License();
            aposeLic.setLicense(is);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * excel 转为pdf 输出。
     *
     * @param sourceFilePath  excel文件
     * @param desFilePathd  pad 输出文件目录
     */
    public static void excel2pdf(String sourceFilePath, String desFilePathd ){
        // 验证License 若不验证则转化出的pdf文档会有水印产生
        if (!getLicense()) {
            return;
        }
        try {
            // 原始excel路径
            Workbook wb = new Workbook(sourceFilePath);

            FileOutputStream fileOS = new FileOutputStream(desFilePathd);
            PdfSaveOptions pdfSaveOptions = new PdfSaveOptions();
            pdfSaveOptions.setOnePagePerSheet(true);


            int[] autoDrawSheets={3};
            //当excel中对应的sheet页宽度太大时，在PDF中会拆断并分页。此处等比缩放。
            //autoDraw(wb,autoDrawSheets);

            int[] showSheets={0};
            //隐藏workbook中不需要的sheet页。
            printSheetPage(wb,showSheets);
            wb.save(fileOS, pdfSaveOptions);
            fileOS.flush();
            fileOS.close();
            System.out.println("完毕");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    /**
     * 设置打印的sheet 自动拉伸比例
     * @param wb
     * @param page 自动拉伸的页的sheet数组
     */
    public static void autoDraw(Workbook wb,int[] page){
        if(null!=page&&page.length>0){
            for (int i = 0; i < page.length; i++) {
                wb.getWorksheets().get(i).getHorizontalPageBreaks().clear();
                wb.getWorksheets().get(i).getVerticalPageBreaks().clear();
            }
        }
    }


    /**
     * 隐藏workbook中不需要的sheet页。
     * @param wb
     * @param page 显示页的sheet数组
     */
    public static void printSheetPage(Workbook wb,int[] page){
        for (int i= 1; i < wb.getWorksheets().getCount(); i++)  {
            wb.getWorksheets().get(i).setVisible(false);
        }
        if(null==page||page.length==0){
            wb.getWorksheets().get(0).setVisible(true);
        }else{
            for (int i = 0; i < page.length; i++) {
                wb.getWorksheets().get(i).setVisible(true);
            }
        }
    }
}
```

# WordUtils

```java
@Slf4j
public class WordUtils
{

	public static void main(String[] args) throws Exception {
		WordUtils.convertDocxToPdf("d://123.docx","d://123.pdf");
	}

	/**
	 * docx文档转换为PDF
	 *
	 * @param pdfPath PDF文档存储路径
	 * @throws Exception 可能为Docx4JException, FileNotFoundException, IOException等
	 */
	public static void convertDocxToPdf(String docxPath, String pdfPath)
	{
		FileOutputStream fileOutputStream = null;
		try {
			File file = new File(docxPath);
			fileOutputStream = new FileOutputStream(new File(pdfPath));
			WordprocessingMLPackage mlPackage = WordprocessingMLPackage.load(file);
			log.info("docx转换:" + mlPackage);
			setFontMapper(mlPackage);
			Docx4J.toPDF(mlPackage, fileOutputStream);
		} catch (Exception e) {
			e.printStackTrace();
			log.error("docx文档转换为PDF失败");
		} finally {
			try {
				fileOutputStream.flush();
			} catch (IOException e) {
				e.printStackTrace();
			}
			IOUtils.closeQuietly(fileOutputStream);
		}
	}


	private static void setFontMapper(WordprocessingMLPackage mlPackage) throws Exception {
		Mapper fontMapper = new IdentityPlusMapper();
		fontMapper.put("隶书", PhysicalFonts.get("LiSu"));
		fontMapper.put("宋体", PhysicalFonts.get("SimSun"));
		fontMapper.put("微软雅黑", PhysicalFonts.get("Microsoft Yahei"));
		fontMapper.put("黑体", PhysicalFonts.get("SimHei"));
		fontMapper.put("楷体", PhysicalFonts.get("KaiTi"));
		fontMapper.put("新宋体", PhysicalFonts.get("NSimSun"));
		fontMapper.put("华文行楷", PhysicalFonts.get("STXingkai"));
		fontMapper.put("华文仿宋", PhysicalFonts.get("STFangsong"));
		fontMapper.put("宋体扩展", PhysicalFonts.get("simsun-extB"));
		fontMapper.put("仿宋", PhysicalFonts.get("FangSong"));
		fontMapper.put("仿宋_GB2312", PhysicalFonts.get("FangSong_GB2312"));
		fontMapper.put("幼圆", PhysicalFonts.get("YouYuan"));
		fontMapper.put("华文宋体", PhysicalFonts.get("STSong"));
		fontMapper.put("华文中宋", PhysicalFonts.get("STZhongsong"));
		fontMapper.put("等线", PhysicalFonts.get("SimSun"));
		fontMapper.put("等线 Light", PhysicalFonts.get("SimSun"));
		fontMapper.put("华文琥珀", PhysicalFonts.get("STHupo"));
		fontMapper.put("华文隶书", PhysicalFonts.get("STLiti"));
		fontMapper.put("华文新魏", PhysicalFonts.get("STXinwei"));
		fontMapper.put("华文彩云", PhysicalFonts.get("STCaiyun"));
		fontMapper.put("方正姚体", PhysicalFonts.get("FZYaoti"));
		fontMapper.put("方正舒体", PhysicalFonts.get("FZShuTi"));
		fontMapper.put("华文细黑", PhysicalFonts.get("STXihei"));
		fontMapper.put("宋体扩展",PhysicalFonts.get("simsun-extB"));
		fontMapper.put("仿宋_GB2312",PhysicalFonts.get("FangSong_GB2312"));
		fontMapper.put("新細明體",PhysicalFonts.get("SimSun"));
		//解决宋体（正文）和宋体（标题）的乱码问题
		PhysicalFonts.put("PMingLiU", PhysicalFonts.get("SimSun"));
		PhysicalFonts.put("新細明體", PhysicalFonts.get("SimSun"));
		mlPackage.setFontMapper(fontMapper);
	}
}
```

# ZipUtil

```java
@Slf4j
public class ZipUtil {

    public static void main(String[] args) {
        createZip("d://test3.pdf","d://test.zip");
    }

    /**
     * 创建ZIP文件
     * @param sourcePath 文件或文件夹路径
     * @param zipPath 生成的zip文件存在路径（包括文件名）
     */
    public static void createZip(String sourcePath, String zipPath) {
        FileOutputStream fos = null;
        ZipOutputStream zos = null;
        try {
            fos = new FileOutputStream(zipPath);
            zos = new ZipOutputStream(fos);
            //此处修改字节码方式
            //zos.setEncoding("UTF-8");
            writeZip(new File(sourcePath), "", zos);
        } catch (FileNotFoundException e) {
            log.error("创建ZIP文件失败",e);
        } finally {
            try {
                if (zos != null) {
                    zos.close();
                }
            } catch (IOException e) {
                log.error("创建ZIP文件失败",e);
            }

        }
    }

    private static void writeZip(File file, String parentPath, ZipOutputStream zos) {
        if(file.exists()){
            //处理文件夹
            if(file.isDirectory()){
                parentPath+=file.getName()+File.separator;
                File [] files=file.listFiles();
                if(files.length != 0){
                    for(File f:files){
                        writeZip(f, parentPath, zos);
                    }
                }else{
                    //空目录则创建当前目录
                    try {
                        zos.putNextEntry(new ZipEntry(parentPath));
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }else{
                FileInputStream fis=null;
                try {
                    fis=new FileInputStream(file);
                    ZipEntry ze = new ZipEntry(parentPath + file.getName());
                    zos.putNextEntry(ze);
                    byte [] content=new byte[1024];
                    int len;
                    while((len=fis.read(content))!=-1){
                        zos.write(content,0,len);
                        zos.flush();
                    }

                } catch (FileNotFoundException e) {
                    log.error("创建ZIP文件失败",e);
                } catch (IOException e) {
                    log.error("创建ZIP文件失败",e);
                }finally{
                    try {
                        if(fis!=null){
                            fis.close();
                        }
                    }catch(IOException e){
                        log.error("创建ZIP文件失败",e);
                    }
                }
            }
        }
    }
}
```