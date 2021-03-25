---
title: Java处理数据库表获取字段名
date: 2017-07-10 20:03:04
tags:
---

利用 Java 处理所提供的数据库表描述，来获取当中的字段名并输出，便于 ARES 应用。

<!--more-->

```java
import java.io.*;

public class Main {

    public static void doSegmentName(){
        String fileLoc = "C:\\LLLLL\\raw.txt";
        try{
            File raw = new File(fileLoc);
            File modified = new File("C:\\LLLLL\\new.txt");
            modified.createNewFile();
            BufferedWriter out = new BufferedWriter(new FileWriter(modified));
            InputStreamReader reader = new InputStreamReader(new FileInputStream(raw));
            BufferedReader buffer = new BufferedReader(reader);
            String line = buffer.readLine();
            while(line!=null){
                line = line.substring(1, line.indexOf("\t", 1));
                out.write(line+","); // \r\n即为换行
                out.flush(); // 把缓存区内容压入文件
                line = buffer.readLine();
            }
            out.close(); // 最后记得关闭文件
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public static void doObjectContent(){
        String fileLoc = "C:\\LLLLL\\uft_sfuturedeal.object";
        String resource = "C:\\LLLLL\\raw.txt";
        try{
            File raw = new File(resource);
            File modified = new File(fileLoc);
            InputStreamReader reader = new InputStreamReader(new FileInputStream(raw));
            BufferedReader buffer = new BufferedReader(reader);
            String line = buffer.readLine();
            BufferedWriter out = new BufferedWriter(new OutputStreamWriter(
                    new FileOutputStream(modified, true)));
            while(line!=null){
                out.write("<properties id=\"" + line + "\"/>\r\n"); // \r\n即为换行
                out.flush(); // 把缓存区内容压入文件
                line = buffer.readLine();
            }
            out.close(); // 最后记得关闭文件
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        doSegmentName();
//        doObjectContent();
    }
}

```