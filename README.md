# OA-CFI
1.安装所需工具
(1)cmake 版本3.5以上
a、下载cmake-3.5.1.tar.gz：https://cmake.org/download/
b、进入cmake-3.5.1  进入命令 cd cmake-3.5.1 
c、安装gcc-c++：sudo apt-get install  build-essential（或者直接执行这两条命令sudo apt-get install gcc,sudo apt-get install g++）
d、执行 sudo ./bootstrap
e、执行sudo make 
f、执行 sudo make install

(2)radare2版本4.0.0
a. git clone https://github.com/radareorg/radare2.git
b. 执行sys/install.sh
c.执行pip install r2pipe

(3)执行sudo apt install cmake g++ gcc python bash git python-pip

(4)安装gold-plugin
a.sudo apt-get install linux-headers-$(uname -r) csh gawk automake libtool bison flex libncurses5-dev
b.# Check 'makeinfo -v'. If 'makeinfo' does not exist
sudo apt-get install apt-file texinfo texi2html
sudo apt-file update
sudo apt-file search makeinfo

(5)下载binutils
cd ~
git clone --depth 1 git://sourceware.org/git/binutils-gdb.git binutils
mkdir build
cd build
../binutils/configure --enable-gold --enable-plugins --disable-werror
make

2.搭建编译器
cd $OACFI_PATH/
mkdir build
cd build
cmake -DLLVM_BINUTILS_INCDIR="path_to_binutils/include" -G "Unix Makefiles" -DLLVM_ENABLE_PROJECTS=“clang;libcxx;libcxxabi;lld;compiler-rt;clang-tools-extra;libunwind;polly;openmp" -DCMAKE_BUILD_TYPE=Realease ../llvm
make -j8

3.备份
cd ~
mkdir backup
cd /usr/bin/
cp ar ~/backup/
cp nm ~/backup/
cp ld ~/backup/
cp ranlib ~/backup/

4.替换执行文件
cd /usr/bin/
sudo cp ~/build/binutils/ar ./
sudo rm nm
sudo cp ~/build/binutils/nm-new ./nm
sudo cp ~/build/binutils/ranlib ./
sudo cp ~/build/gold/ld-new ./ld

5.安装llvm-gold
cd /usr/lib
sudo mkdir bfd-plugins
cd bfd-plugins
sudo cp $OACFI_PATH/build/lib/LLVMgold.so ./
sudo cp $OACFI_PATH/build/lib/libLTO.* ./

6.搭建 SVF-SUPA
a.配置环境变量
export LLVM_SRC=your_path_to_llvm-7.0.0.src
export LLVM_OBJ=your_path_to_llvm-7.0.0.obj
export LLVM_DIR=your_path_to_llvm-7.0.0.obj
export PATH=$LLVM_DIR/bin:$PATH

b.搭建SVF
cd $OACFI_PATH/svf-src
mkdir Debug-build
cd Debug-build
cmake -D CMAKE_BUILD_TYPE:STRING=Debug ../
make -j8
c.配置环境变量
export PATH=$OACFI_PATH/svf-src/Debug-build/bin:$PATH

二 运行测试
SPEC benchmark测试
1.解压spec2006镜像，到cpu2006目录下安装
./install.sh
. ./shrc

2.将test.cfg拷贝到config目录下，需要修改编译器和优化选项，执行runspec
rm -rf benchspec/CPU2006/*/exe/
runspec  --action=run --config=test.cfg --tune=base --size=test --iterations=1 --noreportable all

3.在任一benchmark的build目录下，修改Makefile和Makefile.spec文件
Makefile修改Makefile.defaults路径为具体路径
Makefile.spec：
#add oscfi.c, mpxrt.c, mpxrt-utils.c in the source list, keep others same
SOURCES=oscfi.c mpxrt.c mpxrt-utils.c …

4.执行OA-CFI目录下的run.sh



