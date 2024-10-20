
本文讲述安装了Kali Linux 2024\.3,Gnome桌面，以及安装各种应用与美化的过程。


## 安装操作系统


网络上下载操作系统镜像，写入u盘，制作启动盘
[https://mirrors.tuna.tsinghua.edu.cn/kali\-images/current/kali\-linux\-2024\.3\-installer\-amd64\.iso](https://github.com)


查看硬盘，找到你的U盘



```
sudo fdisk -l

```

找到U盘为/dev/sdb



```
dd if=./kali-linux-2024.3-installer-amd64.iso of=/dev/sdb bs=4M status=progress


```

等待写入完毕，即可重启，进入引导，进U盘操作系统安装到你的硬盘上


## 更换软件源



```
sudo vim /etc/apt/sources.list

```

内容替换为



```
deb https://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main non-free contrib non-free-firmware

```

## 安装一些依赖



```
sudo apt install btrfs-progs xfsprogs (这里因为用了btrfs或者xfs文件系统，所以需要)
sudo apt update
sudo apt upgrade
sudo apt install okular 
sudo apt remove texlive-* 

```

## 安装中文输入法



```
sudo apt remove fcitx5-*
sudo apt install fcitx

sudo apt install qtbase5-dev
sudo apt install libqt5qml5 libqt5quick5 libqt5quickwidgets5 qml-module-qtquick2  
sudo apt install libgsettings-qt1

sudo apt install ./sogoupinyin_4.2.1.145_amd64.deb

```

此时可以使用im\-config来配置系统使用fcitx输入框架，重启后，系统自动启动fcitx输入法框架，使用fcitx\-config进去添加一项sogoupinyin一项即可。


但是，这时你会发现，虽然配置好了，但是不能使用，可以运行`/opt/sogoupinyin/files/bin/sogoupinyin-configtool`来测试一下，会发现存在报错。


这应该是sogou自带的qt5库的问题，需要修改方能使用。


sogou输入法安装在/opt/sogoupinyin/files目录下，可以使用`/opt/sogoupinyin/files/bin/sogoupinyin-configtool`查看其动态库的使用。我们可以发现，其使用了`/opt/sogoupinyin/files/lib/qt5`里面的动态库，存在一定的问题，我们需要将其换成操作系统自带的qt5库里面，我们前文已经安装了所有所需的qt5的库，在`/usr/lib/x86_64-linux-gnu/qt5/`，我们将其引入


将文件/opt/sogoupinyin/files/bin/qt.conf修改为



```
[Paths]
Prefix = /usr/lib/x86_64-linux-gnu/qt5/
Plugins = plugins


```

删除sogou自带的库



```
sudo rm /opt/sogoupinyin/files/lib/qt5 -rf

```

这时基本已经好使了，可以运行`/opt/sogoupinyin/files/bin/sogoupinyin-configtool`来测试一下，一般已经能显示这个界面了，这样一般就不缺少东西了，整个都能用了


## 安装docker


安装由debian维护的docker.io，参考https://www.kali.org/docs/containers/installing\-docker\-on\-kali/



```
sudo apt update
sudo apt install docker.io
sudo systemctl enable docker --now


```

为普通用户添加docker的权限



```
sudo usermod -aG docker $USER

```

更改docker镜像（参考了https://github.com/yuzhihui/p/17461781\.html）



```
sudo vim /etc/docker/daemon.json 

```

添加内容如下



```
{
  "registry-mirrors": [
	  "https://dockerproxy.cn"		
  ]
}

```

然后重启docker容器



```
sudo systemctl restart docker

```

安装docker\-compose



```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.29.7/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

```

测试



```
docker run hello-world

```

## 安装conda


去网上下载Miniconda的安装包



```
sh ./Miniconda3-py38_4.12.0-Linux-x86_64.sh

```

更换conda源，



```
vim ~/.condarc

```

填入以下内容，(from [https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/](https://github.com))



```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud


```

创建conda的环境



```
conda create -n py311 python=3.11
conda activate py311
conda install pytorch torchvision torchaudio pytorch-cuda=12.4 -c pytorch -c nvidia
conda install tensorboard torchmetrics matplotlib numpy

conda install -c conda-forge diffusers accelerate transformers gpustat loguru

```

## 安装texlive


装之前先给系统带的texlive卸载



```
sudo apt remove texlive-*

```

挂载上下载的texlive镜像包（[https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/texlive/Images/texlive2024\-20240312\.iso），进入运行安装程序](https://github.com)



```
sudo ./install-tl

```

一路安装即可。


## 配置Latex编译与双向搜索


为vscode官网下载安装包([https://code.visualstudio.com/docs/?dv\=linux64\_deb](https://github.com):[milou加速器](https://xinminxuehui.org))



```
sudo apt install ./code_1.94.2-1728494015_amd64.deb

```

安装okular



```
sudo apt install okular

```

进入vscode，安装latex workshop插件


配置vscode配置



```
{
    "workbench.colorTheme": "Visual Studio 2017 Light - C++",
    "workbench.iconTheme": "material-icon-theme",
    "editor.fontSize": 16,

    # 这里开始latex配置，自动打开okular来预览
    "latex-workshop.view.pdf.external.viewer.command": "okular",
    "latex-workshop.view.pdf.external.viewer.args": [
      "--unique",
      "%PDF%"
    ],
    "latex-workshop.view.pdf.viewer": "external",
    "latex-workshop.view.pdf.external.synctex.command": "okular",
    "latex-workshop.view.pdf.external.synctex.args": [
      "--unique",
      "%PDF%#src:%LINE%%TEX%"
    ],
    "editor.wordWrap": "on",
    "[html]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
	# 关闭自动保存，关闭自动编译
    "files.autoSaveDelay": 15000,
    "files.autoSave": "afterDelay",
    "latex-workshop.latex.autoBuild.run": "never",
    "latex-workshop.latex.autoBuild.cleanAndRetry.enabled": false
}

```

在okular中设置中，editor中选择其他编辑器，跳转指令设置为



```
code --goto %f:%l

```

即可在vscode中，ctrl\+alt\+j跳转到PDF中，在okular中shift\+click跳转到latex源码对应行


## 安装Zotero


去官网下载安装包，（[https://www.zotero.org/download/client/dl?channel\=release\&platform\=linux\-x86\_64\&version\=7\.0\.8）](https://github.com)


解压到你的目录中，配置desktop来作为程序入口



```
sudo vim /usr/share/applications/zotero.desktop

```

其中填入：



```
[Desktop Entry]
Name=Zotero
Exec=bash -c "/home/abc/APP/Zotero/Zotero_linux-x86_64/zotero -url %U"
Icon=/home/abc/APP/Zotero/Zotero_linux-x86_64/icons/icon128.png
Type=Application
Terminal=false
Categories=Office;
MimeType=text/plain;x-scheme-handler/zotero;application/x-research-info-systems;text/x-research-info-systems;text/ris;application/x-endnote-refer;application/x-inst-for-Scientific-info;application/mods+xml;application/rdf+xml;application/x-bibtex;text/x-bibtex;application/marc;application/vnd.citationstyles.style+xml
X-GNOME-SingleWindow=true

```

给这里的Exec和Icon行修改为你的路径即可


安装zotero插件


翻译插件：


[https://github.com/windingwind/zotero\-pdf\-translate](https://github.com)


可以根据翻译插件去设置API


## 安装服务


对于某些软件，需要开机自启动，可以做成服务



```
sudo vim /usr/lib/systemd/system/***.service

```

填入



```
Description=*** daemon  
[Service] 
Type=simple 
User=root 
ExecStart=/home/abc/APP/***/***-linux-amd64 -d /home/abc/APP/***/
Restart=on-failure  
[Install] 
WantedBy=multi-user.target

```

设置开机自启动



```
sudo systemctl enable ***.service
sudo systemctl start **.service

```

查看状态



```
sudo systemctl status ***.service

```

## 安装flatpak


安装flatpak，并设置flathub，并使用sjtu的flathub镜像



```
sudo apt install flatpak
sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
sudo flatpak remote-modify flathub --url=https://mirror.sjtu.edu.cn/flathub

```

安装flatseal



```
sudo flatpak install flathub com.github.tchx84.Flatseal

```

## 安装WPS


使用lfatpak安装WPS



```
sudo flatpak install flathub com.wps.Office

```

安装缺失字体([https://github.com/dv\-anomaly/ttf\-wps\-fonts](https://github.com))



```
cd /tmp
git clone https://github.com/iamdh4/ttf-wps-fonts.git
cd ttf-wps-fonts
sudo bash install.sh
cd .
rm -rf /tmp/ttf-wps-fonts

```

## 安装QQ 微信



```
sudo flatpak install flathub com.qq.QQ
sudo flatpak install flathub com.tencent.WeChat

```

