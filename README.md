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
cd $EVAL_DIR
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
cd $EVAL_DIR
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

## Step 2 Run demo CVE-2018-10191 (mruby)


Download mruby and rollback to the version where CVE-2018-10191 has not been fixed.
```
cd $EVAL_DIR
git clone https://github.com/mruby/mruby.git
cd mruby
git checkout e9ddb593f3f6c0264563eaf20f5de8cf43cc1c5d
```

Compile mruby twice, for RCA fuzz and RCA analysis respectively.
The first one is compiled with afl-gcc and adds the sanitizer compilation flag, which is used for RCA fuzz.
The second one is compiled with gcc and removes the sanitizer compilation flag, which is used for RCA analysis.
```
CC=$AFL_DIR/afl-gcc CFLAGS="-fsanitize=address -fsanitize-recover=address -ggdb -O0" LDFLAGS="-fsanitize=address"  make -e -j
mv ./bin/mruby ../mruby_fuzz
make clean
CFLAGS="-ggdb -O0" make -e -j
mv ./bin/mruby ../mruby_trace
```


Download and copy the seed folder and files, start the RCA fuzzing process, and collect crash and non-crash test cases.
```
cd $EVAL_DIR
git clone https://github.com/gejingquan/RCA_v1
cp -r RCA_v1/seed ./
timeout 43200 $AFL_DIR/afl-fuzz -C -d -m none -i $EVAL_DIR/seed -o $AFL_WORKDIR -- $EVAL_DIR/mruby_fuzz @@
cp $AFL_WORKDIR/queue/* $EVAL_DIR/inputs/crashes
cp $AFL_WORKDIR/non_crashes/* $EVAL_DIR/inputs/non_crashes
```

Trace all crashing and non-crashing inputs found by the fuzzer's crash exploration mode using Intel Pin.
```
mkdir -p $EVAL_DIR/traces
cd $AURORA_GIT_DIR/tracing/scripts
python3 tracing.py $EVAL_DIR/mruby_trace $EVAL_DIR/inputs $EVAL_DIR/traces
python3 addr_ranges.py --eval_dir $EVAL_DIR $EVAL_DIR/traces
```

Once tracing completed, you can determine predicates as follows (requires Rust Nightly):
```
cd $AURORA_GIT_DIR/root_cause_analysis
cargo build --release --bin monitor
cargo build --release --bin rca
cargo run --release --bin rca -- --eval-dir $EVAL_DIR --trace-dir $EVAL_DIR --monitor --rank-predicates
cargo run --release --bin addr2line -- --eval-dir $EVAL_DIR
```
The result of RCA is ranked_predicates_verbose.txt and ranked_predicates.txt


