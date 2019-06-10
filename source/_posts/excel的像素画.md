---
title: excel的像素画
date: 2019-06-08 18:11:15
categories: 小玩具
tags: [突发奇想]
---

# 缘由

豆瓣上看的在excel上画像素画，之前工作中用到了excel的解析，那时候这一个小需求写了好久。大约两个星期吧！还被带我的小哥嘲讽了一下，不过确实对excel的解析熟悉了不少（也就是poi的使用......），之前工作的那个解析也是可以写写的，这次就说像素画，不扯其他的。

<!--more-->

# 进入正题

## 原理

​	原理就是把图片的每个像素点遍历一下，拿到每个像素的内容，然后设置excel的单元格的合适大小，给每个excel的单元格一一设置背景色。（也不算全部遍历，这是在遍历的时候需要跳跃一下）

## 代码

​	虽然是一个简单的小玩意，使用maven还是方便一些的，依赖如下：

``` xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.9</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.9</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml-schemas</artifactId>
    <version>3.9</version>
</dependency>
```

正式代码

​	应豆瓣友邻的要求写一个教程，就把注释写的详细一点

``` java
public class ToExcel {
    public static void load(String picPath) throws Exception{
        List<Integer> list=new ArrayList<Integer>(); // 用来存图片的像素信息
        BufferedImage pic = ImageIO.read(new File(picPath));
        int height = pic.getHeight(); // 获得图片的高
        int width = pic.getWidth(); // 获得图片的宽
        int index=0; // 计数，真实的excel的行数
        for(int i=0;i<height;i+=4){  // 并非全部遍历，每次以四个像素对高在跳跃，建议你设置大一点
            index++;
            for(int j=0;j<width;j+=4){ // 并非全部遍历，每次以四个像素对宽在跳跃，建议你设置大一点
                int pixel = pic.getRGB(j, i); // 获得跳跃后的像素点
                list.add(pixel); // 添加到list当中
            }
        }
        genExce(list,index);
    }
    public static void genExce(List<Integer> total,int heights){
        int widths=total.size()/heights; // 通过高度获得宽度
        XSSFWorkbook excel = new XSSFWorkbook(); // 创建操作单元格的对象
        XSSFSheet sheet = excel.createSheet("beauty"); // 设置excel的表的名称
        //设置默认的行高和宽，对于每个单元格来说的
        short width= (short) 2;
        short height= (short) (1.50875*256);
        sheet.setDefaultRowHeight(height); 
        sheet.setDefaultColumnWidth(width);
        // 设置一个map用来存放颜色对象，避免过多的创建颜色对象，影响速度（但是创建的以然很多，有点慢）
        Map<String,XSSFColor> map=new HashMap<String,XSSFColor>();
        for (int i = 0; i <heights ; i++) {
            XSSFRow row = sheet.createRow(i);
            for (int j = 0; j <widths ; j++) {
                if(j!=0&&j%widths==0){ // 进行换行处理，要不怎么能成为一个图片
                    break;
                }
                Integer pixel = total.get(i * widths + j); //拿到对应excel位置的图片的像素值
                int red = (pixel & 0xff0000) >> 16; // 三基色，红、绿、蓝，用于下面生成颜色
                int green = (pixel & 0xff00) >> 8;
                int blue = pixel & 0xff;
                XSSFCell cell = row.createCell(j); // 在第i行中添加单元格第j个单元格
                XSSFCellStyle style = excel.createCellStyle(); // 创建单元格样式
                // 判断颜色存在于集合之中，存在直接拿出，不存在创建放入
                if(!map.containsKey(red+"."+green+"."+blue)){
                    map.put(red+"."+green+"."+blue,new XSSFColor(new Color(red,green,blue)));
                }
                // 统一在map中拿取颜色对象
                style.setFillForegroundColor(map.get(red+"."+green+"."+blue));
                //solid 填充  foreground  前景色
                style.setFillPattern(HSSFCellStyle.SOLID_FOREGROUND);
                cell.setCellStyle(style);
                System.out.println(i * widths + j); // 运行时间有点长，给你个输出，不至于让你感觉程序卡死了.....
            }
        }
        FileOutputStream out = null;
        try {
            out = new FileOutputStream("D:/old_color.xlsx");
            excel.write(out);
            out.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) { // 程序的入口
        try {
            load("D:/1.jpg");   // 读取图片地址
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

# tip

图片的大小别使用太大的，手机拍的照片最好在ps里缩小一下，然后再用，如果最后生成的excel打不开，将循环图片的那个间隔值设置大一些，12或者16长宽统一。程序的时间只要都耗费到了，生成颜色对象的那块。主要还是我太菜了，如果路过的大神有什么可以大幅度的提升运行时间的办法，可以留个言我试一下。

运行效果太慢了，我就不贴出来了。相信来看的大部分都是豆瓣的观光团，效果你们都看过了。代码没什么实际意义，就是拿过来玩的。