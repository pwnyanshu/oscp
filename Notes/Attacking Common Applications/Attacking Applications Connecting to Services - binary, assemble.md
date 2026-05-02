
Applications that are connected to services often include connection strings that can be leaked if they are not protected sufficiently.

---
## ELF Executable Examination

Lets say `octopus_checker` a binary is found on a remote machine during the testing. Running the application locally reveals that it connects to database instances in order to verify that they are available.

Run to see what the binary does
```shell
./octopus_checker

# The binary probably connects using a SQL connection string that contains credentials.
```

Using tools like [PEDA](https://github.com/longld/peda) (Python Exploit Development Assistance for GDB) we can further examine the file.
This is an extension of the standard GNU Debugger (GDB), which is used for debugging C and C++ programs.
GDB is a command line tool that lets you step through the code, set breakpoints, and examine and change variables.

```bash
gdb ./octopus_checker
```

Once the binary is loaded, we set the `disassembly-flavor` to define the display style of the code, and we proceed with disassembling the main function of the program.

```shell
gdb-peda$ set disassembly-flavor intel
gdb-peda$ disas main
```
![[Pasted image 20260411015547.png]]

This reveals several call instructions that point to addresses containing strings. They appear to be sections of a SQL connection string, but the sections are not in order, and the endianness entails that the string text is reversed. Endianness defines the order that the bytes are read in different architectures. Further down the function, we see a call to SQLDriverConnect.

![[Pasted image 20260411015615.png]]

Adding a breakpoint at this address and running the program once again, reveals a SQL connection string in the RDX register address, containing the credentials for a local database instance.

```
gdb-peda$ b *0x5555555551b0
gdb-peda$ run
```
![[Pasted image 20260411015743.png]]

Apart from trying to connect to the MS SQL service, penetration testers can also check if the password is reusable from users of the same network.

---
## DLL File Examination

A DLL file is a `Dynamically Linked Library` and it contains code that is called from other programs while they are running. 

The `MultimasterAPI.dll` binary is found on a remote machine during the enumeration process. 

Examination of the file reveals that this is a .Net assembly.

```
C:\> Get-FileMetaData .\MultimasterAPI.dll

<SNIP>
M .NETFramework,Version=v4.6.1 TFrameworkDisplayName.NET Framework 4.6.1    api/getColleagues        ! htt
p://localhost:8081*POST         Ò^         øJ  ø,  RSDSœ»¡ÍuqœK£"Y¿bˆ   C:\Users\Hazard\Desktop\Stuff\Multimast
<SNIP>
```

Using the debugger and .NET assembly editor [dnSpy](https://github.com/0xd4d/dnSpy), we can view the source code directly. 

This tool allows reading, editing, and debugging the source code of a .NET assembly (C# and Visual Basic).

Inspection of `MultimasterAPI.Controllers` -> `ColleagueController` reveals a database connection string containing the password.

![[Pasted image 20260411013040.png]]

Apart from trying to connect to the MS SQL service, attacks like password spraying can also be used to test the security of other services.