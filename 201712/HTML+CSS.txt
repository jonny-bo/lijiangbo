# HTML+CSS

## 描述
- 时间：2017.12.28
- 题目：HTML+CSS

## 内容


HTML
一、简介
  HTML，Hyper Text Markup Language，是一种通过使用标记来描述网页的标记语言，其包含了一整套标记标签。
  HTML+CSS+JavaScript --> HTML
  1.HTML标签
    HTML tag，由“<”和“>”将标签名包裹起来的字符串（“<标签名>”）称为标记标签。
    a.标签是成对出现的。标签分为起始标签和结束标签，即“<a>···</a>”。(单标签除外)
    b.开始标签 --> 开放标签，结束标签 --> 闭合标签
  2.HTML文档
    就是网页，HTML文档描述的就是网页的内容和结构
    Web浏览器就是读取HTML文档，渲染之后呈现给用户的就是网页，只是不会将标签显示出来。
    基本结构
    ##################################################################################################
    <!DOCTYPE html>
    <html>
        <head>
            <title>title</title>
        </head>
        <body>
            <p>This is my First Page</p>
        </body>
    </html>
    
    ##################################################################################################
    <!DOCTYPE>声明
    HTML5
    <!DOCTYPE html>
    <meta>文档描述
    <meta http-equiv="Content-Type" content="text/html" charset="utf-8" />
    1）http-equiv
      HTTP标题信息，主要是告诉浏览器一些用于正确和精确地显示网页内容的信息
      a.Content-Type
        Content：text/html
        Charset：gb2312
      b.Content-Language
        Content:zh-CN
      c.Refresh
        网页多长时间刷新一次，或在多长时间后网页自动链接到其他网页
        Content：时间间隔，单位ms
        Url：如果定义了此项，则在指定的时间后自动链接到此url；如果没有定义，则默认为当前页面
      d.Expire
        网页在缓存中的过期时间，一旦网页过期，必须到服务器上重新调阅。
        Content：GMT时间格式或数字，表示过期时间或时间间隔
     2）name
       页面描述信息
       a.keywords
         为搜索引擎提供关键字列表
         content：关键词列表，关键词之间以英文逗号‘,’隔开
       b.description
         告诉浏览器你的网站的主要内容
         content：网页简述
       c.robots
         告诉搜索机器人哪些页面需要索引，哪些页面不需要索引。
         content：可选值列表{all,none,index,noindex,follow,nofollow}，默认为all
         all，文件将被检索，且页面上的链接可以被查询；
         none，文件将不被检索，且页面上的链接不可以被查询
       d.author
         标注网页的作者或制作组
         content：多个作者之间用英文逗号隔开
       e.copyright
         标注版权
         content：版权声明字符串
       f.generator
         编辑器说明
        
    

    
二、HTML元素
  开始标签和结束标签之间的所有内容，包括空内容。
    
三、HTML属性
  1.HTML中可以编写属性，属性以键值对(KEY-VALUE)的方式书写(大小写不敏感)，比如name="tag1"。
    <a href="www.baidu.com">Click to BaiDu</a>
  2.通用属性
    1）class：设定元素的类名
    2）id：设定元素的唯一id
    3）style：设定元素的行内样式，也叫做内联样式，inline style
    4）title：设定元素的额外信息。
四、HTML标题
  <h1></h1>、<h2></h2>、<h3></h3>、<h4></h4>、<h5></h5>、<h6></h6>，从大到小
  一个文档中，只能有一个<h1>
五、HTML水平线 
  <hr />
  用于分隔内容
六、HTML注释
  使用“<!--”和“-->”将所要注释的内容包裹起来，浏览器对注释的内容不进行解析显示
七、段落
  <p></p>，浏览器会自动在段落的前后添加空行。
  <br />，换行标签。浏览器在显示页面时，会移除源代码中多余的空格和换行，所有连续的空格或空行都会被算作一个空格。
八、文本格式化
  <b></b>,           定义粗体文本
  <big></big>，      定义大号字
  <em></em>，        定义着重文字
  <i></i>，          定义斜体字
  <small></small>，  定义小号字
  <strong></strong>，定义加重语气
  <sub></sub>，      定义下标字
  <ins></ins>，      定义插入子
  <del></del>，      定义删除字
九、区域
  <div></div>，定义文档中节或区域，块元素
  <span></span>，定义文档中的行内小块或区域，内联元素
十、样式
  <style></style>,定义样式
