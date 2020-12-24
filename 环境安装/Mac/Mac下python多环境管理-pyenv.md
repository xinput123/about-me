## 1、安装homebrew
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
Homebrew安装成功后，会自动创建目录 /usr/local/Cellar 来存放Homebrew安装的程序

## 2、安装pyenv
```
brew update
brew install pyenv
pyenv -v # 安装之后查看 pyenv 版本，确认是否安装成功
```

## 3、安装&管理多个Python
```
pyenv install -l  # 查看 pyenv 可以安装哪些Python版本
pyenv install 2.7.15
pyenv install 3.7.3
pyenv versions # 所有已经安装的版本
```

## 4、切换版本
```
pyenv global 3.7.3 # 不建议全局切换

python -V  # 验证一下是否切换成功

pyevn global system  # 切换回系统版本

pyenv local 3.7.3  # 当前目录及其目录切换

python -V  # 验证一下是否切换成功

pyenv local --unset  # 解除local设置

pyenv shell 3.7.3  # 当前shell会话切换

python -V  # 验证一下是否切换成功

pyenv shell --unset  # 解除shell设置
```

## 5、切换不成功
如果遇到切换之后，Python版本还是系统的默认版本的话，就需要配置一下环境变量，在 ~/.zshrc 或 ~/.bash_profile 文件最后写入：
```
export PYENV_ROOT=~/.pyenv
export PATH=$PYENV_ROOT/shims:$PATH
if which pyenv > /dev/null;
  then eval "$(pyenv init -)";
fi
```

使配置生效 
source ~/.bash_profile
