# การเริ่มโพรเซสใหม่ในลินุกซ์

ตัวอย่างโปรแกรม lab2_1.cpp 
-----
```cpp

#ifdef HAVE_CONFIG_H
#include <config.h>
#endif

#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h> // for exit()
#include <wait.h>   // for wait()

int main()
{
	pid_t  pid;

	pid = fork();  //Fork a child process

        if(pid < 0){ //Fork error
		fprintf(stderr,"Fork failed.\n");
                exit(-1);
        }
        else if(pid==0){ // This is the path of child process
            printf("Before replace the child with other code...\n");
            execlp("/usr/bin/nano","nano",NULL); // call a text editor
	}
	else { // This is the path of parent process
	    printf("Before going into the wait  state...\n");
	    wait(NULL);
	    printf("Child process has terminated\n");
            exit(0);
        }
}

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

ตัวแปร pid_t ทำหน้าที่เก็บค่าของ PID โดยค่าเริ่มต้นจะเป็น 0
```cpp
pid_t  pid;
```
ฟั่งชั่น fork()[^2]


[^1]:Text Editor ใน Linux
[^2]:[The fork() System Call](http://www.csl.mtu.edu/cs4411.ck/www/NOTES/process/fork/create.html)