十一、资源
  <link type="text/css" href="filename"></link>
  定义资源引用
  1.type，资源类型
  2.href，资源文件全名（含路径）
十二、链接、锚点
  <a name="tag1" href="url" target="_blank"></a>
  点击可以跳转至另一页面
  1.name，定义锚点名称，页内定位
  2.href，定义链接文档的url
  3.target，定义链接文档的显示位置
十三、图像
  <img src="/images/banner.jpg" alt="images" align="right" width="" height="" />
  空标签，不含有元素
  1.src，定义图像的URL地址 
  2.alt，定义替换文本，浏览器无法载入图片时，将显示替换文本
  3.align，图像的对齐方式，bottom、middle、top、right、left
  4.图像尺寸，width、height定义
十四、表格
  <table border="1" cellpadding="5" cellspacing="10" frame="box" >
      <caption>Title</caption>
      <tbody>
          <tr>
              <th>head-1</th>
              <th>head-2</th>
              <th colspan="2">head-3</th>
              <th>head-4</th>
          </tr>
          <tr>
              <td rowspan="2">col-1</td>
              <td>col-2</td>
              <td>col-3</td>
              <td>col-4</td>
              <td>col-5</td>
          </tr>
          <tr>
              <td>col-2</td>
              <td>col-3</td>
              <td>col-4</td>
              <td>col-5</td>
          </tr>
      </tbody>
  </table>
十五、列表
  <ul>
      <li></li>
      <li></li>
      <li></li>
         ···
  </ul>
  
  <ol>
      <li></li>
      <li></li>
      <li></li>
         ···
  </ol>
  
  <dl>
      <dt></dt>
      <dd></dd>
      <dt></dd>
         ···
  </dl>
十六、表单和输入
  <form action="url" method="get" >
      <input type="text" name=firstname>
      <input type="text" name=secondname>
      <input type="password" name=password>
      <input type="submit" value="submit">
  </form>
  1.文本域
    <input type="text">
  接收文本输入
  2.单选按钮
    <input type="radio" name="sex" value="male">Male
    <br />
    <input type="radio" name="sex" value="female">Female
  3.复选框
    <input type="checkbox" name="bike" />Bike
    <br />
    <input type="checkbox" name="car" />Car    
  4.提交、复位按钮
    <input type="submit" value="Submit">
    <input type="reset" value="Reset">
  5.表单的动作属性（Action）和确认按钮
    当单击提交按钮的时候，web浏览器会默认提交到表单标签属性action所指向的url地址，属性method指定了提交的方法。
  6.下拉列表
    <select name="brand">
        <option value="volvo">Volvo</option>
        <option value="audi" selected="selected">Audi</option>
    </select>
  7.文本域
    多行文本输入
    <textarea rows="5" cols="20">I am liebert</textarea>
  8.控制标签
    <label><input type="radio" name="sex" value="male">Male</label>
    
十七、iframe
  <iframe src="url" frameborder="0" height="" width=""></iframe>
  
  与a标签配合使用，a的超链接页面在指定的iframe中打开
  <iframe name="iframe_a" src="url" frameborder="0"></iframe>
  <a href="url" target="iframe_a"></a>    
  
十八、script脚本
  <script type="text/javascript" src="js/jquery.js"></script>
  
  
  显示结果      描述       实体名称      实体编号 
                空格       &nbsp;        &#160; 
    <           小于号     &lt;          &#60; 
    >           大于号     &gt;          &#62; 
    &           和号       &amp;         &#38; 
    "           引号       &quot;        &#34; 
    '           撇号       &apos; (IE 不支持)  &#39; 
    ￠          分         &cent;        &#162; 
    ￡          镑         &pound;       &#163; 
    ￥          日圆       &yen;         &#165; 
    €           欧元       &euro;        &#8364; 
    §           小节       &sect;        &#167; 
    ?           版权       &copy;        &#169; 
    ?           注册商标   &reg;         &#174; 
    ?           商标       &trade;       &#8482; 
    ×           乘号       &times;       &#215; 
    ÷           除号       &divide;      &#247; 


CSS

一、简介
  Cascading Style Sheets，层叠样式表。设置HTML标签的外观。
二、使用
  1.外部样式表
    <link type="text/css" rel="stylesheet" href="style.css">
  2.内部样式表
    <style type="text/css">
        body{
            color: red;
        }
    </style>
  3.内联样式
    <p style="color: red;"></p>