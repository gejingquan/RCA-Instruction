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


