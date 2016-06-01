sudo yum groupinstall "X Window System"
sudo yum groupinstall "Fonts"
sudo yum install gdm kde-workspace

sudo unlink /etc/systemd/system/default.target
sudo ln -sf /lib/systemd/system/graphical.target /etc/systemd/system/default.target

sudo yum install epel-release
sudo yum install openbox
sudo yum install -y chromium x11-xserver-utils unclutter ttf-mscorefonts-installer
