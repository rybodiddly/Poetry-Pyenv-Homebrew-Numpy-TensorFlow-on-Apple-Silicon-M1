# Setup guide for dev environment on Apple Silicon.

A guide to setup a development environment using Homebrew, Python 3.9.1 & 3.8.6, Pyenv, Poetry, Numpy and Tensorflow on new Apple Silicon M1 macs. Note that while working through this guide, it's best practice to restart your terminal after each installation / major change affecting your .zshrc file.


__Install Xcode:__
- install xcode from app store v12+
- then install CommandLine Tools

`xcode-select --install`

- set CommandLine Tools version under Xcode Preferences > Locations

![](https://github.com/rybodiddly/Poetry-Pyenv-Homebrew-Numpy-TensorFlow-on-Apple-Silicon-M1/blob/main/xcode.jpg)

- add SDKROOT Path to .zshrc file

`export SDKROOT="/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk"`
- restart terminal


__Install Homebrew:__
```
cd /opt
sudo mkdir homebrew
sudo chown -R $(whoami) /opt/homebrew
curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
echo "export PATH=/opt/homebrew/bin:$PATH" >> ~/.zshrc
```
- restart terminal


__Install brew dependencies:__

`brew install openssl readline sqlite3 xz zlib`


__Install Pyenv:__

- install Pyenv using ![Pyenv Installer](https://github.com/pyenv/pyenv-installer) instead of brew (will allow a working 3.9.1 install)

`curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash`

- add the following to youir .zshrc file:
```
# pyenv
export PATH="$HOME/.pyenv/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi
```
- restart terminal


__Install python 3.9.x:__

`brew install python`
- make sure `python3 -V` results in newly installed version


__Install pipx to manage global packages:__
```
python3 -m pip install --user pipx
python3 -m userpath append ~/.local/bin
```

__Install global packages:__
```
python3 -m pipx install flake8
python3 -m pipx install black
```

- make sure your .zshrc file looks similar to this (replace "YOUR_USER_NAME" with your name):
```
export PATH="/opt/homebrew/bin:/Users/YOUR_USER_NAME/Library/Python/3.9/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin:$PATH"
export SDKROOT="/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk"

# pyenv
export PATH="$HOME/.pyenv/bin:$PATH"
if command -v pyenv 1>/dev/null 2>&1; then
  eval "$(pyenv init -)"
fi

# python3 Alias
alias python="python3"

# pipx
export PATH="~/.local/bin:$PATH"
```
- note the creation of a python3 Alias in the snippet above, this will prevent poetry from defaulting to python 2.


__Install Python 3.9.1 in Pyenv:__

`pyenv install 3.9.1`


__Install Python 3.8.6 in Pyenv:__
```
CFLAGS="-I$(brew --prefix openssl)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(xcrun --show-sdk-path)/usr/include" LDFLAGS="-L$(brew --prefix openssl)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib" pyenv install --patch 3.8.6 <<(curl -sSL https://raw.githubusercontent.com/Homebrew/formula-patches/113aa84/python/3.8.3.patch\?full_index\=1)
```

__Set Pyenv Global for Poetry:__

`pyenv global 3.9.1`


__Install Poetry:__
```
curl -sSL https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py | python

poetry config virtualenvs.in-project true
```

__Protect Global & Brew Python Packages from Mishaps:__

- add following to bottom of .zshrc
```
### Add these next lines to protect your system python from
### polution from 3rd-party packages

# pip should only run if there is a virtualenv currently activated
export PIP_REQUIRE_VIRTUALENV=true
 
# commands to override pip restriction above.
# use `gpip` or `gpip3` to force installation of
# a package in the global python environment
# Never do this! It is just an escape hatch.
gpip(){
   PIP_REQUIRE_VIRTUALENV="" pip "$@"
}
gpip3(){
   PIP_REQUIRE_VIRTUALENV="" pip3 "$@"
}
```

__Create Poetry Project__
```
poetry new name_of_project
cd name_of_project
pyenv local 3.8.6
```
- check python version with `python -V`
- should result in `3.8.6`


__Manually Install Numpy & Tensorflow into Poetry Environment:__

- Download Apple's Tensorflow package.
- https://github.com/apple/tensorflow_macos/releases/download/v0.1alpha1/tensorflow_macos-0.1alpha1.tar.gz
- extract
- create a packages folder in your new poetry project
- copy numpy or tensorflow from the apple tensorflow's arm64 folder into the packages folder you created inside you poetry project
```
cd name_of_project
poetry add ./packages/name_of_package_ARM64.whl
```

__Download VSCode for MacOS Arm64:__
- https://code.visualstudio.com/docs/?dv=darwinarm64&build=insiders


__Sources:__
- https://medium.com/@briantorresgil/definitive-guide-to-python-on-mac-osx-65acd8d969d0
- https://gist.githubusercontent.com/nrubin29/bea5aa83e8dfa91370fe83b62dad6dfa/raw/48f48f7fef21abb308e129a80b3214c2538fc611/homebrew_m1.sh
- https://towardsdatascience.com/tensorflow-2-4-on-apple-silicon-m1-installation-under-conda-environment-ba6de962b3b8
- https://github.com/pyenv/pyenv/issues/1767
