### MAC 安装 Emscripten

### MAC安装环境依赖

- Git

- CMake 

	- brew install cmake

- Python 2.7.x 默认安装过


### 一、编译Emscripten

```
git clone https://github.com/juj/emsdk.git

cd emsdk

# 编译源码
./emsdk install latest

# 激活sdk
./emsdk activate latest

#设置环境变量
source ./emsdk_env.sh
```

**问题**

./emsdk install latest 报错

```
likai@likaideMacBook-Pro:~/hisun/resource/emsdk$ ./emsdk install latest
Installing SDK 'sdk-releases-upstream-7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f-64bit'..
Installing tool 'node-12.18.1-64bit'..
Error: Downloading URL 'https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/node-v12.18.1-darwin-x64.tar.gz': <urlopen error unknown url type: https>
Warning: Possibly SSL/TLS issue. Update or install Python SSL root certificates (2048-bit or greater) supplied in Python folder or https://pypi.org/project/certifi/ and try again.
Installation failed!
```

**解决**

#### ./emsdk.py install latest

```
likai@likaideMacBook-Pro:~/hisun/resource/emsdk$ ./emsdk.py install latest
Installing SDK 'sdk-releases-upstream-7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f-64bit'..
Installing tool 'node-12.18.1-64bit'..
Downloading: /Users/likai/hisun/resource/emsdk/zips/node-v12.18.1-darwin-x64.tar.gz from https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/node-v12.18.1-darwin-x64.tar.gz, 20873670 Bytes
Unpacking '/Users/likai/hisun/resource/emsdk/zips/node-v12.18.1-darwin-x64.tar.gz' to '/Users/likai/hisun/resource/emsdk/node/12.18.1_64bit'
Done installing tool 'node-12.18.1-64bit'.
Installing tool 'python-3.7.4-2-64bit'..
Downloading: /Users/likai/hisun/resource/emsdk/zips/python-3.7.4-2-macos.tar.gz from https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/python-3.7.4-2-macos.tar.gz, 25365593 Bytes
Unpacking '/Users/likai/hisun/resource/emsdk/zips/python-3.7.4-2-macos.tar.gz' to '/Users/likai/hisun/resource/emsdk/python/3.7.4-2_64bit'
Done installing tool 'python-3.7.4-2-64bit'.
Installing tool 'releases-upstream-7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f-64bit'..
Downloading: /Users/likai/hisun/resource/emsdk/zips/7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f-wasm-binaries.tbz2 from https://storage.googleapis.com/webassembly/emscripten-releases-builds/mac/7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f/wasm-binaries.tbz2, 69799761 Bytes
Unpacking '/Users/likai/hisun/resource/emsdk/zips/7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f-wasm-binaries.tbz2' to '/Users/likai/hisun/resource/emsdk/upstream'
Done installing tool 'releases-upstream-7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f-64bit'.
Running post-install step: npm ci ...
Done running: npm ci
Done installing SDK 'sdk-releases-upstream-7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f-64bit'.

```

####  ./emsdk.py activate latest

```
likai@likaideMacBook-Pro:~/hisun/resource/emsdk$ ./emsdk.py activate latest
Setting the following tools as active:
   node-12.18.1-64bit
   python-3.7.4-2-64bit
   releases-upstream-7a7f38ca19da152d4cd6da4776921a0f1e3f3e3f-64bit

Next steps:
- To conveniently access emsdk tools from the command line,
  consider adding the following directories to your PATH:
    /Users/likai/hisun/resource/emsdk
    /Users/likai/hisun/resource/emsdk/node/12.18.1_64bit/bin
    /Users/likai/hisun/resource/emsdk/python/3.7.4-2_64bit/bin
    /Users/likai/hisun/resource/emsdk/upstream/emscripten
- This can be done for the current shell by running:
    source "/Users/likai/hisun/resource/emsdk/emsdk_env.sh"
- Configure emsdk in your bash profile by running:
    echo 'source "/Users/likai/hisun/resource/emsdk/emsdk_env.sh"' >> $HOME/.bash_profile

```

####  source ./emsdk_env.sh

```
likai@likaideMacBook-Pro:~/hisun/resource/emsdk$ source ./emsdk_env.sh
Adding directories to PATH:
PATH += /Users/likai/hisun/resource/emsdk
PATH += /Users/likai/hisun/resource/emsdk/upstream/emscripten
PATH += /Users/likai/hisun/resource/emsdk/node/12.18.1_64bit/bin
PATH += /Users/likai/hisun/resource/emsdk/python/3.7.4-2_64bit/bin

Setting environment variables:
EMSDK = /Users/likai/hisun/resource/emsdk
EM_CONFIG = /Users/likai/hisun/resource/emsdk/.emscripten
EM_CACHE = /Users/likai/hisun/resource/emsdk/upstream/emscripten/cache
EMSDK_NODE = /Users/likai/hisun/resource/emsdk/node/12.18.1_64bit/bin/node
EMSDK_PYTHON = /Users/likai/hisun/resource/emsdk/python/3.7.4-2_64bit/bin/python3

```

**参考**

