BootStrap:docker
From: sachet/polysolver:v3

%post

     mkdir /data/
     mv /home/polysolver /usr/local/libexec/polysolver
     mv /home/samtools /usr/local/libexec/samtools
     export PSHOME=/usr/local/libexec/polysolver
     export SAMTOOLS_DIR=/usr/local/libexec/samtools
     export JAVA_DIR=/usr/bin
     export NOVOALIGN_DIR=$PSHOME/binaries

     # IMPORTANT: the below aren't set, I'm not sure where they are inside image
     export GATK_DIR=$PSHOME
     export MUTECT_DIR=$PSHOME
     export STRELKA_DIR=$PSHOME
     chmod 777 -R /usr/local/libexec
