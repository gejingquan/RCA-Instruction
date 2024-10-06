# RCA-Instruction
How to install and run RCA.

## Step 1. Installation

Download AURORA and create related folders
```
git clone https://github.com/RUB-SysSec/aurora
AURORA_GIT_DIR="$(pwd)/aurora"
mkdir evaluation
cd evaluation
EVAL_DIR=`pwd`
AFL_DIR=$EVAL_DIR/afl-fuzz
AFL_WORKDIR=$EVAL_DIR/afl-workdir
mkdir -p $EVAL_DIR/inputs/crashes
mkdir -p $EVAL_DIR/inputs/non_crashes
```

## Step 2.
Download AFL code and patch AFL so that non-crash test cases can also be stored.

```
wget -c https://lcamtuf.coredump.cx/afl/releases/afl-latest.tgz
tar xvf afl-latest.tgz
mv afl-2.52b afl-fuzz
cd afl-fuzz
patch -p1 < ${AURORA_GIT_DIR}/crash_exploration/crash_exploration.patch
cd $EVAL_DIR/afl-fuzz
make -j
cd ..
```