https://blog.csdn.net/cnds123/article/details/106742371

### 二、验证

#### emcc -v不报错就成功了

```
likai@likaideMacBook-Pro:~/hisun/resource/emsdk$ emcc -v
emcc (Emscripten gcc/clang-like replacement + linker emulating GNU ld) 2.0.3
clang version 12.0.0 (/b/s/w/ir/cache/git/chromium.googlesource.com-external-github.com-llvm-llvm--project a39423084cbbeb59e81002e741190dccf08b5c82)
Target: x86_64-apple-darwin19.4.0
Thread model: posix
InstalledDir: /Users/likai/hisun/resource/emsdk/upstream/bin
shared:INFO: (Emscripten: Running sanity checks)

```
#### 获取帮助

#### emcc --help

### 三、示例一

#### 1. 当前目录创建名为demo的目录

```
mkdir demo
```
#### 2. 一个hello world代码

```
cd demo

vim hello_world.c 

```
内容

```
#include <stdio.h>

int main(int argc, char ** argv) {
  printf("Hello World\n");

}

```

#### 3. 编译命令

#### 1. emcc c_file -o out_file

```
emcc  	#是Emscripten编译器行命令
	
c_file	#test.c是我们的输入文件

-o		#可以指定输出文件

out_file #输出的文件名；若为-o test.wasm，则只生成test.wasm这个文件；若为-o test.js，则生成test.js文件 ，还有test.wasm；若为-o test.html，则生成test.html文件，还有test.js和test.wasm。
	
```

#### 2. 执行hello world的例子

```
emcc ./hello_world.c -s WASM=1 -o ./hello_world.html

```

```
likai@likaideMacBook-Pro:~/hisun/resource/emsdk/demo$ emcc ./hello_world.c -s WASM=1 -o ./hello_world.html
shared:INFO: (Emscripten: Running sanity checks)

likai@likaideMacBook-Pro:~/hisun/resource/emsdk/demo$ ls
hello_world.c    hello_world.html hello_world.js   hello_world.wasm

```

#### 3. 启动http服务命令

```
emrun --no_browser --port 8080 ./hello_world.html

```

长这个样子

```
likai@likaideMacBook-Pro:~/hisun/resource/emsdk/demo$ emrun --no_browser --port 8080 ./hello_world.html
Web server root directory: /Users/likai/hisun/resource/emsdk/demo
Now listening at http://0.0.0.0:8080/

```

#### 4. 访问http://localhost:8080/

结果就是浏览器的consul会出现hello world，就这。
 

### 四、示例二

#### 描述: 
	
- 写一个test.c文件，里面是加减乘除计算。
- 编译成.wasm文件
- 写一个html，调用.wasm文件
 
#### 1. test.c 代码如下：

```
char* toChar (char* str) {
  return str;

}

int add (int x, int y) {
  return x + y;

}

int square (int x) {
  return x * x;

}
 
```
#### 2. 编译成.wasm
 
```
emcc ./test.c -Os -s WASM=1 -s SIDE_MODULE=1 -o ./test.wasm

```

#### 3. 写一个test.html

```
<!DOCTYPE html>

<html lang="en">

<head>

  <meta charset="UTF-8">

  <title>Document</title>

</head>

<body>


<script>

  /**

   * @param {String} path wasm 文件路径

   * @param {Object} imports 传递到 wasm 代码中的变量

   */

  function loadWebAssembly (path, imports = {}) {
    return fetch(path) // 加载文件

            .then(response => response.arrayBuffer()) // 转成 ArrayBuffer

            .then(buffer => WebAssembly.compile(buffer))

            .then(module => {
              imports.env = imports.env || {}

 

              // 开辟内存空间

              imports.env.memoryBase = imports.env.memoryBase || 0

              if (!imports.env.memory) {
                imports.env.memory = new WebAssembly.Memory({ initial: 256 })

              }

 

              // 创建变量映射表

              imports.env.tableBase = imports.env.tableBase || 0

              if (!imports.env.table) {
                // 在 MVP 版本中 element 只能是 "anyfunc"

                imports.env.table = new WebAssembly.Table({ initial: 0, element: 'anyfunc' })

              }

 

              // 创建 WebAssembly 实例

              return new WebAssembly.Instance(module, imports)

            })

  }

  // 加载wasm文件

  loadWebAssembly('test.wasm')

          .then(instance => {
            //调用c里面的方法

            const toChar = instance.exports.toChar

            const add = instance.exports.add

            const square = instance.exports.square

           

            console.log('return:   ', toChar("12352324"))

            console.log('10 + 20 =', add(10, 20))

            console.log('3*3 =', square(3))

            console.log('(2 + 5)*2 =', square(add(2 + 5)))

          })

 

</script>

</body>

</html>
 
```
 
#### 4. 运行服务

```
emrun --no_browser --port 8080 ./test.html

```

#### 访问: http://localhost:8080/test.html 查看consul

```
return:    12352324
test.html:88 10 + 20 = 30
test.html:90 3*3 = 9
test.html:92 (2 + 5)*2 = 49

```

 
###END

