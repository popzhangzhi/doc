    <?php
    /**
     * Created by PhpStorm.
     * User: zhangzhi
     * Date: 2018/11/26
     * Time: 16:21
     */

    /**
     * build(打包后文件目录)
     * src
        index.php
        function.php
     * phar-builder.php (本文件)
     *
     * index.php 源码如下
     *
     *  require_once "phar://test.phar/function.php";
        $symbol = "index.php";
        echo $symbol . PHP_EOL;
        $common = new common();
        $common->totell();
     *
     * function.php 源码如下
     *
        class common {

        private  $symbol="";
        function __construct()
        {
        $this->symbol = "function";
        }
        function totell(){
        echo $this->symbol . PHP_EOL;

        }
        }
     */

    $phar_name = "test.phar";
    $phar_alias ="";
    $src_path = "./src/";
    $build_path = "./build/";

    $phar_readonly = ini_get('phar.readonly');

    if((int)$phar_readonly == 0 ){
        echo "phar.readonly需要设置为Off，前往php.ini去设置 "   . PHP_EOL;
    }
    //删除已有包
    if(is_file($build_path.$phar_name)){
        unlink($build_path.$phar_name);
    }

    $phar = new Phar($build_path.$phar_name,
        FilesystemIterator::CURRENT_AS_FILEINFO |FilesystemIterator::KEY_AS_FILENAME,$phar_alias);
    //第一个参数是构建phar的源码路径，第二个是正则过滤，忽略则打包整个目录
    $phar->buildFromDirectory($src_path,'/.php$/');

    //打包后的入口文件，createDefaultStub
    $phar->setStub($phar->createDefaultStub("index.php"));

    /**
     * createDefaultStub
     * 缺省生成类似以下文件
     * Phar::mapPhar();
     * include "phar://myapp.phar/index.php";
     * __HALT_COMPILER();
     *
     * Phar::mapPhar() 用来分析Phar文件的元数据，并初始化它。
     * stub文件的结尾处需要调用 __HALT_COMPILER() 方法，这个方法后不能留空格。
     * __HALT_COMPILER() 会立即终止PHP的运行，防止include的文件在此方法后仍然执行。
     * 这是Phar必须的，没有它Phar将不能正常运行。
     *
     * 可以创建自己的stub文件来执行自定义的初始化过程，像这样加载自定义文件
     * $phar->setStub(file_get_contents("stub.php"))
     *
     */

    /**
     * php phar-builder.php 去生成phar
     *
     * 我们可以直接访问phar，但需要配置服务器。使用以下方法来达到直接访问
     * 建立一个run.php 内容如下
     * <?php
     *  require "test.phar";
     * ?>
     *
     * 这样就免去更改配置服务器
     *
     *
     * 切记！
     *  phar包内源码引用php需要这样
     * require_once "phar://test.phar/function.php";
     * phar://包名/class.php
     */
