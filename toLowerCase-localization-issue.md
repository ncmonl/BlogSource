---
title: toLowerCase()转换把'i'转换为'ı'
categories: 技术栈
tags:
  - Android
  - Java
date: 2019-08-22 23:55:47
---
## 问题描述
最近遇到一个Java API 国际化处理中特别案例，看以下代码
```java
System.out.println("info".equals("INFO".toLowerCase());
```
第一反应，当然会输出true了，太简单了！
正常情况下这里没有问题，结果这里居然也遇到了一个坑，在做国际化过程中，在土耳其语环境下，该判断会输出false了。

## 问题原因
为什么呢，这里直接看下toLowerCase()方法的定义：
```java
    /**
     * Converts all of the characters in this {@code String} to lower
     * case using the rules of the default locale. This is equivalent to calling
     * {@code toLowerCase(Locale.getDefault())}.
     * <p>
     * <b>Note:</b> This method is locale sensitive, and may produce unexpected
     * results if used for strings that are intended to be interpreted locale
     * independently.
     * Examples are programming language identifiers, protocol keys, and HTML
     * tags.
     * For instance, {@code "TITLE".toLowerCase()} in a Turkish locale
     * returns {@code "t\u005Cu0131tle"}, where '\u005Cu0131' is the
     * LATIN SMALL LETTER DOTLESS I character.
     * To obtain correct results for locale insensitive strings, use
     * {@code toLowerCase(Locale.ROOT)}.
     * <p>
     * @return  the {@code String}, converted to lowercase.
     * @see     java.lang.String#toLowerCase(Locale)
     */
    public String toLowerCase() {
        return toLowerCase(Locale.getDefault());
    }
```
看到这里实际是一个重写方法实际上是传入了Locale.getDefault()的参数，经测试和验证发现，Android/Java 
在默认 Locale 被设置成土耳其语的情况下，使用 toLowerCase会导致“I”变成“ı”注释中有也有相关说明。

## 解决方式

解决这个问题也简单，就是如果需要国际化，使用toLowerCase()的时候直接使用重载方法，toLowerCase(Locale.CHINA)使该方法不受语言环境影响。
