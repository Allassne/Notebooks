curl https://releases.rancher.com/install-docker/20.10.sh | sh

sudo systemctl enable docker 
sudo systemctl start docker
sudo systemctl status docker

sudo usermod -aG docker albam