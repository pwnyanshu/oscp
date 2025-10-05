
like every-time - 

```bash
xfreerdp /u:USER /p:PASS /v:10.0.0.5 +clipboard /cert:ignore
```

---

## Useful Flags

- `/u:` → Username
    
- `/p:` → Password
    
- `/d:` → Domain
    
- `/v:` → Target host/IP
    
- `/f` → Fullscreen
    
- `/size:WxH` → Window size
    
- `/sec:rdp` → Force classic RDP (skip NLA)
    
- `/cert:ignore` → Ignore cert warnings
    
- `+clipboard` → Enable clipboard
    
- `/drive:share,PATH` → Share local dir
    
- `/sound:sys:alsa` → Redirect sound