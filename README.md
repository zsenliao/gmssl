GMSSL
========
GmSSL是一个开源的加密包的python实现，支持SM2/SM3/SM4等国密(国家商用密码)算法、项目采用对商业应用友好的类BSD开源许可证，开源且可以用于闭源的商业应用。

### 安装

```shell
pip install gmssl
```

### SM2算法
RSA算法的危机在于其存在亚指数算法，对ECC算法而言一般没有亚指数攻击算法
SM2椭圆曲线公钥密码算法：我国自主知识产权的商用密码算法，是ECC（Elliptic Curve Cryptosystem）算法的一种，基于椭圆曲线离散对数问题，计算复杂度是指数级，求解难度较大，同等安全程度要求下，椭圆曲线密码较其他公钥算法所需密钥长度小很多。

gmssl是包含国密SM2算法的Python实现， 提供了 `encrypt`、 `decrypt`等函数用于加密解密， 用法如下：

#### 1. 初始化`CryptSM2`

```python
import base64
import binascii
from gmssl import sm2, func
#16进制的公钥和私钥
private_key = '00B9AB0B828FF68872F21A837FC303668428DEA11DCD1B24429D0C99E24EED83D5'
public_key = 'B9C9A6E04E9C91F7BA880429273747D7EF5DDEB0BB2FF6317EB00BEF331A83081A6994B8993F3F5D6EADDDB81872266C87C018FB4162F5AF347B483E24620207'
sm2_crypt = sm2.CryptSM2(
    public_key=public_key, private_key=private_key)
# 对接java 时验签失败可以使用
sm2_crypt = sm2.CryptSM2(
    public_key=public_key, private_key=private_key, asn1=True)
```

#### 2. `encrypt`和`decrypt`

```python
#数据和加密后数据为bytes类型
data = b"111"
enc_data = sm2_crypt.encrypt(data)
dec_data =sm2_crypt.decrypt(enc_data)
assert dec_data == data
```

#### 3. `sign`和`verify`
```python
data = b"111" # bytes类型
random_hex_str = func.random_hex(sm2_crypt.para_len)
sign = sm2_crypt.sign(data, random_hex_str) #  16进制
assert sm2_crypt.verify(sign, data) #  16进制
```

#### 4. `sign_with_sm3`和`verify_with_sm3`

```python
data = b"111" # bytes类型
sign = sm2_crypt.sign_with_sm3(data) #  16进制
assert sm2_crypt.verify_with_sm3(sign, data) #  16进制
```


### SM4算法

国密SM4(无线局域网SMS4)算法， 一个分组算法， 分组长度为128bit， 密钥长度为128bit，
算法具体内容参照[SM4算法](https://drive.google.com/file/d/0B0o25hRlUdXcbzdjT0hrYkkwUjg/view?usp=sharing)。

gmssl是包含国密SM4算法的Python实现， 提供了 `encrypt_ecb`、 `decrypt_ecb`、 `encrypt_cbc`、
`decrypt_cbc`等函数用于加密解密， 用法如下：

#### 1. 初始化`CryptSM4`

```python
from gmssl.sm4 import CryptSM4, SM4_ENCRYPT, SM4_DECRYPT

key = b'3l5butlj26hvv313'
value = b'111' #  bytes类型
iv = b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00' #  bytes类型
crypt_sm4 = CryptSM4(padding_mode=3)    # 默认填充为”3-PKCS7“



```
> 原项目中SM4算法使用的是PKCS7填充算法，因开发需求，添加填充0算法
>
> 在原有项目中，扩充了NoPadding、ISO9797M2、PBOC三种填充算法。对于原有的‘padding_mode’取值进行了重新排序。

通过padding_mode参数完成加解密数据填充算法的选择：
+ 0：NoPadding，不做填充处理，需要保证传入数据字节长度为16的倍数。 
+ 1：ZERO，遵循《ANSI X99/X9.19》标准，传入数据补位0x00至16倍数长度。 
+ 2：ISO9797M2，遵循《ISO/IEC 9797-1》标准，传入数据不足16字节，强制补0x80再用0x00补齐16的倍数。当数据为16倍数时，同样需要强制填充1字节0x80，再补15字节0x00。 
+ 3：PKCS7，传入数据不足16字节，需要填充n个字节，补位内容就是n，例如缺5字节，补位“0505050505”。当数据为16倍数时，需要补充16字节‘10’。 
+ 4：PBOC，参考PBOC2018规范第7部分11.1.1章节。如果数据长度不是16的整数倍，则填充1字节0x80，再填充0x00到分组长度的整数倍。如果数据长度是分组长度的整数倍则不填充。

#### 2. `encrypt_ecb`和`decrypt_ecb`

```python

crypt_sm4.set_key(key, SM4_ENCRYPT)
encrypt_value = crypt_sm4.crypt_ecb(value) #  bytes类型
crypt_sm4.set_key(key, SM4_DECRYPT)
decrypt_value = crypt_sm4.crypt_ecb(encrypt_value) #  bytes类型
assert value == decrypt_value

```


#### 3. `encrypt_cbc`和`decrypt_cbc`

```python

crypt_sm4.set_key(key, SM4_ENCRYPT)
encrypt_value = crypt_sm4.crypt_cbc(iv , value) #  bytes类型
crypt_sm4.set_key(key, SM4_DECRYPT)
decrypt_value = crypt_sm4.crypt_cbc(iv , encrypt_value) #  bytes类型
assert value == decrypt_value

```
