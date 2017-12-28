# PHP要点

## 描述
- 时间：2017.12.28
- 题目：PHP要点

## 内容

1.发展历程
  PHP:Personal Home Page -> PHP:Hypertext Preprocessor。1994年由Rasmus Lerdorf创建，起初主要是为维护个人网页制作的工具（用Per语言编写的程序）。处理网页和数据库信息的交互。后来，用C语言重新编写，将程序和表单直译器整合起来，成为PHP/FI。PHP/FI可以和数据库连接，产生简单的动态网站程序。1995年发布PHP 1.0（Perl），随着用户提出的一些要求和意见，比如循环控制、数组变量等，1995年6月8日发布PHP 2.0（C编写的PHP/FI），支持MySQL数据库。1997年两个以色列工程师重写了PHP解析器，更名为PHP:Hypertext Preprocessor，1997年11月发布了PHP/FI 2.0，1998年6月正式发布PHP 3.0。之后，改写PHP核心（软件开发的重构、相当于工作总结，一般大的公司或老的工程师都会做这项工作），199年发布Zend Engine，并且在以色列成立PHP Tec.管理PHP开发。2000年5月22日，以Zend Engine 1.0为核心的PHP 4.0发布。2004年7月13日发布PHP 5.0（面向对象特性、引入PDO数据库存取对象）。2013年6月20日，发布PHP5.5.0。2014年10月16日，发布PHP5.6.0。2015年10月，发布PHP 7.0。
2.特点
  运行速度相比其它语言比较快；
  负载能力好；
  入门简单，但成为高手很难；
  php是当前web服务器端比较流行的语言；
3.功能特性
  语法融合了c、java、perl、和php自己语法，执行网页速度比cgi、perl、asp要快。
  具有很好的开放性和可扩展型。自由软件，源码全部公开，大家都能为其增砖添瓦。
  数据库支持。如：dba、dbase、infomix、msql、mysql、sql server、sybase、postgresql、oracle
  面向对象编程。提供类和对象，引入对象重载、引用技术等。
  功能丰富。对象式设计、结构化特性、数据库处理、网络接口、安全编码等。
4 Apache与PHP
             ________           ________
            |        |         |        |
     +------| Module |---------| Module |--------+
     |      |________|         |________|        |
     |                                     ______|_____
     |              Apache HTTPD          |            |  
     |                                    | PHP Module |
     |                                    |____________|
     |                              ______       |
     +--------------|--------------| MPM  |------+
                   _|______________/------   
                  |                |   |
                  |      APR       |   |
                  |________________|   |
               ____________|___________|_
              |                          | 
              |           OS             |          
              |__________________________|

  PHP作为Apache的一个模块，在Apache运行时加载到内存中。
  配置文件：http.conf、httpd-php-sapi53.conf
  以Windows环境下为例：
  ##############################################################################################

  LoadFile "C:/phpStudy/php53/php5ts.dll"
  LoadModule php5_module "C:/phpStudy/php53/php5apache2_4.dll"

  <IfModule php5_module>
      PHPIniDir "C:/phpStudy/php53/"
  </IfModule>

  <FilesMatch "\.php$">
      SetHandler application/x-httpd-php
  </FilesMatch>

  ##############################################################################################
  
  Apache请求处理流程：
  收到Request --> Post-Read-Request --> URI Translation --> Header Parsing --> Access Control --> Authentication --> Authorization --> MIME type checking --> FixUP --> Response --> Logging --> Clean up --> Wait --> 收到Request --> ......
  Post-Read-Request：
  URI Translation：将请求的URL映射到本地文件系统。mod_alias
  Header Parsing：检查请求的头部。mod_setenvif
  Access Control：根据配置文件检查是否允许访问请求的资源。mod_auth_host
  Authentication：按照配置文件设定的策略对用户进行认证，并设定用户名区域。
  Authorization：根据配置文件检查是否允许用户认证过的用户执行请求的操作。
  MIME Type Checking：根据请求资源的的MIME类型的相关规则，判定将要使用的内容处理函数mod_negotiation、mod_mime
  FixUp：在内容生成器之前的一个钩子
  Response：生成客户端的内容，负责给客户端发送一个恰当的回复。
  Logging：记录事务
  Cleanup：清理本次请求事务处理完成之后的遗留环境，比如文件、目录的处理或者Socket关闭等。请求处理的最后阶段。

  PHP组成：内核、Zend引擎、扩展。内核主要处理请求、文件流、错误处理等等；Zend引擎将源文件转换成机器语言，然后运行，产生的即如果返回给PHP内核，有内核通过SAPI传送给服务器，并最终发送给浏览器；扩展是一组函数、类库和流，比如数据库操作mysqli，解压缩操作zip等。

5.SAPI：Server Application Programming Interface，服务器应用编程接口，服务器和其他应用程序交互接口。常见的SAPI：cgi、fast-cgi、cli、isapi、apache模块的dll。
  cgi：Common Gateway Interface 通用公共网关接口，就是一段将WEB服务器和其他应用程序（数据库）连接起来程序。工作方式：接收到用户请求时，创建一个cgi子进程，激活cgi进程，处理请求（解析php.ini、重现载入全部扩展并初始化数据结构），处理完后结束这个子进程。也就是所谓的“fork and execute”模式。有多少个连接请求，就会有多少个cgi子进程，子进程反复加载造成cgi进程性能低下。当用户请求数量过多时，将会严重占用系统资源，导致服务器性能下降。
  fast-cgi：cgi的升级版本，好比一个常驻的cgi，加载一次。工作流程：web server启动时载入fastcgi管理器 -> fastcgi自身初始化，启动多个解释器进程，等待web server的连接 -> 客户端请求到达，fastcgi进程管理器选择连接到一个cgi解释器，web server将cgi环境变量和标准输入发送到fastcgi子进程php-cgi -> fastcgi子进程完成处理后将标准输出和错误信息从同一连接返回给web server -> fastcgi子进程关闭连接，请求处理完成 -> fastcgi子进程等待处理下一个连接。
  apache handler：php作为apache模块，apache server在系统启动后预先生成多个进程副本驻留在内存中。

