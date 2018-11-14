1：2.1.3 小节

基本命令：

```
# 查看 openssl 版本号
$ openssl version

# 查看 openssl 帮助
$ openssl help 

# 查看某个命令的帮助
$ openssl rsa --help
```

构建特定版本的 OpenSSL 

```
#下载二进制包并解压缩
$ wget https://www.openssl.org/source/openssl-1.1.0f.tar.gz

$ tar xvf openssl-1.1.0f.tar.gz 
$ cd openssl-1.1.0f 

#查看安装手册
$ more INSTALL

#查看config 
./config --help

#安装并编译，安装目录不要和系统目录冲突
$ ./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl 

$ make 
$ make test 
$ make install 
$ make clean

#运行安装的 OpenSSL 
$ /usr/local/openssl/bin/openssl version
OpenSSL 1.1.0f  25 May 2017
```

2：2.2.3 小节

shell 命令生成随机数：

```
# 使用外部熵生成随机数
$ head -c 32 /dev/urandom | openssl enc -base64

# openssl 工具生成随机数
$ openssl rand -base64 24
```

PHP 函数生成随机数：

```
//初始化种子
mt_srand();
$randval = mt_rand();
echo $randval;
```

3：2.4.4 小节

（1）OpenSSL 命令行应用 AES 算法：

```
//显示 enc 子命令帮助
$ openssl enc --help

# 采用 3des 算法 
$ openssl enc des3 

#采用 3des 算法
$ openssl des3 

#显示系统支持的加密算法
$ openssl list -cipher-algorithms

$ echo "hello" >> file.txt
#执行加密参数
$ openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass pass:mypassword -p
#执行解密操作
$ openssl enc -d -aes-256-cbc -in file.enc 
```

（2）PHP 语言应用 AES 算法 

```
class AES128Encryptor
{
    // 128 表示的是分组长度，可以用 MCRYPT_RIJNDAEL_128 表示 AES 算法 
    private $_cipher = MCRYPT_RIJNDAEL_128; 
    
    // 分组模式 
    private $_mode = MCRYPT_MODE_CBC;

    // 密钥 
    private $_key;

    // 初始化向量长度 
    private $_ivSize;

    // 构造函数
    public function __construct($key)
    {
        $this->_key = $key;

        // 可以从算法计算出初始化向量长度
        $this->_ivSize = mcrypt_get_iv_size($this->_cipher, $this->_mode);

        // 获取特定算法和分组模式对应的密钥长度 
        $keyMaxLen = mcrypt_get_key_size($this->_cipher, $this->_mode);

        // 如果输入的密钥长度不合法，则直接报错
        if (strlen($key) > $keyMaxLen) {
            throw new Exception("error");
        }
    }

    // 加密数据
    public function encrypt($data)
    {   
        // 获取特定算法和分组模式的分组长度 
        $blockSize = mcrypt_get_block_size($this->_cipher, $this->_mode);

        // PKCS#7 填充标准
        $pad = $blockSize - (strlen($data) % $blockSize);

        // 生成随机的初始化向量
        $iv = mcrypt_create_iv($this->_ivSize, MCRYPT_DEV_URANDOM);

        // 密文包含初始化向量，而且是不加密的
        return $iv . mcrypt_encrypt(
            $this->_cipher,
            $this->_key,
            $data . str_repeat(chr($pad), $pad),
            $this->_mode,
            $iv
        );
    }

    // 解密
    public function decrypt($encryptedData)
    {
        // 从密文中获取初始化向量，初始化向量的长度等于分组长度
        $iv = substr($encryptedData, 0, $this->_ivSize);

        $data =  mcrypt_decrypt(
            $this->_cipher,
            $this->_key,
            substr($encryptedData, $this->_ivSize),
            $this->_mode,
            $iv
        );

        // 去除填充
        $pad = ord($data[strlen($data) - 1]);
        return substr($data, 0, -$pad);
    }
}

// 密钥基于口令使用 Hash 算法生成，后续章节会介绍口令和 Hash 算法的概念。 
$hash = hash('SHA256', "password", true);

// 密钥长度是 128 比特
$key = substr($hash, 0, 16); 

$obj = new AES128Encryptor($key);
// 加密 
$d = $obj->encrypt("hello");

// 解密
echo $obj->decrypt($d);
```

4：2.5.3 小节

（1）通过 PHP 演示如何使用 HMAC 算法

