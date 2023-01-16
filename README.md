# CFDEM Coupling Compilation - A How To

CFDEM is a coupling software that allows coupled CFD and DEM simulations. It uses OpenFOAM to model the fluid dynamics as well as LIGGGHTS to model the discrete elements. Tutorials and an installation guide can be found under www.cfdem.com. Unfortunately the official guide did not work on my system without some tweaking. Here, the necessary steps I used to get everything working are detailed. They are the combination of the manual (https://www.cfdem.com/media/CFDEM/docu/CFDEMcoupling_Manual.html) and about a weekend of googling. 

# Steps 

```
  cd $HOME
```

mkdir CFDEM
cd CFDEM
git clone https://github.com/CFDEMproject/CFDEMcoupling-PUBLIC.git

cd $HOME
mkdir LIGGGHTS
cd LIGGGHTS
git clone https://github.com/CFDEMproject/LIGGGHTS-PUBLIC.git
git clone https://github.com/CFDEMproject/LPP.git lpp

cd $HOME
mkdir OpenFOAM
cd OpenFOAM
git clone git://github.com/OpenFOAM/OpenFOAM-5.x.git
git clone git://github.com/OpenFOAM/ThirdParty-5.x.git

gedit ~/.bashrc	
# Add the following line to the end of the file
source $HOME/OpenFOAM/OpenFOAM-5.x/etc/bashrc
# Save and exit
source ~/.bashrc
cd $WM_PROJECT_DIR
./Allwmake -j # This compiles using all available processors, just add a number after j if you do not want to annoy your admin if you are compiling this on an hpc

cd $HOME/CFDEM
mv CFDEMcoupling-PUBLIC CFDEMcoupling-PUBLIC-$WM_PROJECT_VERSION

gedit ~/.bashrc
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


source ~/.bashrc
cfdemSysTest

gedit ~/LIGGGHTS/LIGGGHTS-PUBLIC/src/MAKE/Makefile.user # Maybe the file is called Makefile.user_default -> this seems to refer to the same file
# Change the following lines
AUTOINSTALL_VTK = "OFF" # Assuming VTK has not been installed yet
#...
BUILD_LIBRARIES = "NONE"
# to
AUTOINSTALL_VTK = "ON" # Assuming VTK has not been installed yet
#...
BUILD_LIBRARIES = "ALL"
# Save and exit 

# Compile LIGGGHTS -> This will probably fail (you can test this by attempting to run
# lmp_auto in the LIGGGHTS/LIGGGHTs-PUBLIC src directory)
cfdemCompLIG # This command installs VTK as well

# If lmp_auto does not work, find the file where VTK was installed: Probably under
# ~/LIGGGHTS/LIGGGHTS-PUBLIC/lib/vtk/install/lib
# Again edit the bashrc
gedit ~/.bashrc
# Add the following line to the end
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/LIGGGHTS/LIGGGHTS-PUBLIC/lib/vtk/install/lib/
# Save and exit
source ~/.bashrc

# Compile LIGGGHTS again
cfdemCompLIG
# Run the reminaing commands from the installation tutorial:
cfdemCompCFDEMsrc
cfdemCompCFDEMsol # Watch out, a version of the installation tutorail has a typo here 
cfdenCompCFDEMuti

# If you get no error, then the compilation is complete
