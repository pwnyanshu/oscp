
Grant permission

```bash
icacls WService.exe /grant Everyone:F
```




### Basic rights (shorthand)

- **F** → Full access
    
- **M** → Modify access
    
- **RX** → Read & execute
    
- **R** → Read-only
    
- **W** → Write-only
    

---

### Detailed rights

- **D** → Delete
    
- **RC** → Read control (read permissions, read security descriptor)
    
- **WDAC** → Write DAC (change permissions)
    
- **WO** → Write owner (take ownership)
    
- **S** → Synchronize (used for multi-thread/process sync)
    
- **AS** → Access system security (change audit/security settings)
    
- **MA** → Maximum allowed
    

---

### Object access rights

- **GR** → Generic read
    
- **GW** → Generic write
    
- **GE** → Generic execute
    
- **GA** → Generic all
    

---

### Inheritance flags (this is where those single letters like `(I)` show up)

- **(I)** → ACE (Access Control Entry) is inherited from parent
    
- **(OI)** → Object inherit (applies to files in a folder)
    
- **(CI)** → Container inherit (applies to subfolders)
    
- **(IO)** → Inherit only (does not apply to the object itself, only children)
    
- **(NP)** → Do not propagate inherit (stops inheritance beyond one level)
    
- **(ID)** → ACE inherited by default from parent container
    

---

### Execution-specific shorthand

- **X** → Execute
    
- **RX** → Read + Execute
    
- **M** → Modify (basically R/W/X without full control)
    
- **I** (in parentheses only) → Inherited permission

---

- **AD** → _Add File_ (also called **Append Data**)
- **WD** → _Write Data_