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
```
Usage: 2DMotif-Dock [OPTION] <parameter>

Note: Make sure profit\prophecy in your $PATH!

MSAprofit or Pattern parameter:
  -i      Your Interested sequences library in a fasta. <>
           Warning: don't contains any ":" or " " in fasta title!
  -p    Percentage cutoff of MSAprofit. <70>
  -m    A MSA fasta file to define a sequence motif.
          NOTE: MSA Length must be equal to the motif residue number in complex strcuture!
  -M    Sequence pattern to define a seqence motif. such as "..[PG]EE[TS]."
          .     : arbitrary residue.
          [PG]  : arbitrary P or G
          E     : must E.
  -t    Input a Table to ddG calculation. Instead of -m and -i.
            Table Format:  <Motif-Seq>  <Title>  <other...>

Structure parameter:
  -r    A complex strcuture pdb file Corresponding to your motif and receptor!
  -c    The chainname of your sequence motif in Complex structure (-r)! <B>
          NOTE: Must be the second protein chain except HETATM!
          NOTE: Residue number must be equal to the MSA length!
          NOET: If any HETATM record included, all of them should in the last chain.
  -q    relax before mutate, default is false.

Rosetta parameter:
  -n    The Max number of CPU threads available for this job, default is 40.
  -a    Path to Rosetta app, defalut is $rosetta_app </public/home/wanglin3/software/rosetta3.13-mpi/main/source/bin>
  -b    Path to Rosetta db, defalut is $rosetta_db </public/home/wanglin3/software/rosetta3.13-mpi/main/database>
  -v    Rosetta version, static or mpi. <mpi>
```

Usage of 3DMotif-Dock
----

&ensp;&ensp;Run `3DMotif-Dock -h` to show the help information of 3DMotif-Dock.
```
Usage: 3DMotif-Dock [OPTION] <parameter>

Example:
1) PDS library Geneation:
3DMotif-Dock -i INPUT_PDB_LIB -n 40 -1
## This step will return a INPUT_PDB_LIB-Index file.

2) Motif Docking:
## for flexible motif.
3DMotif-Dock -m Motif_1.pdb -r Receptor.pdb -l INPUT_PDB_LIB-Index -n 40 -d 1.0 -3
## for stable motif.
3DMotif-Dock -m Motif_2.pdb -r Receptor.pdb -l INPUT_PDB_LIB-Index -n 40 -d 0.6 -3
## Normal Solution, it takes a lot of time.
3DMotif-Dock -m Motif_2.pdb -r Receptor.pdb -l INPUT_PDB_LIB-Index -n 40 -d 0.8

Note: Make sure julia\MAESTER\Parallel\Python3 in your $PATH!

Expected Output:
1. INPUT-Motif-FINAL_interface_score_OUT.sc: The final socre file.
2. INPUT-Motif-Jd2-OUT: This dir contains the final complexes of your receptor structure with the structure which is in your INPUT lib as well as matched to your motif structure.
3. INPUT-Motif-Relax: The refined complexes for INPUT-Motif-Jd2-OUT.
4. INPUT-Motif-Jd2-?(Target_Chainname).fasta: The sequences of matched motif structures.

Input parameter:
  -m    A pdb file to define a structure motif.
  -i    Your Interested PDB library.
  -r    A receptor strcuture Corresponding to your motif!
  -d    RMSD cutoff for matching.
  -c    Set a chain name of target, default is B.
  -k    Don't change the chain name of target proteins, but set receptor chain to "Z".
            NOTE: it is useful for multiple chain PDB library.
  -f    Set a clash_score_cutoff to the filter for ugly PPI complex, or set to "unlimited". <1.64>
            NOTE: 1.64, 2.44, 6.20. More little, more sensitive to clash.
  -q    How many top complexes will be relax. <500>
  -g    Motif to match with a gap in range. <"5-20">

  -n    The Max number of CPU threads available for this job, default is 40.
  -a    Path to Rosetta app, defalut is $rosetta_app
  -b    Path to Rosetta db, defalut is $rosetta_db
  -v    Rosetta version, static or mpi. <mpi>
  -l    Input a PDS index list from MAESTER.

Control Parameter:
-1 Turn off except PDS Library Generation.
-2 Turn off Rosetta_Jd2 and Rosetta_Interface_Score.
-3 Turn off Rosetta_Relax. It is recommended for the first time to use.
```

Citation
----
Coming soon ...

Acknowledgements
----
_&ensp;&ensp;&ensp;&ensp;We sincerely appreciate the Chinese Rosetta Community, Yuan Liu and Wei-kun Wu, who provide technical communication for this study. We also sincerely thank Joe Greener, the developer of [BioStructures](https://github.com/BioJulia/BioStructures.jl), for his help to our Julia code. We are also grateful for the support from HPC Platform of ShanghaiTech University._
