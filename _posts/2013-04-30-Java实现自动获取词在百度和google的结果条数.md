---
layout: post
title: "Java实现自动获取词在百度和google的结果条数"
description: ""
category: IT在路上
tags: [java,搜索结果获取]
---
前两天由于导师的布置了一项任务，要找8千多个词中的100个流行词，刚开始还以为 直接比较搜索引擎的结果条数就可以了，用程序实现了这个功能，后来才发现这个方法不好用，因为存在了词的歧义性，而且有些词会被拆分，出现的根本就不是那 个词，后来只能人工方法搞掂了，但程序还在，还在贴出来吧，百度和google的都收集起来了。

<pre><code>
package com.cn.words;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.net.MalformedURLException;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import com.gargoylesoftware.htmlunit.FailingHttpStatusCodeException;
import com.gargoylesoftware.htmlunit.WebClient;
import com.gargoylesoftware.htmlunit.html.HtmlDivision;
import com.gargoylesoftware.htmlunit.html.HtmlForm;
import com.gargoylesoftware.htmlunit.html.HtmlInput;
import com.gargoylesoftware.htmlunit.html.HtmlPage;
import com.gargoylesoftware.htmlunit.html.HtmlSpan;
import com.gargoylesoftware.htmlunit.html.HtmlSubmitInput;
import com.gargoylesoftware.htmlunit.html.HtmlTextInput;

import edu.ccnu.nlp.util.RegexUtil;

public class Getwords {
final static Pattern pattern = Pattern.compile("[0-9]");//正则提取搜索结果中的数字

//读取文本的关键词
public String[] readTxt() throws IOException
{
String[] words = new String[9000];
int i = 0;
FileReader fr = new FileReader("H:/mojiaqin.txt");
BufferedReader br = new BufferedReader(fr);
while(br.ready())
{
words[i] = br.readLine();
//System.out.println(br.readLine());
i++;
}
return words;
}
//将搜索结果写到文本中
public void writeTxt(String[] words,String[] times) throws IOException
{
FileWriter fw = new FileWriter("H:/mojiaqin_1.txt");
BufferedWriter bw = new BufferedWriter(fw);
for(int i = 0;i &lt;words.length&amp;&amp;i&lt;times.length;i++)
{
if(words[i]!=null&amp;&amp;times[i]!=null)
{
bw.write(words[i]);
bw.write(",");
bw.write(times[i]);
bw.newLine();
}
else
break;
}
bw.close();
fw.close();
}

//google获取链接
public String LinkGoogle(String word) throws Exception, Exception, Exception
{
final WebClient  webclient = new WebClient();
final HtmlPage htmlpage = webclient.getPage("http://www.google.com.hk/webhp?hl=zh-CN");//中国去网址
webclient.setCssEnabled(false);
webclient.setJavaScriptEnabled(false);
//System.out.println(htmlpage.getTitleText());
final HtmlForm form = htmlpage.getFormByName("f");  //获取表单
//System.out.println(form.asText());
final HtmlSubmitInput button = form.getInputByValue("Google 搜索");//搜索按钮
final HtmlTextInput textField = form.getInputByName("q");//搜索的文本框
textField.setValueAttribute(word);   //设置搜索关键字
final HtmlPage page2 = button.click();   //点击链接
List&lt;?&gt; divs = (List&lt;?&gt;) page2.getByXPath("//div"); //获取搜索结果页面的层
HtmlDivision div =null;
div = (HtmlDivision) divs.get(16);      //获取返回搜索总条数的层
String str = div.asText();
//System.out.println(":"+pickUp(str));
return pickUp(str);
//System.out.println(div.asText());
}

//用百度搜索
public String  LinkBaidu(String word) throws Exception, Exception, Exception
{
final WebClient  webclient = new WebClient();
final HtmlPage htmlpage = webclient.getPage("http://www.baidu.com/");

webclient.setCssEnabled(false);
webclient.setJavaScriptEnabled(false);
//System.out.println(htmlpage.getTitleText());
final HtmlForm form = htmlpage.getFormByName("f");
final HtmlSubmitInput button = form.getInputByValue("百度一下");
final HtmlTextInput textField = form.getInputByName("wd");
textField.setValueAttribute(word);
final HtmlPage page2 = button.click();
//System.out.println(page2.asText());

HtmlSpan span = (HtmlSpan) page2.getByXPath("//span[@class='nums']").get(0);
String str = span.asText();
// pickUp(str);
webclient.closeAllWindows();
return pickUp(str);
}
//提取数字
public static String pickUp(String text) {
Matcher matcher = pattern.matcher(text);
StringBuffer bf = new StringBuffer(64);
while (matcher.find()) {
bf.append(matcher.group()).append("");
}
return bf.toString();
}
public static void main(String[] args) throws IOException
{
Getwords getwords = new Getwords();
String[] words = getwords.readTxt();
String[] times = new String[9000];
String str;
//words.writeTxt(w);
try {
for(int i = 0;i&lt;words.length;i++)
{
if(words[i]!= null)
{
str = getwords.LinkGoogle(words[i]);
times[i] = str;
System.out.println(words[i]+","+str);
}
else
break;
}
System.out.println("搜索完毕！");
getwords.writeTxt(words,times);
} catch (Exception e) {
// TODO Auto-generated catch block
e.printStackTrace();
}
}
}
</code></pre>