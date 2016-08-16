### moco server 配置方法


- **首先启动moco server** 
 
- moco server 解析json文件

解析json数据后以格式化输出前端代码展示。
代码如下：

 ``` 
 function jsonFormat($data, $indent=null){

    // 将urlencode的内容进行urldecode
    $data = urldecode($data);
    //$data= urlencode($data);
    // 缩进处理
    $ret = '';
    $pos = 0;
    $length = strlen($data);
    $indent = isset($indent)? $indent : '    ';
    $newline = "\n";
    $prevchar = '';
    $outofquotes = true;

    for($i=0; $i<=$length; $i++){

        $char = substr($data, $i, 1);

        if($char=='"' && $prevchar!='\\'){
            $outofquotes = !$outofquotes;
        }elseif(($char=='}' || $char==']') && $outofquotes){
            $ret .= $newline;
            $pos --;
            for($j=0; $j<$pos; $j++){
                $ret .= $indent;
            }
        }

        $ret .= $char;

        if(($char==',' || $char=='{' || $char=='[') && $outofquotes){
            $ret .= $newline;
            if($char=='{' || $char=='['){
                $pos ++;
            }

            for($j=0; $j<$pos; $j++){
                $ret .= $indent;
            }
        }

        $prevchar = $char;
    }

    return $ret;
}
$jsonData =  jsonFormat($_POST["message"]);
echo decodeUnicode($jsonData);
//echo $jsonData;
//urdecode 方法
function decodeUnicode($str)
{
    return preg_replace_callback('/\\\\u([0-9a-f]{4})/i',
        create_function(
            '$matches',
            'return mb_convert_encoding(pack("H*", $matches[1]), "UTF-8", "UCS-2BE");'
        ),
        $str);
}
```
以上是代码便可完成json的格式解析,以解析后的json格式显示，==但需注意需要PHP安装mb_convert_encoding的PHP库==，否则报==call to undefined function mb_convert_encoding in。。。的错误==


- **moco server 的启动方式**
-moco Server 启动方式以及json文件的基本配置，如下
   
    详情见++https://github.com/dreamhead/moco++

- 针对业务的json文件布局，以及方法
   
   主要设置为三个文件：
    
1. settngs.json文件，此文件为全局设置文件，用来存放不同的模块uri集合文件，结构如下：其中的include的json文件，分别为不同的模块文件。
   

```   
       [
    {
        "include" : "index.json"
    },
    {

    	"include" :"second.json"
    },
    {

    	"include":"third.json"
    }
        。。。。。。
]
       
```

2. index.json\second.json\third.json这三个列举的文件，分别为不同模块的uri集合，以index.json文件举例，可以设置一个模块下的所有接口uri到一个json文件下，这样便可以模块管理，结构如下：
  ```
    [
    {
        "request" : {
            "uri": 
        {
           "match": "/android/dynamic.php.*mod=mob&ctl=home55.*"
        }
        }, 
        "response" : {
            "file" : "file.json"
        }
    },
    {
        "request" : {
            "uri" : "/test1"
        },
        "response" : {
            "file" : "file1.json"
        }
    },
    {
        "request" : {
            "uri": 
        {
           "contain": "mod=mob&ctl=home55"
        }
        }, 
        "response" : {
            "file" : "file.json"
        }
    }
]
```
3. 在模块文件中，分别设置每个接口的uri,从而在URL中请求server，server根据uri寻找与之对应的uri,获取不同的接口json数据，数据结构如下，以一个接口uri请求的file json 文件为例，如下：
```
{
    "header": {
        "status": "2",
        "markid": "f1e0779e06edd1c5d33c1856c4450ac9"
    },
    "body": {
        
    }
}
```
==file json文件直接存放的为json接口假数据，直接可以返回想要的数据==



  