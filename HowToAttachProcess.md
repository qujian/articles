
## senaro:

I need to debug in Docker use vscode, and my docker is login as a non-root user, so I met the problem: 


```bash
Superuser access is required to attach to a process, but authorisation can't be confirmed since there is an extra line in the terminal.

```

so I tried to attach the process with a gdb command in docker, but it 


```bash
$ gdb --pid 5291
GNU gdb (Ubuntu 12.0.90-0ubuntu1) 12.0.90
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
Attaching to process 5291
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
(gdb)


```

try to change 


```bash
vi /etc/sysctl.d/10-ptrace.conf

```


```bash
kernel.yama.ptrace_scope = 1

```

to:


```bash
kernel.yama.ptrace_scope = 0

```

and got:


```bash
$ sysctl -p /etc/sysctl.d/10-ptrace.conf 
sysctl: setting key "kernel.yama.ptrace_scope", ignoring: Read-only file system

```

the try to change value in the host:


```bash
echo 0 > /proc/sys/kernel/yama/ptrace_scope 

# go to docker
cat /proc/sys/kernel/yama/ptrace_scope 
0

```

and now I can attach debug in my vscode `launch.json`


```json
"configurations": [
      {
        "name": "attach test",
        "type": "cppdbg",
        "request": "attach",
        "program": "${workspaceFolder}/test",
        "args": [],
        "processId": "${command:pickProcess}",
        "miDebuggerPath": "/usr/bin/gdb",
        //"preLaunchTask": "source",
        "stopAtEntry": false,
        "cwd": "${workspaceFolder}",
        //"externalConsole": false,
        //"envFile": "${workspaceFolder}/.vscode/env",
        //"console":"none",
        "MIMode": "gdb",
        "setupCommands": [
          {
            "description": "Enable pretty-printing for gdb",
            "text": "-enable-pretty-printing",
            "ignoreFailures": true
          }
        ]
      }
]


```

## reference:

[/proc/sys/kernel/yama/ptrace_scope keeps resetting to 1](https://unix.stackexchange.com/questions/329504/proc-sys-kernel-yama-ptrace-scope-keeps-resetting-to-1)

[docker中调试失败](https://blog.csdn.net/happytree001/article/details/125696209)

## question link:

[Superuser access is required to attach to a process, but authorisation can't be confirmed since there is an extra line in the terminal.](https://github.com/microsoft/vscode-cpptools/issues/6524)


