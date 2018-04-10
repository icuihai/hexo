title: "Unicode转接码"             #可以随意修改,可以是中文
date: 2017-07-23 18:00:06        #创建的时间,一般不改
categories: blog                 #文章文类
tags: [Utils,Android]                 #文章tag标签,多个标签用逗号隔开
---
## 适用场景（emoji表情转解码等）

## 1.转码成Unicode以及解析

*编码时过滤数字和英文字母*

## 2.转码成UTF-8以及解析

```java
/**
 * description:
 * 1，转码成Unicode以及解析
 * 2，转码成UTF-8以及解析
 * @auther cuihai
 * @since 2017/5/3.
 */

public class Unicode {
    public static String getUTF8XMLString(String xml) {
        // A StringBuffer Object
        StringBuffer sb = new StringBuffer();
        sb.append(xml);
        String xmString = "";
        String xmlUTF8 = "";
        try {
            xmString = new String(sb.toString().getBytes("UTF-8"));
            xmlUTF8 = URLEncoder.encode(xmString, "UTF-8");
            System.out.println("utf-8 编码：" + xmlUTF8);
        } catch (UnsupportedEncodingException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        // return to String Formed
        return xmlUTF8;
    }

    public static String unicode2String(String unicode) {

        StringBuffer string = new StringBuffer();
//        String[] hex = unicode.split("\\\\u");
        String[] hex = unicode.split("\\\\u"); // asdf asdasf  dsgfdsf

        char firstChar = unicode.charAt(0);
        if (firstChar < 127 && firstChar > 0) { // 不是汉字
            int index = hex[0].length();
            string.append(hex[0].substring(0, index));
        }

//        StringBuffer string = new StringBuffer();
////        String[] hex = unicode.split("\\\\u");
//        String[] hex = unicode.split("\\\\u");
        for (int i = 1; i < hex.length; i++) { // asdf  asdk
            String str = hex[i];
            Log.i("TTT", str.toString());
            if (str.length() == 4) {
                // 转换出每一个代码点
                int data = Integer.parseInt(str, 16);
                // 追加成string
                string.append((char) data);
                Log.i("TTT", string.toString());

            } else if (str.length() > 4) {
                String first = str.substring(0, 4);
                String second = str.substring(4, str.length());

                // 转换出每一个代码点
                int data = Integer.parseInt(first, 16);
                // 追加成string
                string.append((char) data);
                string.append(second);
            } else {
                string.append(str);
            }
//            // 转换出每一个代码点
//            int data = Integer.parseInt(hex[i], 16);
//            // 追加成string
//            string.append((char) data);
        }
        return string.toString();
    }

    public static String unicode2UTH8(String str) {
        String chiStr = null;
        try {
            chiStr = URLDecoder.decode(str, "utf-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return chiStr;
    }

    public static String string2Unicode(String string) {
        StringBuffer unicode = new StringBuffer();
        for (int i = 0; i < string.length(); i++) {
            // 取出每一个字符
            char c = string.charAt(i);
            if (c < 127 && c > 0) {
                unicode.append(c);
                Log.i("TAG", unicode.toString());
            } else {
                unicode.append("\\u" + Integer.toHexString(c));
                Log.i("DDD", unicode.toString());
            }
            // 转换为unicode
//            unicode.append("\\u" + Integer.toHexString(c));
        }
        return unicode.toString();
    }
}
```