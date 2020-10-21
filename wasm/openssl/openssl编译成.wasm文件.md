### Openssl 编译成.wasm文件


#### 介绍

这是一个将openssl项目直接编译成openssl.wasm的文档，并且可以在本地直接run openssl.wasm，像使用docker一样让它运行，并且执行openssl的加解密命令。接下来一起尝试下吧。

- 工具安装
- 编译openssl
- 手写脚本测试加解密
- openssl aes加解密与openssl.wasm性能对比

#### 环境

- Python 3.7.3

- Pip3 19.3.1

- Openssl 官网： https://www.openssl.org/source

- 依赖工具 wasienv


	
	
#### 一、安装wasienv
	
	curl https://raw.githubusercontent.com/wasienv/wasienv/master/install.sh | sh

#### 问题1

	无法解析raw.githubusercontent.com

#### 解决办法

	把install.sh复制到本地执行


#### 问题2（巨坑）

	执行过程中 pip3 install wasienv --install-option="--install-scripts=$INSTALL_DIRECTORY/bin" --upgrade --user 报错，不支持这种写法

	WARNING: Skipping wasienv as it is not installed.
	/usr/local/lib/python3.7/site-packages/pip/_internal/commands/install.py:235: UserWarning: Disabling all use of wheels due to the use of --build-option / --global-option / --install-option.
	  cmdoptions.check_install_build_global(options)
	ERROR: Location-changing options found in --install-option: ['--install-scripts'] from command line. This is unsupported, use pip-level options like --user, --prefix, --root, and --target instead.

#### 解决办法

	是因为pip3更新到最新版本导致，不支持--install-option=“”显示传参，回退版本
	
	pip install --upgrade pip==19.3.1

#### 问题3

	执行过程 curl https://get.wasmer.io -sSfL | sh报错，无法解析域名

#### 解决办法

	浏览器访问，把内容复制出来，我保存的文件是install2.sh，修改install.sh的引用。


#### 问题4

	
	执行过程  INSTALL_DIRECTORY/bin/wasienv install-sdk unstable
	
	 File "/usr/local/lib/python3.7/site-packages/urllib3/connectionpool.py", line 659, in urlopen
    conn = self._get_conn(timeout=pool_timeout)
	 File "/usr/local/lib/python3.7/site-packages/urllib3/connectionpool.py", line 279, in _get_conn
    return conn or self._new_conn()
	 File "/usr/local/lib/python3.7/site-packages/urllib3/connectionpool.py", line 948, in _new_conn
	"Can't connect to HTTPS URL because the SSL module is not available."
	
#### 解决办法

**参考文档** 
	
	https://www.jianshu.com/p/f8585da77ed9
	
	https://stackoverflow.com/questions/45954528/pip-is-configured-with-locations-that-require-tls-ssl-however-the-ssl-module-in/59280089#59280089
	
	https://stackoverflow.com/questions/35280956/ignoring-ensurepip-failure-pip-7-1-2-requires-ssl-tls-python-3-x-os-x#35282183
	
	#设置brew源
	https://www.jianshu.com/p/b26c7bc14440
	
	
	
**分析**

	这是说python没有ssl模块，对于依赖的包，无法下载的问题。
	
	经过查阅资料，python ssl模块对于openssl的版本有要求，所以这里尝试升级openssl
	
	# 当前版本
	likai@likaideMacBook-Pro:~/Desktop/wasm$ openssl version
	LibreSSL 2.8.3
	
	# 安装其他版本openssl,第一步很慢
	brew update && brew upgrade
	brew uninstall --ignore-dependencies openssl; brew install https://github.com/tebelorg/Tump/releases/download/v1.0.0/openssl.rb

	
	
#### 问题4.1

安装openssl 错误

	Error: Calling Non-checksummed download of openssl formula file from an arbitrary URL is disabled! Use 'brew extract' or 'brew create' and 'brew tap-new' to create a formula file in a tap on GitHub instead.
	If reporting this issue please do so at (not Homebrew/brew or Homebrew/core):
	https://github.com/tebelorg/Tump/issues/new

#### 解决办法

**参考文档**
	
	https://github.com/kelaberetiv/TagUI/issues/635

