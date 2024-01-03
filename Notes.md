Other Misc Notes:  

# For CRIU Testing

https://blog.openj9.org/2023/06/19/deploying-on-ocp-4-13-with-openj9-criu-support/  
[https://blog.openj9.org/2022/09/26/getting-started-with-openj9-criu-support/. <-- Scripts are outdated]  
  
The following assumes running as root. Clone the InstantOnStartupGuide repo  
  
`git clone https://github.com/ibmruntimes/InstantOnStartupGuide.git`  
`cd InstantOnStartupGuide`   
Run the script to build the image (note this script uses podman)  

`bash Scripts/unprivileged/buildUBI8RH9.sh`  
  
The above scripts will build all the required images  
  
Then run the below command for restore:  
  
`podman run --rm --cap-add=CHECKPOINT_RESTORE --cap-add=SETPCAP --security-opt seccomp=criuseccompprofile.json --name restore_run restorerun`  
  
  
# FOR BUILDING JDK ON UBI8/RHEL 

Look for the docker file: https://github.com/ibmruntimes/semeru-containers/blob/ibm/criu/ubi8/Dockerfile  
Following dependencies might be needed:  
  
`yum install zlib-devel`  
Normally you want to install the -devel packages of the required libs. Additionally you should have installed the "Development tools" package  

`yum groupinstall 'Development Tools`  
Search for packages with  
  
`yum search package-name`  

Or for binaries    

```
yum whatprovides wantedBinary
yum list | grep cups-libs
yum whatprovides autoconf automake
```
```
subscription-manager repos --list
subscription-manager repos --enable rhel-9-for-x86_64-supplementary-rpms
subscription-manager repos --enable codeready-builder-for-rhel-9-x86_64-rpms
yum groupinstall "Development Tools"
yum install ant
yum install autoconf
yum install ca-certificates
yum install cmake
yum install cpio
yum install curl
yum install file
yum install gdb
yum install git
yum install libffi-dev
yum install -y alsa-lib-devel
yum install -y cups-devel
yum install -y desktop-file-utils
yum install -y giflib-devel
yum install elfutils-devel.x86_64
yum install -y expat-devel
yum install -y fontconfig
yum install -y fontconfig-devel
yum install -y freetype 
yum install -y freetype-devel
yum install -y numactl-devel
yum install -y  openssl-devel
yum install -y libX11
yum install -y libXext
yum install -y libXrandr
yum install -y libXrender
yum install -y libXt
yum install -y libXtst
yum install -y libXtst-devel
yum install -y libX11-devel
yum install -y libXt-devel
yum install -y gtk2-devel
yum install -y lcms2-devel
yum install -y libjpeg-devel

yum install  -y make
yum install  -y nasm
yum install  -y openssh
yum install  -y openssh-server
yum install  -y openssh-clients
yum install  -y perl
yum install  -y pkg-config
yum install  -y coreutils
yum install  -y systemtap-sdt-devel
yum install  -y unzip
yum install  -y xorg-x11-server-Xvfb
yum install  -y zlib-devel
yum install  -y nss-devel
yum install  -y xorg-x11-proto-devel

yum install libstdc++-static

yum install g++-7 /
yum install gcc-7 /
yum install build-essential /
yum install ant-contrib /
```
