# Setup guide for dev environment on Apple Silicon.

__Install Xcode:__
- install xcode from app store v12+
- set version under Preferences > Locations

img


__Install Homebrew:__
```
cd /opt
sudo mkdir homebrew
sudo chown -R $(whoami) /opt/homebrew
curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
echo "export PATH=/opt/homebrew/bin:$PATH" >> ~/.zshrc
```

__Install Python 3.9.1:__

`brew install python`


__Install Numpy:__

`poetry add ./path/to/package.whl`
