When doing windows pwn, shellcoding,you often need to call a function in kernel32.dll, or another system dll.<br>

You can leak kernel32.dll base easily with a shellcode beginning like this:<br>

```assembly
xor rdi, rdi            /* RDI = 0x0 */
mul rdi                 /* RAX&RDX =0x0 */
mov rbx, gs:[rax+0x60]  /* RBX = Address_of_PEB */
mov rbx, [rbx+0x18]     /* RBX = Address_of_LDR */
mov rbx, [rbx+0x20]     /* RBX = 1st entry in InitOrderModuleList / ntdll.dll  */
mov rbx, [rbx]          /* RBX = 2nd entry in InitOrderModuleList / kernelbase.dll */
mov rbx, [rbx]          /* RBX = 3rd entry in InitOrderModuleList / kernel32.dll */
mov rbx, [rbx+0x20]     /* RBX = &kernel32.dll ( Base Address of kernel32.dll) */
mov r8, rbx             /* RBX & R8 = &kernel32.dll */
```

this is the classic way to retrieve kernel32.dll base, from PEB,LDR,etc..<br>

then the next step is to search for the address of the wanted function, by searching his ascii name string for example..<br>

that works good with many windows versions, but result in big shellcode<br>

like in this exploit for example:<br>

[https://github.com/nobodyisnobody/write-ups/blob/main/INTENT.CTF.2022/pwn/PwnME/working.exploit.py](https://github.com/nobodyisnobody/write-ups/blob/main/INTENT.CTF.2022/pwn/PwnME/working.exploit.py)

sometimes you need to write shorter shellcode, because of space<br>

another more short way to do it, if you know the kernel32.dll version, or if you can guess it with a leak.<br>

is to just add the function offset to known kernel32.dll base..which result in way shorter shellcode..<br>

so I start making a database of various kernel32.dll versions symbols lists, in text files, that I can parse to quickly find a specific functions offsets with his name, for my own pwn usage.<br>

You can also use it to identify a remote kernel32.dll version by searching for its lower 12bits.. for example (like libc-datase does):<br>

lets say for example you have a leak of VirtualProtect function, and it's ending by `990`, you can just do:<br>

```bash
grep --color=auto -rnw 'windows.dll.symbols/' -ie "VirtualProtect" --color=always 2> /dev/null | grep 990
windows.dll.symbols/kernel32.dll/64.bits/10.0.20348.1070/kernel32.dll.64bits.version.10.0.20348.1070.symbols.txt:1534:  0001B990  1518 VirtualProtect
```

so you will know that the remote kernel32.dll vesion is `10.0.20348.1070`<br>

now you can now calculate the address of any other functions with just an `add` to the leaked function address, and offset of wanted function..<br>

that's the idea..<br>

I will add other versions when I found them, I can not redistribute kernel32.dll binary, so only the symbols list textfile will be put here..<br>

If you have other versions that are missing here, you can fill an issue, I will add them to the list..<br>

I use these simple command script to create the text file:

```bash
VERSION=`peres -v kernel32.dll|tail -n 1 | awk '{print $3}'`
winedump -j export kernel32.dll > kernel32.dll.64bits.version.$VERSION.symbols.txt
```

the `winedump` tool is a part of wine installation

the `peres` tool can be installed on ubuntu (debian) whith:
```bash
sudo apt-get install pev
```

happy hacking...hope this will be useful for you too..