```
//在消息上附加 MAC 值 
function hmacSign($message, $key){
    return hash_hmac('sha256', $message, $key) . $message;
}

function hmacVerify($bundle, $key){   
    //取出 HMAC 值
    $mac = mb_substr($bundle, 0, 64, '8bit');

    //取出原始消息
    $message = mb_substr($bundle, 64, null, '8bit');

    //进行比较
    return hash_equals(
        hash_hmac('sha256', $message, $key),
        $mac
    );
}

$ciphertext = "hello";
$key = "000102030405060708090a0b0c0d0e0f";
$message = hmacSign($ciphertext, $key);
var_dump(hmacVerify($message, $key));
```

（2）OpenSSL 演示 HMAC 算法


操作 Hash 算法：

```
$ echo "hello" >>file.txt
# 对文件 file.txt 通过 SHA-1 算法计算摘要值 
$ openssl dgst -sha1 file.txt 

# 对文件 file.txt 通过 SHA-1 算法计算摘要值 ，并将摘要值保存到文件 digest.txt
$ openssl sha1 -out digest.txt file.txt
```

使用 OpenSSL 生成 HMAC 值：

```
# 使用 HMAC-SHA-1 算法，结合密钥 "mykey" 对文件内容进行运算并生成 HMAC 值 
$ echo "hello" >>file.txt
$ openssl dgst -sha1 -hmac "mykey" file.txt
```

5：2.5.4 小节

（1）举例说明加密算法不能保证消息完整性，下面是破解代码

```
// 密钥 
$key = "000102030405060708090a0b0c0d0e0f";
$obj = new AES128Encryptor($key);
$ciphertext = $obj->encrypt(1);

// 对生成的密文值进行 Base64 编码
$ciphertext = base64_encode($ciphertext) ;
 
//构建替代字符，通过暴力破解方式去修改密文
for ($i=41;$i<=90;$i++) {
    //a-z 字符 
    $s[] =sprintf("%c",$i);
}

//构建替代字符
for ($i=97;$i<=122;$i++) {
    //A-Z 字符
    $s[] =sprintf("%c",$i);
}
// = 和 + 两个字符 
$s[] = "=";
$s[] = "+";

for ($i=0;$i<54;$i++) {
    for ($j=0;$j<23;$j++) {
        $rd = $ciphertext;
        // 对密文进行篡改，每次修改一位 
        $rd[$j] = $s[$i];   

        if ($obj->decrypt(base64_decode($rd)) == 1) {
            echo "成功篡改：" . $ciphertext . "_" . $rd . "\n" ;
            exit;
        }
    }
}
```

（2）保证消息同时具备机密性和完整性

```
// 加密密钥 
$key = "000102030405060708090a0b0c0d0e0f";

// MAC 算法密钥，一般不同于加密密钥
$mackey = "2510c39011c5be704182423e3a695e91";

$obj = new AES128Encryptor($key);

$plaintext = "hello";

// 加密值
$ciphertext = $obj->encrypt($plaintext);

// MAC 值 
$mac = hash_hmac("sha256",$plaintext,$mackey);

//通信的消息包含加密值和 MAC 值 
$message = base64_encode($ciphertext . $mac) ;
echo $message;
```

6：2.6.4 小节

（1）使用 OpenSSL 演示 RSA 算法

使用 genrsa 子命令生成密钥对，密钥对是一个文件：

```
# 生成的密钥长度是 2048 比特 
$ openssl genrsa -out mykey.pem 2048

# 口令结合 3DES 算法保护密钥对
$ openssl genrsa -des3 -out mykey2.pem 2048
```

从密钥对中分离出公钥：

```
$ openssl rsa -in mykey.pem  -pubout -out mypubkey.pem

# 需要输入口令才能分离公钥
$ openssl rsa -in mykey2.pem -pubout -out mypubkey2.pem
```

校验密码对文件是否正确：

```
$ openssl rsa -in mykey.pem -check -noout
```

显示公钥信息：

```
$ openssl rsa  -pubin -in mypubkey.pem -text 
```

使用 rsautl 子命令进行数据加解和解密 ：

```
$ echo "hello" >>plain.txt

# 使用密钥对加密，rsautl 命令默认的填充机制是 PKCS#1 v1.5
$ openssl rsautl -encrypt -inkey mykey.pem -in plain.txt -out cipher.txt

# 使用公钥加密，务必有 -pubin 参数表明 -inkey 参数输入的是公钥文件
$ openssl rsautl -encrypt -pubin  -inkey mypubkey.pem -in plain.txt -out cipher.txt

# 解密
$ openssl rsautl -decrypt  -inkey mykey.pem -in cipher.txt  
```

（2）使用 PHP 演示 RSA 算法

