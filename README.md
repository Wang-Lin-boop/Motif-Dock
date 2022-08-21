# Motif-Dock
&ensp;&ensp;&ensp;&ensp;_Motif-Dock is a protein-protein interaction (PPI) search pipeline for proteins containing **specific substructure motif (3D)** or **sequence motif (2D)**. Motif-Dock can be used to **1)** mining potential PPI partners for specific receptors, such as E3, kinase, phosphatase. **2)** searching protein scaffolds fusing specific motifs such as antigenic epitopes._  


Introduction
----

&ensp;&ensp;&ensp;&ensp;A number of proteins can bind to the same receptor at the same site with similar binding patterns. Most of the time, these proteins belong to the same protein family and have a high degree of sequence similarity. However, these proteins can sometimes be scattered among a diverse family of proteins, maintaining sequence conservation for a short conserved sequence (_shown in Figure 1A_) or even no apparent sequence conservation (_shown in Figure 1B_). For one thing, the **short conserved sequence** between the disordered protein and the receptor is called **sequence motif**, such as the TRAF6 and its substrates. For another thing, some **small sub-structures**, called **structural motifs**, are no apparent sequence conservation and scattered among different protein families, such as CRBN and its substrate proteins with different 3D overall structures. The development of Motif-Dock was inspired by these PPIs that are difficult to search by sequence alignment methods(e.g. Blast).  

