Author: Giulia Ripellino

Contact: giulia.ripellino@cern.ch

This folder will allow you to create your own madgraph sample for exotic Higgs decays.

TO DO: What final states do we want to generate?

First, set up the FCC analysis environment (necessary e.g to enable use of python 3.7 which is needed for MagGraph):
```
source /cvmfs/fcc.cern.ch/sw/latest/setup.sh
export PYTHONPATH=/cvmfs/sw.hsf.org/spackages/linux-centos7-broadwell/gcc-8.3.0/fccanalyses-0.2.0pre02-6kok72w65toi2vvgedijdoqnd4hgg2wu/python:$PYTHONPATH
export PATH=/cvmfs/sw.hsf.org/spackages/linux-centos7-broadwell/gcc-8.3.0/fccanalyses-0.2.0pre02-6kok72w65toi2vvgedijdoqnd4hgg2wu/python/bin:$PATH
```

Now download the latest version of madgraph (http://madgraph.phys.ucl.ac.be/). Here we're using MadGraph5 v3.4.0. 

Copy the Madgraph tarball to your local area on lxplus:

```
scp MG5_aMC_v3.4.0.tar username@lxplus.cern.ch:/path/to/your/dir
```

Then ssh to lxplus and unzip the tarball 
```
tar -xf MG5_aMC_v3.4.0.tar)
```

Then download the HAHM_MG5model_v1 model from http://insti.physics.sunysb.edu/~curtin/hahm_mg.html. Copy to lxplus and unzip with the same procedure.

Move the HAHM_variableMW_UFO and HAHM_variablesw_UFO into the Madgraph5 models directory: `MG5_aMC_v3_4_0/models/`.

Now change to the Madgraph directory and run Madgraph:
```
cd MG5_aMC_v3_4_0
./bin/mg5_aMC
```

Then enter (for Higgs to dark photons):
```
set auto_convert_model T
import model --modelname HAHM_variableMW_UFO
generate h > zp zp 
output PROC_HAHM_variableMW_UFO_DarkPhoton
```

or (for Higgs to scalars)
```
generate h > hs hs 
output PROC_HAHM_variableMW_UFO_DarkScalar
```

Now set parameters in `PROC_HAHM_variableMW_UFO_<Model>/Cards/param_card.dat`:

For scalar case: Set epsilon=1.000000e-09 and mZD=1.000000e+03

For dark photon case: kap = 1.000000e-09 and MHS=1.000000e+03 

Now run
```
./bin/mg5_aMC /absolute/path/to/PROC_HAHM_variableMW_UFO_<Model>/Cards/mg5_proc_card.dat
cd PROC_HAHM_variableMW_UFO_<Model>
./bin/generate_events
```
to create the LHE file.


The resulting events will be stored in `PROC_HAHM_variableMW_UFO_DarkScalar/Events/run_01/unweighted_events.lhe.gz` file.

Unzip it (`gunzip unweighted_events.lhe.gz`) and give the *absolute* path to DarkScalar_pythia.cmnd file to generate the delphes root file.



You also need to grab the latest official Delphes card and edm4hep tcl file:
```
#cd to one directory up from FCCeePhysicsPerformance/
git clone https://github.com/HEP-FCC/FCC-config.git
cd FCC-config/
git checkout spring2021
cd ../FCCeePhysicsPerformance/case-studies/BSM/LLP/ExoticHiggsDecays/SampleGeneration/
```

To create delphes root file you need to do the following on your command line:

```
source /cvmfs/fcc.cern.ch/sw/latest/setup.sh
DelphesPythia8_EDM4HEP ../../../../../../FCC-config/FCCee/Delphes/card_IDEA.tcl ../../../../../../FCC-config/FCCee/Delphes/edm4hep_IDEA.tcl MG5_aMC_v3_4_0/DarkScalar_pythia.cmnd DarkScalar.root
```

This will crash because of some Pythia version issue: https://answers.launchpad.net/mg5amcnlo/+question/263774

Therefore change the first line of header of your .lhe file

LesHouchesEvents version=“3.0” —> LesHouchesEvents version=“2.0”

Do not open the file in an editor, will take forever to open. Change it in emacs or wim or similar.

The resulting DarkScalar.root is your EDM sample!