```
// 原始值 
$data   = "hello";

// 加载公钥文件
$pkeyid = openssl_pkey_get_public("file://mypubkey.pem");

// 使用 PKCS#1 v1.5 标准加密数据
$bool = openssl_public_encrypt( $data, $encdata, $pkeyid, OPENSSL_PKCS1_PADDING );
openssl_free_key($pkeyid);

// 加载私钥文件
$pkeyid = openssl_pkey_get_private("file://mykey.pem");

// 遵循同样的标准解密文件
openssl_private_decrypt($encdata, $descdata, $pkeyid, OPENSSL_PKCS1_PADDING);
openssl_free_key($pkeyid);

// 输出原始值和解密值
echo $data."_".$descdata;
```

7：2.7.2 小节

（1）PHP 演示如何使用 PBKDF2 函数

```
$password = 'yOuR-pAs5w0rd-hERe';
// 随机的盐值
$salt = openssl_random_pseudo_bytes(12);
$keyLength = 40;
$iterations = 10000;
$generated_key = openssl_pbkdf2($password, $salt, $keyLength, $iterations, 'sha256');
//转换为 16 进制
echo bin2hex($generated_key)."\n";
echo base64_encode($generated_key)."\n";
```

（2）OpenSSL 演示口令和 Salt

```
$ echo "hello">>file.txt
$ openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass pass:password -p

# 口令保护 RSA 密钥对
$ openssl genrsa -des3 -out mykey2.pem 2048
```

8：2.8.4 小节

（1）dhparam 和 genpkey 子命令介绍

```
#生成一个 1024 比特的参数文件
$ openssl dhparam -out dhparam.pem -2 1024

#查看参数文件内容
$ openssl dhparam -in dhparam.pem -noout -C 

#基于参数文件生成密钥对
$ openssl genpkey -paramfile dhparam.pem -out dhkey.pem 
 
#查看密钥对文件内容
$ openssl pkey -in dhkey.pem -text -noout
```

（2）完整的 DH 协商例子

```
# 通信双方的任何一方生成 DH 的参数文件，可以对外公开
$ openssl genpkey -genparam -algorithm DH -out dhp.pem 

# 查看参数文件的内容，包括 p 和 g 等参数
$ openssl pkeyparam -in dhp.pem -text

# 发送方 A 基于参数文件生成一个密钥对 
$ openssl genpkey -paramfile dhp.pem -out akey.pem
# 查看密钥对内容
$ openssl pkey -in akey.pem -text -noout

# 发送方 B 基于参数文件生成一个密钥对 
$ openssl genpkey -paramfile dhp.pem -out bkey.pem
# 查看密钥对内容
openssl pkey -in bkey.pem -text -noout

# 发送方 A 拆出公钥文件 akey_pub.pem ，私钥自己保存 
$ openssl pkey -in akey.pem -pubout -out akey_pub.pem 
 
# 发送方 B 拆出公钥文件 bkey_pub.pem ，私钥自己保存 
$ openssl pkey -in bkey.pem -pubout -out bkey_pub.pem 

# 发送方 A 收到 B 发送过来的公钥，将协商出的密钥保存到 data_a.txt 文件
$ openssl pkeyutl -derive -inkey akey.pem -peerkey bkey_pub.pem -out data_a.txt

# 发送方 B 收到 A 发送过来的公钥，将协商出的密钥保存到 data_b.txt 文件
$ openssl pkeyutl -derive -inkey bkey.pem -peerkey akey_pub.pem -out data_b.txt
```

9：2.9.2 小节

了解 ECC 曲线：

```
# 查看系统有多少椭圆曲线，通信双方需要选择一条都支持的命名曲线
$ openssl ecparam -list_curves

# 生成一个参数文件，通过 -name 指定命名曲线
$ openssl ecparam -name secp256k1 -out secp256k1.pem

# 默认的情况下，查看参数文件只会显示曲线的名称
$ openssl ecparam -in secp256k1.pem -text -noout 
ASN1 OID: secp256k1 

# 显示参数文件参数
$ openssl ecparam -in secp256k1.pem -text -param_enc explicit -noout
```

10：2.9.3 小节

ECDH 算法演示：

```
# 生成参数文件 
$ openssl ecparam -name secp256k1 -out secp256k1.pem 

# A 和 B 各自生成一个 ECDH 私钥对文件
$ openssl genpkey -paramfile secp256k1.pem  -out akey.pem 
$ openssl genpkey -paramfile secp256k1.pem  -out bkey.pem 

# A 和 B 分解出公钥，将公钥发送给对方
$ openssl pkey -in akey.pem -pubout -out akey_pub.pem 
$ openssl pkey -in bkey.pem -pubout -out bkey_pub.pem 

# A 和 B 计算出同样的会话密钥
$ openssl pkeyutl -derive -inkey akey.pem -peerkey bkey_pub.pem -out data_a.txt
$ openssl pkeyutl -derive -inkey bkey.pem -peerkey akey_pub.pem -out data_b.txt 
```

