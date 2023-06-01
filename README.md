# MesaOnAmiga (MOA)

Hi, just in case you want follow-up on MesaOnAmiga (MOA) I will share my ideas and approach here just in case you may want to follow or even want to try/improve yourself my "naive" approach.

# Development environment 

My development environment: Debian 12, aarch64 (so yes, no need for a Solaris or #illumos based distro :-))  running within a VM (Fusion) on a MacBook Pro M2. Mesa3D 23.1.1 sources and Bebbo's 13.1.1 gcc toolchain.

# The road ahead

First thing I did,  compiling native mesa-23.1.1 on Linux (gcc 12.2.0-14) to make sure you have the whole shibang of libs, etc required by Mesa to build "natively" with all the bits and pieces on Linux (do not forget libelf and elf-tools depending on your distro).

git clone https://gitlab.freedesktop.org/mesa/mesa/-/tree/23.1

https://gitlab.freedesktop.org/mesa/mesa/-/tree/23.1




Once that worked I went to build amiga-gcc, 13.1.1 (ATTENTION THIS IS ABSOLUTELY WOKR IN PROGRESS)

Create destination directory:

sudo mkdir -p /opt/amiga

sudo chmod 775 /opt/amiga

sudo usermod -a -G users username

sudo chrgp users /opt/amiga


Let's build amiga-gcc:


git clone https://github.com/bebbo/amiga-gcc

cd amiga-gcc

make update-gcc

pushd projects/gcc

git fetch origin amiga13.1:amiga13.1 --depth=10

git checkout amiga13.1

popd

make clean

make all NDK=3.2

At least for me I was getting an error in libgcc so I patched it .. I just added #include <sys/types.h> to amiga-gcc/projects/gcc/libgcc/libgcov.h which fixed it.
Installation as kind of a fav in /opt/amiga

# Mesa3D cross-compilation build

ok, next step .. Mesa3D's meson build system .. go to your mesa src folder, create a nice build-amigaos folder and then start experimenting ..

Trying to build (ini files for cross-compilation in the repo) the barest minimum of Mesa3D with:

meson setup --cross-file m68k.ini --cross-file cross.ini --default-library=static -Dbuildtype=release -Db_ndebug=true -Dllvm=disabled -Dplatforms= -Dosmesa=true -Dglx=disabled -Dgallium-drivers=swrast -Dvulkan-drivers=[]

Failure #1: libatomic cannot be found (well wasn't build as part gcc though in the sources)

<img width="1680" alt="Bildschirmfoto 2023-06-01 um 10 30 38" src="https://github.com/isoriano1968/mesaonamiga/assets/81116077/8c97591f-d411-4583-8a5f-212c945be9d8">
<img width="1680" alt="Bildschirmfoto 2023-06-01 um 10 30 44" src="https://github.com/isoriano1968/mesaonamiga/assets/81116077/b38fbfce-3c6e-4d95-9432-bad1bda0c348">


... to be continued

# What would probably make sense?

Instead of having a "local" dev env using a layered (images) approach it would make more sense to wto have images with core os, amiga-gcc, mesa that we can use as a container to serve as consistent dev environment. 
