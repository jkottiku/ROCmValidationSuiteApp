# ROCmValidationSuiteApp
Simple Application for ROCmValidationSuite (RVS) project.

Application uses API interfaces provided by RVS to execute tests, benchmarks and qualifications from various modules. 

Refer [link](https://github.com/ROCm-Developer-Tools/ROCmValidationSuite) for details on RVS.

## Prerequisites 
Install ROCm stack and RVS before compilation of ROCmValidationSuiteApp.

Ubuntu :

    sudo apt-get -y update && sudo apt-get install -y libpci3 libpci-dev doxygen unzip cmake git libyaml-cpp-dev

CentOS :

    sudo yum install -y cmake3 doxygen pciutils-devel rpm rpm-build git gcc-c++ yaml-cpp-devel
 
RHEL :

    sudo yum install -y cmake3 doxygen rpm rpm-build git gcc-c++ yaml-cpp-devel
        
    wget http://mirror.centos.org/centos/7/os/x86_64/Packages/pciutils-devel-3.5.1-3.el7.x86_64.rpm
        
    sudo rpm -ivh pciutils-devel-3.5.1-3.el7.x86_64.rpm
		
SLES :

    sudo SUSEConnect -p sle-module-desktop-applications/15.1/x86_64
       
    sudo SUSEConnect --product sle-module-development-tools/15.1/x86_64
       
    sudo zypper  install -y cmake doxygen pciutils-devel libpci3 rpm git rpm-build gcc-c++ yaml-cpp-devel

## Install ROCm stack, rocblas and rocm-smi-lib
Install ROCm stack for Ubuntu/CentOS/SLES/RHEL, Refer https://github.com/RadeonOpenCompute/ROCm

_**Note:**_

rocm_smi64 package has been renamed to rocm-smi-lib64 from >= ROCm3.0. If you are using ROCm release < 3.0 , install the package as "rocm_smi64".
rocm-smi-lib64 package has been renamed to rocm-smi-lib from >= ROCm4.1.
 
Install rocBLAS and rocm-smi-lib : 

Ubuntu :

    sudo apt-get install rocblas rocm-smi-lib
   
CentOS & RHEL :

    sudo yum install --nogpgcheck rocblas rocm-smi-lib
   
SUSE :

    sudo zypper install rocblas rocm-smi-lib

_**Note:**_
If  rocm-smi-lib is already installed but "/opt/rocm/rocm_smi/ path doesn't exist. Do below:

Ubuntu :

    sudo dpkg -r rocm-smi-lib && sudo apt install rocm-smi-lib

CentOS & RHEL :

    sudo rpm -e  rocm-smi-lib && sudo yum install  rocm-smi-lib

SUSE :

    sudo rpm -e  rocm-smi-lib && sudo zypper install  rocm-smi-lib

## Building RVS from Source
This section explains how to get and compile current development stream of RVS.

### Clone repository

    git clone https://github.com/ROCm-Developer-Tools/ROCmValidationSuite.git

### Configure:

    cd ROCmValidationSuite
    cmake -B ./build -DROCM_PATH=<rocm_installed_path> -DCMAKE_INSTALL_PREFIX=<rocm_installed_path> -DCPACK_PACKAGING_INSTALL_PREFIX=<rocm_installed_path>

    e.g. If ROCm 5.5 was installed,
    cmake -B ./build -DROCM_PATH=/opt/rocm-5.5.0 -DCMAKE_INSTALL_PREFIX=/opt/rocm-5.5.0 -DCPACK_PACKAGING_INSTALL_PREFIX=/opt/rocm-5.5.0

### Build binary:

    make -C ./build

### Build package:

    cd ./build
    make package

**Note:**_ based on your OS, only DEB or RPM package will be built. You may
ignore an error for the unrelated configuration

### Install built package:

Ubuntu :

    sudo dpkg -i rocm-validation-suite*.deb

CentOS & RHEL & SUSE :

    sudo rpm -i --replacefiles --nodeps rocm-validation-suite*.rpm

## Building ROCmValidationSuiteApp from source
This section explains how to fetch and compile ROCmValidationSuiteApp source code.

### Clone repository

    git clone https://github.com/jkottiku/ROCmValidationSuiteApp.git

### Configure:

    cd ROCmValidationSuiteApp
    cmake -B ./build -DROCM_PATH=<rocm_installed_path> -DCMAKE_INSTALL_PREFIX=<rocm_installed_path> -DCPACK_PACKAGING_INSTALL_PREFIX=<rocm_installed_path>

    e.g. If ROCm 5.5 was installed,
    cmake -B ./build -DROCM_PATH=/opt/rocm-5.5.0 -DCMAKE_INSTALL_PREFIX=/opt/rocm-5.5.0 -DCPACK_PACKAGING_INSTALL_PREFIX=/opt/rocm-5.5.0

### Build binary:

    make -C ./build

## Running ROCmValidationSuiteApp

### Running version built from source code:

    cd ./build/bin
    ./rvs ; Run all the precomplied tests in ROCmValidationSuiteApp source
