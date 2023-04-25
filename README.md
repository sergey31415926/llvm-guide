# llvm-guide
How to build LLVM, make different passes

## Setup LLVM

- Download LLVM 

```
git clone --depth 1 https://github.com/llvm/llvm-project.git
```

- Build LLVM  
In this instruction ninja generator is used. See https://llvm.org/docs/GettingStarted.html for other options.  


```
cd llvm-project && mkdir build
```  

```
cmake -S llvm -B build -G Ninja \  
    -DLLVM_USE_LINKER=lld \  
    -DCMAKE_BUILD_TYPE=Release \  
    -DLLVM_BUILD_TESTS=true \  
    -DLLVM_BUILD_EXAMPLES=true \  
    -DLLVM_ENABLE_PROJECTS=clang
```

!!! From now on all commands executed from llvm-project/build

- Build Clang only - `ninja clang`

## AST Plugin

- Create Plugin

Add line "add_subdirectory(MyPlugin)" in llvm-project/clang/examples/CMakeLists.txt  
mkdir MyPlugin # with example content and register plugin at the end of .cpp

Check out PrintFunctionNames example in llvm-project/clang/examples

- Build Plugin - `ninja MyPlugin.so`

- Run Plugin 

```
/bin/clang++ -cc1 -load llvm-project/build/lib/MyPlugin.so -add-plugin my-registered-plugin /path/to/test_program.cpp
```

## LLVM IR Pass

- Make IR Representation from .cpp (avoid optimizations - "-disable-O0-optnone")

```
./bin/clang++ -Xclang -disable-O0-optnone -emit-llvm -S /some/file.cpp -o /out.ll
```

- Create Pass

As previously copy example content and change directory name and name mentioned in .cpp (Name == "..."). 
Bye example contains some legacy code (static cl:: ...) and legacy PM registration.

- Build Pass - `ninja PassName`

- Run Pass  

```
./bin/opt --load-pass-plugin=./lib/PassName.so -passes="RegisterName" /test.ll -S -o /out.ll
```

## Backend Pass 
- Make sure you've installed llc

Create your own backend pass (or spoil X86ReturnThunks.cpp like I did)

### Prepare your .cpp to link it in llc build   
- Make file in llvm-project/llvm/lib/Target/X86
- Add file in llvm-project/llvm/lib/Target/X86/CMakeLists.txt

### in X86.h

- declare function pass like 
    FunctionPass *createX86ReturnMyBack();
- declare init with registry like 
    void initializeX86ReturnMyBack(PassRegistry &);

### in X86TargetMachine.cpp

- initialize to registry like
    initializeX86ReturnMyBack(PR);
- use it in backend compilation like
    addPass(createX86ReturnMyPass());

### When Backend Pass is writen
- Check that it's executed during compilation:

```
llc -march=x86-64 test.ll -debug-pass=Structure | grep "Your pass desctiption"
```

```
/bin/llc -debug-only=pass_name /.ll -o /.ll
```
```
llc -march=x86-64 test.ll -stop-before=pass_name -o test.mir
```
```
llc -march=x86-64 test.ll -stop-after=pass_name -o test.mir
```
```
llc -march=x86-64 test.mir -run-pass=pass_name
```
