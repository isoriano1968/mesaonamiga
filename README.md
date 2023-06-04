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

Installation as kind of a fav in /opt/amiga

# Mesa3D cross-compilation build

ok, next step .. Mesa3D's meson build system .. go to your mesa src folder, create a nice build-amigaos folder and then start experimenting ..

Trying to build (ini files for cross-compilation in the repo) the barest minimum of Mesa3D with:

meson setup --cross-file m68k.ini --cross-file cross.ini --default-library=static -Dbuildtype=release -Db_ndebug=true -Dllvm=disabled -Dplatforms= -Dosmesa=true -Dglx=disabled -Dgallium-drivers=swrast -Dvulkan-drivers=[]

Failure #1: libatomic cannot be found (well wasn't build as part gcc though in the sources)

Fixed: https://github.com/bebbo/gcc/issues/203

Failure #2: libdl was missing

Fixed: https://github.com/BSzili/libdl-hunk

Failure #3: librt was missing, may be part of libc but also as a stub in librt ... looking for clock_gettime etc

Failure #4: libdrm missng, NOW THIS IS where the beef is .. dependencies on X11 libs .. welcome to hell!

... to be continued

# Mesa DRI/DRM annotations

- src/loader/loader.c (loader_get_driver_for_fd) is the one responsible for detecting the right driver (fd as input parameter is acquired previously by calling DRI2Connect() as part of the DRI bring up process. Then the actual driver file is loaded in glx/dri_common.c (driOpenDriver).

- LIBGL_DEBUG=verbose LIBGL_ALWAYS_SOFTWARE=1 glxgears (forcing software renderer)

# Mesa source repository structure

- In src/egl/ we have the implementation of the EGL standard. If you are working on EGL-specific features, tracking down an EGL-specific problem or you are simply curious about how EGL links into the GL implementation, this is the place you want to visit. This includes the EGL implementations for the X11, DRM and Wayland platforms. 
- In src/glx/ we have the OpenGL bits relating specifically to X11 platforms, known as GLX. So if you are working on the GLX layer, this is the place to go. Here there is all the stuff that takes care of interacting with the XServer, the client-side DRI implementation, etc. 
- src/glsl/ contains a critical aspect of Mesa: the GLSL compiler used by all Mesa drivers. It includes a GLSL parser, the definition of the Mesa IR, also referred to as GLSL IR, used to represent shader programs internally, the shader linker and various optimization passes that operate on the Mesa IR. The resulting Mesa IR produced by the GLSL compiler is then consumed by the various drivers which transform it into native GPU code that can be loaded and run in the hardware.
- src/mesa/main/ contains the core Mesa elements. This includes hardware-independent views of core objects like textures, buffers, vertex array objects, the OpenGL context, etc as well as basic infrastructure, like linked lists.
- src/mesa/drivers/ contains the actual classic drivers (not Gallium). DRI drivers in particular go into src/mesa/drivers/dri. The code here is, for the most part, very specific to the underlying hardware platforms.
- src/mesa/swrast*/ and src/mesa/tnl*/ provide software implementations for things like rasterization or vertex transforms. Used by some software drivers and also by some hardware drivers to implement certain features for which they don’t have hardware support or for which hardware support is not yet available in the driver. For example, the i965 driver implements operations on the accumulation and selection buffers in software via these modules.
- src/mesa/vbo/ is another important module. Across its various versions, OpenGL has specified many ways in which a program can tell OpenGL about its vertex data, from using functions of the glVertex*() family inside glBegin()/glEnd() blocks, to things like vertex arrays, vertex array objects, display lists, etc… The drivers, however, do not need to deal with all this, Mesa makes it so that they always receive their vertex data as collection of vertex arrays, significantly reducing complexity on the side of the driver implementator. This is the module that takes care of managing all this, so no matter what type of drawing you GL program is doing or how it specifies its vertex data, it will always go through this module before it reaches the driver.
- src/loader/ contains the Mesa driver loader, which provides the logic necessary to decide which Mesa driver is the right one to use for a specific hardware so that Mesa’s libGL.so can auto-select the right driver when loaded.
- src/gallium/ contains the Gallium3D framework implementation. If working on a classic driver, you don’t need to care about the contents of this at all. If you are working on Gallium drivers however, this is the place where you will find the various Gallium drivers in development (inside src/gallium/drivers/), like the various Gallium ATI/AMD drivers, Nouveau or the LLVM based software driver (llvmpipe) and the Gallium state trackers.

So with this in mind, one should have enough information to know where to start looking for something specific:
 - If are interested in how vertex data provided to OpenGL is manipulated and uploaded to the GPU, the vbo module is probably the right place to look.
 - If we are looking to work on a specific aspect of a concrete hardware driver, we should go to the corresponding directory in src/mesa/drivers/ if it is a classic driver, or src/gallium/drivers if it is a Gallium driver.
 - If we want to know about how Mesa, the framework, abstracts various OpenGL concepts like textures, vertex array objects, shader programs, etc. we should look into src/mesa/main/.
 - If we are interested in the platform specific support, be it EGL or GLX, we want to look into src/egl or src/glx.
 - If we are interested in the GLSL implementation, which involves anything from the compiler to the intermediary IR and the various optimization passes, we need to look into src/glsl/.

# What would probably make sense?

Instead of having a "local" dev env using a layered (images) approach it would make more sense to have images with core os, amiga-gcc, mesa that we can use as a container to serve as consistent dev environment. 
