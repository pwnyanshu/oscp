
### Install full docker engine
```bash
sudo apt update
sudo apt install docker.io -y
```

### Enable and start docker
```bash
sudo systemctl enable docker
sudo systemctl start docker
```


### Download the docker file
```bash
git clone https://github.com/lgandx/PCredz.git

cd PCredz

sudo docker build -t pcredz .
```


### Run it
```bash
sudo docker run --rm -v $(pwd):/data pcredz -f /data/demo.pcapng
```

![[Pasted image 20260323182922.png]]