



# shell编程

## for循环之批量解压缩

例子

```
解压软件包的脚本1

[root@localhost tar]# cat tar.sh
#!/bin/bash

cd /tar		#进入目录

ls *.tar.gz > tar.log		#查看当前目录的末尾是.tar.gz的文件有就输出到tar.log
ls *.tgz >> tar.log &>/dev/null	#也是一样但是如果没有就将报错输出到/dev/null中这是一个类似垃圾箱的文件

aa=$(cat tar.log|wc -l)		#查看tar.log 的行数	

for (( i=1;i<="$aa";i=i+1 ))	#当i小于变量aa的时候执行以下再自增
do
        bb=$(cat tar.log | awk 'NR=='$i' {print $1}' )	#查看tar.log中的行数为i的输出出来赋值给变量bb
        tar -zxvf $bb -C /tar	#将bb的行数解压
done
rm -rf  tar.log		#完成后删除了因为没有用了
```

```
解压软件包的脚本2
上面的例子步骤太多了
下面的简单一点
[root@localhost tar]# cat tar2.sh
#!/bin/bash

cd /tar		#进入目录

ls *.tar.gz > tar.log   #查看当前目录的末尾是.tar.gz的文件有就输出到
ls *.tgz >> tar.log &>/dev/null     #也是一样但是如果没有就将报错输出到/dev/null中这是一个类似垃圾箱的文件

for i in $(cat tar.log)   #查看tar.log 的行数有多少复制给变量i
do
        tar -zxvf $i		解压变量i的当前行有多少行就解压多少行
done

rm -rf tar.log		#完成后删除了因为没有用了
```



## for循环之合法ip判断

例子1

```
原始数据
[root@localhost sh]# cat ip.txt
192.168.1.200
202.106.0.20
300.36.190.5
222222222222
192.168.1.300
200.2.2
192.168.100.100

#!/bin/bash

grep "^[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}$" /root/sh/ip.txt > /root/sh/ip_test1.txt
#先通过正则，把明显不符合规则的ip过滤，把结果保存在ip_test1.txt临时文件中
line=$(wc -l /root/sh/ip_test1.txt  | awk '{print $1}')
#统计test1中有几行IP
echo "" > /root/sh/ip_test.txt
#清空最终数据文件

for (( i=1;i<=$line;i=i+1 ))
#有几行IP，循环几次
do
        cat /root/sh/ip_test1.txt | awk 'NR=='$i'{print}' > /root/sh/ip_test2.txt
        #第几次循环，就把第几行读入ip_test2.txt文件（此文件中只有一行IP）
        a=$(cat  /root/sh/ip_test2.txt | cut -d '.' -f 1)
        b=$(cat  /root/sh/ip_test2.txt | cut -d '.' -f 2)
        c=$(cat  /root/sh/ip_test2.txt | cut -d '.' -f 3)
        d=$(cat  /root/sh/ip_test2.txt | cut -d '.' -f 4)
        #分别把IP地址的四个数值分别读入变量a,b,c,d

        if [ "$a" -lt 1 -o "$a" -gt 255 ]
        #如果第一个数值小于1或大于等于255
                then
                        continue
                        #则退出本次循环
        fi

        if [ "$b" -lt 0 -o "$b" -gt 255 ]
                then
                        continue
        fi    

        if [ "$c" -lt 0 -o "$c" -gt 255 ]
                then
                        continue
        fi
        if [ "$d" -lt 0 -o "$d" -gt 255 ]
                then
                        continue
        fi
        #依次判断四个IP数值是否超出范围，如果超出，退出本次循环

        cat /root/sh/ip_test2.txt >> /root/sh/ip_test.txt
        #如果四个IP数值都符合要求，则把合法IP记录在文件中
done
rm -rf /root/sh/ip_test1.txt
rm -rf /root/sh/ip_test2.txt
#删除临时文件

执行之后的结果
[root@localhost sh]# cat ip_test.txt

192.168.1.200
202.106.0.20
192.168.100.100

```

例子2

```
#!/bin/bash
#检查ip格式
grep "^[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}$" /root/sh/ip.txt > /root/sh/ip_test1.txt
#先通过正则，把明显不符合的过滤掉。保存到临时文件中
echo "" > /root/sh/ip_valid.txt
#清空保存数据的文件

for i in $(cat /root/sh/ip_test1.txt)
do
        a=$(echo "$i" | cut -d "." -f 1 )
        b=$(echo "$i" | cut -d "." -f 2 )
        c=$(echo "$i" | cut -d "." -f 3 )
        d=$(echo "$i" | cut -d "." -f 4 )
        #分别把IP地址的四个数值分别读入变量a,b,c,d
        if [ "$a" -lt 1 -o "$a" -gt 255 ]
        #如果第一个数值大于1或大于等于255
                then
                        continue
                        #退出本次循环
        fi

        if [ "$b" -lt 0 -o "$b" -gt 255 ]
                then
                        continue
        fi

        if [ "$c" -lt 0 -o "$c" -gt 255 ]
                then
                        continue
        fi

        if [ "$d" -lt 0 -o "$d" -gt 255 ]
                then
                        continue
        fi

        echo "$i" >> /root/sh/ip_valid.txt
	#把合法IP地址写入/root/sh/ip_valid.txt文件
done
rm -rf /root/sh/ip_test1.txt

```





