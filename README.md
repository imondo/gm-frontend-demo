# gm-frontend-demo
国密加解密

## 前言

由于公司业务需要国密加解密，查遍全网也只找到 [https://blog.csdn.net/taisuiyu6397/article/details/107086902/](https://blog.csdn.net/taisuiyu6397/article/details/107086902/) 这位博主自己写的算法；没办法，谁叫我算法差 😭

## 注意

不过在使用的时候还是碰到了一些问题

- **前后端密钥需要一致**，可是全网大部分解决方案都是密钥不一致，因为前后端的实现方式不同，导致密钥的算法也不一样

- 解密时需要注意后端返回的数据是否有**换行符**（检查了好久😥）

- [SM2 加解密](https://github.com/imondo/gm-crypto)前后端密钥生成不一致，所以采用 SM4 加密，SM3 对称比较方法

## 加解密流程

- 前端对传输参数 SM4 加密，且加密后字符串与当前时间戳 SM3 加密形成如下参数值传输

```json
{
 id: SM4(id)

 name: SM4(name)

 date: new Date().getTime()
}
```

或者

```json
{

  id: id,
  name: name,
  date: new Date().getTime(),
  encryptStr: SM3(id + name + date)
}
```

- 后端接受参数依次像前端一样加密再与前端 `encryptStr` 对比是否一样

## 用法

```js
<script src="./crypto/base64.js"></script>
<script src="./utils/hex.js"></script>
<script src="./utils/byteUtil.js"></script>
<script src="./crypto/sm4.js"></script>

const key = '1234567890123456' // 密钥需要前后端一致
const msg = {
  pageResult: {
    endRow: 10, firstPage: true, hasNextPage: true, hasPrePage: false, index: 0,
    rows: [
      {
        account: "admin",
        admin: true,
        enable: true,
        extend1: 0,
        extend2: 0,
        extend3: "2021-08-22 15:21:02",
        from: "system",
        fullname: "系统管理员",
        groupId: "",
        id: "1",
        identityType: "user",
        orgList: [],
        preLoginTime: 1629614882000,
        roleList: [],
        status: 1,
        userId: "1",
      }
    ],
    total: 12
  }
}
const encodeData = encode({
  input: msg,
  key
})
console.log(encodeData)
console.log(decode({ input: encodeData, key }))
// 加密
function encode({ input, key }) {
  const _input = JSON.stringify(input) // 需要对加密数据序列化
  let decryptedData = SM4.encode({
    input: _input,
    key: key
  })
  return decryptedData
}

// 解密
function decode({ input, key }) {
  const _input = input.replace(/\r\n/g, "") // 去掉可能存在的换行符
  let decryptedData = SM4.decode({
    input: _input,
    key: key
  })
  return JSON.parse(decryptedData.replace(/[\u0000]+./g, '')) // 由于算法补全，需要去掉 [\u0000]
}

```