![**Figure 1. Kinds of PPI protein motifs.**](https://user-images.githubusercontent.com/58931275/174751397-d529dfaf-f970-43f2-a0fe-0f3d99c006f7.png)  
**Figure 1. Kinds of PPI protein motifs.**  

Installation
----
Motif-Dock needs to be installed on the Linux system. Although the webserver is available at [here](https://bailab.siais.shanghaitech.edu.cn/services/motif-dock), we highly recommend that users install the Motif-Dock system locally to use the more complete capabilities.  

## 1. Install dependent packages.  
&ensp;&ensp;a. _(Required)_ Intsall the GNU parallel, Julia and Python3.   
>_Parallel:_  
```
sudo apt install parallel
```
>_Python3 (by miniconda):_  
```
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
sh Miniconda3-latest-Linux-x86_64.sh
```
>_Julia:_   
```  
wget https://julialang-s3.julialang.org/bin/linux/x64/1.5/julia-1.5.3-linux-x86_64.tar.gz
tar zxvf julia-1.5.3-linux-x86_64.tar.gz  
cd julia-1.5.3/bin
echo "export PATH=${PWD}:\$PATH" >> ~/.bashrc
source ~/.bashrc
julia
] # in Julia REPL
add BioStructures # in Julia REPL
add PDBTools # in Julia REPL
exit()
```
&ensp;&ensp;b. _(Required)_ Intsall [MASTER](https://grigoryanlab.org/index.php?sec=download&soft=MASTER) packages and add to `PATH` environment variables.   

&ensp;&ensp;c. _(Optional)_ Intsall the [Rosetta](https://www.rosettacommons.org/docs/latest/build_documentation/Build-Documentation) packages and set up environment variables.  
```
echo "export rosetta_app=<path to rosetta apps>" >> ~/.bashrc
echo "export rosetta_db=<path to rosetta database>" >> ~/.bashrc
echo "export rosetta_version=<rosetta versions, mpi or static>" >> ~/.bashrc
```  
&ensp;&ensp;d. _(Optional)_ Intsall the [EMBOSS](http://emboss.open-bio.org/html/adm/ch01s01.html) packages and add to `PATH` environment variables.  

## 2. Download Motif-Dock and configure environment variables.   
```
git clone https://github.com/Wang-Lin-boop/Motif-Dock/
cd Motif-Dock/scripts/
echo " # Scripts of Motif-Dock ,added by Motif-Dock, refer to https://github.com/Wang-Lin-boop/Motif-Dock/
export PATH=${PWD}:\$PATH" >> ~/.bashrc
source ~/.bashrc
```

Database
----
**1. Download the library of prepared human protein structures and disorder sequeneces.**

&ensp;&ensp;First, download the 3DMotif-Dock structure library, which may take a bit longer.  
```
wget https://bailab.siais.shanghaitech.edu.cn/service/3DMotif-Dock-pds-Library.zip
unzip 3DMotif-Dock-pds-Library.zip
for pds in `ls 3DMotif-Dock-pds-Library`;do
  echo "${PWD}/3DMotif-Dock-pds-Library/${pds}" >> 3DMotif-Dock-pds-Library.list
done
echo " ## Database of 3DMotif-Dock, added by Motif-Dock, refer to https://github.com/Wang-Lin-boop/Motif-Dock/
export HumanProteinPDS=${PWD}/3DMotif-Dock-pds-Library.list" >> ~/.bashrc
```
&ensp;&ensp;Next, switch to the ``Motif-Dock/Database`` and execute the following command.
```
echo " ## Database of 2DMotif-Dock, added by Motif-Dock, refer to https://github.com/Wang-Lin-boop/Motif-Dock/
export HumanDisorderSequence50=${PWD}/Human-Disorder50-protein.fasta 
export HumanDisorderSequence70=${PWD}/Human-Disorder70-protein.fasta" >> ~/.bashrc
```
&ensp;&ensp;Finally, three environment variables will be added to your ``~/.bashrc`` file.

**2. Use the 3DMotif-Dock to generate your own structure libraries.**

&ensp;&ensp;First, you need to build your own library of single chain protein structures, which may be obtained from PDB (please refer to [GetPDB](https://github.com/Wang-Lin-boop/GetPDB)), AlphaFold DB (recommend to pre-precessed by [Post-AlphaFold](https://github.com/Wang-Lin-boop/AlphaFoldDB_Processing)) or some conformational ensemble generated by molecular simulation. Next, run the following command to generate the processed structure library.  
```
3DMotif-Dock -i <INPUT_PDB_LIB> -n 40 -1 
```
&ensp;&ensp;The program generates a file named INPUT_PDB_LIB-Index, which will be used in the next calculations as the database index file.  

**3. Use the scripts to generate your own disorder sequences libraries.**

&ensp;&ensp;Please refer to [AlphaFold Processing](https://github.com/Wang-Lin-boop/AlphaFoldDB_Processing).   

Usage of 2DMotif-Dock
----

&ensp;&ensp;Run `2DMotif-Dock -h` to show the help information of 2DMotif-Dock.  

&ensp;&ensp; 1. Search for disordered human protein sequence library using sequence pattern.   
```
2DMotif-Dock -i ${HumanDisorderSequence70} -M "..[PG]EE[TS]." -r <receptor.pdb> -c "B" 
```

&ensp;&ensp; 2. Search for disordered human protein sequence library using MSAprofit (need to EMBOSS).  

```
2DMotif-Dock -i ${HumanDisorderSequence70} -m <msa.fasta> -p 65 -r <receptor.pdb> -c "B" 
```

Usage of 3DMotif-Dock
----

&ensp;&ensp;Run `3DMotif-Dock -h` to show the help information of 3DMotif-Dock.

&ensp;&ensp; 1. Search for flexible structural motif.   
```
3DMotif-Dock -m Motif_1.pdb -r Receptor.pdb -l INPUT_PDB_LIB-Index -n 40 -d 1.0 -2
``` 
&ensp;&ensp; 2. Search for stable structural motif.   
```
3DMotif-Dock -m Motif_1.pdb -r Receptor.pdb -l INPUT_PDB_LIB-Index -n 40 -d 0.6 -2
```
&ensp;&ensp; 3. Perform motif searching and optimizing the searched structure (Required rosetta).   
```
3DMotif-Dock -m Motif_1.pdb -r Receptor.pdb -l INPUT_PDB_LIB-Index -n 40 -d 0.6
```
# Expected Output of 3DMotif-Dock:  
_1. INPUT-Motif-FINAL_interface_score_OUT.sc: The final socre file._  
_2. INPUT-Motif-Jd2-OUT: This dir contains the final complexes of your receptor structure with the structure which is in your INPUT lib as well as matched to your motif structure._  
_3. INPUT-Motif-Relax: The refined complexes for INPUT-Motif-Jd2-OUT._  
_4. INPUT-Motif-Jd2-?(Target_Chainname).fasta: The sequences of matched motif structures._  

Citation
----
Coming soon ...

Acknowledgements
----
_&ensp;&ensp;&ensp;&ensp;We sincerely appreciate the Chinese Rosetta Community, Yuan Liu and Wei-kun Wu, who provide technical communication for this study. We also sincerely thank Joe Greener, the developer of [BioStructures](https://github.com/BioJulia/BioStructures.jl), for his help to our Julia code. We are also grateful for the support from HPC Platform of ShanghaiTech University._
