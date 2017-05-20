http://www.cnblogs.com/linzhenjie/p/5485475.html

PHP优化加速之Opcache使用总结
发表于2016/4/12 16:31:36  3533人阅读
分类： PHP

PHP优化加速之Opcache使用总结：
Opcache是一种通过将解析的PHP脚本预编译的字节码存放在共享内存中来避免每次加载和解析PHP脚本的开销，解析器可以直接从共享内存读取已经缓存的字节码，从而大大提高PHP的执行效率。PS: 需要区别于Xcache机制，后续总结中会介绍其使用。
 
·     如何安装
·     如何配置
·     如何使用
·     显示分析
·     注意事项
 
一、如何安装
在PHP 5.5.0及后续版本中，PHP已经将Opcache功能以拓展库形式内嵌在发布版本中了，默认未开启Opcache加速，需要开发人员在php.ini中添加或解注释Opcache相关配置即可。对于之前的老版本，可以将Opcache作为PECL拓展库进行安装和配置，可以参考PHP拓展安装及配置：
http://blog.csdn.net/why_2012_gogo/article/details/51120645
 
NOTE:
如果你使用 --disable-all 参数 禁用了默认扩展的构建， 那么必须使用--enable-opcache 选项来开启 Opcache。
 
二、如何配置
php.ini:
[opcache]
; 启动操作码缓存
opcache.enable=1
; 针对支持CLI版本PHP启动操作码缓存 一般被用来测试和调试
opcache.enable_cli=1
; 共享内存大小，单位为MB
opcache.memory_consumption=128
; 存储临时字符串缓存大小，单位为MB，PHP5.3.0以前会忽略此项配置
opcache.interned_strings_buffer=8
; 缓存文件数最大限制，命中率不到100%，可以试着提高这个值
opcache.max_accelerated_files=4000
; 一定时间内检查文件的修改时间, 这里设置检查的时间周期, 默认为 2, 单位为秒
opcache.revalidate_freq=60
; 开启快速停止续发事件，依赖于Zend引擎的内存管理模块，一次释放全部请求变量的内存，而不是依次释放内存块
opcache.fast_shutdown=1
;启用检查 PHP 脚本存在性和可读性的功能，无论文件是否已经被缓存，都会检查操作码缓存,可以提升性能。 但是如果禁用了 opcache.validate_timestamps选项， 可能存在返回过时数据的风险。
opcache.enable_file_override=1
 
; 拓展库so文件关联加载
zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20131226/opcache.so
 
NOTE:
上面列出的配置项是常用且重要的配置项，实际不止上面这些配置项。
 
三、如何使用
实际上，对于Opcache的使用，主要体现在其提供的几个函数：
1、opcache_get_configuration；
形式:array opcache_get_configuration(void);
获取设置的缓存配置信息，以数组形式返回配置信息、黑名单及版本号。
 
2、opcache_get_status；
形式:array opcache_get_status(void);
获取设置的缓存状态信息。
 
3、opcache_invalidate；
形式:boolean opcache_invalidate (string);
该函数的作用是使得指定脚本的字节码缓存失效。如果force 没有设置或者传入的是 FALSE，那么只有当脚本的修改时间 比对应字节码的时间更新，脚本的缓存才会失效。
 
4、opcache_reset;
形式:boolean opcache_reset(void);
该函数将重置整个字节码缓存。在调用 opcache_reset() 之后，所有的脚本将会重新载入并且在下次被点击的时候重新解析。
 
5、opcache_compile_file；
形式:boolean opcache_compile_file (string);
无需运行，就可以编译并缓存脚本。
 
6、opcache_is_script_cached
形式:boolean opcache_is_script_cached (string);
判断某个脚本是否已经缓存到Opcache。
 
下面我编写一个PHP脚本，囊括上面的几个函数的封装，这样也方便日后对Opcache的维护和管理，具体如下：
<?php
/**
 * 这个文件是对opcache优化器的几个
 * 函数的封装，作为一个工具脚本使用
 */
 
if(!extension_loaded("ZendOpcache")) {
      echo "You do nothave the Zend OPcache extension loaded , please open it up,then retry!";
}
 
/**
 * 函数操作封装类
 * 数组形式的结果，会转为json格式返回,不做显示上的处理
 * 这里主要处理的是影响Opcache缓存状态的操作，对于查看
 * Opcache各项指标的处理，可查看项目：opcache-status
 */
