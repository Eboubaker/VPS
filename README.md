```sh
#? Windows pwsh
#!!! ssh root@eboubaker.xyz


# Set Correct Time
#? root
timedatectl set-timezone Africa/Algiers

# Increase Swap file to add more 4GB of SSD as memory
#? root
swapoff /swapfile && \
dd if=/dev/zero of=/swapfile bs=1M count=2048 oflag=append conv=notrunc && \
chmod 600 /swapfile && \
mkswap /swapfile && \
swapon /swapfile && \
printf "/swapfile  none                    swap    sw                   0 0\n" >> /etc/fstab  && \
free -h #check


# Update System & Packages
#? root
yum install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
yum -y update


# add myself
#? root
useradd -m eboubaker \
passwd eboubaker \
usermod -G wheel eboubaker \
exit

# Disable PasswordAuthentication and enable PublicKeyAuthentication
#? Windows pwsh
#!! ssh-keygen -t rsa -b 4096 #Choose name RockyVPS
#!! scp "C:\Users\me\.ssh\RockyVPS\RockyVPS.pub" eboubaker@eboubaker.xyz:
#!! ssh eboubaker@eboubaker.xyz

#? eboubaker
mkdir ~/.ssh && \
cat ~/RockyVPS.pub >> ~/.ssh/authorized_keys && \
rm ~/RockyVPS.pub && \
chown -R eboubaker:eboubaker ~/.ssh && \
chmod -R go= ~/.ssh && \
sudo nano /etc/ssh/sshd_config # set passwordauth no | publickeyauth yes | rootauth no
sudo systemctl restart sshd
# check if it works?
#? Windows pwsh
#!! ssh -i "C:\Users\me\.ssh\RockyVPS\RockyVPS" eboubaker@eboubaker.xyz
# Then Restart the VPS

#fireWall stuff
#? eboubaker 
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --zone=public --add-port=8000-10000/tcp 
sudo firewall-cmd --reload


# Install some stuff
#? eboubaker 
sudo yum group install -y "Development Tools" && \
sudo yum install -y wget yum-utils java-11-openjdk-devel util-linux-user goaccess && \
sudo yum module enable -y nodejs:16 && \
sudo yum install -y npm && \
sudo npm i -g pnpm && \
sudo yum module enable -y php:remi-8.1/devel && \
sudo yum install -y vim nginx php composer php-pear php-devel curl-devel && \
sudo pear config-set php_ini /etc/php.ini && \
sudo pecl channel-update pecl.php.net && \
printf "y\ny\ny\ny\ny\ny\n" | sudo pecl install swoole && \
sudo wget -O /etc/cacert.pem https://curl.haxx.se/ca/cacert.pem && \
echo curl.cainfo="/etc/cacert.pem" | sudo tee -a /etc/php.ini && \
echo openssl.cafile="/etc/cacert.pem" | sudo tee -a /etc/php.ini
# Then Restart the VPS


#Install ohmyzsh & autosuggestions plugin
#? eboubaker
sudo yum install -y zsh && \
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)" && \
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions && \
echo source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh >> ~/.zshrc && \
. ~/.zshrc
# logout and login from ssh


# Wireguard VPN Server setup, commit 842361bc6 was choosed(for security not installing from master)
#? root
wget -O ~/wgconfig.sh https://raw.githubusercontent.com/angristan/wireguard-install/842361bc6ed9686371baa4b5727c7e501ddee487/wireguard-install.sh && \
chmod +x ~/wgconfig.sh && \
# Restart system before adding a new client
printf "\n\n\n\n\n\n\n\n\n" | bash ~/wgconfig.sh


# Install Docker
#? root
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum update -y
yum install -y docker-ce docker-ce-cli containerd.io
curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o /usr/bin/docker-compose
chmod +x /usr/bin/docker-compose


# SSL certificate
sudo yum install -y snapd certbot python3-certbot-nginx
sudo certbot --nginx

# Install Cockpit
#? eboubaker
sudo yum install -y cockpit && \
sudo systemctl start cockpit.socket && \
sudo systemctl enable --now cockpit.socket && \
sudo firewall-cmd --add-service=cockpit --permanent && \
sudo firewall-cmd --reload
# Follow these guides to proxy the cockpit panel into nginx (enables ssl and logs)
# https://github.com/cockpit-project/cockpit/wiki/Proxying-Cockpit-over-nginx
# https://cockpit-project.org/guide/latest/listen.html
```
