# Polar Workshop Workflow
How you choose to create a dataset of orthologous gene alignments for your work is up to you. If you have questions about this step, I suggest that you look into the following methods. 

A) Transdecoder: https://github.com/TransDecoder/TransDecoder/wiki

B) OrthoFinder: https://github.com/davidemms/OrthoFinder

C) PhyloTreePruner:http://sourceforge.net/projects/phylotreepruner/

Your final alignments should onlu have each of your species represented once in each alignment (no paralogs).

Once you have your alignment peptide files there will likely be a need to trim the names of species using some form of sed command and/or perl script that keeps everyting up to the first space on the Species ID line (e.g., see Trimtospace.pl). You do not want to overwrite your original files. One possible solution is listed below.

for file in *.fa; do sed 's///g' $file > "$(basename "$file" .tex)_1"; done

for file in *.fa_1; do perl Trimtospace.pl $file > "$(basename "$file" .tex)_2"; done

Keep in mind that you cannot trim the names too much, as you will need to Species_Name_contig_ID_gene_ID to gather the correct cds sequences for each alignment file.

There are numerous ways to accomplish this using various unix/linux or perl scripts, below I have included one example.

Your peptide alignment files will likely have species_gene_IDs similar to the following.

>Camptaug|Gene_17689__TR19742_0_g1_i1__g_17689__m_17689
peptide sequence
>Ccrass|Gene_62889__TR4866_0_g1_i1__g_62889__m_62889
peptide sequence

Make a header list for each alignment using grep.

for file in *.fa_pruned.fa_2; do grep '>' $file > "$(basename "$file" .tex)_headlist.txt"; done

Using this header list you can use the provided perl script (subset_fasta.pl) to gather the necessary cds sequences for each alignment file. You will need to combine all the individual cds files for earch species into one file using the cat commnand.

cat *.cds.fa > ALLspecies_cds.fa

for file in *.fa_pruned.fa_2_headlist.txt; do perl subset_fasta.pl -i $file Allspecies_cds.fasta > "$(basename "$file" .tex)_cds.fa"; done

Now you will have a directory of alignment.peptide.fa files with the corresponding cds.fa files with the same alignment basenames.

OG0009035.pep.fa_2 OG0009035._cds.fa   #Remember the headers for each species must be identical within each peptide alignment and cds file.

You can now make two lists 1) List for the peptide alignments and 2) List for the corresponding cds files.

ls *.pep.fa_2 > ALN_list.txt
ls *._cds.fa > CDS_list.txt

You will need to install PAL2NAL. It can be downloaded here http://www.bork.embl.de/pal2nal/. Make sure the prgram is accessible from your account path.

Use the perl script CDS_AA_Hash_for_PAL2NAL.pl with the command usage. Be sure to understand the usage of tthe screen command (https://thesystemadministrator.net/cpanel/how-to-install-and-use-screen-in-linux) so that long processes do not stop when you exit your server.

perl CDS_AA_Hash_for_PAL2NAL.pl CDS_list.txt ALN_list.txt

the perl script is set up to output the CDS alignment file for PAML = Phyllip format with no gaps nor inframe stop codons in the alignment. You should run this to produce files for PAML and also modifiy the command line for pal2nal below to output fasta files for HYPHY.

print "perl pal2nal.pl $a_files[$i] $c_files[$i] -output paml -nogap -codontable 1 > $c_files[$i]_align.phy\n";

system "perl pal2nal.pl $a_files[$i] $c_files[$i] -output paml -nogap -codontable 1 > $c_files[$i]_align.phy\n";

change paml to fasta

Now you have all the aligned cds files that you will need for PAML:CODEML and HYPHY.

1)  Running CODEML through PAML

PAML may be downloaded here: http://abacus.gene.ucl.ac.uk/software/paml.html

We will be using the Branch-Site Model within codeml that allows for 2 or more foreground branches to be tested for positive (episodic) selection.

You will need to create a control file *.ctl for running CODEML using the null hypothesis where omega=1 on the foreground branches and also the alternative hypothesis where omega is allowed to be > 1 and estimated from the data provdided using maximum likelihood.

You will need to create a list of the aligned cds files you plan to test as described above. You should also creat a list that repeats the corresponding species tree file for as many aligned cds files that you wish to test. The perl script below will allow you to either use the same species tree for all your tests are try out different species trees. 

You will also need to label the foreground branches (Polar Species) on your species tree (using a newick format) with #1, for example see the tree file below. Your tree should include branch lengths and be UNROOTED. Particular nodes may also be labeled in the foreground.

(((Bugner:0.159527,(Camptaug#1:0.077304,Ausvul#1:0.071029):0.030435):0.055321,(Cellarlat#1:0.116033,((WatersipFL:0.127171,Ccrass#1:0.070258):0.042494,SchizoFL:0.143287):0.020109):0.055403):0.0498,(Amvid:0.259907,(Horn#1:0.976799,(Alflab#1:0.28738,Alcyohauf:0.24686);

The perl script CODEML_TREE_ALN_NULL.pl will create the control file for running the null model.

The perl script CODEML_TREE_ALN_ALT.pl will create the control file for running the alternative model.

Again make a list of the CTL files that you wish to run, then use the perl script below.

The perl script CODEML_CTL_RUN .pl will run the control files for the null and alternative models. I suggest that you run all the ALT models first and all the NULL models second. You will need to modify the line below to include the complete path to your installation of CODEML.

system("/YOUR_PATH/codeml/paml4.9d/bin/codeml /YOUR_PATH/MCL_CTL_Files/$ctl_files[$i]\n");



















