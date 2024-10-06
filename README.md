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

Download Intel Pin source code and compile aurora_tracer.so dynamic library, then create related folders.

```
wget -c http://software.intel.com/sites/landingpage/pintool/downloads/pin-3.15-98253-gb56e429b1-gcc-linux.tar.gz
tar -xzf pin*.tar.gz
export PIN_ROOT="$(pwd)/pin-3.15-98253-gb56e429b1-gcc-linux"
mkdir -p "${PIN_ROOT}/source/tools/AuroraTracer"
cp -r ${AURORA_GIT_DIR}/tracing/* ${PIN_ROOT}/source/tools/AuroraTracer
cd ${PIN_ROOT}/source/tools/AuroraTracer
make obj-intel64/aurora_tracer.so
cd -
mkdir -p $EVAL_DIR/traces

```

## Step 2
