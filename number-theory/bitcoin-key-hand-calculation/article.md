---
id: number-theory.bitcoin-key-hand-calculation
title: 一把 Bitcoin 钥匙是怎样算出来的？从私钥到地址和签名完整走一遍
slug: bitcoin-key-hand-calculation
domain: number-theory
level: bridge
status: published
summary: 在一条可手算的微型椭圆曲线上生成私钥、公钥和 ECDSA 签名，再用 Python 复现真实 secp256k1 公钥、WIF 与传统地址，逐步定位可逆编码、哈希丢失和离散对数难题。
estimated_minutes: 60
prerequisites:
  - number-theory.bitcoin-key-security
related:
  - discrete-math.ssh-hand-calculated-session
demos: []
---

## 从一张纸上的数字开始

这次我们不从“256 位很大”开始，而是亲手制造一把钥匙。

实验分成两个世界：

1. 在微型曲线 \(y^2=x^3+7\pmod{97}\) 上，用私钥 13 手算公钥和签名；
2. 在真实 secp256k1 上，用公开且极不安全的示例私钥 2，通过 Python 算出公钥、WIF 和传统 P2PKH 地址。

> **绝对不要向文中的地址转入 Bitcoin。** 私钥 2 已经在全文公开，任何人都能控制对应地址。这里的所有密钥只用于教学。

本文要追踪的不是一条含糊的“加密链”，而是：

\[
\text{随机私钥}
\longrightarrow
\text{椭圆曲线公钥}
\longrightarrow
\text{公钥哈希}
\longrightarrow
\text{地址文本},
\]

以及花费时的：

\[
\text{交易摘要}+\text{私钥}+\text{一次性数}
\longrightarrow
\text{签名}
\longrightarrow
\text{公钥验签}.
\]

每经过一步，我们都问三个问题：

- 能不能从右边精确还原左边？
- 如果不能，是因为信息真的丢了，还是因为已知算法太慢？
- 攻击者究竟缺少哪种数学工具？

## 先把“不可逆”拆成三种

日常语言里的“不可逆”很容易把三件事混为一谈。

### 第一种：完全可逆的编码

十六进制、Base58Check 和公钥压缩格式主要是编码。只要规则公开，任何人都能解码回来。

这类似把十进制 255 写成 FF：字符变了，信息没有丢。

### 第二种：数学上存在原值，但目前反求太慢

从私钥 \(d\) 算公钥 \(Q=dG\) 很快；从 \(Q\) 找 \(d\) 在真实参数上没有已知的快速经典算法。

这里不是信息凭空消失了。公钥确实对应某个私钥，困难在于没有足够快的求解工具。这就是椭圆曲线离散对数问题。

### 第三种：映射主动压缩，信息确实丢失

哈希把很长输入压成固定长度输出。理论上必然有许多输入得到同一输出，因此一般不存在唯一的逆函数。

攻击者能做的是寻找一个满足目标哈希的输入。这叫原像搜索，不是“调用哈希的解密函数”。

这三类必须分开，否则很容易把 Base58 当成加密，把哈希碰撞当成私钥恢复，或者把“尚无快速算法”说成“数学上绝对不能反向”。

## 手算舞台：模 97 的曲线

我们使用曲线

\[
E:\quad y^2\equiv x^3+7\pmod{97}.
\]

意思是点 \((x,y)\) 必须满足：\(y^2-(x^3+7)\) 能被 97 整除。

取基点

\[
G=(1,28).
\]

先检查它确实在曲线上：

\[
28^2=784\equiv8\pmod{97},
\]

\[
1^3+7=8.
\]

左右同余，所以 \(G\) 合法。

这条微型曲线一共有 79 个点，包括无穷远点 \(\mathcal O\)。79 是素数，因此任意非无穷远点都会生成整个群。私钥可以取

\[
1\le d\le78.
\]

这当然极不安全：攻击者最多试 78 次。它的优点是我们真的能看见每一步。

真实 secp256k1 也使用

\[
y^2\equiv x^3+7\pmod p,
\]

但 \(p\) 和基点阶 \(n\) 都是约 256 位的大数。微型实验保留曲线形状和群运算，舍弃安全规模。

## 点加法究竟怎样算？

普通实数曲线常画成光滑曲线，用直线交点理解点加法；有限域里图像散成离散点，但代数公式仍能工作。

