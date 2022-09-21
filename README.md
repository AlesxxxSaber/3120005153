# PaperCheck
- 一个简单的 Java 论文查重程序,
- 使用的是 SimHahsh + 海明距离 算法

| 这个作业属于哪个课程 | [广工软件工程课程学习社区](https://bbs.csdn.net/forums/gdut-ryuezh) |
| ------ | ------ |
| 这个作业要求在哪里 | [个人项目作业-论文查重](https://bbs.csdn.net/topics/608092799) |
| Github地址 |  |

[TOC]

### 一、仓库地址
[Github地址](https://github.com/AlesxxxSaber/3120005153)
### 二、PSP表
| PSP2.1 | 	Personal Software Process Stages | 预估耗时（分钟） | 实际耗时（分钟） |
| ------ | ------ | ------ | ------ |
| Planning | 计划 | 20 | 20 |
| · Estimate | 	· 估计这个任务需要多少时间 | 10 | 10 |
| Development | 	开发 | 120 | 600 |
| · Analysis | · 需求分析 (包括学习新技术) | 40 | 30 |
| · Design Spec | 	· 生成设计文档 | 20 | 30 |
| · Design Review | · 设计复审 | 30 | 20 |
| · Coding Standard | · 代码规范 (为目前的开发制定合适的规范) | 50 | 60 |
| · Design | · 具体设计 | 150 | 120 |
| · Coding | · 具体编码 | 120 | 150 |
| · Code Review | · 代码复审 | 40 | 30 |
| · Test | · 测试（自我测试，修改代码，提交修改）  |120 | 90 |
| Reporting | 报告 |90 | 120 |
| · Test Repor | · 测试报告 | 30 | 40 |
| · Size Measurement | · 计算工作量 | 20 | 15 |
| · Postmortem & Process Improvement Plan | 	· 事后总结, 并提出过程改进计划 | 60 | 55 |
|   | · 合计 | 920 | 1490 |

### 三、需求分析
#### 3.1需求
题目：论文查重

描述如下：

设计一个论文查重算法，给出一个原文文件和一个在这份原文上经过了增删改的抄袭版论文的文件，在答案文件中输出其重复率。

原文示例：今天是星期天，天气晴，今天晚上我要去看电影。
抄袭版示例：今天是周天，天气晴朗，我晚上要去看电影。
要求输入输出采用文件输入输出，规范如下：

从命令行参数给出：论文原文的文件的绝对路径。
从命令行参数给出：抄袭版论文的文件的绝对路径。
从命令行参数给出：输出的答案文件的绝对路径。
我们提供一份样例，使用方法是：orig.txt是原文，其他orig_add.txt等均为抄袭版论文。

注意：答案文件中输出的答案为浮点型，精确到小数点后两位
#### 3.2需求分析
根据题目要求，对比论文原文和抄袭版论文并检测文本相似度，输出相似度。
流程图：
![img](https://img-community.csdnimg.cn/images/628b19c6ff5542a1a80ab300866d1301.png "#left")

### 四、设计与实现
#### 4.1读写txt文件的模块
类： TxtlOUtils 
包含两个静态方法：
1、readTxt：读取txt文件
2、writeTxt：写入txt文件
实现：都是调用 Java.io 包提供的接口

#### 4.2 SimHash模块（核心模块）
类：SimHashUtils

包含了两个静态方法：
1、getHash：传入String，计算出它的hash值，并以字符串形式输出，（使用了MD5获得hash值）
2、getSimHash：传入String，计算出它的simHash值，并以字符串形式输出，（需要调用 getHash 方法）
getSimHash 是核心算法，主要流程如下：
1、分词（使用了外部依赖 hankcs 包提供的接口）
```java
List<String> keywordList = HanLP.extractKeyword(str, str.length());//取出所有关键词
```
2、获取hash值

```java
String keywordHash = getHash(keyword);
           if (keywordHash.length() < 128) {
               // hash值可能少于128位，在低位以0补齐
               int dif = 128 - keywordHash.length();
               for (int j = 0; j < dif; j++) {
                   keywordHash += "0";
               }
           }
```
3、加权合并

```java
for (int j = 0; j < v.length; j++) {
             // 对keywordHash的每一位与'1'进行比较
             if (keywordHash.charAt(j) == '1') {
                 //权重分10级，由词频从高到低，取权重10~0
                 v[j] += (10 - (i / (size / 10)));
             } else {
                 v[j] -= (10 - (i / (size / 10)));
             }
         }
```
4、降维
```java
String simHash = "";// 储存返回的simHash值
        for (int j = 0; j < v.length; j++) {
            // 从高位遍历到低位
            if (v[j] <= 0) {
                simHash += "0";
            } else {
                simHash += "1";
            }
        }
```
#### 4.3 海明距离模块
类：HammingUtils
包含了两个静态方法：
1、getHammingDistance：输入两个 simHash 值，计算出它们的海明距离 distance
```
for (int i = 0; i < simHash1.length(); i++) {
                // 每一位进行比较
                if (simHash1.charAt(i) != simHash2.charAt(i)) {
                    distance++;
                }
            }
```
2、getSimilarity：输入两个 simHash 值，调用 getHammingDistance 方法得出海明距离 distance，在由 distance 计算出相似度
```
return 0.01 * (100 - distance * 100 / 128);
```
#### 4.4 main主模块
main 方法的主要流程：
从命令行输入的路径名读取对应的文件，将文件的内容转化为对应的字符串
由字符串得出对应的 simHash值
由 simHash值求出相似度
把相似度写入最后的结果文件中
退出程序
### 五、接口部分的性能改进
#### 5.1 性能分析
Overview

![img](https://img-community.csdnimg.cn/images/aaa958655ac2470d865f07dac197af2f.png "#left")

![img](https://img-community.csdnimg.cn/images/b338cd408a1a4151a70678363210fca9.png "#left")
从分析图可以看到：

调用次数最多的是com.hankcs.hanlp包提供的接口， 即分词、取关键词与计算词频花费了最多的时间。

所以在性能上基本没有什么需要改进的。
### 六、单元测试
#### 6.1读写txt文件的模块的测试
基本思路：
1、测试正常读取
2、测试正常写入
3、测试错误读取
4、测试错误写入

```
public class TxtIOUtilsTest {
    @Test
    public void readTxtTest() {
        // 路径存在，正常读取
        String str = TxtIOUtils.readTxt("D:/test/orig.txt");
        String[] strings = str.split(" ");
        for (String string : strings) {
            System.out.println(string);
        }
    }
    @Test
    public void writeTxtTest() {
        // 路径存在，正常写入
        double[] elem = {0.11, 0.22, 0.33, 0.44, 0.55};
        for (int i = 0; i < elem.length; i++) {
            TxtIOUtils.writeTxt(elem[i], "D:/test/ans.txt");
        }
    }
    @Test
    public void readTxtFailTest() {
        // 路径不存在，读取失败
        String str = TxtIOUtils.readTxt("D:/test/none.txt");
    }
    @Test
    public void writeTxtFailTest() {
        // 路径错误，写入失败
        double[] elem = {0.11, 0.22, 0.33, 0.44, 0.55};
        for (int i = 0; i < elem.length; i++) {
            TxtIOUtils.writeTxt(elem[i], "User:/test/ans.txt");
        }
    }
}

```
测试结果
![img](https://img-community.csdnimg.cn/images/5b37f3657fb4453a8b4d322672b28baa.png "#left")
代码覆盖率
![img](https://img-community.csdnimg.cn/images/5d19c17289d0420b90bbfad3189d6cdc.png "#left")
#### 6.2 SimHash模块的测试

```
public class SimHashUtilsTest {
    @Test
    public void getHashTest(){
        String[] strings = {"余华", "是", "一位", "真正", "的", "作家"};
        for (String string : strings) {
            String stringHash = SimHashUtils.getHash(string);
            System.out.println(stringHash.length());
            System.out.println(stringHash);
        }
    }
    @Test
    public void getSimHashTest(){
        String str0 = TxtIOUtils.readTxt("D:/test/orig.txt");
        String str1 = TxtIOUtils.readTxt("D:/test/orig_0.8_add.txt");
        System.out.println(SimHashUtils.getSimHash(str0));
        System.out.println(SimHashUtils.getSimHash(str1));
    }
}
```
测试结果
![img](https://img-community.csdnimg.cn/images/3575aa8c270e4224978f084460cbd2af.png "#left")
代码覆盖率

![img](https://img-community.csdnimg.cn/images/0b877744d7fa4a238aea2788b43c4832.png "#left")
#### 6.4海明距离模块的测试

```
public class HammingUtilsTest {
    @Test
    public void getHammingDistanceTest() {
        String str0 = TxtIOUtils.readTxt("D:/test/orig.txt");
        String str1 = TxtIOUtils.readTxt("D:/test/orig_0.8_add.txt");
        int distance = HammingUtils.getHammingDistance(SimHashUtils.getSimHash(str0), SimHashUtils.getSimHash(str1));
        System.out.println("海明距离：" + distance);
        System.out.println("相似度: " + (100 - distance * 100 / 128) + "%");
    }
}

```
测试结果
![img](https://img-community.csdnimg.cn/images/3f2256b7127748909fdcf934ed9c58bc.png "#left")
代码覆盖率
![img](https://img-community.csdnimg.cn/images/9a343c9264e245f4b156f7422cf713cb.png "#left")

### 七、异常处理
#### 7.1 设计与实现
当文本长度太短时，HanLp无法取得关键字，需要抛出异常。
```
        try{
            if(str.length() < 200) throw new ShortStringException("文本过短！");
        }catch (ShortStringException e){
            e.printStackTrace();
            return null;
        }



```
实现了一个处理这个异常的类：ShortStringException（继承了Exception）
```
public ShortStringException(String message) {
        super(message);
    }



```
测试结果
![img](https://img-community.csdnimg.cn/images/c151523ef4fd42eb8e1729858b40081e.png "#left")
