# 加密

## 对称加密

采用单钥密码系统的加密方法，同一个密钥可以同时用作信息的加密和解密，这种加密方法称为对称加密，也称为单密钥加密。

### AES 加密

对称密钥加密中最流行的算法之一，Advanced Encryption Standard，安全级别不高的自动登录一般用AES加密，AES加密是比较快的。 [动画演示](http://coolshell.cn/articles/3161.html)

#### 代码

#### 生成key

```text
byte[] key = (SALT2 + username + password).getBytes("UTF-8");
MessageDigest sha = MessageDigest.getInstance("SHA-1");
key = sha.digest(key);
key = Arrays.copyOf(key, 16); // use only first 128 bit
SecretKeySpec secret = new SecretKeySpec(key, "AES");
```

#### 加密

```text
/* Encrypt the message. */
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, secret);
AlgorithmParameters params = cipher.getParameters();
byte[] iv = params.getParameterSpec(IvParameterSpec.class).getIV();
byte[] ciphertext = cipher.doFinal("Hello, World!".getBytes("UTF-8"));
```

#### 解密

```text
/* Decrypt the message, given derived key and initialization vector. */
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, secret, new IvParameterSpec(iv));
String plaintext = new String(cipher.doFinal(ciphertext), "UTF-8");
System.out.println(plaintext);
```

## 非对称加密

需要两个密钥来进行加密和解密，这两个秘钥是公开密钥（public key，简称公钥）和私有密钥（private key，简称私钥）。

### RSA 加密

非对称加密,在公开密钥加密和电子商业中RSA被广泛使用，如github 传输到服务器的代码用的ssl 就是 rsa加密，RSA加密是比较慢的。 公钥：加密 私钥：解密 [详解](http://blog.csdn.net/bbld_/article/details/38777491)

## md5散列码

MD5 是 Message-Digest Algorithm 5（消息摘要算法第五版）的简称。 是一种加密算法。它的返回值是一个32位长度的字符串。 md5是散列函数，提取数据的特征，输出是不可逆的散列值，用于代表某信息A而又不暴露信息A的内容。

### 特点

#### 1. 和内容绑定的加密算法，就是说内容变，结果一定变。

这一特点可以用于内容验证，防止内容被攥改。俗称MD5校验。应用非常广泛。 如：下载Android 源码后，如果不放心，就可以进行一下MD5校验。

**2. MD5 的算法是不可逆的。**

这一特点可用于加密。应用非常广泛。 如： 用户的QQ密码，由于安全需要不能明文传输。就可以用MD5加密（或多次加密）后传输和保存密文。 android手机的唯一标识，用户不希望明文传输，也可以用MD5加密（或多次加密）后传输和保存密文。

#### 代码

```text
byte[] bytesOfMessage = yourString.getBytes("UTF-8");
MessageDigest md = MessageDigest.getInstance("MD5");
byte[] thedigest = md.digest(bytesOfMessage);
```

## base 64

base64的作用不是加密，而是用来避免“字节”中不能转换成可显示字符的数值。由于某些系统中只能使用ASCII字符。Base64就是用来将非ASCII字符的数据转换成ASCII字符的一种方法。 Base64中是一种基于64个可打印字符来表示二进制数据的表示方法。Base64中的可打印字符包括字母A-Z、a-z、数字0-9，这样共有62个字符，此外两个可打印符号在不同的系统中而不同。

### 代码

```text
String str = "Hello!";     

// 加密 
String enToStr = Base64.encodeToString(str.getBytes(), Base64.DEFAULT);  

// 解密  
String str = new String(Base64.decode(enToStr.getBytes(), Base64.DEFAULT)));
```

