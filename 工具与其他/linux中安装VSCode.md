# linux中安装VSCode



---

## １.从官网下载
访问Visual Studio Code官网 https://code.visualstudio.com/docs?dv=linux64
或者直接用下列命令下载64位安装包：
```
wget https://az764295.vo.msecnd.net/stable/7ba55c5860b152d999dda59393ca3ebeb1b5c85f/code-stable-code_1.7.2-1479766213_amd64.tar.gz
```
## 2.解压
```
tar xzvf code-stable-code_1.7.2-1479766213_amd64.tar.gz
```
## 3.移动到local/usr下
```
mv VSCode-linux-x64 /usr/local/
```
## 4.可能还需要给可执行的权限, 然后就已经可以运行了
```
chmod +x /usr/local/VSCode-linux-x64/code
```
## 5.复制一个VScode图标文件到 /usr/share/icons/ 目录
```
cp /usr/local/VSCode-linux-x64/resources/app/resources/linux/code.png /usr/share/icons/
```
## 6.创建启动器, 在/usr/share/applications/ 目录, 也可以将它复制到桌面目录
```
vim /usr/share/applications/VSCode.desktop
```
输入下列文字;
```
[Desktop Entry]
Name=Visual Studio Code
Comment=Multi-platform code editor for Linux
Exec=/usr/local/VSCode-linux-x64/code
Icon=/usr/share/icons/code.png
Type=Application
StartupNotify=true
Categories=TextEditor;Development;Utility;
MimeType=text/plain;
```
保存后退出, 然后可以复制到桌面:
cp /usr/share/applications/VSCode.desktop ~/桌面/
之后 就会发现 桌面和 应用程序菜单都有了 VSCode的快捷方式了

