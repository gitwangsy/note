```shell
# 查看python版本(后面操作需要3.7以上版本)
python --version
# 如果版本低于3.7，先查看系统中有哪些python版本
ls /usr/bin/python*
# 如果没有python3.7以上版本，安装python3.8
sudo apt-get install python3.8
ls /usr/bin/python*
# 为某个特定用户修改python默认版本，在~/.bashrc文件中创建alias(别名)(如不使用bash,则修改对应的rc配置文件)
vim ~/.bashrc
alias python='/usr/bin/python3.8'
# 使配置生效
source ~/.bashrc
python --version # 查看python版本
# 安装pip (pip 是 Python 包管理工具，该工具提供了对 Python 包的查找、下载、安装、卸载的功能。)
sudo apt-get install python3-pip
# 升级pip
python -m pip install --upgrade pip
pip --version # 查看pip版本，要对应3.8
# 安装build/lite (ohos-build的依赖)
python -m pip install --user build/lite # -m参数表示在当前环境下安装 --user参数表示安装到用户目录下即~/.local/bin
# 安装ohos-build (ohos-build是一个用于构建HarmonyOS的工具)
python -m pip install --user --upgrade ohos-build
hb -h # 查看ohos-build的帮助信息
# 以上任意一步出现权限问题，可使用以下命令
sudo chown -R $USER ~/.local
# 出现python问题，可使用以下命令
source py38/bin/activate
# 配置环境变量
vim ~/.bashrc
export PATH=~/.local/bin:$PATH
source ~/.bashrc
```
