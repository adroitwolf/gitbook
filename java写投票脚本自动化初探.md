---
title: java写投票脚本自动化初探
date: 2019-06-07 10:51:25
categories:
- java
- 自动化
tags:
- java
- selenium
- 自动化
- orc
- opencv
---

# java写投票脚本自动化初探

由于大学里面有一些任务需要投票，而又不想劳烦他人去帮我，所以打算自己写一个脚本来实现自动投票的功能。这里记录一下我的整个过程。


## 技术选型

java8
selenium3 实现chrome自动化的jar包
AUTolt 模拟键盘操作的软件

* 导入selenium3

这里我用的maven导入的，代码如下：

```xml
      <dependency>
           <groupId>org.seleniumhq.selenium</groupId>
           <artifactId>selenium-server</artifactId>
           <version>3.141.59</version>
       </dependency>
       <dependency>
       <groupId>org.seleniumhq.selenium</groupId>
       <artifactId>selenium-java</artifactId>
       <version>3.14.0</version>
   </dependency>
```


## 代码结构

![](https://s2.ax1x.com/2019/06/08/VDUrOx.png)


## 利用代码启动chrome

由于我这里是用的chrome实现的自动化，所以我们先要下chrome的[启动器](http://chromedriver.storage.googleapis.com/index.html)

![](https://s2.ax1x.com/2019/06/08/VDNzdO.png)


我们需要下载对应版本的启动器，不然会失效，首先查看自己的chrome版本，然后打开每个版本的文件夹，里面有notes.txt,显示对应的支持版本范围。

![](https://s2.ax1x.com/2019/06/08/VDU8O0.png)

下载好之后随便放到一个位置，后面根据路径引入即可，为了方便我将程序放到我的resource目录下了，如上图所示，代码如下：

```java
        // 设置webdirver路径  这个BrushTicket是指的这个代码所在的类。
        System.setProperty("webdriver.chrome.driver", BrushTicket.class.getClassLoader().getResource("chromedriver.exe").getPath());


// 创建ChromeOptions，options可以设置一些网页请求头啥的
        ChromeOptions options = new ChromeOptions();

        //        指定本机chrome安装位置
        options.setBinary("C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe");

          // 创建WebDriver对象
         ChromeDriver driver = new ChromeDriver(options);

         // 打开那个网站
         driver.get("https://www.wjx.cn/m/36384473.aspx");
```

要记住，这个driver是最核心的类，我们后面根据这个driver在网页上做各种操作


## 网页自动点击操作


这个做法很简单，基本就是定位需要点击的位置，然后点击。


我截取了一小段代码，基本上所有点击的功能就都会了


```java

 driver.findElement(By.xpath("//div[@id='divSubmit']/div[2]")).click();
```


嗯哼，这里其实就需要注意一下这个xpath的定位问题，//div[@id='xxx'],就是定位这个id为啥的元素，其实也可以用通配符，比如说 //* [@id='xxx']。后面再像洋葱一样，一层层的扒网页就行。

这个findElement的返回值是WebElement,这个元素又可以做各种操作，比如取css的属性，或者自己的属性，也可以点击和输入值(sendKeys)



## 验证码(未完全解决)


本来以为点完各种按钮，提交之后就完事，结果不！突然出现了验证码。

挣扎了很久，我选择了将验证码下载到本地后，然后解析出验证码这个解决办法。


### 解决弹窗问题


我们没有管验证码直接提交，会弹出一个警告框，我们先解决弹窗，后来发现问卷星的网页结构是，弹窗由一个div构成的，如果没有弹过的时候是没有这个结构的，第二次弹出之前，会将之前的div由display:none属性改回block属性，这样就简单了。具体代码如下：


```java
/**
    * 功能描述: //判断是否由弹出框
    * @Param: [driver]
    * @Return: boolean
    * @Author: WHOAMI
    * @Date: 2019/6/8 22:21
     */
private boolean ifAlert(ChromeDriver driver){
       try
       {
         //定位元素，如果没有这个元素直接报错
           WebElement  webElement = driver.findElement(By.xpath("//div[@id='alert_box']"));

           Thread.sleep(300);

           if(!webElement.getCssValue("display").equals("none")) {
               return true;
           }
       }
       catch (Exception Ex)
       {
           Ex.printStackTrace();
           return false;
       }
       return false;
   }

```

根据有没有Alert框判断是不是让我们输入验证码。

### 下载图片


首先我们需要利用Action解决右键另存图片这个操作

具体代码如下：


```java

  // 这个driver是ChromeDriver
      Actions action = new Actions(driver);

// 定位到验证码图片的位置
      WebElement element = driver.findElement(By.xpath("//div[@id='tdCode']/table/tbody/tr/td[3]/img"));

// action移动到指定位置
      action.moveToElement(element);
// 休眠等待网页反应
      Thread.sleep(200);

// 这里模拟右键 打开菜单
      action.contextClick(element).build().perform();


      //模拟键盘操作（这里是移动向下方向键）  这个Robot是java.awt包下面的
      Robot robot = new Robot();
      Thread.sleep(400);
      // 模拟键盘下方向键
      robot.keyPress(KeyEvent.VK_DOWN);

      Thread.sleep(400);
      robot.keyPress(KeyEvent.VK_DOWN);
// 模拟回车键
      Thread.sleep(400);
      robot.keyPress(KeyEvent.VK_ENTER);
```

**后面需要了解一下java.awt这个包**

由于后面的选择保存位置然后下载操作是selenium3实现不了的，所以我们选择了AUTolt这个软件，[下载网址](https://www.autoitscript.com/site/autoit/downloads/)
,然后安装即可，这里不再赘述。


软件安装好之后，是这个样子的

![](https://s2.ax1x.com/2019/06/08/VDcYiq.png)


#### 操作AUTolt定位窗口
先打开游览器准备上面另存为的操作
![](https://s2.ax1x.com/2019/06/08/VDgCYq.png)

定位另存为窗口需要这个数据 Class: 比如上面的窗口时#32770，下面写脚本需要用。

![](https://s2.ax1x.com/2019/06/08/VDgklT.png)


定位按钮啊，编辑框啥的需要这个数据 CLassNameNN

![](https://s2.ax1x.com/2019/06/08/VDge0J.png)

> 其实我这里为了方便保存图片都是一个名字，而且为了方便都没有删除图片，所以这里出现了一个替换的问题，解决办法其实就是在定位一下替换的窗口和确定按钮就行


#### 写脚本


打开写脚本的工具，我的代码是这样的，改改就能用


```bash
ControlFocus("另存为", "","Edit1")
;ControlFocus("title","text",controlID) Edit1=Edit instance 1
; Wait 10 seconds for the Upload window to appear

WinWait("[CLASS:#32770]","",10)

; Set input focus to the edit control of Upload window using the handle returned by WinWait

  ControlFocus("另存为","","Edit1")

  Sleep(1000)

; Set the File name text on the Edit field

  ControlSetText("另存为", "", "Edit1", "buff.gif")

  Sleep(200)

; Click on the Open button

ControlClick("另存为", "","Button2");

ControlFocus("确认另存为", "","Edit2")

 WinWait("[CLASS:#32770]","",10)

 sleep(200)


ControlClick("确认", "","Button1");
```



写完脚本后，保存成au3格式的文件，准备合成exe

#### 合成exe文件


打开合成exe程序,确定脚本和生成exe路径即可。

![](https://s2.ax1x.com/2019/06/08/VDg2As.png)


#### java引用该文件

```java
//调用你使用Compile Script to.exe生成的可执行exe文件 这个download.exe就是我生成的exe文件
           //对Windows窗体进行操作：更换文件名，并保存到指定文件夹
     Runtime.getRuntime().exec("D:\\code\\java\\download.exe");
//代码等待程序完成在进行接下来的操作。
     Thread.sleep(4000);
```


### 验证码去除干扰线


由于我没有学**openCV**算法，所以我就直接粘代码了。


```java
/**
   * 功能描述: //验证码去除干扰线
   * @Param: [sfile：图片文件路径, destDir：文件名字]
   * @Return: void
   * @Author: WHOAMI
   * @Date: 2019/6/8 22:11
    */
   public static void cleanLinesInImage(File sfile, String destDir)  throws IOException {
       File destF = new File(destDir);
       if (!destF.exists())
       {
           destF.mkdirs();
       }

       BufferedImage bufferedImage = ImageIO.read(sfile);
       int h = bufferedImage.getHeight();
       int w = bufferedImage.getWidth();

       // 灰度化
       int[][] gray = new int[w][h];
       for (int x = 0; x < w; x++)
       {
           for (int y = 0; y < h; y++)
           {
               int argb = bufferedImage.getRGB(x, y);
               // 图像加亮（调整亮度识别率非常高）
               int r = (int) (((argb >> 16) & 0xFF) * 1.1 + 30);
               int g = (int) (((argb >> 8) & 0xFF) * 1.1 + 30);
               int b = (int) (((argb >> 0) & 0xFF) * 1.1 + 30);
               if (r >= 255)
               {
                   r = 255;
               }
               if (g >= 255)
               {
                   g = 255;
               }
               if (b >= 255)
               {
                   b = 255;
               }
               gray[x][y] = (int) Math
                       .pow((Math.pow(r, 2.2) * 0.2973 + Math.pow(g, 2.2)
                               * 0.6274 + Math.pow(b, 2.2) * 0.0753), 1 / 2.2);
           }
       }

       // 二值化
       int threshold = ostu(gray, w, h);
       BufferedImage binaryBufferedImage = new BufferedImage(w, h, BufferedImage.TYPE_BYTE_BINARY);
       for (int x = 0; x < w; x++)
       {
           for (int y = 0; y < h; y++)
           {
               if (gray[x][y] > threshold)
               {
                   gray[x][y] |= 0x00FFFF;
               } else
               {
                   gray[x][y] &= 0xFF0000;
               }
               binaryBufferedImage.setRGB(x, y, gray[x][y]);
           }
       }

       //去除干扰线条
       for(int y = 1; y < h-1; y++){
           for(int x = 1; x < w-1; x++){
               boolean flag = false ;
               if(isBlack(binaryBufferedImage.getRGB(x, y))){
                   //左右均为空时，去掉此点
                   if(isWhite(binaryBufferedImage.getRGB(x-1, y)) && isWhite(binaryBufferedImage.getRGB(x+1, y))){
                       flag = true;
                   }
                   //上下均为空时，去掉此点
                   if(isWhite(binaryBufferedImage.getRGB(x, y+1)) && isWhite(binaryBufferedImage.getRGB(x, y-1))){
                       flag = true;
                   }
                   //斜上下为空时，去掉此点
                   if(isWhite(binaryBufferedImage.getRGB(x-1, y+1)) && isWhite(binaryBufferedImage.getRGB(x+1, y-1))){
                       flag = true;
                   }
                   if(isWhite(binaryBufferedImage.getRGB(x+1, y+1)) && isWhite(binaryBufferedImage.getRGB(x-1, y-1))){
                       flag = true;
                   }
                   if(flag){
                       binaryBufferedImage.setRGB(x,y,-1);
                   }
               }
           }
       }


       // 矩阵打印
       for (int y = 0; y < h; y++)
       {
           for (int x = 0; x < w; x++)
           {
               if (isBlack(binaryBufferedImage.getRGB(x, y)))
               {
                   System.out.print("*");
               } else
               {
                   System.out.print(" ");
               }
           }
           System.out.println();
       }

       ImageIO.write(binaryBufferedImage, "jpg", new File(destDir, sfile
               .getName()));
   }

   public static boolean isBlack(int colorInt)
   {
       Color color = new Color(colorInt);
       if (color.getRed() + color.getGreen() + color.getBlue() <= 300)
       {
           return true;
       }
       return false;
   }

   public static boolean isWhite(int colorInt)
   {
       Color color = new Color(colorInt);
       if (color.getRed() + color.getGreen() + color.getBlue() > 300)
       {
           return true;
       }
       return false;
   }

   public static int isBlackOrWhite(int colorInt)
   {
       if (getColorBright(colorInt) < 30 || getColorBright(colorInt) > 730)
       {
           return 1;
       }
       return 0;
   }

   public static int getColorBright(int colorInt)
   {
       Color color = new Color(colorInt);
       return color.getRed() + color.getGreen() + color.getBlue();
   }

   public static int ostu(int[][] gray, int w, int h)
   {
       int[] histData = new int[w * h];
       // Calculate histogram
       for (int x = 0; x < w; x++)
       {
           for (int y = 0; y < h; y++)
           {
               int red = 0xFF & gray[x][y];
               histData[red]++;
           }
       }

       // Total number of pixels
       int total = w * h;

       float sum = 0;
       for (int t = 0; t < 256; t++)
           sum += t * histData[t];

       float sumB = 0;
       int wB = 0;
       int wF = 0;

       float varMax = 0;
       int threshold = 0;

       for (int t = 0; t < 256; t++)
       {
           wB += histData[t]; // Weight Background
           if (wB == 0)
               continue;

           wF = total - wB; // Weight Foreground
           if (wF == 0)
               break;

           sumB += (float) (t * histData[t]);

           float mB = sumB / wB; // Mean Background
           float mF = (sum - sumB) / wF; // Mean Foreground

           // Calculate Between Class Variance
           float varBetween = (float) wB * (float) wF * (mB - mF) * (mB - mF);

           // Check if new maximum found
           if (varBetween > varMax)
           {
               varMax = varBetween;
               threshold = t;
           }
       }

       return threshold;
   }

```


粘贴完代码后运行就会在相同的路径下覆盖原来的验证码图片，但是据我测试，这个代码效果不是很理想，所以才是未完成的操作。


## 图像识别


图片识别我打算利用tess4j。

maven导入项目

```xml
        <dependency>
            <groupId>net.sourceforge.tess4j</groupId>
            <artifactId>tess4j</artifactId>
            <version>4.3.1</version>
        </dependency>
```

导入语言包，这一步操作需要我们下载tess4j的源代码，然后解压之后将tessdata移出来，供下面导入。

接着我们利用tess4j进行图像识别


我发现根据上面图像处理之后，图像放大10倍后tess4j的识别率是最高的，所以有了下面的代码


```java

  /**
  * 功能描述: //将代码放大10倍
  * @Param: [file]
  * @Return: java.awt.image.BufferedImage
  * @Author: WHOAMI
  * @Date: 2019/6/8 22:24
   */
    private static BufferedImage change(File file){

        // 读取图片字节数组
        BufferedImage textImage = null;
        try{
            InputStream in = new FileInputStream(file);
            BufferedImage image = ImageIO.read(in);
            in.close();
            textImage = ImageHelper.convertImageToGrayscale(ImageHelper.getSubImage(image, 0, 0, image.getWidth(), image.getHeight()));  //对图片进行处理
            textImage = ImageHelper.getScaledInstance(image, image.getWidth() * 10, image.getHeight() * 10);  //将图片扩大5倍

        }catch (IOException e) {
            e.printStackTrace();
        }

        return textImage;
    }

```


>接着就是我们的核心代码了


```java
public static String deal(String pathname,String fileName){
        //打开需要处理的图片
        File imageFile = new File(pathname+"\\"+fileName);

        try {
          //去干扰线
            cleanLinesInImage(imageFile,pathname);
        } catch (IOException e) {
            e.printStackTrace();
        }


        //开始图像识别
        Tesseract instance = new Tesseract();



          // 这是我的语言包的路径
        instance.setDatapath("D:\\code\\java\\ChromeOperation\\tessdata");

        instance.setLanguage("eng");//选择字库文件（只需要文件名，不需要后缀名）
        //将验证码图片的内容识别为字符串
        try {
          // 打开处理后的图像
            File image = new File(pathname+"\\"+fileName);

            String result = instance.doOCR(change(image));


          //利用正则表达式过滤掉非法字符
            String correct = result.replaceAll("[^0-9a-zA-Z]J*","");
            // 返回正确的字符串
            return correct;
        } catch (TesseractException e) {
            e.printStackTrace();
        }

        return null;
    }

```


### 重复验证


由于上面的代码并不理想，所以我打算利用死循环直到他输入正确为止，代码如下：

```java
          while(ifAlert(driver)){

               driver.findElement(By.xpath("//div[@id='alert_box']/div[2]/div[2]/div[2]")).click();
               dealImg(driver);
           }

```