**执行**

	brew uninstall openssl
	brew tap-new $USER/old-openssl
	brew extract --version=1.0.2t openssl $USER/old-openssl
	brew install openssl@1.0.2t

### 二、openssl 生成 .wasm文件

- 下载相关文件
	
	git clone https://github.com/wapm-packages/OpenSSL.git

- 参考脚本

	build.sh

```
#!/usr/bin/env sh

# Based on code from https://github.com/TrueBitFoundation/wasm-ports/blob/master/openssl.sh

OPENSSL_VERSION=1.1.0l
PREFIX=`pwd`

DIRECTORY="openssl-${OPENSSL_VERSION}"

if [ ! -d "$DIRECTORY" ]; then
  echo "Download source code"
  wget https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz
  tar xf openssl-${OPENSSL_VERSION}.tar.gz
fi

cd openssl-${OPENSSL_VERSION}

echo "Configure"
make clean
wasiconfigure ./Configure linux-x32 -no-asm -static -no-sock -no-afalgeng -DOPENSSL_SYS_NETWARE -DSIG_DFL=0 -DSIG_IGN=0 -DHAVE_FORK=0 -DOPENSSL_NO_AFALGENG=1 -DOPENSSL_NO_SPEED=1 || exit $?

cp ../progs.h apps/progs.h

sed -i 's|^CROSS_COMPILE.*$|CROSS_COMPILE=|g' Makefile

echo "Build"
wasimake make -j12 build_generated libssl.a libcrypto.a apps/openssl

rm -rf ${PREFIX}/include
mkdir -p ${PREFIX}/include
cp -R include/openssl ${PREFIX}/include

cp -R apps/openssl.wasm ../

# echo "Generate libraries .wasm files"
# wasicc libcrypto.a -o ${PREFIX}/libcrypto.wasm
# wasicc libssl.a -o ${PREFIX}/libssl.wasm

# echo "Link"
# wasicc apps/*.o libssl.a libcrypto.a \
#   -o ${PREFIX}/openssl.wasm

# chmod +x ${PREFIX}/openssl.wasm || exit $?

echo "Done"

```



### 三、版本问题

**openssl openssl-1.1.0l 以下版本编译都正常，可以理解为1.1.0系列。**

### 四、验证openssl.wasm

	# 进入命令行
	wasmer run openssl.wasm
	
#### 错误信息

	OpenSSL>	enc -aes-128-ecb -in ./aa.txt -out ./bb-en.txt
	
	0:error:24064064:random number generator:RAND_bytes:PRNG not seeded:crypto/rand/md_rand.c:506:You need to read the OpenSSL FAQ, https://www.openssl.org/docs/faq.html
	error in enc
	OpenSSL> exit
	
#### 排错

**官方解释**

	https://www.openssl.org/docs/faq.html#USER1
	
	这里加上--nosalt参数，不使用随机数即可。
	
	
**测试脚本**

生成文件的命令,生成100M，300M，500M文件：

dd if=/dev/urandom of=file100 bs=1m count=100

dd if=/dev/urandom of=file300 bs=1m count=300

dd if=/dev/urandom of=file500 bs=1m count=500