若 \(P=(x_1,y_1)\) 与 \(Q=(x_2,y_2)\) 不同，斜率是

\[
\lambda=(y_2-y_1)(x_2-x_1)^{-1}\pmod{97}.
\]

这里的除法变成乘模逆元。新点 \(R=P+Q=(x_3,y_3)\) 为

\[
x_3=\lambda^2-x_1-x_2\pmod{97},
\]

\[
y_3=\lambda(x_1-x_3)-y_1\pmod{97}.
\]

若做倍点 \(P+P\)，斜率改为

\[
\lambda=(3x_1^2)(2y_1)^{-1}\pmod{97}.
\]

所谓 \(dG\)，就是在这套加法下把 \(G\) 加 \(d\) 次。程序使用“倍点加点”，因此只需约 \(\log_2 d\) 轮，不会真的加几百亿亿次。

## 第一幕：私钥 13 怎样变成公钥？

选择教具私钥

\[
d=13.
\]

公钥定义为

\[
Q=dG=13G.
\]

由于

\[
13=8+4+1,
\]

我们只需算 \(2G,4G,8G\)，再组合。

### 算 \(2G\)

对 \(G=(1,28)\) 倍点：

\[
(2y)^{-1}=56^{-1}\equiv26\pmod{97},
\]

因为

\[
56\times26=1456=15\times97+1.
\]

斜率为

\[
\lambda=3\cdot1^2\cdot26=78\pmod{97}.
\]

所以

\[
x_{2G}=78^2-1-1=6082\equiv68\pmod{97},
\]

\[
y_{2G}=78(1-68)-28=-5254\equiv81\pmod{97}.
\]

得到

\[
2G=(68,81).
\]

可以验点：

\[
81^2\equiv62\pmod{97},
\]

\[
68^3+7\equiv62\pmod{97}.
\]

### 算 \(4G\)

对 \(2G=(68,81)\) 倍点。因为

\[
(2\cdot81)^{-1}=162^{-1}\equiv65^{-1}\equiv3\pmod{97},
\]

斜率

\[
\lambda=3\cdot68^2\cdot3\equiv3\pmod{97}.
\]

于是

\[
x_{4G}=3^2-68-68=-127\equiv67\pmod{97},
\]

\[
y_{4G}=3(68-67)-81=-78\equiv19\pmod{97}.
\]

所以

\[
4G=(67,19).
\]

### 算 \(8G\)

对 \(4G=(67,19)\) 倍点：

\[
(2\cdot19)^{-1}=38^{-1}\equiv23\pmod{97},
\]

\[
\lambda=3\cdot67^2\cdot23\equiv20\pmod{97}.
\]

\[
x_{8G}=20^2-67-67=266\equiv72\pmod{97},
\]

\[
y_{8G}=20(67-72)-19=-119\equiv75\pmod{97}.
\]

所以

\[
8G=(72,75).
\]

### 组合成 \(13G\)

先算 \(12G=8G+4G\)。两点不同，所以

\[
\lambda=(19-75)(67-72)^{-1}
=(-56)(-5)^{-1}\pmod{97}.
\]

\[
(-5)^{-1}\equiv58\pmod{97},
\]

因此

\[
\lambda\equiv(-56)\cdot58\equiv50\pmod{97}.
\]

\[
x_{12G}=50^2-72-67=2361\equiv33\pmod{97},
\]

\[
y_{12G}=50(72-33)-75=1875\equiv32\pmod{97}.
\]

所以

\[
12G=(33,32).
\]

最后

\[
13G=12G+G.
\]

\[
\lambda=(28-32)(1-33)^{-1}
=(-4)(-32)^{-1}\pmod{97}.
\]

\[
(-32)^{-1}\equiv3\pmod{97},
\]

\[
\lambda\equiv-12\equiv85\pmod{97}.
\]

\[
x_{13G}=85^2-33-1=7191\equiv13\pmod{97},
\]

\[
y_{13G}=85(33-13)-32=1668\equiv19\pmod{97}.
\]

最终公钥是

\[
Q=(13,19).
\]

这不是巧合地“把 13 原样写进了横坐标”。换别的点或私钥不会保留这种表面关系；本例只是数值碰巧相等。

## 第一处真正的卡点：从 \(Q\) 怎样找回 \(d\)？