## for循环之批量添加用户

```
#!/bin/bash
#批量添加指定数量的用户
# Author: shenchao （E-mail: shenchao@atguigu.com）

read -p "Please input user name: " -t 30 name
#让用户输入用户名，把输入保存入变量name
read -p "Please input the number of users: " -t 30 num 
#让用户输入添加用户的数量，把输入保存入变量num
read -p "Please input the password of users: " -t 30 pass
#让用户输入初始密码，把输入保存如变量pass

if [ ! -z "$name" -a ! -z "$num" -a ! -z "$pass" ]
#判断三个变量不为空
        then
        y=$(echo $num | sed 's/[0-9]//g')
		#定义变量的值为后续命令的结果
		#后续命令作用是，把变量num的值替换为空。如果能替换为空，证明num的值为数字
		#如果不能替换为空，证明num的值为非数字。我们使用这种方法判断变量num的值为数字
        if [ -z "$y" ]
		#如果变量y的值为空，证明num变量是数字
                then
                for (( i=1;i<=$num;i=i+1 ))
				#循环num变量指定的次数
                    do  
                        /usr/sbin/useradd $name$i &>/dev/null
						#添加用户，用户名为变量name的值加变量i的数字
                        echo $pass | /usr/bin/passwd --stdin $name$i &>/dev/null
						#给用户设定初始密码为变量pass的值
                    	chage -d 0 $name$i &>/dev/null
						#强制用户登录之后修改密码
done
        fi  
fi

```





## for循环之批量删除用户

```

[root@localhost ~]# cat  usedel.sh
#!/bin/bash
#给变量name赋值查看/etc/passwd中包含/bin/bash除了root的数据有哪些再根据分号做制表符显示第一列
name=$( cat /etc/passwd | grep "/bin/bash" | grep -v root | cut -d ":" -f 1)		

#循环删除由变量name赋值给变量i的用户
for i in $name
        do
                userdel -r $i &>/dev/null	
        done

```



## while循环

```
语法：
while [ 条件判断式 ]
	do	
		程序
	done
对while循环来讲，只要条件判断式成立，循环就会一直继续，直到条件判断式不成立，循环才会停止，
```

例子

```
1加到100
[root@localhost ~]# cat whle.sh
#!/bin/bash
i=1
s=0
while [ $i -le 100 ]
        do
                s=$(( $s+$i ))
                i=$(( $i+i ))
        done

echo $s

结果
[root@localhost ~]# ./whle.sh
5050

```



## until循环

until和while循环相反，until循环时只要条件判断式不成立则进行循环，并执行循环程序，一旦循环条件成立，则终止循环



例子

```
[root@localhost ~]# cat until.sh
#!/bin/bash

i=1
s=0

until [ $i -gt 100 ]
do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
done
echo $s

结果
[root@localhost ~]# ./until.sh
5050

```

‘

## exit语句

```
exit语句
系统是有exit命令的，用于退出当前用户的登录状态，可以是shell脚本中，exit语句是用来退出当前脚本，也就是说在shell脚本中，只要碰到exit语句，后续的程序就不再执行，而直接退出脚本
语法：
exit [ 返回值 ]
```

例子

```
[root@localhost ~]# cat exit.sh
#!/bin/bash

read -t 30 -p "please input number num: " num	#输出值
y=$( echo $num | sed 's/[0-9]//g')	#给变量y赋值如果为数字就变成0
if [ -n "$y" ]		#如果输出有值就输出error
then
        echo "error please input number"
        exit 19
else		#没值就输出num变量
        echo $num
fi


[root@localhost ~]# ./exit.sh
please input number num: asdf
error please input number
[root@localhost ~]# echo $?
19
[root@localhost ~]#

```



## break语句和continue语句

```
break语句
当程序执行到break语句时，会结束整个当前循环，而continue与也是结束循环的语句，不过continue语句单次当前循环，下次循环会继续
```

![1716450102428](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1716450102428.png)

例子

```
原始输出
[root@localhost ~]# cat break.sh
#!/bin/bash

for ((i=1;i<=10;i++  ))
do
        echo $i
done

[root@localhost ~]# ./break.sh
1
2
3
4
5
6
7
8
9
10

加了break之后
[root@localhost ~]# cat break.sh
#!/bin/bash

for ((i=1;i<=10;i++  ))
do
        if [ "$i" -eq 4  ]
        then
                break
        fi
        echo $i
done

结果
[root@localhost ~]# ./break.sh
1
2
3
```



```
continue语句
contune也是结束流程控制的语句，如果在循环汇总，continue语句只会结束单次当前循环，
```

![1716450295438](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1716450295438.png)

例子

```
加入continue
[root@localhost ~]# cat break.sh
#!/bin/bash

for ((i=1;i<=10;i++  ))
do
        if [ "$i" -eq 4 ]
        then
                continue
        fi
        echo $i
done

结果会少了一个4
[root@localhost ~]# ./break.sh
1
2
3
5
6
7
8
9
10
```