class OpcacheScriptModel{
      private $_configuration;
      private $_status;
     
      function __construct() {
            $this->_configuration =opcache_get_configuration();
            $this->_status =opcache_get_status();
      }
     
      // 获取配置信息
      public function getConfigDatas(){
            echo json_encode($this->_configuration);
      }
     
      // 获取状态信息
      public function getStatusDatas(){
            echo json_encode($this->_status);
      }
     
      // 指定某脚本文件字节码缓存失效
      public function invalidate($script){
            return opcache_invalidate($script);
      }
     
      // 重置或清除整个字节码缓存数据
      public function reset() {
            return opcache_reset();
      }
     
      // 无需运行，就可以编译并缓存脚本
      public function compile($file){
            return opcache_compile_file($file);
      }
     
      // 判断某个脚本是否已经缓存到Opcache
      public function isCached($script){
            return opcache_is_script_cached($script);
      }
}
 
// 获得对象
function getOpcacheDataModel(){
      // 初始化对象
      $dataModel = NULL;
      if(NULL ==$dataModel) {
            $dataModel = new OpcacheScriptModel();
      }
     
      return $dataModel;
}
 
?>
 
上面的脚本工具比较简单，一般可放在项目中或是单独作为工具使用，需要时去解析编译即可，其实下面的显示分析开源项目，也是使用上面的API函数，只不过其将获得数据以图形化形式展示出来，这样更加直观，请继续往下看。
 
四、显示分析
我们知道PHP脚本的执行机制是，解析器解析PHP脚本文件，并将其解析为字节码数据，而Opcache优化器的作用就是缓存被解析的字节码数据，做到直接从缓存中读取而不需要每次都重复PHP脚本的加载和解析工作，所以对于Opcache的使用，我们一般只需要做两件事儿：
1、使用Opcache优化器，加快PHP程序的执行速度；
2、通过Opcache各项指标参数，实时了解当前PHP程序的性能状态；
 
那么，我们如何去查看和分析当前的Opcache加速效果那？答案是可以使用下Github上开源的项目：https://github.com/rlerdorf/opcache-status
将下载下来的项目放入到当前的Web服务器根目录下，直接访问即可，先看效果：
 
从上面的截图及项目文件看出，该Opcache工具是一个简化的GUI版本，使用它可以清楚了解和分析下面的内容：
1、缓存使用情况、剩余情况及内存浪费情况及比例；
2、缓存的keys、剩余的keys数；
3、缓存命中数以及未命中数；
4、缓存配置、状态以及缓存捕获脚本；
5、缓存的脚本文件，以视图形式划分直观显示；
 
好了，Opcache的可视化就说到这里，下面看下几项注意点。
 
五、注意事项
1、不建议Xcache和Opcache同时启用PHP优化；
因为PHP 5.5.0及后续版本已经内嵌对Opcache的支持，所以PHP意识到其重要性，相对于Xcache等第三方的PHP优化器来说，使用Opcache会是更好的选择。另外，两者同时存在的话，会使Opcache的缓存命中数大大降低，而且增加不必要的开销。
 
2、不建议在开发过程中开启Opcache
原因很明显，开启了Opcache之后，开发人员修改的内容不会立即显示和生效，因为受到opcache.revalidate_freq=60的影响，所以建议在开发并测试之后，测试性能时再行打开测试，当然，生产环境一直都要开着Opcache了哦。
 
3、不建议将Opcache指标设置太大
Opcache各项指标配置大小或是否开启，需要结合项目实际情况需求及Opcache官方建议的配置，项目的实际情况分析，可结合上面第四部分的可视化缓存信息分析调整。
 
4、不建议长期使用老版本的Opcache
建议及时关注Opcache官网动态，实时了解其的bugs修复，功能优化及新增功能，以便更好的将其应用在自己的项目中。
 
5、不建议在生产环境中，将上面介绍的开源项目放入Web服务根目录
原因很简单，因为这个开源项目并未做访问的限制和安全处理，也就是说凡是可以访问外网的用户，只要知道了访问地址就可以直接访问，所以不安全。一般下，这个开源工具只是帮助可视化分析PHP的性能，通常在开发调试阶段使用。如果就是想在生产环境开启使用，那么就必须做好安全限制工作。
 
 
 
 
好了，到这里已经总结并介绍完了对于Opcache的使用，如需进一步优化底层，可以参看和修改源代码，不过一般情况下不需要哦！
 
