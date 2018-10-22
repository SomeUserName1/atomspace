## Git Repo
### Git Submodules
Split into Language definition + Compiler/Interpreter Stack using LLVM and Atomese as IR  
and  
Database so save those IRs for compiled programs, extracted features for images/sound, other static knowledge  

# Compiler features  
## Static Code Analysis of C++ in LLVM:  
[Paper](https://pdfs.semanticscholar.org/55c8/a6fb28edea5640a3310d3c11b4f5d85c41c6.pdf)  
[Default checkers](https://clang-analyzer.llvm.org/available_checks.html#default_checkers)  

## Tutorials on clang code analyzer:
[1](https://wiki.documentfoundation.org/Development/Clang_Code_Analysis)  
[2](https://github.com/ingve/awesome-clang)  
[LLVM Tooling setup](http://clang.llvm.org/docs/HowToSetupToolingForLLVM.html)  

## Safety
### Data Flow Analysis
[Phasar](https://phasar.org/)  

### Low Level Bounded Model Checker
[Webpage KIT](http://llbmc.org/)

### KLEE: High-coverage test generation (theorem prover to evaluate all possible
paths)
__Sources:__
- [Webpage](http://klee.github.io/)
- [GitHub](https://github.com/klee/klee/tree/master)
- [Tutorial for coreutils](http://klee.github.io/tutorials/testing-coreutils/)


### SAFECode: Static code analysis: array bounds checking, memory object
validation, dangeling pointer detection
__Sources:__
- [Webpage](http://safecode.cs.illinois.edu/index.html)

### LLVM/Clang Flags: Adress, Memory, Thread, Leak sanitizers & data flow analysis 
__Sources:__
- [compiler-rt webpage](http://compiler-rt.llvm.org/)
e.g.
'''
 -Wthread-safety -fsanitize=address ASAN_OPTIONS=check_initialization_order=1  ASAN_OPTIONS=detect_leaks=1 -fsanitize=thread -fsanitize=memory fsanitize=undefined -fsanitize=cfi
'''

## Optimization
### better linker:
[LLD](http://lld.llvm.org/)
### BOLT: call analysis
- [BOLT](https://github.com/facebookincubator/BOLT)
### Polly: Cache locality and auto-parallelism 
__Sources:__
- [Webpage](http://polly.llvm.org/)

### OpenMP: Thread-Safe parallelization by compiler directives & OpenCL
__Sources:__
- [Webpage](http://openmp.llvm.org/)
- [Standard](https://www.openmp.org/wp-content/uploads/OpenMP3.1.pdf)
- [Webpage](http://libclc.llvm.org/)

### GPGPU/OpenACC
__Sources:__
- [Medusa src](https://github.com/JianlongZhong/Medusa)
- [GRute src](https://github.com/groute/groute)
- [OpenACC 2 LLVM IR Paper](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7550823)
- [Auto-Vectorization](http://cuda.dcc.ufmg.br/dawn/)



## CMake Generation & Deployment  
#### Kubernets?  
### Override CMake project compiler settings and set the binutils prefix to llvm
__Sources:__
- [StackOverflow](https://stackoverflow.com/questions/7031126/switching-between-gcc-and-clang-llvm-using-cmake)
- [FreeBSD Manual](https://www.freebsd.org/cgi/man.cgi?query=clang++&sektion=1&manpath=FreeBSD+9.0-RELEASE)

```
-DCMAKE_USER_MAKE_RULES_OVERRIDE=~/ClangOverrides.txt 
-D_CMAKE_TOOLCHAIN_PREFIX=llvm-
```

Use update-alternatives to enable system-wide
```
sudo update-alternatives --config c++
sudo update-alternatives --config cc
```

### Generate dependency graphs
__Sources:__
- [StackOverflow](https://stackoverflow.com/questions/42577241/cmake-graphviz-auto-generated)
- [CMake Manual](https://cmake.org/cmake/help/v3.0/module/CMakeGraphVizOptions.html)

CMakeLists.txt:
```
add_custom_target(graphviz ALL
                  "${CMAKE_COMMAND}" "--graphviz=foo" .
                  WORKING_DIRECTORY "${CMAKE_BINARY_DIR}")
```
                  
CLI:
```
-graphviz=basename
```


### Generate Compile Symbol database for Language Servers/LSP
__Sources:__
- [StackOverflow](https://stackoverflow.com/questions/20059670/how-to-use-cmake-export-compile-commands)
- [CMake Manual](https://cmake.org/cmake/help/v3.5/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html)

```
-DCMAKE_EXPORT_COMPILE_COMMANDS=1
```


### CMake & PyPi Packages_
__Sources:__
- [PyPi Packaging tutorial](https://packaging.python.org/tutorials/packaging-projects/)
- [setuptools docs](https://setuptools.readthedocs.io/en/latest/setuptools.html#developer-s-guide)

Prequisites:
```
setuptools wheel twine
```

1. check directory structure
2. write a propper setup.py
```
from setuptools import setup, find_packages
setup(
    name="HelloWorld",
    version="0.1",
    packages=find_packages(),
    scripts=['say_hello.py'],

    # Project uses reStructuredText, so ensure that the docutils get
    # installed or upgraded on the target machine
    install_requires=['docutils>=0.3'],

    package_data={
        # If any package contains *.txt or *.rst files, include them:
        '': ['*.txt', '*.rst'],
        # And include any *.msg files found in the 'hello' package, too:
        'hello': ['*.msg'],
    },

    # metadata to display on PyPI
    author="Me",
    author_email="me@example.com",
    description="This is an Example Package",
    license="PSF",
    keywords="hello world example examples",
    url="http://example.com/HelloWorld/",   # project home page, if any
    project_urls={
        "Bug Tracker": "https://bugs.example.com/HelloWorld/",
        "Documentation": "https://docs.example.com/HelloWorld/",
        "Source Code": "https://code.example.com/HelloWorld/",
    }

    # could also include long_description, download_url, classifiers, etc.
)
```
3. Add License and README
4. Generate distr archives
```
python3 setup.py sdist bdist_wheel
```
5. Upload to PyPi
``` 
twine upload --repository-url https://test.pypi.org/legacy/ dist/*
```


### Creating .deb package
__Sources:__
- [Debian manual](https://www.debian.org/doc/manuals/debmake-doc/ch08.en.html#cmake-multi)
- [CMake extention](https://github.com/IvanSafonov/cmake-deb-packaging)
- [CPack](https://gitlab.kitware.com/cmake/community/wikis/doc/cpack/Configuration)

