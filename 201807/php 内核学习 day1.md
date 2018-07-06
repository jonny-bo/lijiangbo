# php 内核学习 day1
## 准本工作
1. 参考网站： [深入理解PHP内核](https://www.php-internals.com) and [PHP 核心：骇客指南](http://php.net/manual/zh/internals2.php)
2. 克隆代码：`git clone git://github.com/php/php-src.git`
3. 切换代码分支到指定版本的PHP：`cd php-src && git checkout PHP-5.3 # 签出5.3分支`
4. 准备编译环境: `sudo apt-get install build-essential` (ubuntu系统需要安装)
5. 编译：`./buildconf` 然后 `./configure --disable-all` 最后 `make`

## 目录结构
- `/` 根目录，这个目录包含的东西比较多，主要包含一些说明文件以及设计方案。
- `build` 顾名思义，这里主要放置一些和源码编译相关的一些文件，比如开始构建之前的buildconf脚本等文件，还有一些检查环境的脚本等。
- `ext` 官方扩展目录，包括了绝大多数PHP的函数的定义和实现，如array系列，pdo系列，spl系列等函数的实现，都在这个目录中。
- `main` 这里存放的就是PHP最为核心的文件
- `Zend` Zend引擎的实现目录，比如脚本的词法语法解析，opcode的执行以及扩展机制的实现等等
- `pear` “PHP 扩展与应用仓库”，包含PEAR的核心文件
- `sapi` 包含了各种服务器抽象层的代码，例如apache的mod_php，cgi，fastcgi以及fpm等等接口
- `TSRM` PHP的线程安全是构建在TSRM库之上的，PHP实现中常见的*G宏通常是对TSRM的封装，TSRM(Thread Safe Resource Manager)线程安全资源管理器
- `tests` PHP的测试脚本集合，包含PHP各项功能的测试文件
- `win32` 这个目录主要包括Windows平台相关的一些实现

## 常用代码

php经常使用一些`宏`来定义代码，对复杂的代码进行简单的封装。

`##` 被称为 连接符（concatenator），它是一种预处理运算符， 用来把两个语言符号(Token)组合成单个语言符号。

`#` 是一种预处理运算符，它的功能是将其后面的宏参数进行字符串化操作。

`#line` 用于强制指定新的行号和编译文件名。

`#error` 用于自定义一条编译错误信息。

`#waring` 用于自定义一条编译警告信息。

PHP中的全局变量宏，全局变量为如下的一个结构体，各字段的意义如字段后的注释：

```
struct _php_core_globals {
        zend_bool magic_quotes_gpc; //  是否对输入的GET/POST/Cookie数据使用自动字符串转义。
        zend_bool magic_quotes_runtime; //是否对运行时从外部资源产生的数据使用自动字符串转义
        zend_bool magic_quotes_sybase;  //   是否采用Sybase形式的自动字符串转义
 
        zend_bool safe_mode;    //  是否启用安全模式
 
        zend_bool allow_call_time_pass_reference;   //是否强迫在函数调用时按引用传递参数
        zend_bool implicit_flush;   //是否要求PHP输出层在每个输出块之后自动刷新数据
 
        long output_buffering;  //输出缓冲区大小(字节)
 
        char *safe_mode_include_dir;    //在安全模式下，该组目录和其子目录下的文件被包含时，将跳过UID/GID检查。
        zend_bool safe_mode_gid;    //在安全模式下，默认在访问文件时会做UID比较检查
        zend_bool sql_safe_mode;
        zend_bool enable_dl;    //是否允许使用dl()函数。dl()函数仅在将PHP作为apache模块安装时才有效。
 
        char *output_handler;   // 将所有脚本的输出重定向到一个输出处理函数。
 
        char *unserialize_callback_func;    // 如果解序列化处理器需要实例化一个未定义的类，这里指定的回调函数将以该未定义类的名字作为参数被unserialize()调用，
        long serialize_precision;   //将浮点型和双精度型数据序列化存储时的精度(有效位数)。
 
        char *safe_mode_exec_dir;   //在安全模式下，只有该目录下的可执行程序才允许被执行系统程序的函数执行。
 
        long memory_limit;  //一个脚本所能够申请到的最大内存字节数(可以使用K和M作为单位)。
        long max_input_time;    // 每个脚本解析输入数据(POST, GET, upload)的最大允许时间(秒)。
 
        zend_bool track_errors; //是否在变量$php_errormsg中保存最近一个错误或警告消息。
        zend_bool display_errors;   //是否将错误信息作为输出的一部分显示。
        zend_bool display_startup_errors;   //是否显示PHP启动时的错误。
        zend_bool log_errors;   // 是否在日志文件里记录错误，具体在哪里记录取决于error_log指令
        long      log_errors_max_len;   //设置错误日志中附加的与错误信息相关联的错误源的最大长度。
        zend_bool ignore_repeated_errors;   //   记录错误日志时是否忽略重复的错误信息。
        zend_bool ignore_repeated_source;   //是否在忽略重复的错误信息时忽略重复的错误源。
        zend_bool report_memleaks;  //是否报告内存泄漏。
        char *error_log;    //将错误日志记录到哪个文件中。
 
        char *doc_root; //PHP的”根目录”。
        char *user_dir; //告诉php在使用 /~username 打开脚本时到哪个目录下去找
        char *include_path; //指定一组目录用于require(), include(), fopen_with_path()函数寻找文件。
        char *open_basedir; // 将PHP允许操作的所有文件(包括文件自身)都限制在此组目录列表下。
        char *extension_dir;    //存放扩展库(模块)的目录，也就是PHP用来寻找动态扩展模块的目录。
 
        char *upload_tmp_dir;   // 文件上传时存放文件的临时目录
        long upload_max_filesize;   // 允许上传的文件的最大尺寸。
 
        char *error_append_string;  // 用于错误信息后输出的字符串
        char *error_prepend_string; //用于错误信息前输出的字符串
 
        char *auto_prepend_file;    //指定在主文件之前自动解析的文件名。
        char *auto_append_file; //指定在主文件之后自动解析的文件名。
 
        arg_separators arg_separator;   //PHP所产生的URL中用来分隔参数的分隔符。
 
        char *variables_order;  // PHP注册 Environment, GET, POST, Cookie, Server 变量的顺序。
 
        HashTable rfc1867_protected_variables;  //  RFC1867保护的变量名，在main/rfc1867.c文件中有用到此变量
 
        short connection_status;    //  连接状态，有三个状态，正常，中断，超时
        short ignore_user_abort;    //  是否即使在用户中止请求后也坚持完成整个请求。
 
        unsigned char header_is_being_sent; //  是否头信息正在发送
 
        zend_llist tick_functions;  //  仅在main目录下的php_ticks.c文件中有用到，此处定义的函数在register_tick_function等函数中有用到。
 
        zval *http_globals[6];  // 存放GET、POST、SERVER等信息
 
        zend_bool expose_php;   //  是否展示php的信息
 
        zend_bool register_globals; //  是否将 E, G, P, C, S 变量注册为全局变量。
        zend_bool register_long_arrays; //   是否启用旧式的长式数组(HTTP_*_VARS)。
        zend_bool register_argc_argv;   //  是否声明$argv和$argc全局变量(包含用GET方法的信息)。
        zend_bool auto_globals_jit; //  是否仅在使用到$_SERVER和$_ENV变量时才创建(而不是在脚本一启动时就自动创建)。
 
        zend_bool y2k_compliance;   //是否强制打开2000年适应(可能在非Y2K适应的浏览器中导致问题)。
 
        char *docref_root;  // 如果打开了html_errors指令，PHP将会在出错信息上显示超连接，
        char *docref_ext;   //指定文件的扩展名(必须含有’.')。
 
        zend_bool html_errors;  //是否在出错信息中使用HTML标记。
        zend_bool xmlrpc_errors;   
 
        long xmlrpc_error_number;
 
        zend_bool activated_auto_globals[8];
 
        zend_bool modules_activated;    //  是否已经激活模块
        zend_bool file_uploads; //是否允许HTTP文件上传。
        zend_bool during_request_startup;   //是否在请求初始化过程中
        zend_bool allow_url_fopen;  //是否允许打开远程文件
        zend_bool always_populate_raw_post_data;    //是否总是生成$HTTP_RAW_POST_DATA变量(原始POST数据)。
        zend_bool report_zend_debug;    //  是否打开zend debug，仅在main/main.c文件中有使用。
 
        int last_error_type;    //  最后的错误类型
        char *last_error_message;   //  最后的错误信息
        char *last_error_file;  //  最后的错误文件
        int  last_error_lineno; //  最后的错误行
 
        char *disable_functions;    //该指令接受一个用逗号分隔的函数名列表，以禁用特定的函数。
        char *disable_classes;  //该指令接受一个用逗号分隔的类名列表，以禁用特定的类。
        zend_bool allow_url_include;    //是否允许include/require远程文件。
        zend_bool exit_on_timeout;  //  超时则退出
#ifdef PHP_WIN32
        zend_bool com_initialized;
#endif
        long max_input_nesting_level;   //最大的嵌套层数
        zend_bool in_user_include;  //是否在用户包含空间
 
        char *user_ini_filename;    //  用户的ini文件名
        long user_ini_cache_ttl;    //  ini缓存过期限制
 
        char *request_order;    //  优先级比variables_order高，在request变量生成时用到，个人觉得是历史遗留问题
 
        zend_bool mail_x_header;    //  仅在ext/standard/mail.c文件中使用，
        char *mail_log;
 
        zend_bool in_error_log;
};
```