攻击者已知：

\[
G=(1,28),\qquad Q=(13,19),
\]

要解：

\[
Q=dG.
\]

在这个小群里，他可以列举：

\[
G,2G,3G,\ldots,78G,
\]

直到第 13 项遇见 \(Q\)。所以玩具私钥能被破解。

但真实 secp256k1 的基点阶是

\[
n=
\mathtt{FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141}_{16},
\]

大约有 \(2^{256}\) 个候选。通用 Pollard rho 类攻击的工作量约为

\[
\sqrt n\approx2^{128}
\]

次曲线群运算。

攻击者缺少的数学工具可以准确表述为：

> 一个能在 secp256k1 基点生成的群上，高效求解椭圆曲线离散对数的经典算法。

普通代数里的除法帮不上忙，因为 \(Q=dG\) 中的“乘法”是重复的曲线点加法，不是坐标分别乘 \(d\)。不存在

\[
d=\frac{x_Q}{x_G}
\]

这样的公式。

这不是已经证明“任何算法永远做不到”。准确说法是：公开研究至今没有已知的足够快经典算法；若未来可用的大规模容错量子计算机运行 Shor 算法，情况会改变。

## 第二幕：公钥怎样压缩成字符？

曲线上同一个横坐标通常对应两个纵坐标：

\[
y\quad\text{和}\quad -y\bmod p.
\]

在教具曲线上，公钥 \(Q=(13,19)\) 的另一个同横坐标点是

\[
(13,-19\bmod97)=(13,78).
\]

只要保存横坐标 13，再保存“纵坐标是奇数还是偶数”这一位，就能选回正确的 19。

真实压缩公钥使用：

- 前缀 02：纵坐标为偶数；
- 前缀 03：纵坐标为奇数；
- 后面 32 字节：横坐标。

因此“压缩公钥”并不是哈希。对合法曲线点，给定横坐标和奇偶位，可以通过曲线方程恢复纵坐标。它是**可逆编码**。

公钥压缩不会把私钥也压进去。解码只能恢复公钥点，仍然无法高效解决 \(Q=dG\)。

## 第三幕：从公钥到传统地址，哪里真的丢了信息？

为了给出一条具体路径，本文选择传统压缩公钥 P2PKH 地址。Bitcoin 还有 P2SH、SegWit、Taproot 等其他地址类型，变换规则并不相同。

传统 P2PKH 的核心链条是：

\[
\text{compressed\_pubkey}
\overset{\mathrm{SHA256}}{\longrightarrow}
h_1
\overset{\mathrm{RIPEMD160}}{\longrightarrow}
h_{160}.
\]

这通常写作

\[
\operatorname{HASH160}(P)
=\operatorname{RIPEMD160}(\operatorname{SHA256}(P)).
\]

随后构造：

\[
\text{payload}=\text{网络版本字节}\|\operatorname{HASH160}(P),
\]

\[
\text{checksum}
=\operatorname{SHA256}(\operatorname{SHA256}(\text{payload}))[0:4],
\]

最后把

\[
\text{payload}\|\text{checksum}
\]

做 Base58 编码。

逐项判断：

| 步骤 | 能否逆回去？ | 原因 |
| --- | --- | --- |
| Base58 文本 → 字节 | 能 | 只是进制编码 |
| 去掉版本和校验和 | 能 | 字段长度和格式公开 |
| HASH160 → 原公钥 | 一般不能有效完成 | 哈希压缩并依赖原像抗性 |
| 公钥 → 私钥 | 数学上有对应值，但无已知快速经典算法 | ECDLP |

HASH160 只有 160 位，而压缩公钥有 264 位。鸽巢原理已经告诉我们：不可能为每个公钥哈希保留唯一可逆结果。

但“存在碰撞”不等于“攻击者已找到能花费这笔钱的公钥”。攻击者需要找到一个满足指定目标哈希、又能配合自己私钥通过脚本验证的输入，这是目标原像问题。

在传统 P2PKH 输出尚未花费前，链上主要公开的是公钥哈希；花费时会公开公钥和签名。公钥公开后，哈希这层遮挡不再是主要障碍，剩下的核心仍是 ECDLP。

## 第四幕：用私钥 13 签一条教具交易

Bitcoin 不是用私钥解密收到的币。所谓“花费”，本质是对由交易规则产生的消息摘要签名。

