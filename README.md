# 网安导论实验——[隐私保护]()

## 实验要求

### 实验目的

1. 编写Paillier算法（密钥生成、加密和解密算法）并验证其加法同态性质
2. 模拟实现基于Paillier 算法的匿名电子投票流程，了解该算法的应用，加深对同态加密算法的认识

### 实验原理

1. Paillier算法是一种满足加法同态性质（密文相乘的结果等于对应明文相加的结果）的加密算法
2. 统计票数使用加法累加进行统计，Paillier算法可被用于匿名电子投票系统，保护投票人的投票信息

![image.png](https://s2.loli.net/2024/07/03/98LdzEUsZtrAHSl.png)

### 实验步骤

1. 根据下图所示的Paillier算法编写Paillier.py文件，其中包含keygen函数、encrypt函数以及decrypt函数，它们分别被用于密钥生成、加密和解密。

![image.png](https://s2.loli.net/2024/07/03/iKBdGWIZFDRCpSg.png)

2. 算法编写完成后，输入两个明文333和444，观察两个明文被加密后的密文值，对密文进行相乘再解密后发现结果是777，相当于对应明文相加，因此该算法具有加法同态性。
3. 编写ElectronicVoting.py文件，在该文件中import Paillier，通过引用第1步中写好的Paillier算法模拟实现基于Paillier 算法的匿名电子投票流程。

## 实验准备——solved by Enjun Du

### 实验题目

基于Paillier 算法的匿名电子投票流程实现

### 硬件环境

```
磁盘驱动器：NVMe KIOXIA- EXCERIA G2 SSD
NVMe Micron 3400 MTFDKBA1TOTFH
显示器：NVIDIA GeForce RTX 3070 Ti Laptop GPU
系统型号	ROG Strix G533ZW_G533ZW
系统类型	基于 x64 的电脑
处理器	12th Gen Intel(R) Core(TM) i9-12900H，2500 Mhz，14 个内核，20 个逻辑处理器
BIOS 版本/日期	American Megatrends International, LLC. G533ZW.324, 2023/2/21
BIOS 模式	UEFI
主板产品	G533ZW
操作系统名称	Microsoft Windows 11 家庭中文版
```

### 软件环境

```
PyCharm 2023.2 专业版
python 3.11
```

## 开始实验——蓝天°

### 理解实验原理

1. 1.1 在密钥生成阶段，我们选择两个大素数p 和 q，这两个素数的选择要足够随机且相互独立。然后计算N=pq 和 λ=lcm(p−1,q−1)，其中lcm 是最小公倍数函数。这些值构成了公钥和私钥的一部分。

2. 我们还需要选择一个随机数 g，确保 g 与 N²互素。这样做的目的是为了确保在加密和解密过程中存在一个特定的模逆 μ，使得解密过程能够正确地还原明文

3. 1.1 加密过程涉及将明文 m 加密为密文 c。为了确保加密的安全性，我们首先选择一个随机数 r∈ZN，然后应用加密公式，将明文映射到密文空间中。

4. 解密过程是加密过程的逆过程，即从密文中还原出原始的明文。在解密过程中，我们首先将密文的 λ 次幂取模 N²，然后将其乘以预先计算的模逆 μ，最后将结果取模 N，得到原始的明文。

   ###  安全性分析

   ​	Paillier加密算法的安全性基于两个数论难题：大素数分解问题和离散对数问题。攻击者需要能够分解 N 为其素因子 p 和 q，以获得 λ 的值。但是，由于 N 是两个大素数的乘积，目前尚未找到高效的算法来解决这个问题。离散对数问题：攻击者需要能够从 gm 中还原出 m，即求解离散对数问题。但是，由于 g 是随机选择的，没有已知的有效算法可以在合理的时间内解决这个问题。因此，Paillier加密算法被认为是安全的，并且在广泛的应用场景中得到了验证。

   ### 代码编写

   首先我们尝试编写Paillier.py：

   ```python
   import random
   import math
   def keygen(input_bits):
       def is_prime(n, k=14):
           #The accuracy of the Miller-Rabin test after k iterations is 1 - (1/2^k). Empirical testing has shown that when k is set to 14, the accuracy of the test reaches 99.99%.
           if n <= 1:
               return False
           if n <= 3:
               return True
           if n % 2 == 0:
               return False
           r = 0
           d = n - 1
           while d % 2 == 0:
               r += 1
               d //= 2
           for i in range(k):
               a = random.randint(2, n - 2)
               x = pow(a, d, n)
               if x == 1 or x == n - 1:
                   continue
               for j in range(r - 1):
                   x = pow(x, 2, n)
                   if x == n - 1:
                       break
               else:
                   return False
           return True
       def generate_prime(bits):
           while True:
               p = random.getrandbits(bits)
               # 确认p是素数
               p |= 1
               if is_prime(p):
                   return p
   
       def create_pq(bits):
           while True:
               p = generate_prime(bits)
               q = generate_prime(bits)
               if p != q:
                   # 计算 gcd(p*q, (p-1)*(q-1))
                   if math.gcd(p * q, (p - 1) * (q - 1)) == 1:
                       return p, q
       def calculate_miu(p, q, N, lambda_1):
           # mo zhi shu yun suan
           def mod_exp(base, exponent, modulus):
               result = 1
               base = base % modulus
               while exponent > 0:
                   if exponent % 2 == 1:
                       result = (result * base) % modulus
                   exponent = exponent >> 1
                   base = (base * base) % modulus
               return result
   
           # mo ni yun suan
           def mod_inv(a, m):
               m0, x0, x1 = m, 0, 1
               while a > 1:
                   q = a // m
                   m, a = a % m, m
                   x0, x1 = x1 - q * x0, x0
               return x1 + m0 if x1 < 0 else x1
   
           # calculate miu
           g = random.randint(2, N - 1)
           miu = mod_inv(((mod_exp(g, lambda_1, N ** 2) - 1) // N), N)
   
           return miu,g
   
       p, q = create_pq(input_bits)
       N=p*q
       lambda_1=abs((p-1)*(q-1))//math.gcd((p-1),(q-1))
       miu,g = calculate_miu(p, q, N, lambda_1)
   
       return N, g ,lambda_1,miu
   
   def encrypt(m,N,g):
       r = random.randint(1, N**2 - 1)
       while math.gcd(r, N**2) != 1:
           r = random.randint(1, N**2 - 1)
       c = (pow(g, m, N**2) * pow(r, N, N**2)) % (N**2) #加密密文
       return c
   
   def decrypt(c,lambda_1,N,miu):
       decrypted_m = ((pow(c, lambda_1, N**2) - 1) * miu // N) % N
       return decrypted_m
   
   def main():
       N,g,lambda_1,miu=keygen(1024)
   
       m1=eval(input("请输入第一个明文"))
       m2=eval(input("请输入第二个明文"))
   
       c1=encrypt(m1,N,g)
       c2=encrypt(m2,N,g)
   
       print("对第一个明文加密后得到密文:{}".format(c1))
       print("对第二个明文加密后得到密文:{}".format(c2))
   
       c=c1*c2
   
       print("两密文相乘得到:{}".format(c))
   
       m=decrypt(c,lambda_1,N,miu)
   
       print("两密文相乘后解密得到的明文为:{}".format(m))
   
   if __name__ == "__main__":
       main()
   ```

   首先我们编写keygen函数，用于生成密钥。我采用Miller-Rabin素性检测来检验，根据计算得知，运行k次后判断一个数是素数的正确率大于(1-1/2^k),经验证，当k=14时，判断的准确率大于99.99%。根据公式来计算p，q，λ，μ和N。然后根据公司和得到的私钥、公钥、明文来进行encrypt和decrypt函数的编写。然后编写main函数测试输入明文333和444后程序的输出结果。

   然后我们编写ElectronicVoting.py

   ```python
   import time
   import Paillier
   
   print("***************************此程序模拟了基于Paillier算法的匿名电子投票的流程:*****************************************")
   print("首先每位投票者为候选人投票并将结果加密发送给计票人。每人只有1张选票，选票上被投票的候选者得到1张选票，其他候选者得到0张选票;")
   print("然后计票人将所有选票上对应候选人的加密的投票结果相乘，并将加密的统计结果发送给公布人;")
   print("最后公布人对统计的票数进行解密并公布;")
   print("***************************************************************************************************************************")
   
   houxuan_num=eval(input("请设置候选人数："))
   toupiao_num=eval(input("请设置投票人数："))
   
   houxuanpiao=[0 for i in range (houxuan_num+1)]
   
   for i in range(1,toupiao_num+1):
       print("----------请第{}名投票者为候选者投票----------".format(i))
       for j in range(1,houxuan_num+1):
           houxuanpiao[j]+=eval(input("请为第{}名候选者投票:".format(j)))
   
   print("对该投票结果进行加密并发送给计票人")
   print("-------计票人计票完成并将加密后的投票结果发给公布人-------")
   print("加密后的投票结果为：")
   
   N, g, lambda_1, miu = Paillier.keygen(1024)
   encrypted_m=[0 for i in range (houxuan_num+1)]
   decrypt_c=[0 for i in range (houxuan_num+1)]
   
   for i in range (1,houxuan_num+1):
       encrypted_m[i]=Paillier.encrypt(houxuanpiao[i],N,g)
       print("第{}位候选人获得的选票票数的加密结果为：{}".format(i,encrypted_m[i]))
       time.sleep(1)
   
   print("-------公布人解密计票结果并公布最终的投票结果-------")
   
   for i in range (1,houxuan_num+1):
       decrypt_c[i]=Paillier.decrypt(encrypted_m[i],lambda_1,N,miu)
       print("第{}位候选人获得了{}张选票".format(i,decrypt_c[i]))
       time.sleep(1)
   
   max_index=decrypt_c.index(max(decrypt_c))
   
   print("最终第{}位候选人获得的选票最多，为{}张".format(max_index,decrypt_c[max_index]))
   ```

     我将p和q的长度设置为1024比特，然后根据题目要求编写代码，引用了time库使得程序运行更流程，引用Paillier里的三个函数完成密钥生成、加密和解密的过程。

   

   ​	最终将两个代码进行测试，**代码完美完成预计任务**

   ## 结论与体会

   ​	在本次实验中，通过编写和测试Paillier算法及其在匿名电子投票系统中的应用，我深刻理解了同态加密算法的原理和实际应用价值。通过实践，我掌握了Paillier算法的核心机制，包括密钥生成、加密、解密过程以及如何利用算法的加法同态性质。

   ​	首先，在密钥生成阶段，我学习到了选择合适的大素数p和q，以及如何计算N和λ，这些都是保证加密算法安全性的基础。选择随机数g的过程也让我认识到了在加密算法中引入随机性的重要性，这有助于提高算法的安全性。在编写加密和解密函数时，我通过实践深入理解了Paillier算法的数学原理，以及如何将理论应用到实际编程中。

   ​	通过将Paillier算法应用于模拟的匿名电子投票系统中，我体会到了同态加密算法在保护隐私信息方面的巨大潜力。在这个系统中，选民的投票选择得到了加密保护，同时还能够在不解密的情况下进行票数统计，这充分展示了Paillier算法加法同态性质的强大用途。

   ​	此外，我还学习到了算法的安全性分析，包括大素数分解问题和离散对数问题，这些都是算法安全性的理论基础。通过对这些数论难题的了解，我认识到了为什么Paillier算法被认为是安全的，并且可以在实际应用中提供强有力的数据保护。

   ​	总的来说，这次实验不仅让我对同态加密算法有了深入的了解，而且还让我体会到了将理论知识应用到实践中的重要性。我意识到，理论和实践相结合是理解和掌握复杂概念的关键。通过这次实验，我对加密算法的原理、实现以及在现实世界中的应用有了更加全面的认识，这将对我的未来学习和研究产生积极影响。

    

    

    

   
