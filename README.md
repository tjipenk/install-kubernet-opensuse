# install-kubernet-opensuse
## install docker
```sh
sudo zypper update
sudo zypper refresh

#ensure the swap is disable
sudo swapon --show 


sudo zypper install docker

#enable the docker service started on boot
sudo systemctl enable docker
sudo systemctl start docker

sudo docker ps
```