我们继续使用阶

\[
n=79
\]

的微型曲线。假设待签交易按规则序列化并哈希后，化成教具整数

\[
z=42.
\]

私钥仍是

\[
d=13,
\]

公钥是

\[
Q=(13,19).
\]

ECDSA 签名还需要每次签名的一次性秘密数。选择

\[
k=5.
\]

> 真实钱包绝不能直接使用本文固定的 \(k\)。安全实现通常按规范从私钥和消息确定性地产生不可预测的 nonce，或使用可靠随机源。nonce 泄露、可预测或复用都可能泄露私钥。

### 计算临时点和 \(r\)

已经知道

\[
4G=(67,19).
\]

所以

\[
5G=4G+G.
\]

斜率：

\[
\lambda=(28-19)(1-67)^{-1}
=9(-66)^{-1}\pmod{97}.
\]

\[
(-66)^{-1}\equiv25\pmod{97},
\]

\[
\lambda\equiv9\cdot25=225\equiv31\pmod{97}.
\]

\[
x_{5G}=31^2-67-1=893\equiv20\pmod{97},
\]

\[
y_{5G}=31(67-20)-19=1438\equiv80\pmod{97}.
\]

这里需要停一下复核。将 \((20,80)\) 代回曲线：

\[
80^2\equiv95\pmod{97},
\]

\[
20^3+7\equiv53\pmod{97},
\]

两边不相等，说明上面的逆元算错了。这正是手算必须验点的原因。

重新求逆元：

\[
-66\equiv31\pmod{97},
\]

\[
31^{-1}\equiv72\pmod{97},
\]

因为

\[
31\times72=2232=23\times97+1.
\]

正确斜率是

\[
\lambda=9\cdot72=648\equiv66\pmod{97}.
\]

于是

\[
x_{5G}=66^2-67-1=4288\equiv20\pmod{97},
\]

\[
y_{5G}=66(67-20)-19=3083\equiv76\pmod{97}.
\]

因此

\[
R=kG=5G=(20,76).
\]

ECDSA 取横坐标模 \(n\)：

\[
r=x_R\bmod n=20.
\]

### 计算 \(s\)

ECDSA 签名第二部分是

\[
s=k^{-1}(z+rd)\bmod n.
\]

先求

\[
k^{-1}=5^{-1}\equiv16\pmod{79},
\]

因为

\[
5\times16=80\equiv1\pmod{79}.
\]

然后

\[
z+rd=42+20\times13=302.
\]

\[
302\bmod79=65.
\]

所以

\[
s=16\times65\bmod79=1040\bmod79=13.
\]

最终教具签名是

\[
(r,s)=(20,13).
\]

签名、消息、公钥都可以公开；没有发送 \(d=13\) 或 \(k=5\)。

## 第五幕：只用公钥怎样验签？

验证者拥有：

\[
Q=(13,19),\quad z=42,\quad(r,s)=(20,13).
\]

先算

\[
w=s^{-1}\bmod n=13^{-1}\bmod79=73,
\]

因为

\[
13\times73=949=12\times79+1.
\]

再算

\[
u_1=zw\bmod n=42\times73\bmod79=64,
\]

\[
u_2=rw\bmod n=20\times73\bmod79=38.
\]

验证点为

\[
X=u_1G+u_2Q.
\]

通过同样的倍点加点运算可得：

\[
64G=(14,36),
\]

\[
38Q=(45,7),
\]

\[
X=(14,36)+(45,7)=(20,76).
\]

最后核对：

\[
x_X\bmod n=20=r.
\]

签名通过。

为什么一定能回到 \(R=kG\)？把公式摊开：

\[
s=k^{-1}(z+rd)\pmod n,
\]

因此

\[
s^{-1}=k(z+rd)^{-1}\pmod n.
\]

于是

\[
u_1G+u_2Q
=zs^{-1}G+rs^{-1}(dG),
\]

\[
=s^{-1}(z+rd)G,
\]

\[
=k(z+rd)^{-1}(z+rd)G,
\]

\[
=kG=R.
\]

验证者能组合出 \(R\)，但没有因此得到 \(d\)。验证是一条检查等式，不是私钥恢复算法。

## 一个致命反例：nonce 重复时，私钥直接被消元

