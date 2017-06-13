# pdf2img
1. 使用icepdf, convert pdf to images
```java
/**
   * 将指定pdf文件的首页转换为指定路径的缩略图
   * 
   * @param filepath 原文件路径，例如d:/test.pdf
   * @param imagepath 图片生成路径，例如 d:/
   * @param zoom 缩略图显示倍数，1表示不缩放，0.3则缩小到30%
   * @return 生成图片名称列表
   */
  public void tranfer(String filepath, String imagepath, float zoom) throws PDFException,
      PDFSecurityException, IOException {
    // ICEpdf document class
    Document document = null;
    float rotation = 0f;
    document = new Document();
    document.setFile(filepath);
    // maxPages = document.getPageTree().getNumberOfPages();
    for (int i = 0; i < document.getNumberOfPages(); i++) {
      BufferedImage img =
          (BufferedImage) document.getPageImage(i, GraphicsRenderingHints.SCREEN,
              org.icepdf.core.pobjects.Page.BOUNDARY_CROPBOX, rotation, zoom);
      Iterator iter = ImageIO.getImageWritersBySuffix("jpg");
      ImageWriter writer = (ImageWriter) iter.next();
      File outFile =
          new File(imagepath + (filepath.replace(imagepath, "").split("\\.")[0]) + "-" + (i + 1)
              + ".jpg");
      FileOutputStream out = new FileOutputStream(outFile);
      ImageOutputStream outImage = ImageIO.createImageOutputStream(out);
      writer.setOutput(outImage);
      writer.write(new IIOImage(img, null, null));
    }
  }
```

2. 使用**pdfbox2.x** convert pdf to images  &  合同多个图片为一个图片
```java
/**
   * pdf to big image
   * 
   * @param pdfPath pdf文件路径
   * @return big image file
   * @throws Exception
   */
  private File pdf2image(File pdfFile) throws Exception {
    // 拼成图片后的宽度和高度
    int w = 0;
    int h = 0;
    List<BufferedImage> images = Lists.newArrayList();
    // 生成图片后的路径
    String path = pdfFile.getParent() + File.separator;
    String fileName = pdfFile.getName().replace(".pdf", "");

    File destinationFile = new File(path);
    if (!destinationFile.exists()) {
      destinationFile.mkdir();
    }
    PDDocument document = PDDocument.load(pdfFile);
    PDPageTree list = document.getDocumentCatalog().getPages();
    int pageCounter = 0;
    for (PDPage page : list) {
      PDFRenderer pdfRenderer = new PDFRenderer(document);
      BufferedImage image = pdfRenderer.renderImageWithDPI(pageCounter, 100, ImageType.RGB);
      String target = path + fileName + "-" + (pageCounter++) + ".png";
      ImageIOUtil.writeImage(image, target, 100);

      w = image.getWidth();
      h += image.getHeight();
      images.add(image);
      new File(target).delete();
    }
    document.close();

    BufferedImage combined = new BufferedImage(w, h, BufferedImage.TYPE_INT_ARGB);
    Graphics g = combined.getGraphics();

    int y = 0;
    for (BufferedImage image : images) {
      g.drawImage(image, 0, y, null);
      y += image.getHeight();
    }

    // Save as new image
    File image = new File(path, fileName + ".png");
    ImageIO.write(combined, "PNG", image);
    return image;
  }
```
pom.xml
```xml
<dependency>
  <groupId>org.apache.pdfbox</groupId>
  <artifactId>pdfbox</artifactId>
  <version>2.0.6</version>
</dependency>
<dependency>
  <groupId>org.apache.pdfbox</groupId>
  <artifactId>pdfbox-tools</artifactId>
  <version>2.0.6</version>
</dependency>
```

