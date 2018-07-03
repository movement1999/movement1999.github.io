# การเริ่มโพรเซสใหม่ในลินุกซ์

ตัวอย่างโปรแกรม lab2_1.cpp 
-----
```cpp
  1 #ifdef HAVE_CONFIG_H
  2 #include <config.h>
  3 #endif
  4
  5 #include <sys/types.h>
  6 #include <stdio.h>
  7 #include <unistd.h>
  8 #include <stdlib.h> // for exit()
  9 #include <wait.h>   // for wait()
 10
 11 int main(){
 12         pid_t  pid;
 13
 14         pid = fork();  //Fork a child process
 15
 16         if(pid < 0){ //Fork error
 17                 fprintf(stderr,"Fork failed.\n");
 18                 exit(-1);
 19         }
 20         else if(pid==0){ // This is the path of child process
 21             printf("Before replace the child with other code...\n");
 22             execlp("/usr/bin/nano","nano",NULL); // call a text editor
 23         }
 24         else { // This is the path of parent process
 25             printf("Before going into the wait  state...\n");
 26             wait(NULL);
 27             printf("Child process has terminated\n");
 28             exit(0);
 29         }
 30 }
```
โปรแกรมนี้จะสร้าง process ลูกที่เรียกโปรแกรม "nano"[^1] ขี้นมาและ process แม่จะรอจนกว่า



```console 

Movement@PC:~/code/c/os$ ls
lab2_1.cpp
Movement@PC:~/code/c/os$ g++ lab2_1.cpp -o lab2_1
Movement@PC:~/code/c/os$ ls
lab2_1  lab2_1.cpp
Movement@PC:~/code/c/os$
```
g++ เป็น compiler ภาษาซีพลัสพลัส วิธีใช้งานพิมพ์คำสัง "g++ file.c" ส่วนออฟชั่น "-o"เป็นการสังให้ 
output ออกมาเป็นชื่อที่เรากำหนด ในที่นี้เป็น "lab2_1"

---
## ทดสอบการทำงานของโปแกรม lab2_1

```console
Movement@PC:~/code/c/os$ ./lab2_1
```

 กรณีที่ nano ไม่ถูกเปิดขึ้นมาแสดงว่า nano ไม่ได้อยู่ใน "/usr/bin/nano" 
 ```cpp
 execlp("/usr/bin/nano","nano",NULL);
 ```
 ใช้คำสัง "which" ในการหาที่อยู่
```consolse
Movement@PC:~/code/c/os$ which nano
/bin/nano
```
เป็นที่อยู่ในฟังชั่น execlp จาก "/usr/bin/nano" เป็นที่อยู่ที่ได้มาใหม่กรณีเป็น
```cpp
execlp("/bin/nano","nano",NULL);
```
หลังรันโปรแกรม lab2_1 nanoจะถูกเปิดขึ้นมา
```console
  GNU nano 2.5.3                New Buffer                                      





















^G Get Help  ^O Write Out ^W Where Is  ^K Cut Text  ^J Justify   ^C Cur Pos
^X Exit      ^R Read File ^\ Replace   ^U Uncut Text^T To Spell  ^_ Go To Line
```

ดู process ที่กำลังทำงานอยู่โดยใช้คำสัง "ps -ef"
```console
Movement@PC:~$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 16:33 ?        00:00:00 /init ro
root         3     1  0 16:33 tty1     00:00:00 /init ro
Movement     4     3  0 16:33 tty1     00:00:00 -bash
Movement    30     4  0 16:33 tty1     00:00:00 vim fork-01
root        84     1  0 18:42 tty2     00:00:00 /init ro
Movement    85    84  0 18:42 tty2     00:00:00 -bash
Movement   105    85  0 18:42 tty2     00:00:00 vim
Movement   152    85  0 18:44 tty2     00:00:00 ./lab2_1
Movement   153   152  0 18:44 tty2     00:00:00 nano
root       154     1  0 18:44 tty3     00:00:00 /init ro
Movement   155   154  2 18:44 tty3     00:00:00 -bash
Movement   169   155  0 18:44 tty3     00:00:00 ps -ef
Movement@PC:~$
```
PID หมายถึง ไอดีของ process (Process ID)
PPID หมายถึง ไอดีแม่ของ process (Parent Process ID)

```console
Movement   152    85  0 18:44 tty2     00:00:00 ./lab2_1
Movement   153   152  0 18:44 tty2     00:00:00 nano
```
ในบรรทัดนี้เราจะว่า PID ของ lab2_1 เป็น 152
ส่วน nano มี PID 153 และ PPID 152 แสดงว่า nano เป็น process ลูกของ lab2_1

----
## อธิบาย code

ตัวแปร pid_t ทำหน้าที่เก็บค่าของ PID 
```cpp
pid_t  pid;
```
#### fork
ฟั่งชั่น fork[^2] จะสร้าง process ลูกขึ้นมาแล้วเริ่มทำงานเหมือน code ของ process แม่ แต่จะเริ่มทำงานที่บรรทัดที่มีการเรียกใช้งาน fork()
 - fork() จะคืนค่า PID ของ process ลูก
 - fork() จะคืนค่าลบถ้าสร้าง process ลูกไม่สำเสร็จ
 - fork() process ลูกที่ถูกสร้างที่ขึ้นมาโดย fork จะคืนค่า 0

 ตัวอย่าง fork.cpp
 ```cpp
  1 #include<sys/types.h>
  2 #include<stdio.h>
  3 #include<unistd.h>
  4
  5
  6 int main()
  7 {
  8     pid_t pid;
  9
 10     printf("Before fork()\n");
 11     pid=fork();
 12     printf("After fork() pid=%d\n",pid);
 13
 14     return 0;
 15
 16 }
 ```
 ทดลองรัน

```console
Movement@PC:~/code/c/os$ ./fork
Befor fork()
After fork() pid=268
After fork() pid=0
Movement@PC:~/code/c/os$
```
จะเห็นว่า "Before fork()" ถูกแสดงแค่ครั้งเดียวโดย process แม่ แต่"After fork()" ถูกแสดงถึงสองครั้งโดย process แม่และลูก 
เพราะว่า process ลูกจะเริ่มทำงานที่บรรทัดที่ fork ถูกเรียกใช้ทำให้ไม่มีการปริ้น "Before fork()" เก็บขึ้น


"After fork() pid=268" เป็นของ process แม่เก็บค่า PID ของลูกไว้ <br /> 
"After fork() pid=0" เป็นของ Process ลูก fork จะคืนค่า PID เป็นศูนย์


[^1]:Text Editor ใน Linux
[^2]:[The fork() System Call](http://www.csl.mtu.edu/cs4411.ck/www/NOTES/process/fork/create.html)
