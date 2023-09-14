---
开始使用MLIR
---

MLIR教程相关！
[幻灯片](https://llvm.org/devmtg/2020-09/slides/MLIR_Tutorial.pdf) -
[视频](https://www.youtube.com/watch?v=Y4SvqTtOIDk) -
[入门教程](https://mlir.llvm.org/docs/Tutorials/Toy/)


LLVM相关请参考[LLVM Getting Started](https://llvm.org/docs/GettingStarted.html)

通常情况下，构建LLVM需要一些时间。以下是构建带有LLVM的MLIR的快速指南

以下编译和测试MLIR的流程需要您具备以下条件：
`git`, [`ninja`](https://ninja-build.org/),和C++工具链(可参考[LLVM requirements](https://llvm.org/docs/GettingStarted.html#requirements)).

作为初学者, 你可以尝试 [入门教程](/Tutorials/Toy/Ch-1.md), 该教程将帮助您构建一个Toy编译器。


### 在类Unix系统上进行编译测试:

```sh
git clone https://github.com/llvm/llvm-project.git
mkdir llvm-project/build
cd llvm-project/build
cmake -G Ninja ../llvm \
   -DLLVM_ENABLE_PROJECTS=mlir \
   -DLLVM_BUILD_EXAMPLES=ON \
   -DLLVM_TARGETS_TO_BUILD="X86;NVPTX;AMDGPU" \
   -DCMAKE_BUILD_TYPE=Release \
   -DLLVM_ENABLE_ASSERTIONS=ON
# 使用clang和lld可以加快构建速度，我们建议添加以下选项:
#  -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DLLVM_ENABLE_LLD=ON
# CCache可以大幅加快重建速度，可以尝试添加以下选项:
#  -DLLVM_CCACHE_BUILD=ON
# 可选项，使用ASAN/UBSAN可以在开发早期发现错误，可以通过以下选项启用:
# -DLLVM_USE_SANITIZER="Address;Undefined" 
# 可选项，还可以启用集成测试
# -DMLIR_INCLUDE_INTEGRATION_TESTS=ON
cmake --build . --target check-mlir
```

建议您在计算机上安装clang和lld（例如，在Ubuntu上运行sudo apt-get install clang lld），然后取消上面cmake命令中的最后一部分的注释。

---

### 在Windows系统上进行编译测试:
要在Windows上使用Visual Studio 2017进行编译和测试，可以按照以下步骤操作:

```bat
REM 在已设置Visual Studio环境的Shell中执行以下命令，例如
REM   $visual-studio-install\Auxiliary\Build\vcvarsall.bat" x64
REM invoked.
git clone https://github.com/llvm/llvm-project.git
mkdir llvm-project\build
cd llvm-project\build
cmake ..\llvm -G "Visual Studio 15 2017 Win64" -DLLVM_ENABLE_PROJECTS=mlir -DLLVM_BUILD_EXAMPLES=ON -DLLVM_TARGETS_TO_BUILD="host" -DCMAKE_BUILD_TYPE=Release -Thost=x64 -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON
cmake --build . --target tools/mlir/test/check-mlir
```