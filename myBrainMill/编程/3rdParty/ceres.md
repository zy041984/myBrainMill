# 源代码 2.2.0
https://github.com/ceres-solver/ceres-solver/releases/tag/2.2.0
http://ceres-solver.org/installation.html#customizing-the-build
# 依赖
## eigen 3.4.0
https://gitlab.com/libeigen/eigen/-/releases
要用cmake生成以下，不用设置什么选项，因为后面eigen用的是源代码，这里只是要找到cmake文件

## BLAS and LAPACK
试一下intel的oneMKL版本2024.0

https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit-download.html?operatingsystem=window&distributions=offline

Intel® oneAPI Base Toolkit System Requirements里提到安装前需要删除vs2022的windowsPerformanceToolkit，
https://www.intel.com/content/www/us/en/developer/articles/system-requirements/intel-oneapi-base-toolkit-system-requirements.html
https://www.intel.com/content/www/us/en/docs/oneapi-base-toolkit/get-started-guide-windows/2024-0/run-a-sample-project-using-an-ide.html
1. 从vs2022中删除WindowsPerformanceToolkit
2. 安装IntelOneMKL
3. 添加一个用户变量`SETVARS_CONFIG`，值为一个空格
5. 下载sample代码，https://github.com/oneapi-src/oneAPI-samples/releases/tag/2024.0.0

# 编译
取消EIGENMETIS、GLFLAGS、SUITESPARSE、USECUDA、、
选择EIGENSPARSE、CUSTOMBLAS、MINILOG、
把选项`BLAS_mkl_intel_lp64_dll_library`定义为`C:\Program Files (x86)\Intel\oneAPI\mkl\2024.0\lib\mkl_intel_lp64_dll.lib`
intel说带dll后缀的lib都是动态链接用的
cmake_build_type选择release