## Kernel 6.1.21 - 2023-05-03 Image

Flash: 
Full OS: [Index of /raspios\_armhf/images/raspios\_armhf-2023-05-03](https://downloads.raspberrypi.org/raspios_armhf/images/raspios_armhf-2023-05-03/)
Lite: [Index of /raspios\_lite\_armhf/images/raspios\_lite\_armhf-2023-05-03](https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2023-05-03/)

---

> [!warning]
> boot config location change
> 
> ```
> sudo nano /boot/config.txt
> ```
> 
> ->
> 
> ```bash
> sudo nano /boot/firmware/config.txt
> ```
> 
> **Add the 32-bit Kernel Setting**: Scroll to the bottom of the file and add the following line:
> 
> ```
> arm_64bit=0
> ```
> reboot:
> ```bash
> sudo reboot
> ```

---
```bash
hexapod@hexapod:~$ uname -r
```
:
6.1.21-v7l+

---
Remove previous ssh key:
```bash
ssh-keygen -R hexapod
ssh-keygen -R 192.168.0.122
sss hexapod
```

---
Copy SSH key to GitHub settings
```bash
 ssh-keygen -t ed25519 -C "krystian.glodek1717@gmail.com"
 cat ~/.ssh/id_ed25519.pub 
```

---
```bash
sudo apt update && sudo apt install git -y
git clone git@github.com:Gl0dny/hexapod.git
cd hexapod/
git submodule update --init --recursive
```

---
Enable UART (No -> then Yes), I2C, SPI
```bash
sudo raspi-config
sudo reboot
```

---
Install ODAS and seeed driver
```bash
cd ~
./hexapod/lib/odas/install.sh 
source ~/.bashrc # ~.zshrc
cd hexapod/firmware/seeed-voicecard
git checkout hexapod
sudo ./install.sh --compat-kernel
sudo reboot
```

---

Confirm the output and block kernel
```bash
hexapod@hexapod:~ $ uname -r
5.4.51-v7l+

hexapod@hexapod:~ $ arecord -l
**** List of CAPTURE Hardware Devices ****
card 3: seeed8micvoicec [seeed-8mic-voicecard], device 0: bcm2835-i2s-ac10x-codec0 ac10x-codec.1-0035-0 [bcm2835-i2s-ac10x-codec0 ac10x-codec.1-0035-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0

# Block kernel and headers
sudo apt-mark hold raspberrypi-kernel # raspberrypi-kernel-headers
apt-mark showhold
```
You should see raspberrypi-kernel in the list.

---
Update system 
```bash
sudo apt update && sudo apt --fix-broken install -y
sudo apt update && sudo apt upgrade -y
```

---
Install Python 3.12.0
```bash
sudo apt update
sudo apt install -y \
  build-essential \
  libssl-dev \
  libbz2-dev \
  libreadline-dev \
  libsqlite3-dev \
  libncursesw5-dev \
  libffi-dev \
  liblzma-dev \
  zlib1g-dev \
  libgdbm-dev \
  libnss3-dev \
  libssl-dev \
  libncurses-dev \
  libreadline-dev

curl https://pyenv.run | bash

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
source ~/.zshrc  
	pyenv install --list | grep "3.12"
	pyenv install 3.12.0
	pyenv versions
	pyenv global 3.12.0
	pyenv shell 3.12.0
	python --version
```

---
Create virtual env and install requirements
```bash
python -m venv workspace
echo "alias activate='source ~/workspace/bin/activate'" >> ~/.bashrc
echo "alias activate='source ~/workspace/bin/activate'" >> ~/.zshrc
source ~/.zshrc
activate
cd hexapod
pip install --upgrade pip
pip install -r requirements.txt
```

## Test audio:

```bash
arecord -D hw:CARD=seeed8micvoicec,DEV=0 -d 3 -r 48000 -c 8 -f s32_le test.wav
```

Output file: 2 i 3 channel zmutowane ( powinen byc chyba 7 i 8 ) ale ODAS działa prawidłowo

#### Zsh setup

```bash
sudo apt update
sudo apt install -y \
zsh \
bat \
fzf \
unzip \
p7zip-full \
bzip2 \
gzip \
ripgrep \
htop \
procps \
tree \
zoxide \
stow

sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-completions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions

git clone git@github.com:Gl0dny/dotfiles.git
cd dotfiles/
git checkout hexapod
rm ~/.zshrc 
stow -R --adopt zsh

curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustc --version
cargo --version

cargo install eza
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
eza --version

git clone https://github.com/eth-p/bat-extras.git
cd bat-extras
./build.sh
sudo cp bin/* /usr/local/bin/
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
rm -rf bat-extras 
source ~/.zshrc

error:
/home/hexapod/.oh-my-zsh/plugins/emoji/emoji-char-definitions.zsh:25: character not in range (anon):29: character not in range
:

export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
sudo sed -i 's/^# *en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
sudo locale-gen
sudo update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8

source ~/.zshrc
```

#### Install LLVM 15 Manually on 32-bit RPi

```bash
sudo apt update
sudo apt install build-essential cmake ninja-build git python3-pip \
  libncurses5-dev libxml2-dev libedit-dev zlib1g-dev
```

```bash
git clone https://github.com/llvm/llvm-project.git
cd llvm-project
git checkout llvmorg-15.0.7
```

```bash
mkdir build && cd build
cmake -G Ninja ../llvm \
  -DCMAKE_BUILD_TYPE=Release \
  -DLLVM_TARGETS_TO_BUILD="ARM" \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DCMAKE_INSTALL_PREFIX=/usr/local
```

```bash
ninja -j$(nproc)
sudo ninja install
```

```bash
export LLVM_CONFIG=/usr/local/bin/llvm-config
pip install llvmlite numba resampy
```


## Flash Legacy 2021-05-07 buster Image ( Raw and old way )

Download image: [2021-05-07-buster](https://files.seeedstudio.com/linux/Raspberry%20Pi%204%20reSpeaker/2021-05-07-raspios-buster-armhf-lite-respeaker.img.xz)

## Remove the Preinstalled Driver

After booting the Pi with the flashed image:
```bash
cd seeed-voicecard
sudo ./uninstall.sh
sudo reboot
```

## Check Out  the rel-v5.5 Driver Branch ( ODAS Doa Fix )

```bash
git clone git@github.com:Gl0dny/hexapod.git
cd firmware/seeed-voicecard
git checkout hexapod_odas_doa_fix
sudo ./install.sh --compat-kernel
sudo reboot
```

### ( Optional - manual ) Check Out the rel-v5.5 Driver Branch

```bash
git clone https://github.com/respeaker/seeed-voicecard.git
cd seeed-voicecard
git checkout rel-v5.5
sudo ./install.sh --compat-kernel
sudo reboot
```

## gcc 8.4 for numpy  

```bash
sudo apt update sudo apt upgrade
sudo apt install build-essential libgmp-dev libmpfr-dev libmpc-dev
cd ~
wget https://ftp.gnu.org/gnu/gcc/gcc-8.4.0/gcc-8.4.0.tar.gz
tar -xf gcc-8.4.0.tar.gz
cd gcc-8.4.0
./contrib/download_prerequisites
cd ..
mkdir gcc-8.4.0-build
cd gcc-8.4.0-build

../gcc-8.4.0/configure \
    --prefix=/usr/local/gcc-8.4.0 \
    --enable-languages=c,c++ \
    --disable-multilib \
    --program-suffix=-8.4 \
    --with-arch=armv7-a \
    --with-fpu=vfp \
    --with-float=hard \
    --build=armv7l-linux-gnueabihf \
    --host=armv7l-linux-gnueabihf

make -j$(nproc)
sudo make install
```

1. **--with-float=hard**  
    Explicitly sets hard-float ABI (required for Raspberry Pi's ARMv7)
2. **--with-arch=armv7-a --with-fpu=vfp**  
    Targets correct ARM architecture and floating-point unit
3. **--build and --host Parameters**  
    Ensures correct platform identification (armv7l-linux-gnueabihf)
4. **Development Libraries**  
    libc6-dev-armhf-cross provides missing hard-float headers

### Verification After Install:

```bash
/usr/local/gcc-8.4.0/bin/gcc-8.4 -v 2>&1 | grep "Target"
# Should show: Target: armv7l-linux-gnueabihf
/usr/local/gcc-8.4.0/bin/gcc-8.4 -dM -E - </dev/null | grep -i float
# Should contain: #define __ARM_PCS_VFP 1
```

If you still encounter issues, check your system's header files:

```bash
ls /usr/include/gnu/stubs-hard.h  # Should exist after libc6-dev install
/usr/local/gcc-8.4.0/bin/gcc-8.4 --version
# Expected Output:
# gcc-8.4.0 (GCC) 8.4.0
# Copyright (C) 2018 Free Software Foundation, Inc.
# This is free software; see the source for copying conditions. There is NO
# warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
ls /usr/local/gcc-8.4.0/bin/
# You should see executables like:
# - gcc-8.4
# - g++-8.4
# - cpp-8.4
/usr/local/gcc-8.4.0/bin/gcc-8.4 -v 2>&1 | grep "Target"
# Target: armv7l-linux-gnueabihf
# (If it says gnueabi instead of gnueabihf, the hard-float configuration failed.)

# use gcc 8.4:
export PATH=/usr/local/gcc-8.4.0/bin:$PATH
export CC=gcc-8.4
export CXX=g++-8.4

# or instead to make permanent:  
sudo update-alternatives --install /usr/bin/gcc gcc /usr/local/gcc-8.4.0/bin/gcc-8.4 80 \
    --slave /usr/bin/g++ g++ /usr/local/gcc-8.4.0/bin/g++-8.4 \
    --slave /usr/bin/gcov gcov /usr/local/gcc-8.4.0/bin/gcov-8.4 # Adding gcov as a slave for completeness

sudo update-alternatives --config gcc

gcc --version
g++ --version

```


## Python 12 - pyenv


```bash
sudo apt update
sudo apt install -y \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libxml2-dev \
    libxmlsec1-dev \
    libffi-dev \
    liblzma-dev

curl https://pyenv.run | bash

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc  
pyenv install --list | grep "3.12"
pyenv install 3.12
pyenv versions
pyenv global 3.12
pyenv shell 3.12
python --version

sudo apt install libatlas-base-dev gfortran
pip install numpy --no-cache-dir --force-reinstall

```


## Test audio:

```bash
arecord -D hw:CARD=seeed8micvoicec,DEV=0 -d 3 -r 48000 -c 8 -f s32_le test.wav
```

Output file: 2 i 3 channel zmutowane ( powinen byc chyba 7 i 8 ) ale ODAS działa prawidłowo