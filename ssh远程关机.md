# ssh远程关机

使用场景：管理集群环境（linux系统），需要通过ssh进行远程关机。

方法：使用**expect**（tcl的扩展）命令编写脚本并执行。



## 知识点

**expect：**在 Ubuntu 终端中执行一些命令时总是需要交互式的输入信息，如sudo命令，需要输入用户密码等等，这可以起到提醒用户的作用，也更加保险。但是有的时候在执行自动化脚本时并不希望一直进行交互式的操作，所以 expect命令便显得极为有用。expect 是一个免费的编程工具，可以完成自动化交互式任务，无需人为干预。



##### 脚本开头：

`#!/usr/bin/expect -f`

##### 变量命名：

`set variable value`

例如：

直接赋值：`set timeout 10` 或者 `set pwd "123456"`

通过标准输入赋值：`set ip [lindex $argv 0]`

$argv 0代表执行脚本的第一个变量，比如执行，  sshpoweroff.sh  jsj  123456  192.168.1.1，其中的变量分别为：

argv 0——jsj

argv 1——123456  

argv 2——192.168.1.1



##### 控制语句：

if语句：

expect有if语句，比如，

```bash
if {$index == [llength $passwords]} {
            error "ran out of possible passwords"
        }
```

但是，一般情况下，我们可以通过模式匹配来进行if判断，比如，通过ping命令判断某个服务器是否开启，可以这么写：

```bash
spawn ping -c 2 -i 3 -W 1 192.168.1.1
expect {
				#相当于如果执行命令之后出现以下字符串：
        " 0.0% packet loss" {
            puts "192.168.1.1 is running!\n"
        }
        " 100% packet loss" {
            puts "192.168.1.1 is not running!\n"
        }
    }
```

注：以上的模式匹配了字符串” 0.0% packet loss“和" 100% packet loss"，具体的匹配字符串要根据实际情况来。



for语句：

```bash
for {set ip 172} {$ip<181} {incr ip} {
			
}
```



##### 远程关机完整命令如下：（IP地址范围在192.168.1.172~192.168.1.180，如果IP地址不连续，可以考虑将IPs放入文件，一行一个IP，然后通过读取文件来执行命令）

```bash
#!/usr/bin/expect -f
set timeout 10
set pwd "123456"
set usr "jsj"
set ip [lindex $argv 0]
set ipPre "192.168.1."
for {set ip 172} {$ip<181} {incr ip} {
    spawn ping -c 2 -i 3 -W 1 $ipPre$ip
    expect {
        " 0.0% packet loss" {
            spawn ssh $usr@$ipPre$ip
            expect "*pass*"
            send "$pwd\r"
            expect "*:~*"
            send "sudo poweroff\r"
            expect "*pass*"
            send "$pwd\r"
            expect eof
        }
        " 100% packet loss" {
            puts "$ipPre$ip is not running!\n"
        }
    }
}

puts "done"
```



## 回车换行

最后讲讲回车符\r和换行符\n，这2个符号其实是有历史渊源的，在计算机还没有出现之前，有一种叫“电传打字机（Teletype Model 33）”的东东，刚开始在使用打字机的时候，每1秒钟可以打10个字符，即，每个字打印需要0.1s，但是，在回车换行的时候，需要等0.2s，这样就导致，在此时传入的字符（0.2s之内）没办法捕获并打印，为了弥补这一问题，于是，人们想到用2个字符代表回车和换行，刚好消耗这0.2s。

回车符（\r）：将光标指向一行的开头。

换行符（\n）：将光标指向下一行。

后来，计算机出现了，我们将回车换行的概念引入到了计算机，但是那时候的存储很贵，有的计算机科学家觉得，用2个字符表示回车换行很浪费，于是出现了用一个符号表示回车换行的情况，这也使得回车换行的表示出现了分歧：

Unix/Linux：\n表示回车换行

Mac：\r表示回车换行

Windows：维持原来的情况，\r表示会车，\n表示换行。

因此，常见的现象是，如果一个文件在Unix环境下生成，在Windows中重新打开之后，会发现所有文字都在同一行；如果在Windows环境下生成的文件，在Unix或Mac下打开，每行的结尾会多出一个^M字符。