```
#!/bin/bash

key="A665A45920422F9D417E4867EFDC4FB8"
iv="4632527467615238616971576a466653"
fileName="file500"
ecbEnFileName="aes/ecb-en-${fileName}"
ecbDeFileName="aes/ecb-de-${fileName}"
cbcEnFileName="aes/cbc-en-${fileName}"
cbcDeFileName="aes/cbc-de-${fileName}"
ctrEnFileName="aes/ctr-en-${fileName}"
ctrDeFileName="aes/ctr-de-${fileName}"

# 创建需要的文件，wasm文件运行，不会把数据写入磁盘，所以需要提前准备文件
touch ${ecbEnFileName}
touch ${ecbDeFileName}
touch ${cbcEnFileName}
touch ${cbcDeFileName}
touch ${ctrEnFileName}
touch ${ctrDeFileName}


echo "------start openssl-aes-128 encryption-------"
echo "------ECB Encryption-------"
wasmer run openssl.wasm --dir=. -- enc -aes-128-ecb -in ${fileName} -out ${ecbEnFileName} --nosalt -K ${key}
echo "------CBC Encryption-------"
wasmer run openssl.wasm --dir=. -- enc -aes-128-cbc -in ${fileName} -out ${cbcEnFileName} --nosalt -K ${key} -iv ${iv}
echo "------CTR Encryption-------"
wasmer run openssl.wasm --dir=. -- enc -aes-128-ctr -in ${fileName} -out ${ctrEnFileName} --nosalt -K ${key} -iv ${iv}

echo "#############################################################"
echo "------start openssl-aes-128 Decrypt-------"
echo "------ECB Decrypt-------"
wasmer run openssl.wasm --dir=. -- enc -aes-128-ecb -d -in ${ecbEnFileName} -out ${ecbDeFileName} --nosalt -K ${key}
echo "------CBC Decrypt-------"
wasmer run openssl.wasm --dir=. -- enc -aes-128-cbc -d -in ${cbcEnFileName} -out ${cbcDeFileName} --nosalt -K ${key} -iv ${iv}
echo "------CTR Decrypt-------"
wasmer run openssl.wasm --dir=. -- enc -aes-128-ctr -d -in ${ctrEnFileName} -out ${ctrDeFileName} --nosalt -K ${key} -iv ${iv}

echo "#############################################################"
echo "-------Gen RSA private.key 1024----------"
wasmer run openssl.wasm --dir=. -- genrsa -out rsa_private_key.pem 1024

echo "-------Gen RSA public.key 1024"
touch ./rsa_public_key_1024.pem
wasmer run openssl.wasm --dir=. -- rsa -in rsa_private_key.pem -pubout -out rsa_public_key_1024.pem

echo "-------RSA EN----------"
touch ./hello.en.txt
wasmer run openssl.wasm --dir=. --  rsautl -encrypt -in hello.txt -inkey rsa_public_key_1024.pem -pubin -out hello.en.txt

echo "-------RSA DE----------"
touch ./hello.de.txt
wasmer run openssl.wasm --dir=. --  rsautl -decrypt -in hello.en.txt -inkey rsa_private_key.pem -out hello.de.txt

echo "Done"

```

	
### 五、openssl和openssl.wasm性能对比

#### openssl 

**加密**

| 模式/文件大小    | 100M  |   300M  |  500M | key数据大小 |
|  ----  | ----  |   ----  | ----- |----  | 
| ECB  | 0.197s |   0.970s | 1.418s | 128bit  |
| CBC  | 0.244s |   1.421s | 1.278s | 128bit  |
| CTR  | 0.203s |   0.587s | 1.145s | 128bit  |

**解密**

|  模式/文件大小   | 100M  |   300M  |  500M | key数据大小 |
|  ----  | ----  |   ----  | ----- |----  | ----|
| ECB  | 0.309s |  1.014s   | 1.275s |128bit  |
| CBC  | 0.260s |   0.946s  | 1.630s |128bit  |
| CTR  | 0.322s |   1.081s  | 1.591s |128bit  |

#### openssl.wasm
**加密**

| 模式/文件大小    | 100M  |   300M  |  500M | key数据大小 |
|  ----  | ----  |   ----  | ----- |----  | 
| ECB  | 1.775s |   5.326s | 8.996s | 128bit  |
| CBC  | 1.805s |   5.563s | 9.452s | 128bit  |
| CTR  | 2.011s |   6.133s | 10.350s | 128bit  |

**解密**

| 模式/文件大小    | 100M  |   300M  |  500M | key数据大小 |
|  ----  | ----  |   ----  | ----- |----  | 
| ECB  | 1.894s |   5.780s | 9.186s | 128bit  |
| CBC  | 1.880s |   5.892s | 10.123s | 128bit  |
| CTR  | 2.168s |   6.297s | 11.424s | 128bit  |



![](./openssl对比.png)


### 六、写在最后
	
**参考文档**
	
	https://github.com/wapm-packages/OpenSSL
	
	# WebAssembly工作原理
	https://cunzaizhuyi.github.io/webassembly/
	
	#.wast文件和.wasm文件的转换工具
	https://github.com/WebAssembly/wabt
	
	# js调用.wasm 错误分析
	https://cunzaizhuyi.github.io/WebAssembly-LinkError/

#### END