
like RDP windows app can save creds, or a remoteshare so it can use it again and again

we in BEO have literally windows RDP password stored in the app, so that just on double click it opens the rdp machine.

Windows allows us to use other users' credentials. This function also gives the option to save these credentials on the system. The command below will list saved credentials:

```shell-session
cmdkey /list
```

![[Pasted image 20250910002244.png]]

While you can't see the actual passwords, if you notice any credentials worth trying, you can use them with the `runas` command and the `/savecred` option, as seen below.

```shell-session
runas /savecred /user:admin cmd.exe
```

![[Pasted image 20250910002259.png]]