11：2.9.4 小节

显示系统存在的命名曲线：

```
$ openssl ecparam -list_curves
```

12：2.10.4 小节

（1）使用 OpenSSL 演示数字签名算法

使用 RSAES-PKCS1-V1_5 标准：

```
# 生成一个 RSA 密钥对，密钥长度 1024 比特 
$ openssl genrsa -out rsaprivatekey.pem 1024  

# 从密钥对中分离出公钥
$ openssl rsa -in rsaprivatekey.pem -pubout -out rsapublickey.pem

# 对 plain.txt 文件使用 sha256 Hash 算法和签名算法生成签名文件 signature.txt
$ echo "hello" >>plain.txt 
$ openssl dgst -sha256  -sign rsaprivatekey.pem -out  signature.txt  plain.txt 

# 用相同的摘要算法和签名算法校验签名文件，需要对比签名文件和原始文件
$ openssl dgst -sha256  -verify rsapublickey.pem  -signature signature.txt plain.txt 
```

使用 RSASSA-PSS 标准：

```
# 生成 RSA 密钥对 
$ openssl genrsa -out rsaprivatekey.pem 1024  
$ openssl rsa -in rsaprivatekey.pem -pubout -out rsapublickey.pem

# 指定 RSASSA-PSS 标准 
$ openssl dgst -sha256  -sign rsaprivatekey.pem  -sigopt rsa_padding_mode:pss -out  signature.txt  plain.txt 

# 验证签名 
$ openssl dgst -sha256  -verify rsapublickey.pem -sigopt rsa_padding_mode:pss  -signature signature.txt plain.txt
```

（2）PHP 演示数字签名

```
// 签名和验签函数
// $data 表示原始信息
// $privatekeyFile 表示私钥文件，$publickeyFile 表示公钥文件
function sign_verify($data, $privatekeyFile, $publickeyFile)
{
    //口令，有的时候密钥对文件被口令和对称加密算法保护了
    $passphrase = '';

    //摘要算法
    $digestAlgo = 'sha512';

    //加载私钥文件
    $privatekey = openssl_pkey_get_private(file_get_contents($privatekeyFile), $passphrase);

    //最终生成的签名值
    $signature = '';

    //生成签名
    openssl_sign($data, $signature, $privatekey, $digestAlgo);

    //释放资源
    openssl_free_key($privatekey);

    //对签名值 base64 编码，以便在网络上传输
    $signature = base64_encode($signature);

    //加载公钥
    $publickey = openssl_pkey_get_public(file_get_contents($publickeyFile));
     
    //验签
    $verify = openssl_verify($data, base64_decode($signature), $publickey, $digestAlgo);

    //释放资源
    openssl_free_key($publickey);

    return $verify;
}

//RSA 私钥和公钥文件位置
$rsaprivatekey  =  dirname(__FILE__) . "/rsaprivatekey.pem";
$rsapublickey  =  dirname(__FILE__) . "/rsapublickey.pem";

//假如返回 1 表示验证成功
var_dump(sign_verify("hello", $rsaprivatekey, $rsapublickey));

```

13：2.11.2 小节

生成参数文件和密钥对文件：

```
# 生成参数文件，类似于 DH 参数文件一样
$ openssl dsaparam -out dsaparam.pem 1024

# 通过参数文件生成密钥对 dsaprivatekey.pem  
$ openssl gendsa -out dsaprivatekey.pem dsaparam.pem

# 对私钥使用 des3 算法进行加密
$ openssl gendsa -out dsaprivatekey2.pem -des3 dsaparam.pem

# 通过密钥对文件拆分出公钥
$ openssl dsa -in dsaprivatekey.pem -pubout -out dsapublickey.pem 
```

显示文件的信息:

```
# 包括三个公共参数、公钥、私钥 
$ openssl dsa -in dsaprivatekey.pem -text 

# 公钥信息
$ openssl dsa  -pubin -in dsapublickey.pem -text 
```

生成和验证签名：

```
$ echo "hello" >plain.txt

# 进行签名，可以查看 signature.txt 可以发现有原始信息 hello 
$ openssl dgst -sha256 -sign dsaprivatekey.pem -out signature.txt plain.txt 

# 验证签名
$ openssl dgst -sha256 -verify dsapublickey.pem -signature signature.txt plain.txt 
Verified OK
```

