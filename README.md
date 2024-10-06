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

## Step 2 Run demo

We now can run the demo CVE-2018-10191 (mruby).


```
git clone https://github.com/mruby/mruby.git
cd mruby
git checkout e9ddb593f3f6c0264563eaf20f5de8cf43cc1c5d
CC=$AFL_DIR/afl-gcc CFLAGS="-fsanitize=address -fsanitize-recover=address -ggdb -O0" LDFLAGS="-fsanitize=address"  make -e -j
mv ./bin/mruby ../mruby_fuzz
make clean
CFLAGS="-ggdb -O0" make -e -j
mv ./bin/mruby ../mruby_trace
cd $EVAL_DIR
rm -rf mruby_fuzz mruby_trace
echo "@@" > arguments.txt
```