ECDLP 很难，不代表签名实现可以随便写。

同一私钥对两个不同摘要 \(z_1,z_2\) 使用同一个 \(k\)，会得到同一个 \(r\)：

\[
s_1=k^{-1}(z_1+rd)\pmod n,
\]

\[
s_2=k^{-1}(z_2+rd)\pmod n.
\]

两式相减：

\[
s_1-s_2=k^{-1}(z_1-z_2)\pmod n.
\]

若相关逆元存在，就能直接求：

\[
k=(z_1-z_2)(s_1-s_2)^{-1}\pmod n.
\]

再代回：

\[
d=(s_1k-z_1)r^{-1}\pmod n.
\]

此时攻击者根本不必破解椭圆曲线离散对数。他使用普通模运算和两份泄露关系就能得到私钥。

这个反例标出了安全边界：数学难题保护的是正确使用的签名方案，不保护错误生成的一次性数。

## 在真实 secp256k1 上走一遍

真实曲线参数包括：

\[
p=
\mathtt{FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFC2F}_{16},
\]

\[
y^2\equiv x^3+7\pmod p.
\]

真实私钥是满足

\[
1\le d<n
\]

的整数，其中 \(n\) 是基点 \(G\) 的阶。

为了让输出可以独立复核，我们故意使用公开示例：

\[
d=2.
\]

它的 32 字节十六进制形式是：

~~~text
0000000000000000000000000000000000000000000000000000000000000002
~~~

计算

\[
Q=2G
\]

得到压缩公钥：

~~~text
02c6047f9441ed7d6d3045406e95c07cd85c778e4b8cef3ca7abac09b95c709ee5
~~~

它的哈希链为：

~~~text
SHA256(pubkey)
b1c9938f01121e159887ac2c8d393a22e4476ff8212de13fe1939de2a236f0a7

RIPEMD160(SHA256(pubkey))
06afd46bcdfd22ef94ac122aa11f241244a37ecc
~~~

加入主网 P2PKH 版本字节、四字节校验和并做 Base58Check 编码，地址为：

~~~text
1cMh228HTCiwS8ZsaakH8A8wze1JR5ZsP
~~~

把私钥 2 加入主网私钥版本字节和“使用压缩公钥”标志，再做 Base58Check，得到 WIF：

~~~text
KwDiBf89QgGbjEhKnhXJuH7LrciVrZi3qYjgd9M7rFU74NMTptX4
~~~

WIF 看起来像密码，但它不是加密。任何人都能从这串字符解出私钥 2。WIF 的校验和只能发现常见输入错误，不能阻止窃取。

再次提醒：这是一把所有读者都知道的钥匙。

## 可直接运行的 Python 复算程序

下面的程序只使用 Python 标准库。前半段复算微型曲线的公钥、签名和验签；后半段计算真实 secp256k1 示例公钥、WIF 和传统地址。

~~~python
import hashlib


def point_add(P, Q, prime, a=0):
    if P is None:
        return Q
    if Q is None:
        return P

    x1, y1 = P
    x2, y2 = Q

    if x1 == x2 and (y1 + y2) % prime == 0:
        return None

    if P == Q:
        slope = (3 * x1 * x1 + a) * pow(2 * y1, -1, prime)
    else:
        slope = (y2 - y1) * pow((x2 - x1) % prime, -1, prime)

    slope %= prime
    x3 = (slope * slope - x1 - x2) % prime
    y3 = (slope * (x1 - x3) - y1) % prime
    return x3, y3


def scalar_mul(k, P, prime, a=0):
    result = None
    addend = P

    while k:
        if k & 1:
            result = point_add(result, addend, prime, a)
        addend = point_add(addend, addend, prime, a)
        k >>= 1

    return result


# 一、微型曲线
toy_p = 97
toy_n = 79
toy_G = (1, 28)
toy_d = 13
toy_Q = scalar_mul(toy_d, toy_G, toy_p)
assert toy_Q == (13, 19)

z = 42
k = 5
R = scalar_mul(k, toy_G, toy_p)
r = R[0] % toy_n
s = (pow(k, -1, toy_n) * (z + r * toy_d)) % toy_n
assert R == (20, 76)
assert (r, s) == (20, 13)

