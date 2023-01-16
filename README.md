# CFDEM Coupling Compilation - A How To
CFDEM is a coupling software that allows coupled CFD and DEM simulations. It uses OpenFOAM to model the fluid dynamics as well as LIGGGHTS to model the discrete elements. Tutorials and an installation guide can be found under www.cfdem.com. The CFDEM developers did a good job making the compilation as easy as possible. Unfortunately the official guide did not work on my system without some tweaking. Here, the necessary steps I used to get everything working are detailed. They are a combination of the manual (https://www.cfdem.com/media/CFDEM/docu/CFDEMcoupling_Manual.html) and about a weekend of googling. Hopefully this guide will spare some headaches!

## Steps for compilation
For the compilation to work on a Debian based version of linux, just open a terminal and type the following, highlighted commands.

First, move into the home directory
```
cd $HOME
```

Here, we create a folder for the CFDEM files and change into that directory	. Once this is done, clone the CFDEM git repository.

```
mkdir CFDEM
cd CFDEM
git clone https://github.com/CFDEMproject/CFDEMcoupling-PUBLIC.git
```

Change back to your home directory and make a repository for the DEM solver, LIGGGHTS. We change into that folder and then download the necessary LIGGGHTS and post processing tools.

```
cd $HOME
mkdir LIGGGHTS
cd LIGGGHTS
git clone https://github.com/CFDEMproject/LIGGGHTS-PUBLIC.git
git clone https://github.com/CFDEMproject/LPP.git lpp
```

Move back to the home directory to download the CFD solver, OpenFOAM as well as the third party software.

```
cd $HOME
mkdir OpenFOAM
cd OpenFOAM
git clone git://github.com/OpenFOAM/OpenFOAM-5.x.git
git clone git://github.com/OpenFOAM/ThirdParty-5.x.git
```
The bashrc file is hidden in your home folder and is executed every time a new terminal window is opened or when the source command is called on it. Here, we edit it to source an OpenFOAM file.
```
gedit ~/.bashrc	
# Add the following line to the end of the file
source $HOME/OpenFOAM/OpenFOAM-5.x/etc/bashrc
```
Save and exit gedit and then refresh your environment variables by typing
```
source ~/.bashrc
```
We are now ready to compile OpenFOAM. For this move into the OpenFOAM project directory and call the executable:
```
cd $WM_PROJECT_DIR
./Allwmake -j # This compiles using all available processors, just add a number after j if you do not want to annoy your admin if you are compiling this on an hpc
```
Move back into the CFDEM directory created earlier and change the folder name
```
cd $HOME/CFDEM
mv CFDEMcoupling-PUBLIC CFDEMcoupling-PUBLIC-$WM_PROJECT_VERSION
```
Once again open the bashrc in your home directory and add some lines to the end
```
gedit ~/.bashrc
# Add the following lines to the end of the file
#================================================#
#- source cfdem env vars
export CFDEM_VERSION=PUBLIC
export CFDEM_PROJECT_DIR=$HOME/CFDEM/CFDEMcoupling-$CFDEM_VERSION-$WM_PROJECT_VERSION
export CFDEM_PROJECT_USER_DIR=$HOME/CFDEM/$LOGNAME-$CFDEM_VERSION-$WM_PROJECT_VERSION
export CFDEM_bashrc=$CFDEM_PROJECT_DIR/src/lagrangian/cfdemParticle/etc/bashrc
export CFDEM_LIGGGHTS_SRC_DIR=$HOME/LIGGGHTS/LIGGGHTS-PUBLIC/src
export CFDEM_LIGGGHTS_MAKEFILE_NAME=auto
export CFDEM_LPP_DIR=$HOME/LIGGGHTS/lpp/src
. $CFDEM_bashrc
#================================================#
```
Renew your environment and test your system:
```
source ~/.bashrc
cfdemSysTest # If LIGGGHTS has not been compiled yet, this line might raise an error, just continue with the guide either way
```
For me, the LIGGGHTS compilation seemed to present the biggest trouble. I was able to overcome this by changing the Makefile.user/Makefile.user_default file to allow for autoinstallation of VTK and allowing all LIGGGHTS libraries to be built.
```
gedit ~/LIGGGHTS/LIGGGHTS-PUBLIC/src/MAKE/Makefile.user # Maybe the file is called Makefile.user_default -> this seems to refer to the same file
# Find the following two lines
	AUTOINSTALL_VTK = "OFF" # Assuming VTK has not been installed yet
	BUILD_LIBRARIES = "NONE"
# and change them to
	AUTOINSTALL_VTK = "ON" # Assuming VTK has not been installed yet
	BUILD_LIBRARIES = "ALL"
```
Save and exit gedit. We now call the CFDEM program that compiles LIGGGHTS for us.
```
# Compile LIGGGHTS -> This will probably fail (you can test this by attempting to run lmp_auto in the LIGGGHTS/LIGGGHTs-PUBLIC src directory after compilation)
cfdemCompLIG # This command compiles LIGGGHTS and installs VTK as well
```
If lmp_auto does not work, find the file where VTK was installed: Probably under ~/LIGGGHTS/LIGGGHTS-PUBLIC/lib/vtk/install/lib if VTK was installed as it was here. Again, we edit the bashrc file and add the VTK path:
```
gedit ~/.bashrc
# Add the following line to the end
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/LIGGGHTS/LIGGGHTS-PUBLIC/lib/vtk/install/lib/
```	
Save, exit and renew the nevironment
```
source ~/.bashrc
```
Compile LIGGGHTS again
```
cfdemCompLIG
```
Finally, we continue by running the reminaing commands from the installation tutorial, although watch out, a version that I used had a type 
```
cfdemCompCFDEMsrc
cfdemCompCFDEMsol # Watch out, a version of the installation tutorail has a typo here 
cfdenCompCFDEMuti
```
If you get no error, then the compilation is complete and should work as intended. 
