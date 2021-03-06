# 為exec調用設置catchpoint
## 例子
	#include <unistd.h>
	
	int main(void) {
	    execl("/bin/ls", "ls", NULL);
	    return 0;
	}



## 技巧
使用gdb調試程序時，可以用“`catch exec`”命令為`exec`系列系統調用設置`catchpoint`，以上面程序為例：  

	(gdb) catch exec
	Catchpoint 1 (exec)
	(gdb) r
	Starting program: /home/nan/a
	process 32927 is executing new program: /bin/ls
	
	Catchpoint 1 (exec'd /bin/ls), 0x00000034e3a00b00 in _start () from /lib64/ld-linux-x86-64.so.2
	(gdb) bt
	#0  0x00000034e3a00b00 in _start () from /lib64/ld-linux-x86-64.so.2
	#1  0x0000000000000001 in ?? ()
	#2  0x00007fffffffe73d in ?? ()
	#3  0x0000000000000000 in ?? ()


可以看到當`execl`調用發生後，gdb會暫停程序的運行。  
注意：目前只有HP-UX和GNU/Linux支持這個功能。  
參見[gdb手冊](https://sourceware.org/gdb/onlinedocs/gdb/Set-Catchpoints.html).

## 貢獻者

nanxiao