w = pow(s, -1, toy_n)
u1 = z * w % toy_n
u2 = r * w % toy_n
X = point_add(
    scalar_mul(u1, toy_G, toy_p),
    scalar_mul(u2, toy_Q, toy_p),
    toy_p,
)
assert X[0] % toy_n == r

# 暴力反推玩具私钥：真实曲线上绝不能这样做完
recovered = next(
    candidate
    for candidate in range(1, toy_n)
    if scalar_mul(candidate, toy_G, toy_p) == toy_Q
)
assert recovered == 13

print("toy public key:", toy_Q)
print("toy signature:", (r, s))
print("toy recovered private key:", recovered)


# 二、真实 secp256k1 示例
real_p = 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFC2F
real_G = (
    0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798,
    0x483ADA7726A3C4655DA4FBFC0E1108A8FD17B448A68554199C47D08FFB10D4B8,
)
real_d = 2  # 公开且不安全，只用于教学
real_Q = scalar_mul(real_d, real_G, real_p)

prefix = 2 + (real_Q[1] & 1)
compressed_pubkey = bytes([prefix]) + real_Q[0].to_bytes(32, "big")

sha = hashlib.sha256(compressed_pubkey).digest()
hash160 = hashlib.new("ripemd160", sha).digest()

alphabet = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"


def base58_encode(raw):
    leading_zeroes = len(raw) - len(raw.lstrip(b"\x00"))
    number = int.from_bytes(raw, "big")
    encoded = ""

    while number:
        number, remainder = divmod(number, 58)
        encoded = alphabet[remainder] + encoded

    return "1" * leading_zeroes + encoded


def base58check(payload):
    checksum = hashlib.sha256(
        hashlib.sha256(payload).digest()
    ).digest()[:4]
    return base58_encode(payload + checksum)


address = base58check(b"\x00" + hash160)
private_bytes = real_d.to_bytes(32, "big")
wif = base58check(b"\x80" + private_bytes + b"\x01")

print("private hex:", private_bytes.hex())
print("compressed public key:", compressed_pubkey.hex())
print("SHA256(public key):", sha.hex())
print("HASH160(public key):", hash160.hex())
print("P2PKH address:", address)
print("WIF:", wif)

assert compressed_pubkey.hex() == (
    "02c6047f9441ed7d6d3045406e95c07cd85c778e4b8cef3ca7abac09b95c709ee5"
)
assert address == "1cMh228HTCiwS8ZsaakH8A8wze1JR5ZsP"
assert wif == "KwDiBf89QgGbjEhKnhXJuH7LrciVrZi3qYjgd9M7rFU74NMTptX4"
~~~

这段程序适合核对数学结构，不是生产钱包：

- 没有实现安全内存、侧信道防护和可靠随机数；
- 没有实现完整交易序列化和共识规则；
- 没有实现现代描述符、SegWit 或 Taproot 钱包；
- 不应该拿来保管任何真实资产。

真实软件应使用经过审查的 Bitcoin 钱包和 secp256k1 库，而不是复制教学代码造钱包。

## 一张“可逆性地图”

| 路径 | 分类 | 反向时缺少什么 |
| --- | --- | --- |
| 私钥整数 ↔ 32 字节十六进制 | 可逆编码 | 不缺数学工具 |
| 私钥整数 ↔ WIF | 可逆编码加校验和 | 不缺数学工具；WIF 泄露就是私钥泄露 |
| 私钥 \(d\) → 公钥 \(Q=dG\) | 正向快、反向计算困难 | 高效 ECDLP 算法 |
| 完整公钥 ↔ 压缩公钥 | 对合法点可逆编码 | 由曲线方程求平方根并选奇偶分支 |
| 公钥 → HASH160 | 有损哈希 | 指定目标的有效原像 |
| HASH160 ↔ Base58Check 地址 | 外层编码可逆 | 可还原版本、哈希和校验和，但不能反哈希 |
| 交易字节 → 消息摘要 | 有损哈希 | 指定摘要的有效原像 |
| 私钥、摘要、nonce → ECDSA 签名 | 可公开验证，不能直接求私钥 | ECDLP；但 nonce 泄露会绕开它 |
| 地址 → 余额和历史 | 区块链索引查询 | 这不是密码学逆运算 |

## 攻击者究竟卡在哪里？

### 只知道 WIF

完全不卡。Base58Check 解码后就得到私钥。因此 WIF 必须像私钥一样保密。