6.PHP脚本执行流程
  开始 --> Scanning（将PHP代码转换成代码片段，词法分析，建立符号表） --> Parsing（将代码片段转换成表达式，语法分析，建立表达式生成树） --> Compilation（将表达式转换成操作码） --> Excution（执行操作码）
  
7.需要掌握的知识
  PHP语法，熟悉常用类库
  面向对象编程思想
  常用设计模式
  数据库知识，常用MySQL
  编程工具：PHP Storm
  框架：Symfony
  
8.PHP文本
  ##############################################################################################
  <?php echo'<p>welcome!</p>'; ?>
  ----------------------------------------------------------------------------------------------
  <?php
  
  
  echo'<p>welcome!</p>';
  
  
  
  ?>
  ----------------------------------------------------------------------------------------------
  <html>
      <head>
          <title>PHP 测试</title>
      </head>
      <body>
         <?php echo '<p>Hello World</p>'; ?> <!-- 内嵌php代码-->
      </body>
  </html> 
  
  ##############################################################################################
  
9.基本语法：
  1）PHP标记
     起始标记 <?php 、结束标记 ?>，告诉PHP开始和结束解析代码。也可以使用开始标记<? 和 结束标记?>
     如果文件内容是纯PHP代码，建议在文件末尾删除PHP结束标记，避免PHP结束标记之后意外输入空格或换行，导致PHP输出这些空白，如下所示
     ##############################################################################################
     <?php  
  
         echo'<p>welcome!</p>';
  
  
  
     ##############################################################################################
  2）指令分隔符
     用于将多条语句分割开来，在每条语句结束之后添加 ‘;’
  3）注释
     用于标注解释代码的意图，不会被解释器解释执行。单行注释‘//’，多行注释/*  中间加入注释文字   */