14：2.11.4 小节

（1）OpenSSL 演示 ECDSA 算法

直接生成 ECDSA 私钥：

```
$ openssl ecparam -name secp256k1 -genkey -out ecdsa_priv.pem 
```

显示私钥信息：

```
$ openssl ec -in ecdsa_priv.pem -text -noout
```

拆分密钥对获取公钥：

```
# 生成私钥对应的公钥
$ openssl ec -in ecdsa_priv.pem -pubout -out ecdsa_pub.pem

# 显示公钥信息
$ openssl ec -in ecdsa_pub.pem -pubin -text -noout
```

生成签名值：

```
$ echo "hello" >>plain.txt 

# Hash 算法使用 sha256 算法 
$ openssl dgst -sha256 -sign ecdsa_priv.pem -out signature.txt plain.txt 
```

校验签名： 

```
$ openssl dgst -sha256 -verify ecdsa_pub.pem -signature signature.txt plain.txt 
Verified OK
```

（2）PHP 演示 ECDSA 算法

使用 PHP 语言生成 ECDSA 密钥对和签名 :

```
//对于 ECDSA 来说，增加了 curve_name 命名曲线参数
$config = array(
    "private_key_bits" => 1024,
    "private_key_type"=>OPENSSL_KEYTYPE_EC,
    "curve_name"=>openssl_get_curve_names()[0],
);
     
//生成密钥对
$new_key_pair  = openssl_pkey_new($config);

//从密钥对中导出私钥 
openssl_pkey_export($new_key_pair, $ecdsaprivate_key);

//从密钥对中导出公钥 
$details = openssl_pkey_get_details($new_key_pair);
$ecdsapublic_key = $details['key'];

//将私钥写入文件
$ecdsaprivatekey  =  dirname(__FILE__) . "/ecdsaprivatekey.pem";
file_put_contents($ecdsaprivatekey, $ecdsaprivate_key);

//将公钥写入文件
$ecdsapublickey  =  dirname(__FILE__) . "/ecdsapublickey.pem";
file_put_contents($ecdsapublickey, $ecdsapublic_key);
```

校验签名：

```
var_dump(sign_verify("hello", $ecdsaprivatekey, $ecdsapublickey));
```

15：2.12.2 小节

（1）OpenSSL speed 子命令参数


```
$ openssl speed aes-128-cbc
$ openssl speed -evp  aes-128-cbc

显示非 evp 模式可以运行的密码学算法
$ openssl list -cipher-commands

# evp 模式
$ openssl speed -evp aes-128-gcm

$ openssl speed -multi 2
```

（2）对称加密算法性能测试

```
$ openssl speed -evp aes-128-gcm
```

比较不同对称加密算法的运算速度：

```
$ openssl speed -evp rc4
$ openssl speed -evp des-ede3-cbc
$ openssl speed -evp seed-cbc 
$ openssl speed -evp camellia-128-cbc 
$ openssl speed -evp aes-128-cbc 
```

（3）AES 不同密钥长度的性能测试 

```
$ openssl speed -evp aes-128-cbc 
$ openssl speed -evp aes-192-cbc 
$ openssl speed -evp aes-256-cbc 
```

（4）AES-NI 指令集测试

查看机器是否支持 AES-NI 指令集：

```
$ cat /proc/cpuinfo | grep aes
```

测试：

```
# 禁止使用 AES-NI 指令集
$ OPENSSL_ia32cap="~0x200000200000000" openssl speed -evp aes-128-cbc 

# 使用 AES-NI 指令集
$ openssl speed -evp aes-128-cbc
```

（5）加密模式的性能测试 

```
$ openssl speed -evp aes-128-cbc
$ openssl speed -evp aes-128-gcm 
$ openssl speed -evp chacha20-poly1305 
```

（6）公开密钥算法性能测试

```
$ openssl speed  rsa1024
```

（7）不同密钥长度的 RSA 加密解密测试

```
$ openssl speed  rsa
```

（8）不同数字签名算法性能测试 

```
$ openssl speed dsa 
$ openssl speed rsa 
$ openssl speed ecdsa
```

（9）密钥协商算法性能测试

```
$ openssl speed  ecdh 
```

（10）测试不同 Hash 算法性能 

```
$ openssl speed -evp md5
$ openssl speed -evp sha1 
$ openssl speed -evp sha256 
$ openssl speed -evp sha384 
$ openssl speed -evp sha512
```