### 只知道未花费 P2PKH 地址

可以解码出版本字节和 20 字节公钥哈希，但没有已知高效方法从指定 HASH160 找到可用公钥与对应私钥。

### 已经知道完整公钥

问题缩短为：

\[
Q=dG.
\]

攻击者缺少 secp256k1 ECDLP 的高效经典求解算法。已知通用算法仍需不可行的约 \(2^{128}\) 级工作。

### 已经知道许多正确签名

只要 nonce 安全、实现正确，公开签名本来就是设计目标。攻击者仍不能从普通签名直接解出私钥。

但若 nonce 重复、可预测、部分泄露，或者钱包随机数与侧信道实现有问题，攻击者可能建立额外方程，绕过 ECDLP。

### 已经拿到助记词、私钥文件或解锁后的钱包

也完全不卡。现实盗币往往发生在这个层面，而不是有人发明了 ECDLP 快速算法。

## 黎曼猜想这次出现在哪里？

仍然不在直接安全链上。

本篇遇到的两个主要困难是：

1. 椭圆曲线离散对数：从 \(Q=dG\) 反求 \(d\)；
2. 哈希原像：从指定哈希寻找满足条件的输入。

黎曼猜想研究 zeta 函数零点与质数分布误差。它不会自动提供上面两个问题的高效算法。

secp256k1 使用素数有限域，这说明数论是舞台；不能由此推出“所有涉及素数的密码系统都依赖黎曼猜想”。证明黎曼猜想与构造可行的 secp256k1 离散对数算法之间，目前没有这样的直接推论。

## 常见误解

### “Bitcoin 地址就是公钥”

不总是。本文的 P2PKH 地址编码的是公钥 HASH160 以及网络版本、校验和；其他脚本和地址类型又有不同含义。

### “WIF 把私钥加密了”

错。WIF 是可逆编码。没有额外口令加密层时，拿到 WIF 就拿到了私钥。

### “压缩公钥丢掉了纵坐标，所以不可逆”

错。曲线方程和前缀奇偶位足以恢复正确纵坐标。它丢掉的是冗余表示，不是必要信息。

### “哈希不可逆，所以地址绝对不会碰撞”

错。有限输出必然存在碰撞。安全说法是：寻找指定原像或有用碰撞在当前参数和已知算法下不可行。

### “从公钥不能除以 \(G\)，是因为计算器功能不够”

不是普通除法功能的问题。需要的是对巨大有限椭圆曲线群求离散对数的高效算法。

### “签名中有私钥参与，所以签名是加密后的私钥”

错。签名是满足验证方程的证明值。正确 nonce 下，公开签名不会提供已知的高效私钥恢复法。

### “只要曲线安全，随机数差也没关系”

错。ECDSA nonce 复用能用初等模代数直接消元恢复私钥，完全绕开 ECDLP。

## 练习

1. 验证教具公钥 \(Q=(13,19)\) 确实在曲线 \(y^2=x^3+7\pmod{97}\) 上。
2. 在教具曲线上计算 \(-Q\)，并验证 \(Q+(-Q)=\mathcal O\)。
3. 为什么私钥转十六进制与私钥转公钥虽然都叫“转换”，安全性质完全不同？
4. 从地址 Base58Check 解码后，为什么仍不能直接得到原公钥？
5. 验证 \(5^{-1}\equiv16\pmod{79}\) 与 \(13^{-1}\equiv73\pmod{79}\)。
6. 将摘要从 \(z=42\) 改成 \(z=43\)，仍使用 \(d=13,k=5\)，计算新的 \(s\)。为什么真实系统不应这样重复 \(k\)？
7. 已知同一 \(k\) 下两份签名，说明如何先求 \(k\)，再求 \(d\)。
8. 对比“地址哈希反求公钥”和“公钥反求私钥”：它们分别是哪两个困难问题？
9. 为什么私钥空间有 \(2^{256}\) 级候选，而经典通用攻击强度常说约 \(2^{128}\)？

## 练习提示

### 题 1

分别计算 \(19^2\bmod97\) 和 \(13^3+7\bmod97\)。

### 题 2

横坐标不变，纵坐标取模意义下的相反数。

### 题 3

问有没有信息丢失，以及反向是否存在公开的高效算法。

### 题 4

Base58 可逆，但它解出的 payload 中已经是 HASH160。