10.变量基础
  PHP中的变量用'$[变量名]'表示，变量名区分大小写。
  命名规则：一个有效的变量名由字母或者下划线开头，后面跟上任意数量的字母，数字，或者下划线。正则表示：[a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*
  $this：特殊变量，不能被赋值。
  传值赋值：$var = 12;      单独分配一块存储空间，用于存储数据。
  引用赋值：$var1 = &$var;  指向$var的存储空间。
  初始化：PHP中不需要初始化变量，但初始化变量是一个好习惯。isset()可以检测变量是否已经初始化
      
11.类型
  支持8种数据类型，包括：四种标量类型（boolean、integer、float、string），两种复合类型（array、object），两种特殊类型（resource、NULL）
  伪类型，主要用于代码的助记和可读性，包括：mixed、number、callback
  1）boolean
     取值范围：TRUE或FALSE。不区分大小。占用内存大小：1B
     $flag = true; $flag = TRUE; $flag = True;
     默认值：false
  2）integer
     取值范围：-2^31 ~ 2^31-1。整型，整数。占用内存大小：4B
     默认值：0
  3）float（double）
     取值范围：
     默认值：0.0
  4）string
     取值范围：0 ~ 2GB
     默认值：空字符
     单引号''，字符串中变量和特殊字符不会被转义
     $str = 'i am liebert';
     双引号""，字符串中变量和特殊字符会被转义
     $age = 12;
     $str = 'i am liebert \r\n i am $age';
  5）array
     
     
     
  6）object
  
  
12.变量进阶：预定义变量 --> PHP预置的变量
  
  $GLOBALS                ：一个包含了全部变量的全局组合数组。变量的名字就是数组的键。
  $_SERVER                ：个包含了诸如头信息(header)、路径(path)、以及脚本位置(script locations)等等信息的数组。这个数组中的项目由 Web 服务器创建。
  $_GET                   ：通过 URL 参数传递给当前脚本的变量的数组。
  $_POST                  ：通过 HTTP POST 方法传递给当前脚本的变量的数组。 
  $_FILES                 ：通过 HTTP POST 方式上传到当前脚本的项目的数组。
  $_REQUEST               ：默认情况下包含了 $_GET，$_POST 和 $_COOKIE 的数组。
  $_SESSION               ：当前脚本可用 SESSION 变量的数组。
  $_ENV                   ：通过环境方式传递给当前脚本的变量的数组。
  $_COOKIE                ：通过 HTTP Cookies 方式传递给当前脚本的变量的数组。 
  $php_errormsg           ：包含由 PHP 生成的最新错误信息。这个变量只在错误发生的作用域内可用，并且要求 track_errors 配置项是开启的（默认是关闭的）。
  $HTTP_RAW_POST_DATA     ：原生POST数据。
  $http_response_header   ：HTTP 响应头。
  $argc                   ：传递给脚本的参数数目。
  $argv                   ：传递给脚本的参数数组。
  
13.常量
  常量只能是标量。
  常量只能用define([常量名],[常量值])定义，5.3版本以，可以使用const定义，示例：const PRE = 123; 。
  常量一旦定义就不能被中心定义或者取消定义。
  魔术常量
  __LINE__      ：文件中的当前行号。  
  __FILE__      ：文件的完整路径和文件名。如果用在被包含文件中，则返回被包含的文件名。自 PHP 4.0.2 起，__FILE__ 总是包含一个绝对路径（如果是符号连接，则是解析后的绝对路径），而在此之前的版本有时会包含一个相对路径。  
  __DIR__       ：文件所在的目录。如果用在被包括文件中，则返回被包括的文件所在的目录。它等价于 dirname(__FILE__)。除非是根目录，否则目录中名不包括末尾的斜杠。（PHP 5.3.0中新增） =  
  __FUNCTION__  ：函数名称（PHP 4.3.0 新加）。自 PHP 5 起本常量返回该函数被定义时的名字（区分大小写）。在 PHP 4 中该值总是小写字母的。  
  __CLASS__     ：类的名称（PHP 4.3.0 新加）。自 PHP 5 起本常量返回该类被定义时的名字（区分大小写）。在 PHP 4 中该值总是小写字母的。类名包括其被声明的作用区域（例如 Foo\Bar）。注意自 PHP 5.4 起 __CLASS__ 对 trait 也起作用。当用在 trait 方法中时，__CLASS__ 是调用 trait 方法的类的名字。  
  __TRAIT__     ：Trait 的名字（PHP 5.4.0 新加）。自 PHP 5.4 起此常量返回 trait 被定义时的名字（区分大小写）。Trait 名包括其被声明的作用区域（例如 Foo\Bar）。  
  __METHOD__    ：类的方法名（PHP 5.0.0 新加）。返回该方法被定义时的名字（区分大小写）。  
  __NAMESPACE__ ：当前命名空间的名称（区分大小写）。此常量是在编译时定义的（PHP 5.3.0 新增）。 
14.运算符
  对数据进行操作的符号
  1）赋值运算符
     '='   ：将运算符右边的值赋给左边的变量，$a = 12;
     '=>'  ：为数组的键赋值，基本形式为key => value，$arr = array(
                                                                  "name"  => "liebert",
                                                                  "age"   => "24",
                                                                  "sex"   => "male",
                                                                  "party" => "PLA"
                                                                  );
  2）算数运算符
     +、-、*、/ 、%       $a + $b、$a - $b、$a * $b、$a / $b、$a % $b
     除法运算总是返回浮点数;
     取模运算符的操作数在运算之前都会转换成整数（除去小数部分）,取模运算符 % 的结果和被除数的符号（正负号）相同。
  3）位运算符
     & ：按位与
     | ：按位或
     ^ ：按位异或
     ~ ：按位取反
     <<：左移
     >>：右移
  4）比较运算符
     ==   ：等于，类型转换后值相等为TRUE，否则为FALSE
     ===  ：全等于，类型和值均相同为TRUE，否则为FALSE
     !=   ：不等，类型转换后值不相等为TRUE，否则为FALSE
     <>   ：不等，和!=相同
     !==  ：不全等，类型或者值不相同为TRUE，否则为FALSE
     <    ：小于，严格小于为TRUE，否则为FALSE
     >    ：大于，严格大于为TRUE，否则为FALSE
     <=   ：小于等于，严格小于等于为TRUE，否则为FALSE
     >=   ：大于等于，严格大于等于为TRUE，否则为FALSE
     示例
     ############################################################################################## 
     <?php
         var_dump(0 == '0');        //true，数字和字符串比较或者设计到数字内容的字符串，字符串会被转化为字符所表示的数值。
         var_dump('1' == '01');     //true
         var_dump(0 === '0');       //false
     ##############################################################################################
  5）错误控制运算符
     @ ：将其放置在表达式之前，该表达式可能产生的任何错误信息将会被忽略 @$a+$b;
  6）执行运算符
     ``,反引号包括起来，括号中的内容作为外壳命令来执行，并将外壳输出的信息返回。
     ############################################################################################## 
     <?php
         $var = `dir`;//windows平台shell外壳命令
         echo "$var";
     ############################################################################################## 
  7）递增/递减运算符
     ++：变量加1。
     --：变量减1。
     前递增、前递减
     ++$a、--$a：先对$a进行加1或减1，然后返回$a的值。
     $a++、$a--：先返回$a的值，然后对$a进行加1或减1。
     ############################################################################################## 
     <?php
         $a = 5;
         echo "$a++\r\n";
         echo "++$a\r\n";
     ############################################################################################## 
  8）逻辑运算符
     and  ：逻辑与
     or   ：逻辑或
     xor  ：逻辑异或
     !    ：逻辑非
     ||   ：逻辑或
  9）字符串运算符
     .: 字符串连接字符,将两个字符串拼接起来
     ############################################################################################## 
     <?php
         $str    = "";
         $str1   = "my name is";
         $str2   = "liebert";
         $str    = $str1.$str2;
         echo $str."\r\n";
         echo $str1.$str2."\r\n";
         echo $str1."liebert\r\n";
         echo "$str1"."$str2"."\r\n";
         echo '$str1'.'$str2'."\r\n";
     ############################################################################################## 
  10）类型运算符
     instanceof：确定php变量是否属于某一类的实例，是返回TRUE,否返回FALSE
     ##############################################################################################
     <?php
         class myclass{
         }
         class myclass2{
         }
         $cls = new myclass();
         
         var_dump($cls instanceof myclass);
         var_dump($cls instanceof myclass2);
     ##############################################################################################
  11）表达式
     变量与运算符的组合就是表达式
  12）运算符优先级
     下面按照优先级从高到低的顺序列出了运算符
      无 clone new                                                   clone 和 new 
      左 [                                                           array() 
      右 ++ -- ~ (int) (float) (string) (array) (object) (bool) @    类型和递增／递减  
      无 instanceof                                                  类型  
      右 !                                                           逻辑运算符  
      左 * / %                                                       算术运算符  
      左 + - .                                                       算术运算符和字符串运算符 
      左 << >>                                                       位运算符  
      无 == != === !== <>                                            比较运算符  
      左 &                                                           位运算符和引用 
      左 ^                                                           位运算符  
      左 |                                                           位运算符  
      左 &&                                                          逻辑运算符  
      左 ||                                                          逻辑运算符  
      左 ? :                                                         三元运算符  
      右 = += -= *= /= .= %= &= |= ^= <<= >>= =>                     赋值运算符  
      左 and                                                         逻辑运算符  
      左 xor                                                         逻辑运算符  
      左 or                                                          逻辑运算符  
      左 ,                                                           多处用到 
     ##############################################################################################
     <?php
         $a = 0;
         $b = 1;
         $c = 0;
         if( $c = $a++ == 0 || --$b == 0){
             echo"<div>first</div><br />";
         }else{
             echo"<div>second</div><br />";
         }
         echo"<div>a:$a</div><br /><div>b:$b</div><br /><div>c:$c</div>";
     ##############################################################################################
15.流程控制
  1）简介
  语句由表达式组成，通常以分号结束。语句可以使用{语句1;语句2;...}封装成为语句组。
  对语句执行顺序的控制称为流程控制
  语句类型：条件语句、循环语句、开关语句、跳转语句
  2）条件语句
     先判断表达式的值，再执行语句
     if(expr) statement1;     
     ----------------------------------------------------------------------------------------------
     if(expr) statement1;
     else     statement2;
     ----------------------------------------------------------------------------------------------
     if(expr){
         statement1;
         statement2;
     }
     ----------------------------------------------------------------------------------------------
     if(expr){
         statement1;
         statement2;
     }else{
         statement3;
         statement4;
     }
     ----------------------------------------------------------------------------------------------
     if(expr){
         statement1;
         statement2;
     }elseif{
         statement3;
         statement4;
     }else{
         statement5;
         statement6;
     }
     ----------------------------------------------------------------------------------------------
     if(expr){
         statement1;
         statement2;
     }else if{
         statement3;
         statement4;
     }else{
         statement5;
         statement6;
     } 
  
  3）循环语句
     a）for
     for(expr1; expr2; expr3) statement1;
     ----------------------------------------------------------------------------------------------
     for(expr1; expr2; expr3){
         statement1;
     }
     expr1：每次循环开始前执行
     expr2：每次循环开始前进行判断
     expr3：每次循环最后执行
     
     
     b）foreach
     仅用于遍历数组和对象
     foreach (array_expression as $value)
         statement1;
         
     foreach (array_expression as $key => $value)
         statement1;

     
     c）while
     先判断表达式的值，再执行语句
     ----------------------------------------------------------------------------------------------
     while(expr) statement1;
     ---------------------------------------------------------------------------------------------- 
     while(expr) {
         statement1;
         statement2;
     }     
     ---------------------------------------------------------------------------------------------- 
     while(expr):
         statement1;
         statement2;
     endwhile;
     ----------------------------------------------------------------------------------------------
     d）
     先执行语句，再判断表达式的值
     do{
         statement1;
     }while(expr);
     
     e）continue
       
     
      
  4）开关语句
     switch ($i) {
        case 0:
            echo "i equals 0";
            break;//如果没有break，那就继续往下执行
        case 1:
            echo "i equals 1";
            break;
        case 2:
            echo "i equals 2";
            break;
     }
  
  
  5）跳转语句
     跳转到目标位置标记处。目标位置也有一定的限制，只能位于同一个文件和作用域，也就是说无法跳出一个函数或类方法，以及任何循环或者switch结构中。可以跳出循环或者switch
     goto label;
     
  6）其他特殊控制语句
     include        ：包含并运行指定文件。
     inculde_once   ：包含并运行指定文件。检查文件是否已经被包含进来，如果有则不进行重复加载。
     require        ：包含并运行指定文件，出错时产生错误，终止脚本执行
     require_once   ：包含并运行指定文件，出错时产生错误，终止脚本执行。检查文件是否已经被包含进来，如果有则不进行重复加载。
16.函数
  1）理解
     什么是函数？函数就是事物之间的一种映射关系。y = f(x);
     函数组成：输入、输出、映射关系（对数据的加工和操作等处理操作，编程里边就是通过指令来进行处理）
  2）定义函数
     function 函数名（参数1,参数2,...）{
         函数体（语句组）
         
         return 返回值;
     }
     函数调用：函数名();
     ##############################################################################################
     <?php
         function add($a,$b){
           var $result = 0;
           if(is_integer($a) && is_integer($b)){
               $result = $a + $b;
           }else{
               $result = NULL;
           }
           return $result;//如果省略此句，默认返回为NULL
         }
         
     ##############################################################################################
  3）参数
      值传递
      引用传递
      
  4）可变函数
     var $func = NULL;
     function func1(){
         echo'this is func1';
     }
     function func2(){
         echo'this is func2';
     }
     $func = 'func1';
     func();
     $func = 'func2';
     func();
  5）匿名函数
     闭包函数，临时创建一个没有指定名称的函数。作用域
17.类与对象
   1）结构化编程
      以功能模块和处理过程设计为主的程序设计原则，也叫做函数式编程。
      三种基本结构：顺序结构、选择结构、循环结构
      常用的设计方法：程序流图
      将一个复杂的问题分解为多个基本子问题，这些基本问题是常见的、一般化的问题，通过对这些子问题的求解进而解决最终问题。
      在程序设计早期，都是按照程序顺序执行步骤来编写程序的，后来发现这种情况产生了的大量的重复代码，为了解决这一问题，就出现了结构化程序设计。主要是将大量的功能完善的代码封装为一个个功能函数
      优点：整体思路清晰、目标明确
      缺点：难以在系统分析阶段准备定义。
      ##############################################################################################
      电子温度计
      检测硬件：传感器、显示器
      读取数据：向传感器发送请求、传感器返回数据
      处理数据：多次测量求平均值
      显示数据：将最终计算的温度值显示到显示器上
      
      早期程序设计：
      start:
      host send ask to sensor_temp;
      wait for the ack of sensor_temp;
      sensor_temp send ack to host;
      sensor_temp work well;
      
      host send ask to displayer;
      wait for the ack  of displayer;
      displayer send ack to host;
      displayer work well;
      
      send ask_data to sensor_temp;
      sensor_temp send data_temp to host;
      host save the data_1;
      
      send ask_data to sensor_temp;
      sensor_temp send data_temp to host;
      host save the data_2;
      
      send ask_data to sensor_temp;
      sensor_temp send data_temp to host;
      host save the data_3;
      
      send ask_data to sensor_temp;
      sensor_temp send data_temp to host;
      host save the data_4;
      
      send ask_data to sensor_temp;
      sensor_temp send data_temp to host;
      host save the data_5;
      
      host calculate the average;
      
      host send final data_temp to displayer;
      displayer output data_temp;
      
      goto start;
      
      结构化程序设计
      checkHardware(){
          host send ask to sensor_temp;
          wait for the ack of sensor_temp;
          sensor_temp send ack to host;
          sensor_temp work well;
          
          host send ask to displayer;
          wait for the ack  of displayer;
          displayer send ack to host;
          displayer work well;
      }
      getData(){
        send ask_data to sensor_temp;
        sensor_temp send data_temp to host;
        host save the data;
      }
      calData(){
          host calculate the average;
      }
      displayData(){
        host send final data_temp to displayer;
        displayer output data_temp;
      }
      int MAX;
      int count;
      float result,data_buffer[10];
      
      for(;;){
        if(checkHardware() == "ERROR"){
            displayData("ERROR HARDWARE!");
        }
        for(count = 0; count < MAX; count++){
            data_buffer[count] = getData();
        }
        result = calData(data_buffer);
        displayData(result);
      }
      ##############################################################################################
   2）面向对象编程
      以对象为中心的编程思想。结构化设计中以函数为主。
      程序：数据+算法
      函数要实现意义就必须对数据进行操作，但函数并不是对全部数据进行操作，也只是对具有特定意义的数据进行操作。
      函数关心的是底层功能实现的细节，却忽略了函数是对特定数据进行加工处理的。
      编程问题来源现实世界，程序最终要回归于现实世界，解决现实问题，不仅要有骨架，还有有血有肉。
      之前，咱们编程时将现实世界的数据和操作剥离开来对待，现在将数据和操作统一起来作为一个整体对象来对待的方法就是面向对象设计，程序设计者的关注点就从底层细节设计转移到上层的对象设计，层次提高了。
      什么是对象，万物皆对象。什么是类，万物之共性（人的意识领域）。类就是对象的共性部分的抽象描述，对象就是具象后的类（也就是实例化的类，即按照此类的的结构分配了独立的内存，这块内存就是一个对象）。
      类的组成：属性（特定意义的数据）和行为（特定意义的操作）。
      
      复用：
      稳定：需求更改后，代码接口更改量小
   3）类的定义
      class 类名{
          类的属性；
          类的方法；
      }
      命名规则：一个合法类名以字母或下划线开头，后面跟着若干字母，数字或下划线，正则表示：[a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*。
      类的属性：常量、变量。
      类的方法：函数                   其他编程语言中也分为过程和函数，统一称为行为
      
      ##############################################################################################
      class myClass{
          public $name = "myClass";
          
          public function getName(){
              return $this->name;
          }
      }
      $me = new myClass();
      echo $me->getName();
      ##############################################################################################
      接口：一种类的规约或约束，不包含具体实现。用interface关键字定义。主要用于定义多组类之间的调用接口规范。
      多态：接口的多种不同的实现方式即为多态。相同的调用规范或约束，但是具体的执行过程不一样，功能不一样。统一操作作用于不同对象，可以有不同的解释，产生不同的执行结果。
      ##############################################################################################
      interface ITool{
          function travel();
      }

      class TravelByCar implements ITool{
          function travel(){
              echo'<div>Travel by Car</div>';
              return 200;
          }
      }

      class TravelByTrain implements ITool{
          function travel(){
              echo'<div>Travel by Train</div>';
              return 100;
          }
      }

      class person{
          var $money = 0;
          function __construct($m){
              if(is_integer($m) && ($m > 0)){
                  $this->money = $m;
              }
          }
          function goTravel(ITool $t){
              if($t == NULL) return;
              $this->money -= $t->travel();
              echo '<div>the rest of money is'. $this->money.'</div><br />';
          }
      }

      $p   = new person(1000);
      $p->goTravel(new TravelByCar());
      $p->goTravel(new TravelByTrain());       
      
      ##############################################################################################
      类与类之间的关系：
      a继承：被继承的类称为父类，继承的类称为子类。子类包含父类所有的公有和受保护成员,子类在父类基础上可以进行扩充。使用关键字extends来定义这种继承关系。带空心三角箭头的实线表示
      final：PHP5中新增关键字，如果父类中的方法被声明为 final，则子类无法覆盖该方法。如果一个类被声明为 final，则不能被继承。
      Tips：
           类必须在使用之前被定义；
           父类必须在子类之前被声明；
      ##############################################################################################
      class Controller{
      
      }
      
      class IndexController extends Controller{
          public function index(){
             echo "Welcome to LEOS Studio! ";
          }
      }
      ##############################################################################################
      b实现：类对接口的实现，类与接口之间的关系，用implements关键字定义。带空心三角箭头的虚线表示。
      c依赖：类之间的使用关系，一个类使用到了另一个类，便对此类产生了依赖关系，但这种依赖是临时性、暂时的。带箭头的虚线表示。
      d关联：类之间的强依赖关系，依赖是长期的、持久的。表现在代码层面就是一个类作为另一个类的成员。带箭头的实现表示。
      e聚合：关联关系的一种特殊情况，一个类作为另一个类的一部分存在，是整体-部分关系，是has-a的关系，但是两个类具有各自的生命周期。空心菱形加实线表示。
      f组合：关联关系的一种特殊情况，比聚合关系更强，整体的生命周期结束意味着部分生命周期结束，是contains-a的关系。实心菱形加实现表示。
      抽象类：不能被实例化，任何一个类，如果它里面至少有一个方法是被声明为抽象的，那么这个类就必须声明为抽象的。被定义为抽象的方法只是声明了其调用方式，不能定义其具体的功能实现
      
   4）类成员访问性
      对类的属性或方法的访问控制，称为类成员访问性
      private  ：只能被其定义所在的类访问
      protected：被其自身以及其子类和父类访问
      public   ：可以在任何地方被访问
      一般地，没有指定访问性，默认为public
   5）静态成员
      静态成员：可以直接访问，不需要实例化为对象。用static关键字进行定义。静态属性不能通过一个类已实例化的对象来访问，但是静态方法可以。
      类名::属性名;
      类名::函数名();
   6）属性访问
      静态属性：类内部访问 $this->property 、类外部访问 $me->name
      动态属性：类内部访问 self::property 、类外部访问 myClass::name
      ##############################################################################################
      class myClass{
          public $name = "myClass";
          
          public function getName(){
              return $this->name;
          }
      }
      $me = new myClass();
      $me->name ="liebert";
      ##############################################################################################
   7）自动加载类
      类要使用，必须要先加载类的文件，加载方式：include、include_once、require、require_once。
      PHP5之后，支持自动加载，也就是说当调用到某个类时，首先会判断是否已经加载了该类的源文件，如果没有加载，会执行__autoload()函数（不建议使用）。
      这就PHP内核对外提供的一个钩子，用户可以在此处写入自己的代码，以增强系统的功能。
      
      spl_autoload_register([callback $autoload_function])
      ##############################################################################################
      function autoload(){
      }
      
      spl_autoload_register('autoload');
      
      ##############################################################################################
   8）构造函数和析构函数
      a.构造函数：类在每次创建对象时最先调用此方法，主要用于初始化。
        void __construct([mixed $args[,$...]]){}
        如果子类中定义了构造函数，不会隐式调用其父类的构造函数。要执行父类构造函数，需要在子类构造函数中调用parent::__construct();
      b.析构函数：析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。PHP引擎内部有引用计数器，当引用计数等于0时便销毁此对象。
        void __destruct(void);
        如果子类中定义了析构函数，不会隐式调用其父类的析构函数。要执行父类析构函数，需要在子类析构函数中调用parent::__destruct();
        析构函数即使在使用 exit() 终止脚本运行时也会被调用。在析构函数中调用 exit() 将会中止其余关闭操作的运行。 
   9）重载
      动态“创建”类属性和方法。主要通过魔术方法实现这种重载机制。
      属性重载：
      public void  __set(string $name, mixed $value);             //当给不可访问的属性赋值时，__set()会被调用；
      public mixed __get(string $name);                           //当读取不可访问的属性值时，__get()会被调用；
      public bool  __isset(string $name);                         //当对不可访问的属性调用isset()或empty()时，__isset()会被调用；
      public void  __unset(string $name);                         //当对不可访问的属性调用unset()时，__unset()会被调用。
      方法重载：
      public mixed        __call(string $name, array $arguments); //当对象调用一个不可访问的方法时，__call()会被调用
      public static mixed __callStatic();                         //通过静态调用的方式调用一个不可访问的方法时，__callStatic()会被调用。
      
   10）魔术方法
      PHP将所有以 __（两个下划线）开头的类方法保留为魔术方法。所以在定义类方法时，除了上述魔术方法，建议不要以 __ 为前缀。
      __construct()
      __destruct()
      __call()
      __callStatic()
      __get()
      __set()
      __isset()
      __unset()
      __sleep()
      __wakeup()
      __toString()
      __invoke()
      __set_state()
      __clone()
      __debugInfo()
   11）对象比较
      == ：如果两个对象的属性和属性值都相等，而且两个对象是同一个类的实例，那么这两个对象变量相等。
      ===：这两个对象变量一定要指向某个类的同一个实例（即同一个对象）。
   12）后期静态绑定
      用于在继承范围内引用静态调用的类
   13）对象和引用
      在php5，一个对象变量已经不再保存整个对象的值。只是保存一个标识符来访问真正的对象内容。 当对象作为参数传递，作为结果返回，或者赋值给另外一个变量，另外一个变量跟原来的不是引用的关系，只是他们都保存着同一个标识符的拷贝，这个标识符指向同一个对象的真正内容。 
   14）对象序列化
       serialize()：返回一个包含字节流的字符串。序列化一个对象将会保存对象的所有变量，但是不会保存对象的方法，只会保存类的名字。
       unserialize()：返回一个对象。这个对象的类必须已经定义过。
       
18.命名空间
   1）概述
      命名空间是一种封装事物的方法。
      a.用户编写的代码与PHP内部的类/函数/常量或第三方类/函数/常量之间的名字冲突。
      b.为很长的标识符名称(通常是为了缓解第一类问题而定义的)创建一个别名（或简短）的名称，提高源代码的可读性。
      Tips:名为PHP或php的命名空间，以及以这些名字开头的命名空间（例如PHP\Classes）被保留用作语言内核使用，而不应该在用户空间的代码中使用。
   2）定义
      使用namespace关键字定义命名空间。
      namespace Frame;
      namespace Frame\Driver\DB;  //子命名空间
      Tips：所有非 PHP 代码包括空白符都不能出现在命名空间的声明之前，命名空间必须是程序脚本的第一条语句；同一个命名空间可以定义在多个文件中，即允许将同一个命名空间的内容分割存放在不同的文件中。
   3）__NAMESPACE__：包含当前命名空间名称的字符串。在全局的，不包括在任何命名空间中的代码，它包含一个空的字符串。
   4）使用命名空间
      使用use操作符导入/使用别名：use My\Full\Classname as Another;
      通过use操作符导入/使用别名，一行中包含多个use语句：use My\Full\Classname as Another, My\Full\NSname;
   5）全局空间
      如果没有定义任何命名空间，所有的类与函数的定义都是在全局空间，在名称前加上前缀 \ 表示该名称是全局空间中的名称。
   6）名称解析规则
      1.对完全限定名称的函数，类和常量的调用在编译时解析。例如 new \A\B 解析为类 A\B。  
      2.所有的非限定名称和限定名称（非完全限定名称）根据当前的导入规则在编译时进行转换。例如，如果命名空间 A\B\C 被导入为 C，那么对 C\D\e() 的调用就会被转换为 A\B\C\D\e()。  
      3.在命名空间内部，所有的没有根据导入规则转换的限定名称均会在其前面加上当前的命名空间名称。例如，在命名空间 A\B 内部调用 C\D\e()，则 C\D\e() 会被转换为 A\B\C\D\e() 。  
      4.非限定类名根据当前的导入规则在编译时转换（用全名代替短的导入名称）。例如，如果命名空间 A\B\C 导入为C，则 new C() 被转换为 new A\B\C() 。  
      5.在命名空间内部（例如A\B），对非限定名称的函数调用是在运行时解析的。例如对函数 foo() 的调用是这样解析的：  
        1.在当前命名空间中查找名为 A\B\foo() 的函数  
        2.尝试查找并调用 全局(global) 空间中的函数 foo()。  
      6. 在命名空间（例如A\B）内部对非限定名称或限定名称类（非完全限定名称）的调用是在运行时解析的。下面是调用 new C() 及 new D\E() 的解析过程： new C()的解析:  
        1.在当前命名空间中查找A\B\C类。  
        2. 尝试自动装载类A\B\C。  
        new D\E()的解析:  
        1.在类名称前面加上当前命名空间名称变成：A\B\D\E，然后查找该类。  
        2.尝试自动装载类 A\B\D\E。  
        为了引用全局命名空间中的全局类，必须使用完全限定名称 new \C()。
19.数组函数
   /*************************************************************************************************
    *                                     array_change_key_case
    * description        ：返回字符串键名全为小写或大写的数组
    * param array $input ：待操作的数组
    * param int   $case  ：可选项，取值为CASE_UPPER 或 CASE_LOWER，默认为CASE_LOWER。
    * return array       ：字符串键名全为小写或大写的数组
    ************************************************************************************************/
   funtcion array array_change_key_case(array $input, [, int $case = CASE_LOWER ]);
   
   /*************************************************************************************************
    *                                       array_chunk
    * description               ：将一个数组分割成多个数组
    * param array $input        ：待操作的数组
    * param int   $size         ：每个数组的单元数目
    * param bool  $preserve_keys：是否保留保留输入数组中原来的键名，可选项，默认为FALSE，每个结果数组
    *                             将用从零开始的新数字索引
    * return array              ：一个多维数组，其索引从零开始，每一维包含了 size 个元素
    ************************************************************************************************/
   function array array_chunk(array $input, int $size [, bool $preserve_keys = false ]);
    
   /*************************************************************************************************
    *                                       array_column
    * description            ：返回数组中指定的一列
    * param array $input     ：待操作的数组
    * param mixed $column_key：需要返回的列，它可以是索引数组的列索引，或者是关联数组的列的键，也可以
    *                          是NULL，此时将返回整个数组
    * param mixed $index_key ：作为返回数组的索引/键的列，它可以是该列的整数索引，或者字符串键值
    * return array           ：单列数组
    ************************************************************************************************/
   function array array_column(array $input, mixed $column_key [, mixed $index_key ]);
    
   /*************************************************************************************************
    *                                        array_combine
    * description         ：创建一个数组，用一个数组的值作为其键名，另一个数组的值作为其值
    * param array $keys   ：将被作为新数组的键
    * param array $values ：将被作为数组的值
    * return array        ：返回合并的 array，如果两个数组的单元数不同则返回 FALSE
    ************************************************************************************************/
   function array array_combine(array $keys, array $values );
    
   /*************************************************************************************************
    *                                        array_keys
    * description               ：返回数组中部分的或所有的键名
    * param array $array        ：待操作的数组
    * param mixed $search_value ：如果指定了这个参数，只有包含这些值的键才会返回
    * param bool  $strict       ：判断在搜索的时候是否该使用严格的比较（===）
    * return array              ：返回 input 里的所有键
    ************************************************************************************************/
   function array array_keys(array $array [, mixed $search_value [, bool $strict = false ]]);
    
   /*************************************************************************************************
    *                                         array_map
    * description           ：将回调函数作用到给定数组的单元上
    * param array $callback ：对每个数组的每个元素作用的回调函数
    * param array $arr1     ：将被回调函数（callback）执行的数组
    * return array          ：返回一个数组，该数组的每个元素都数组（arr1）里面的每个元素经过回调函数
    *                        （callback）处理了的
    ************************************************************************************************/
   function array array_map(callable $callback , array $arr1 [, array $... ]);
    
   /*************************************************************************************************
    *                                        array_merge
    * description         ：合并一个或多个数组。如果输入的数组中有相同的字符串键名，则该键名后面的
    *                       值将覆盖前一个值；如果数组包含数字键名，后面的值将不会覆盖原来的值，而
    *                       是附加到后面，键名会以连续方式重新索引。
    * param array $array1 ：待合并的初始数组
    * param array ...     ：待合并的数组
    * return array        ：合并后的数组
    ************************************************************************************************/
   function array array_merge(array $array1 [, array $... ]);
     
   /*************************************************************************************************
    *                                        array_pop
    * description         ：将数组最后一个单元弹出
    * param array $array1 ：需要做出栈的数组
    * return mixed        ：数组中最后一个单元值
    ************************************************************************************************/
   function mixed array_pop(array &$array);
      
      
   /*************************************************************************************************
    *                                        array_push
    * description         ：将一个或多个单元压入数组的末尾
    * param array $array1 ：接收压入单元的数组
    * param mixed $var    ：待压入的值
    * return int          ：返回处理之后数组的元素个数
    ************************************************************************************************/
   function int array_push(array &$array , mixed $var [, mixed $... ]);
      
   /*************************************************************************************************
    *                                      array_replace
    * description         ：数组元素相同key的值替换array1数组的值。如果一个键存在于第一个数组同时也存
    *                       在于第二个数组，它的值将被第二个数组中的值替换。如果一个键存在于第二个数组，
    *                       但是不存在于第一个数组，则会在第一个数组中创建这个元素。如果一个键仅存在于
    *                       第一个数组，它将保持不变。如果传递了多个替换数组，它们将被按顺序依次处理，
    *                       后面的数组将覆盖之前的值。 
    * param array $array1 ：待替换的数组
    * param array $array2 ：替换数组
    * return int          ：返回一个数组。如果发生错误，将返回 NULL
    ************************************************************************************************/
   function array array_replace(array $array1 , array $array2 [, array $... ]);
      
   /*************************************************************************************************
    *                                        array_shift
    * description          ：将数组开头的单元移出数组
    * param  array $array1 ：待操作的数组
    * return mixed         ：返回移出的值，如果 array 为 空或不是一个数组则返回 NULL
    ************************************************************************************************/
   function int array_shift(array &$array);
      
   /*************************************************************************************************
    *                                        array_slice
    * description                 ：将数组开头的单元移出数组
    * param  array $array         ：待操作的数组
    * param  int   $offset        ：如果 offset 非负，则序列将从 array 中的此偏移量开始；如果 offset
    *                                  为负，则序列将从 array 中距离末端这么远的地方开始
    * param  int   $length        ：如果给出了 length 并且为正，则从数组中取出length个单元。如果给出
    *                               了 length 为负，则序列将终止在距离数组末端这么远的地方。如果省略
    *                               则序列将从 offset 开始一直到 array 的末端。
    * param  bool  $preserve_keys ：是否保留原有的键值，默认为FALSE，会重新排序并重置数组的数字索引
    * return mixed                ：返回数组中移除的部分
    ************************************************************************************************/
   function array array_slice(array $array, int $offset [, int $length = NULL [, bool $preserve_keys = false ]]);  

20.文件系统
   /*************************************************************************************************
    *                                        file_exists
    * description            ：检查文件或目录是否存在
    * param string $filename ：文件或目录的路径字符串
    * return bool            ：存在返回TRUE，否则返回FALSE
    ************************************************************************************************/
   function bool file_exists(string $filename);
    
   /*************************************************************************************************
    *                                     file_get_contents
    * description            ：将整个文件读入一个字符串
    * param string $filename ：文件名
    * return string          ：读取的数据
    ************************************************************************************************/
   function string file_get_contents(string $filename [, bool $use_include_path = false [, resource $context [, int $offset = -1 [, int $maxlen ]]]]);
    
   /*************************************************************************************************
    *                                     file_put_contents
    * description            ：将一个字符串写入文件
    * param string $filename ：文件名
    * param mixed  $data     ：要写入的数据，类型为string、array、stream
    * return int             ：返回写入到文件内数据的字节数，失败时返回FALSE
    ************************************************************************************************/
   function bool file_put_contents(string $filename , mixed $data [, int $flags = 0 [, resource $context ]]);
   
   /*************************************************************************************************
    *                                         is_dir
    * description            ：判断给定文件名是否是一个目录
    * param string $filename ：文件名
    * return bool            ：如果文件名存在并且为目录返回TRUE，否则返回FALSE
    ************************************************************************************************/
   function bool is_dir(string $filename);
   
   /*************************************************************************************************
    *                                        is_file
    * description            ：判断给定文件名是否为一个正常的文件
    * param string $filename ：文件名
    * return bool            ：如果文件存在且为正常的文件则返回 TRUE，否则返回 FALSE
    ************************************************************************************************/
   function bool is_file(string $filename);

   /*************************************************************************************************
    *                                        filemtime
    * description            ：取得文件修改时间
    * param string $filename ：文件名
    * return int             ：返回文件上次被写入时间
    ************************************************************************************************/
   function int filemtime(string $filename);
   
21.字符串处理
   /*************************************************************************************************
    *                                        md5
    * description           ：计算字符串的MD5散列值
    * param string $str     ：原始字符串
    * param bool $raw_output：是否已16字节长度的原始二进制返回
    * return string         ：返回文件上次被写入时间
    ************************************************************************************************/
   function string md5(string $str [, bool $raw_output = false ]);
   
   /*************************************************************************************************
    *                                     strtolower
    * description           ：将字符串转化为小写
    * param string $str     ：原始字符串
    * return string         ：返回转换后的字符串
    ************************************************************************************************/
   function string strtolower(string $str);
   
   /*************************************************************************************************
    *                                     strtoupper
    * description           ：将字符串转化为大写
    * param string $str     ：原始字符串
    * return string         ：返回转换后的字符串
    ************************************************************************************************/
   function string strtoupper(string $str);
   
   /*************************************************************************************************
    *                                     str_replace
    * description          ：子字符串替换
    * param mixed $search  ：原始字符串
    * param mixed $replace ：search 的替换值。一个数组可以被用来指定多重替换
    * param mixed $subject ：目标字符串，执行替换的数组或字符串
    * return string        ：该函数返回一个字符串或者数组。该字符串或数组是将 subject 中全部的search
                             都被 replace 替换之后的结果
    ************************************************************************************************/
   function mixed str_replace(mixed $search, mixed $repalce, mixed $subject [, int &$count]);



22.正则表达式




23.图像处理



24.变量和类型扩展



25.数据库操作(MySQL)
   


26.XML操作




27.内存管理





28.php.ini配置




设计模式
工厂模式、单例模式、注册器模式

适配器模式
  定义一个接口，多个类之间的接口约束
策略模式
  解耦
  
数据对象模式

观察者模式--监听者模式
  当一个对象状态发生改变时，依赖它的对象全部会接收到通知