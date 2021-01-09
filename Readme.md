# Setup guide for dev environment on Apple Silicon.

A guide to setup a development environment using Homebrew, Python 3.9.1 & 3.8.6, Pyenv, Poetry, Tensorflow, Numpy and Pandas on new Apple Silicon M1 macs running Big Sur 11.1. Note that while working through this guide, it's best practice to restart your terminal after each installation / major change affecting your .zshrc file.


__Install Xcode:__
- install xcode from app store v12+
- then install CommandLine Tools

`xcode-select --install`

- set CommandLine Tools version under Xcode Preferences > Locations

![](https://github.com/rybodiddly/Poetry-Pyenv-Homebrew-Numpy-TensorFlow-on-Apple-Silicon-M1/blob/main/xcode_prefs.jpg)

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


__Manually Install Tensorflow (which will also install Numpy and other dependencies) into Poetry Environment:__

- Download Apple's Tensorflow package.
- https://github.com/apple/tensorflow_macos/releases/download/v0.1alpha1/tensorflow_macos-0.1alpha1.tar.gz
- create a `packages` folder inside your new poetry project
- extract then copy all of the `whl` files from the apple tensorflow release into the `your_new_project/packages/` folder
- download the [Repack of apples tensorflow](https://github.com/rybodiddly/Poetry-Pyenv-Homebrew-Numpy-TensorFlow-on-Apple-Silicon-M1/releases/download/2.4.0rc0-Repack/tensorflow-2.4.0rc0-cp38-cp38-macosx_11_0_arm64.whl) from the releases section of this repository and copy it into the `your_new_project/packages/` folder (the Repack resolves dependency issues and includes a rebuild of pandas for M1)
- edit the `[tool.poetry.dependencies]` section of the `project.toml` file inside your project folder to look like this:
```
[tool.poetry.dependencies]
python = "^3.8"
numpy = {path = "packages/numpy-1.18.5-cp38-cp38-macosx_11_0_arm64.whl"}
grpcio = {path = "packages/grpcio-1.33.2-cp38-cp38-macosx_11_0_arm64.whl"}
h5py = {path = "packages/h5py-2.10.0-cp38-cp38-macosx_11_0_arm64.whl"}
tensorflow = {path = "packages/tensorflow-2.4.0rc0-cp38-cp38-macosx_11_0_arm64.whl"}
tensorflow_addons = {path = "packages/tensorflow_addons-0.11.2+mlcompute-cp38-cp38-macosx_11_0_arm64.whl"}
pandas = {path = "packages/pandas-1.2.0-cp38-cp38-macosx_11_1_arm64.whl"}
```
- also note the version python is set to `^3.8`, this is important for apples packages, as they are not compatible with `3.9`
- Note: if you want all your poetry projects to default to `3.8` instead of `3.9`, just use `pyenv local 3.8.6` in the base directory where you create and house your poetry projects. Or you could use `pyenv global 3.8.6`. Your call.
- then install the packages while inside your project folder:
`poetry add ./packages/tensorflow-2.4.0rc0-cp38-cp38-macosx_11_0_arm64.whl`


__Download VSCode for MacOS Arm64:__
- https://code.visualstudio.com/docs/?dv=darwinarm64&build=insiders
- you might have to temporarily disable the pip protections in your .zshrc while setting up some of the python modules in vscode
```
#export PIP_REQUIRE_VIRTUALENV=true
#gpip(){
#   PIP_REQUIRE_VIRTUALENV="" pip "$@"
#}
#gpip3(){
#   PIP_REQUIRE_VIRTUALENV="" pip3 "$@"
#}
```


__Sources:__
- https://medium.com/@briantorresgil/definitive-guide-to-python-on-mac-osx-65acd8d969d0
- https://gist.githubusercontent.com/nrubin29/bea5aa83e8dfa91370fe83b62dad6dfa/raw/48f48f7fef21abb308e129a80b3214c2538fc611/homebrew_m1.sh
- https://towardsdatascience.com/tensorflow-2-4-on-apple-silicon-m1-installation-under-conda-environment-ba6de962b3b8
- https://github.com/pyenv/pyenv/issues/1767
- https://github.com/python-poetry/poetry/issues/76