### 题 5

把乘积写成 79 的倍数加 1。

### 题 6

只替换签名公式中的 \(z\)。

### 题 7

先把两条 \(s=k^{-1}(z+rd)\) 方程相减。

### 题 8

一个是哈希原像，一个是椭圆曲线离散对数。

### 题 9

考虑平方根级 Pollard rho 算法。

## 练习答案

### 题 1

\[
19^2=361\equiv70\pmod{97}.
\]

\[
13^3+7=2204\equiv70\pmod{97}.
\]

左右相等，所以 \(Q\) 在曲线上。

### 题 2

\[
-Q=(13,-19\bmod97)=(13,78).
\]

两点横坐标相同，纵坐标之和

\[
19+78=97\equiv0\pmod{97},
\]

按群定义它们相加得到无穷远点 \(\mathcal O\)。

### 题 3

十六进制只是同一个整数的可逆表示；公钥是曲线标量乘法的结果。后者数学上对应某个私钥，但反向需要解决在真实规模下没有已知快速经典算法的 ECDLP。

### 题 4

Base58Check 解码只能恢复网络版本、公钥哈希和校验和。公钥在进入 Base58 之前已经经过 SHA-256 与 RIPEMD-160 的有损压缩；恢复指定原像仍然困难。

### 题 5

\[
5\times16=80=79+1,
\]

\[
13\times73=949=12\times79+1.
\]

所以两个乘积模 79 都等于 1。

### 题 6

\[
s=5^{-1}(43+20\cdot13)\bmod79,
\]

\[
=16\cdot303\bmod79.
\]

\[
303\bmod79=66,
\]

\[
s=16\cdot66=1056\equiv29\pmod{79}.
\]

它与第一份签名拥有相同的 \(r=20\)，暴露了相同 \(k\) 的事实；攻击者可联立两式恢复 nonce 和私钥。

### 题 7

\[
k=(z_1-z_2)(s_1-s_2)^{-1}\pmod n,
\]

求得 \(k\) 后：

\[
d=(s_1k-z_1)r^{-1}\pmod n.
\]

这只需模逆元和四则运算，不需解决 ECDLP。

### 题 8

地址哈希反求公钥属于指定目标的哈希原像问题；完整公钥反求私钥属于 secp256k1 椭圆曲线离散对数问题。两者不是同一种“反向算法”。

### 题 9

群规模约为 \(2^{256}\)，但 Pollard rho 等通用离散对数攻击具有平方根级复杂度：

\[
\sqrt{2^{256}}=2^{128}.
\]

因此常说 secp256k1 约提供 128 位经典安全强度，而不是 256 位。

## 如果你真的理解了

请试着不看表格，自己画出下面这条链，并在每个箭头上写“可逆编码”“哈希原像”或“离散对数”：

\[
\text{私钥整数}
\rightarrow
\text{WIF},
\]

\[
\text{私钥整数}
\rightarrow
\text{压缩公钥}
\rightarrow
\text{HASH160}
\rightarrow
\text{P2PKH 地址}.
\]

然后回答：

1. 哪一串字符泄露后可以直接解出私钥？
2. 哪一步不是信息丢失，而是反向算法太慢？
3. 哪一步因压缩输出而不存在唯一逆函数？
4. 为什么正确签名可验证，却不会自然泄露私钥？
5. 为什么 nonce 重复能绕过最难的数学问题？

如果你能回答这五问，Bitcoin 私钥的安全边界就从“一个很大的数”变成了一张可以逐项检查的地图。

## 资料来源

- [SEC 2：secp256k1 推荐参数](https://www.secg.org/sec2-v2.pdf)。
- [SEC 1：椭圆曲线密码学与公钥编码](https://www.secg.org/sec1-v2.pdf)。
- [Bitcoin 开发者指南：钱包、公钥与 P2PKH](https://developer.bitcoin.org/devguide/wallets.html)。
- [Bitcoin Core libsecp256k1](https://github.com/bitcoin-core/secp256k1)：Bitcoin 使用的曲线运算、ECDSA 和 Schnorr 实现。
- [BIP 66：Bitcoin ECDSA 严格 DER 签名编码](https://bips.dev/66/)。
- [BIP 340：secp256k1 上的 Schnorr 签名](https://bips.dev/340/)。
