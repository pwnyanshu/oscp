### On Linux/macOS

For _viewing_ (no editing, just reading):

- `cat filename` → spits the whole file at once (fast, but not friendly for long files).
    
- `less filename` → scrollable, search-able, doesn’t dump everything. This is the sweet spot.
    
- `more filename` → older cousin of `less`, step-through instead of scroll.
    
- `head filename` → shows first 10 lines (use `-n 20` for 20 lines, etc.).
    
- `tail filename` → shows last 10 lines (useful for logs, `tail -f file` follows in real time).
    

For _editing_ without vim/nano headaches:

- `micro filename` → modern, user-friendly editor (install separately).
    
- `gedit filename` → GUI text editor (if you’re in a desktop environment).
    

### On Windows (CMD/PowerShell)

Viewing options:

- `type filename.txt` → like Linux’s `cat`, dumps the whole file.
    
- `more filename.txt` → page-by-page view.
    
- `Get-Content filename.txt` (PowerShell) → same as `cat`. You can even `gc filename.txt` (short alias).
    

Editing options:

- `notepad filename.txt` → opens in Notepad GUI.
    
- `code filename.txt` (if you have VS Code installed and added to PATH).