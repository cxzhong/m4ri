Currently libcpucycles supports the following cycle counters. Some
cycle counters are actually other forms of counters that libcpucycles
scales to imitate a cycle counter. There is
[separate documentation](selection.html)
for how libcpucycles makes a choice of cycle counter.
There is a `cpucycles-open` command that, when run as `root`, often
succeeds in enabling higher-quality cycle counters until reboot. See the
[security considerations](security.html) regarding enabling or disabling
counters and regarding overclocking (Turbo Boost, Turbo Core, etc.).

`amd64-perfpmcff`: Requires a 64-bit Intel platform; the Linux
`perf_event` interface; `/proc/sys/kernel/perf_event_paranoid` at most 2;
any existing files among `/sys/devices/{cpu,cpu_core,cpu_atom}/rdpmc` at
least 2. Accesses a cycle counter through RDPMC (using Intel-specific
counter `0x40000001`, which is why this ends up not running on AMD CPUs).
This counter runs at the clock frequency of the CPU core.

`amd64-perfpmc`: Requires a 64-bit AMD platform; the Linux `perf_event`
interface; `/proc/sys/kernel/perf_event_paranoid` at most 2; any
existing files among `/sys/devices/{cpu,cpu_core,cpu_atom}/rdpmc` at
least 2. Accesses a cycle counter through RDPMC (using the `perf_event`
index); actively refuses to run on Intel (since some kernels have a
`perf_event` bug that returns the wrong index on E-cores). This counter
runs at the clock frequency of the CPU core.

`amd64-pmcff`: Requires a 64-bit Intel platform and an OS that enables
user-level RDPMC access. Accesses a cycle counter through RDPMC (using
Intel-specific counter `0x40000001`, which is why this ends up not running
on AMD CPUs). This counter runs at the clock frequency of the CPU core.

`amd64-tsc`, `amd64-tscasm`: Requires a 64-bit Intel/AMD platform.
Requires RDTSC to be enabled, which it is by default. Uses RDTSC to
access the CPU's time-stamp counter. On current CPUs, this is an
off-core clock rather than a cycle counter, but it is typically a very
fast off-core clock, making it adequate for seeing cycle counts if
overclocking and underclocking are disabled. The difference between
`tsc` and `tscasm` is that `tsc` uses the compiler's `__rdtsc()` while
`tscasm` uses inline assembly.

`arm32-cortex`: Requires a 32-bit ARMv7-A platform. Uses
`mrc p15, 0, %0, c9, c13, 0` to read the cycle counter. Requires user
access to the cycle counter, which is not enabled by default but can (sometimes) be
enabled under Linux via
[a kernel module](https://github.com/thoughtpolice/enable_arm_pmu).
This counter is natively 32 bits, but libcpucycles watches how the
counter and `gettimeofday` increase to compute a 64-bit extension of the
counter.

`arm32-1176`: Requires a 32-bit ARM1176 platform. Uses
`mrc p15, 0, %0, c15, c12, 1` to read the cycle counter. Requires user
access to the cycle counter, which is not enabled by default but can (sometimes) be
enabled under Linux via
[a kernel module](https://bench.cr.yp.to/cpucycles/n810.html).
This counter is natively 32 bits, but libcpucycles watches how the
counter and `gettimeofday` increase to compute a 64-bit extension of the
counter.

`arm64-perfpmc`: Requires a 64-bit ARMv8-A platform; the Linux
`perf_event` interface; on current Linux kernels, 1 in
`/proc/sys/kernel/perf_user_access`; on older Linux kernels,
[a kernel module](https://github.com/rdolbeau/enable_arm_pmu)
to enable PMCCNTR access. Uses `mrs %0, PMCCNTR_EL0` to read the cycle
counter.

`arm64-pmc`: Requires a 64-bit ARMv8-A platform and an OS that enables
user-level PMCCNTR access, for example an older Linux kernel with
[a kernel module](https://github.com/rdolbeau/enable_arm_pmu).
Uses `mrs %0, PMCCNTR_EL0` to read the cycle counter.

`arm64-vct`: Requires a 64-bit ARMv8-A platform. Uses
`mrs %0, CNTVCT_EL0` to read a "virtual count" timer. This is an
off-core clock, typically running at 24MHz. Results are scaled by
libcpucycles.

`loong64-rdtime`: Requires a 64-bit LoongArch platform.
Uses `rdtime.d` to read the "constant frequency timer".
This is an off-core clock, typically running at 100MHz.

`mips64-cc`: Requires a 64-bit MIPS platform. (Maybe the same code would
also work as `mips32-cc`, but this has not been tested yet.) Uses RDHWR
to read the hardware cycle counter (hardware register 2 times a constant
scale factor in hardware register 3). This counter is natively 32 bits,
but libcpucycles watches how the counter and `gettimeofday` increase to
compute a 64-bit extension of the counter.

`ppc32-mftb`: Requires a 32-bit PowerPC platform. Uses `mftb` and
`mftbu` to read the "time base". This is an off-core clock, typically
running at 24MHz.

`ppc64-mftb`: Requires a 64-bit PowerPC platform. Uses `mftb` and
`mftbu` to read the "time base". This is an off-core clock, typically
running at 24MHz.

`riscv32-perfrdcycle`: Requires a 32-bit RISC-V platform; the Linux
`perf_event` interface; on current Linux kernels, 1 in
`/proc/sys/kernel/perf_user_access`. Uses `rdcycle` and `rdcycleh` to
read a cycle counter.

`riscv32-rdcycle`: Requires a 32-bit RISC-V platform and an OS that
enables user-level `rdcycle` access. Uses `rdcycle` and `rdcycleh` to
read a cycle counter.

`riscv64-perfrdcycle`: Requires a 64-bit RISC-V platform; the Linux
`perf_event` interface; on current Linux kernels, 1 in
`/proc/sys/kernel/perf_user_access`. Uses `rdcycle` to read a cycle
counter.

`riscv64-rdcycle`: Requires a 64-bit RISC-V platform and an OS that
enables user-level `rdcycle` access. Uses `rdcycle` to read a cycle
counter.

`s390x-stckf`: Requires a 64-bit z/Architecture platform. Uses `stckf`
to read the TOD clock, which is documented to run at 4096MHz. On the
z15, this looks like a doubling of an off-core 2048MHz clock. Results
are scaled by libcpucycles.

`sparc64-rdtick`: Requires a 64-bit SPARC platform. Uses `rd %tick`
to read a cycle counter.

`x86-tsc`, `x86-tscasm`: Same as `amd64-tsc` and `amd64-tscasm`, but
for 32-bit Intel/AMD platforms instead of 64-bit Intel/AMD platforms.

`default-gettimeofday`: Reasonably portable. Resolution is limited to 1
microsecond. Results are scaled by libcpucycles.

`default-mach`: Requires an OS with `mach_absolute_time()`. Typically
runs at 24MHz. Results are scaled by libcpucycles.

`default-monotonic`: Requires `CLOCK_MONOTONIC`. Reasonably portable,
although might fail on older systems where `default-gettimeofday` works.
Resolution is limited to 1 nanosecond. Can be almost as good as a cycle
counter, or orders of magnitude worse, depending on the OS and CPU.
Results are scaled by libcpucycles.

`default-perfevent`: Requires the Linux `perf_event` interface, and a
CPU where `perf_event` supports `PERF_COUNT_HW_CPU_CYCLES`. Similar
variations in quality to `default-monotonic`, without the 1-nanosecond
limitation.

`default-zero`: The horrifying last resort if nothing else works.

## Examples

These are examples of `cpucycles-info` output on various machines. The
machines named `cfarm*` are from the
[cfarm compile farm](https://portal.cfarm.net/).

A `median` line saying, e.g., `47 +47+28+0+2-5+0+2-5...` means that the
differences between adjacent cycle counts were 47+47, 47+28, 47+0, 47+2,
47−5, 47+0, 47+2, 47−5, etc., with median difference 47. The first few
differences are typically larger because of cache effects.
Current versions of libcpucycles use `iqm` (interquartile mean)
instead of `median`.

`berry0`,
Broadcom BCM2835:
```
cpucycles version 20240114
cpucycles tracesetup 0 arm32-cortex precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 arm32-1176 precision 22 scaling 1.000000 only32 1
cpucycles tracesetup 2 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-monotonic precision 1199 scaling 1.000000 only32 0
cpucycles tracesetup 5 default-gettimeofday precision 1200 scaling 1000.000000 only32 0
cpucycles tracesetup 6 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 1000000000
cpucycles implementation arm32-1176
cpucycles median 720 +942+124+1+0+2+0+0+0+0+0+0+0+0+0+0+0+0+0+0+1+2+0+0+2+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+222+300+1+0+0+2+0+0+0+0+0+0+0+0+0+0+0+0+0
cpucycles observed persecond 798307692...2045181819 with 1024 loops 12 microseconds
cpucycles observed persecond 915478260...1260523810 with 2048 loops 22 microseconds
cpucycles observed persecond 947809523...1106100000 with 4096 loops 41 microseconds
cpucycles observed persecond 966353658...1129037500 with 8192 loops 81 microseconds
cpucycles observed persecond 988490566...1030019109 with 16384 loops 158 microseconds
cpucycles observed persecond 995169327...1002034063 with 32768 loops 2379 microseconds
cpucycles observed persecond 996871019...1012568691 with 65536 loops 627 microseconds
cpucycles observed persecond 997832134...1004212170 with 131072 loops 1250 microseconds
cpucycles observed persecond 997740918...1000887780 with 262144 loops 5009 microseconds
cpucycles observed persecond 998528349...1001961164 with 524288 loops 5537 microseconds
cpucycles observed persecond 999202882...1001166794 with 1048576 loops 10547 microseconds
```

`berry2`,
Broadcom BCM2836:
```
cpucycles version 20251226
cpucycles tracesetup 0 arm32-cortex precision 310 scaling 1.000000 only32 1
cpucycles tracesetup 1 arm32-1176 precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 default-perfevent precision 254 scaling 1.000000 only32 0
cpucycles tracesetup 3 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-monotonic precision 814 scaling 0.900000 only32 0
cpucycles tracesetup 5 default-gettimeofday precision 1200 scaling 900.000000 only32 0
cpucycles tracesetup 6 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 900000000
cpucycles implementation default-perfevent
cpucycles iqm 154 +208+9+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+173+172+29+0-4+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0
cpucycles observed persecond 218360000...420565218 with 1024 loops 24 microseconds
cpucycles observed persecond 327781250...402566667 with 2048 loops 31 microseconds
cpucycles observed persecond 423204081...468085107 with 4096 loops 48 microseconds
cpucycles observed persecond 490369047...517231708 with 8192 loops 83 microseconds
cpucycles observed persecond 544033112...560208054 with 16384 loops 150 microseconds
cpucycles observed persecond 450227397...457187328 with 32768 loops 364 microseconds
cpucycles observed persecond 584518716...588967800 with 65536 loops 560 microseconds
cpucycles observed persecond 592214995...594508598 with 131072 loops 1106 microseconds
cpucycles observed persecond 595894545...597049591 with 262144 loops 2199 microseconds
cpucycles observed persecond 598008895...598558878 with 524288 loops 4383 microseconds
cpucycles observed persecond 595624105...595918873 with 1048576 loops 8802 microseconds
```

`pi3aplus`,
Broadcom BCM2837B0:
```
cpucycles version 20251226
cpucycles tracesetup 0 arm64-perfpmc precision 8 scaling 1.000000 only32 0
cpucycles tracesetup 1 arm64-pmc precision 0 scaling 1.000000 only32 0
cpucycles tracesetup 2 arm64-vct precision 173 scaling 73.000000 only32 0
cpucycles tracesetup 3 default-perfevent precision 191 scaling 1.000000 only32 0
cpucycles tracesetup 4 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 5 default-monotonic precision 372 scaling 1.400000 only32 0
cpucycles tracesetup 6 default-gettimeofday precision 1700 scaling 1400.000000 only32 0
cpucycles tracesetup 7 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 1400000000
cpucycles implementation arm64-perfpmc
cpucycles iqm 10 +209+13+3+8+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0
cpucycles observed persecond 1030250000...2148500000 with 2048 loops 3 microseconds
cpucycles observed persecond 1176285714...1705800000 with 4096 loops 6 microseconds
cpucycles observed persecond 1264076923...1519818182 with 8192 loops 12 microseconds
cpucycles observed persecond 1216777777...1324600000 with 16384 loops 26 microseconds
cpucycles observed persecond 1366187500...1431586957 with 32768 loops 47 microseconds
cpucycles observed persecond 1324939393...1355443299 with 65536 loops 98 microseconds
cpucycles observed persecond 1219818604...1232802817 with 131072 loops 214 microseconds
cpucycles observed persecond 1379857894...1387955027 with 262144 loops 379 microseconds
cpucycles observed persecond 1344465384...1348359898 with 524288 loops 779 microseconds
cpucycles observed persecond 1305948318...1307784913 with 1048576 loops 1605 microseconds
```

`pi5`,
Broadcom BCM2712,
Debian 12,
Linux kernel 6.12.25+rpt-rpi-2712,
running in power-saving mode
(`echo powersave > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor`):
```
cpucycles version 20251226
cpucycles tracesetup 0 arm64-perfpmc precision 4 scaling 1.000000 only32 0
cpucycles tracesetup 1 arm64-pmc precision 0 scaling 1.000000 only32 0
cpucycles tracesetup 2 arm64-vct precision 144 scaling 44.500000 only32 0
cpucycles tracesetup 3 default-perfevent precision 158 scaling 1.000000 only32 0
cpucycles tracesetup 4 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 5 default-monotonic precision 432 scaling 2.400000 only32 0
cpucycles tracesetup 6 default-gettimeofday precision 2700 scaling 2400.000000 only32 0
cpucycles tracesetup 7 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2400000000
cpucycles implementation arm64-perfpmc
cpucycles iqm 6 +0+1+4-4+1+2+1+2+0+3+0+3+0+3+0+3+0+3+1+4-1-1+5-4+7-3+8-4-1+4-3+6-3+10-4-4+6+0-4+7-3+8-4-4+7-3+8-4-4+7-3+8-4-2+5-3+12-4-4+7-3+8-4-4
cpucycles observed persecond 1176571428...1676600000 with 8192 loops 6 microseconds
cpucycles observed persecond 1369750000...1658400000 with 16384 loops 11 microseconds
cpucycles observed persecond 1426826086...1569238096 with 32768 loops 22 microseconds
cpucycles observed persecond 1457444444...1528418605 with 65536 loops 44 microseconds
cpucycles observed persecond 1473314606...1508655173 with 131072 loops 88 microseconds
cpucycles observed persecond 1489755681...1507614943 with 262144 loops 175 microseconds
cpucycles observed persecond 1498117142...1507094828 with 524288 loops 349 microseconds
cpucycles observed persecond 1498035714...1502511462 with 1048576 loops 699 microseconds
```

`bblack`,
TI Sitara XAM3359AZCZ100:
```
cpucycles version 20251226
cpucycles tracesetup 0 arm32-cortex precision 308 scaling 1.000000 only32 1
cpucycles tracesetup 1 arm32-1176 precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-monotonic precision 1424 scaling 1.000000 only32 0
cpucycles tracesetup 5 default-gettimeofday precision 1300 scaling 1000.000000 only32 0
cpucycles tracesetup 6 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 1000000000
cpucycles implementation arm32-cortex
cpucycles iqm 1251 +1355+63+24+3-1+9+9-1+9-1-1+9-1-1-1+1+9+7-1-1-1-1-1-1-1-1-1+9-1-1-1+9-1-1-1+9-1+1+7-1-1-1-1-1-1-1-1-1-1+9-1-1-1-1-1+9-1+9+9+9-1+9+1+9
cpucycles observed persecond 702100000...3153750000 with 1024 loops 9 microseconds
cpucycles observed persecond 785666666...1626615385 with 2048 loops 14 microseconds
cpucycles observed persecond 837222222...4387840000 with 4096 loops 26 microseconds
cpucycles observed persecond 923108695...1106590910 with 8192 loops 45 microseconds
cpucycles observed persecond 957068965...1055564706 with 16384 loops 86 microseconds
cpucycles observed persecond 981711864...1019555556 with 32768 loops 235 microseconds
cpucycles observed persecond 987004210...1010238901 with 65536 loops 474 microseconds
cpucycles observed persecond 993804582...1010810811 with 131072 loops 741 microseconds
cpucycles observed persecond 996266338...1006985926 with 262144 loops 1422 microseconds
cpucycles observed persecond 997860246...1002197464 with 524288 loops 2761 microseconds
cpucycles observed persecond 999151671...1001252031 with 1048576 loops 9480 microseconds
```

`titan0`,
Intel Xeon E3-1275 V3:
```
cpucycles version 20260105
cpucycles tracesetup 0 amd64-perfpmcff precision 37 scaling 1.000000 only32 0
cpucycles tracesetup 1 amd64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 amd64-pmcff precision 49 scaling 1.000000 only32 0
cpucycles tracesetup 3 amd64-tsc precision 144 scaling 1.000000 only32 0
cpucycles tracesetup 4 amd64-tscasm precision 154 scaling 1.000000 only32 0
cpucycles tracesetup 5 default-perfevent precision 228 scaling 1.000000 only32 0
cpucycles tracesetup 6 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 7 default-monotonic precision 416 scaling 3.500000 only32 0
cpucycles tracesetup 8 default-gettimeofday precision 3860 scaling 3500.000000 only32 0
cpucycles tracesetup 9 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3500000000
cpucycles implementation amd64-perfpmcff
cpucycles iqm 41 +21+20+20-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1+2-2+0-1
cpucycles observed persecond 2065750000...4226000000 with 8192 loops 3 microseconds
cpucycles observed persecond 2748833333...4175500000 with 16384 loops 5 microseconds
cpucycles observed persecond 3288500000...4134000000 with 32768 loops 9 microseconds
cpucycles observed persecond 3285250000...3660555556 with 65536 loops 19 microseconds
cpucycles observed persecond 3452394736...3649333334 with 131072 loops 37 microseconds
cpucycles observed persecond 3450723684...3546486487 with 262144 loops 75 microseconds
cpucycles observed persecond 3472900662...3520758390 with 524288 loops 150 microseconds
cpucycles observed persecond 3472519867...3496290000 with 1048576 loops 301 microseconds
```

`nucnuc`,
Intel Pentium N3700:
```
cpucycles version 20260105
cpucycles tracesetup 0 amd64-perfpmcff precision 18 scaling 1.000000 only32 0
cpucycles tracesetup 1 amd64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 amd64-pmcff precision 40 scaling 1.000000 only32 0
cpucycles tracesetup 3 amd64-tsc precision 140 scaling 1.000000 only32 0
cpucycles tracesetup 4 amd64-tscasm precision 150 scaling 1.000000 only32 0
cpucycles tracesetup 5 default-perfevent precision 517 scaling 1.000000 only32 0
cpucycles tracesetup 6 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 7 default-monotonic precision 470 scaling 1.600000 only32 0
cpucycles tracesetup 8 default-gettimeofday precision 1960 scaling 1600.000000 only32 0
cpucycles tracesetup 9 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 1600000000
cpucycles implementation amd64-perfpmcff
cpucycles iqm 58 +7+15+0+15+0+15+15+15+0+15+0+0+15+0+0+0+0+0+0+0+0+0+0+0+1+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0
cpucycles observed persecond 1059750000...2314000000 with 2048 loops 3 microseconds
cpucycles observed persecond 1192428571...1782800000 with 4096 loops 6 microseconds
cpucycles observed persecond 1502363636...1923222223 with 8192 loops 10 microseconds
cpucycles observed persecond 1495954545...1674450000 with 16384 loops 21 microseconds
cpucycles observed persecond 1563761904...1655775000 with 32768 loops 41 microseconds
cpucycles observed persecond 1580843373...1627061729 with 65536 loops 82 microseconds
cpucycles observed persecond 1589612121...1612828221 with 131072 loops 164 microseconds
cpucycles observed persecond 1594012158...1605529052 with 262144 loops 328 microseconds
cpucycles observed persecond 1593228826...1603244412 with 524288 loops 672 microseconds
cpucycles observed persecond 1598524390...1601417558 with 1048576 loops 1311 microseconds
```

`saber214`,
AMD FX-8350:
```
cpucycles version 20260105
cpucycles tracesetup 0 amd64-perfpmcff precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 amd64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 amd64-pmcff precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 amd64-tsc precision 171 scaling 1.000000 only32 0
cpucycles tracesetup 4 amd64-tscasm precision 180 scaling 1.000000 only32 0
cpucycles tracesetup 5 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 6 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 7 default-monotonic precision 492 scaling 4.013176 only32 0
cpucycles tracesetup 8 default-gettimeofday precision 4343 scaling 4013.176000 only32 0
cpucycles tracesetup 9 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 4013176000
cpucycles implementation amd64-tsc
cpucycles iqm 72 +44-2+0+0-3-2+0+0+0-1+0-1+1-1+0-1+0+0+0-1+0-1+1-1+0-1+0+0+0-1+0+0+0-1+0-1+1-1+0-1+0+0+0-1+0-1+1-1+0-1+0+0+0-1+0+0+0-1+0-1+1-1+0-1
cpucycles observed persecond 3017750000...6236000000 with 4096 loops 3 microseconds
cpucycles observed persecond 3442714285...4913400000 with 8192 loops 6 microseconds
cpucycles observed persecond 3697846153...4411727273 with 16384 loops 12 microseconds
cpucycles observed persecond 3841000000...4194913044 with 32768 loops 24 microseconds
cpucycles observed persecond 3916836734...4092510639 with 65536 loops 48 microseconds
cpucycles observed persecond 3997562500...4088127660 with 131072 loops 95 microseconds
cpucycles observed persecond 3996822916...4041315790 with 262144 loops 191 microseconds
cpucycles observed persecond 4006892950...4029146982 with 524288 loops 382 microseconds
cpucycles observed persecond 4008403645...4019469974 with 1048576 loops 767 microseconds
```

`phoenix`,
AMD Ryzen 5 7640HS:
```
cpucycles version 20260105
cpucycles tracesetup 0 amd64-perfpmcff precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 amd64-perfpmc precision 33 scaling 1.000000 only32 0
cpucycles tracesetup 2 amd64-pmcff precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 amd64-tsc precision 111 scaling 1.000000 only32 0
cpucycles tracesetup 4 amd64-tscasm precision 121 scaling 1.000000 only32 0
cpucycles tracesetup 5 default-perfevent precision 274 scaling 1.000000 only32 0
cpucycles tracesetup 6 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 7 default-monotonic precision 426 scaling 4.301000 only32 0
cpucycles tracesetup 8 default-gettimeofday precision 4651 scaling 4301.000000 only32 0
cpucycles tracesetup 9 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 4301000000
cpucycles implementation amd64-perfpmc
cpucycles iqm 34 +36+0+0+0+0+0+0+472+4+0+0+0+0+0+0+0+0+0+0+448-1+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+35-1+0+0+0+0+0+0+0+0+0+0+0
cpucycles observed persecond 3289600000...5536000000 with 16384 loops 4 microseconds
cpucycles observed persecond 3648000000...4719857143 with 32768 loops 8 microseconds
cpucycles observed persecond 3858823529...4384266667 with 65536 loops 16 microseconds
cpucycles observed persecond 3973818181...4235483871 with 131072 loops 32 microseconds
cpucycles observed persecond 4097000000...4231806452 with 262144 loops 63 microseconds
cpucycles observed persecond 4161523809...4229967742 with 524288 loops 125 microseconds
cpucycles observed persecond 4144822134...4178501993 with 1048576 loops 252 microseconds
```

`cfarm14`,
Intel Xeon E5-2620 v3,
Debian 12,
Linux kernel 6.1.0-26-amd64:
```
cpucycles version 20260105
cpucycles tracesetup 0 amd64-perfpmcff precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 amd64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 amd64-pmcff precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 amd64-tsc precision 118 scaling 1.000000 only32 0
cpucycles tracesetup 4 amd64-tscasm precision 128 scaling 1.000000 only32 0
cpucycles tracesetup 5 default-perfevent precision 179 scaling 1.000000 only32 0
cpucycles tracesetup 6 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 7 default-monotonic precision 410 scaling 3.200000 only32 0
cpucycles tracesetup 8 default-gettimeofday precision 3540 scaling 3200.000000 only32 0
cpucycles tracesetup 9 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3200000000
cpucycles implementation amd64-tsc
cpucycles iqm 23 +28+22+4+10+4-5-5+4-5+4-5+4-5+4-5+4-5+4-5-5+4-5+4-5+4-5+4-5+4-5+4-5-5+4-5+4-5+4-5+4-5+4-5+4-5-5+4-5+4-5+4-5+4-5+4-5+4-5-5+4-5+4-5+4
cpucycles observed persecond 1902750000...3861000000 with 8192 loops 3 microseconds
cpucycles observed persecond 2175000000...3075600000 with 16384 loops 6 microseconds
cpucycles observed persecond 2170285714...2542500000 with 32768 loops 13 microseconds
cpucycles observed persecond 2248444444...2432760000 with 65536 loops 26 microseconds
cpucycles observed persecond 2333307692...2428860000 with 131072 loops 51 microseconds
cpucycles observed persecond 2362645161...2403049181 with 262144 loops 123 microseconds
cpucycles observed persecond 2384497536...2408805971 with 524288 loops 202 microseconds
cpucycles observed persecond 2390051851...2402225807 with 1048576 loops 404 microseconds
```

`cfarm26`,
Intel Core i5-4570 in 32-bit mode under KVM,
Debian 12.12,
Linux kernel 6.1.0-41-686-pae:
```
cpucycles version 20260105
cpucycles tracesetup 0 x86-tsc precision 118 scaling 1.000000 only32 0
cpucycles tracesetup 1 x86-tscasm precision 128 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-perfevent precision 627 scaling 1.000000 only32 0
cpucycles tracesetup 3 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-monotonic precision 2549 scaling 3.192606 only32 0
cpucycles tracesetup 5 default-gettimeofday precision 3532 scaling 3192.606000 only32 0
cpucycles tracesetup 6 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3192606000
cpucycles implementation x86-tsc
cpucycles iqm 22 +13+10+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1
cpucycles observed persecond 1840750000...5895500000 with 8192 loops 3 microseconds
cpucycles observed persecond 2488666666...4844500000 with 16384 loops 5 microseconds
cpucycles observed persecond 2680545454...3770888889 with 32768 loops 10 microseconds
cpucycles observed persecond 2797857142...3326315790 with 65536 loops 20 microseconds
cpucycles observed persecond 3112333333...3345913044 with 131072 loops 47 microseconds
cpucycles observed persecond 3146089887...3218136364 with 262144 loops 177 microseconds
cpucycles observed persecond 3170931972...3245579311 with 524288 loops 146 microseconds
cpucycles observed persecond 3170571428...3207561644 with 1048576 loops 293 microseconds
```

`cfarm27`,
Intel Core i5-4570 in 32-bit mode under KVM,
Alpine 3.19.9,
Linux kernel 6.6.117-0-lts:
```
cpucycles version 20260105
cpucycles tracesetup 0 x86-tsc precision 118 scaling 1.000000 only32 0
cpucycles tracesetup 1 x86-tscasm precision 128 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-perfevent precision 651 scaling 1.000000 only32 0
cpucycles tracesetup 3 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-monotonic precision 1189 scaling 3.192606 only32 0
cpucycles tracesetup 5 default-gettimeofday precision 3532 scaling 3192.606000 only32 0
cpucycles tracesetup 6 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3192606000
cpucycles implementation x86-tsc
cpucycles iqm 22 +123-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1-4+1+1
cpucycles observed persecond 1836250000...4507500000 with 8192 loops 3 microseconds
cpucycles observed persecond 2599833333...4344000000 with 16384 loops 5 microseconds
cpucycles observed persecond 2661272727...3445666667 with 32768 loops 10 microseconds
cpucycles observed persecond 2918250000...3340444445 with 65536 loops 19 microseconds
cpucycles observed persecond 3131859375...3266387097 with 131072 loops 63 microseconds
cpucycles observed persecond 3144478723...3232380435 with 262144 loops 93 microseconds
cpucycles observed persecond 3161572916...3206657895 with 524288 loops 191 microseconds
cpucycles observed persecond 3181324232...3209116839 with 1048576 loops 292 microseconds
```

`cfarm29`,
IBM POWER9,
Debian 13.1,
Linux kernel 6.12.57+deb13-powerpc64le:
```
cpucycles version 20260105
cpucycles tracesetup 0 ppc64-mftb precision 220 scaling 7.500000 only32 0
cpucycles tracesetup 1 default-perfevent precision 367 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 419 scaling 3.800000 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 4130 scaling 3800.000000 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3800000000
cpucycles implementation ppc64-mftb
cpucycles iqm 131 +26+27-4-3+4-4+4-3+4-4-3+11-3+4-4-3+4-4+4-3+4-4+4-3+4-4+4+4-3-4+4-3+4+11-3-11+4-4+4-3+4-4-3-4+12-4+4-3+4-11+4-4+4-3-4+12+4-11+4-4-3+4-4-3
cpucycles observed persecond 2535000000...5449000000 with 2048 loops 3 microseconds
cpucycles observed persecond 3042000000...5350000000 with 4096 loops 4 microseconds
cpucycles observed persecond 3244222222...4275000000 with 8192 loops 8 microseconds
cpucycles observed persecond 3587533333...4187846154 with 16384 loops 14 microseconds
cpucycles observed persecond 3714333333...3981290323 with 32768 loops 32 microseconds
cpucycles observed persecond 3693138888...3924264706 with 65536 loops 35 microseconds
cpucycles observed persecond 3795971428...3913897059 with 131072 loops 69 microseconds
cpucycles observed persecond 3821978417...3880839417 with 262144 loops 138 microseconds
cpucycles observed persecond 3821845323...3851086957 with 524288 loops 277 microseconds
cpucycles observed persecond 3835166064...3849797102 with 1048576 loops 553 microseconds
```

`cfarm45`,
AMD Athlon II X4 640,
Debian 8.11,
Linux kernel 3.16.0-11-686-pae:
```
cpucycles version 20230105
cpucycles tracesetup 0 x86-tsc precision 199 scaling 1.000000 only32 0
cpucycles tracesetup 1 x86-tscasm precision 199 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-perfevent precision 170 scaling 1.000000 only32 0
cpucycles tracesetup 3 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-monotonic precision 941 scaling 3.000000 only32 0
cpucycles tracesetup 5 default-gettimeofday precision 3200 scaling 3000.000000 only32 0
cpucycles tracesetup 6 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3000000000
cpucycles implementation default-perfevent
cpucycles median 72 +12+0+0+0+0+0+0+0+5+0+0+0+0+0+0+0+2+0+0+0+0+0+0+0+1+0+0+0+0+0+0+0+2+0+0+0+0+0+0+0+1+0+0+0+0+0+0+0+2+0+0+0+0+0+0+0+1+0+0+0+0+0+0
cpucycles observed persecond 541500000...1812000000 with 1024 loops 3 microseconds
cpucycles observed persecond 712333333...1212250000 with 2048 loops 5 microseconds
cpucycles observed persecond 1193285714...1733600000 with 4096 loops 6 microseconds
cpucycles observed persecond 1689176470...1804562500 with 8192 loops 33 microseconds
cpucycles observed persecond 1713074626...1770600000 with 16384 loops 66 microseconds
cpucycles observed persecond 1765107692...1795140625 with 32768 loops 129 microseconds
cpucycles observed persecond 1785369649...1800603922 with 65536 loops 256 microseconds
cpucycles observed persecond 1781377862...1796288462 with 131072 loops 261 microseconds
cpucycles observed persecond 1772647398...1778247827 with 262144 loops 691 microseconds
cpucycles observed persecond 1789670493...1794149598 with 524288 loops 870 microseconds
cpucycles observed persecond 1860276211...1861561332 with 1048576 loops 3156 microseconds
```

`cfarm91`,
StarFive JH7100,
Debian trixie/sid,
Linux kernel 5.18.11-starfive:
```
cpucycles version 20250925
cpucycles tracesetup 0 riscv64-rdcycle precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 default-perfevent precision 2702 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 1351 scaling 2.399988 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 2599 scaling 2399.987654 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2399987654
cpucycles implementation default-monotonic
cpucycles iqm 1476 +828-324+60+60+60+60-324+60-324+60+60-324+60+60-324+60+60-324+60-324+60+60-324+62-324+60+60-324+60+60-324+60-324+60+60-324+60+60-324+60-324+60+60-324+60+60-324+60-324+60+60+60+60+60-324+60-324+62+60+444+60-324+60+60
cpucycles observed persecond 1440250000...3968333334 with 1024 loops 7 microseconds
cpucycles observed persecond 1920181818...3072222223 with 2048 loops 10 microseconds
cpucycles observed persecond 2162473684...2733294118 with 4096 loops 18 microseconds
cpucycles observed persecond 2218777777...2530000000 with 8192 loops 35 microseconds
cpucycles observed persecond 2338014705...2490318182 with 16384 loops 67 microseconds
cpucycles observed persecond 2358559701...2440840910 with 32768 loops 133 microseconds
cpucycles observed persecond 2380905660...2419452472 with 65536 loops 264 microseconds
cpucycles observed persecond 2390815939...2410163810 with 131072 loops 526 microseconds
cpucycles observed persecond 2393901140...2403580953 with 262144 loops 1051 microseconds
cpucycles observed persecond 2397546190...2402577217 with 524288 loops 2099 microseconds
cpucycles observed persecond 2398864140...2401570967 with 1048576 loops 4327 microseconds
```

`cfarm92`,
SiFive Freedom U740,
Ubuntu 22.04.3,
Linux kernel 5.19.0-1021-generic:
```
cpucycles version 20240114
cpucycles tracesetup 0 riscv64-rdcycle precision 8 scaling 1.000000 only32 0
cpucycles tracesetup 1 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 2599 scaling 2.399988 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 2599 scaling 2399.987654 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2399987654
cpucycles implementation riscv64-rdcycle
cpucycles median 8 +168+20+2+2+0+0+0+0+570+0+0+0+0+0+0+0+144+0+0+0+0+0+0+0+160+0+0+0+0+0+0+0+160+0+0+0+0+0+0+0+154+0+0+0+0+0+0+0+154+0+0+0+0+0+0+0+152+0+0+0+0+0+0
cpucycles observed persecond 571500000...2198000000 with 1024 loops 3 microseconds
cpucycles observed persecond 833600000...2094000000 with 2048 loops 4 microseconds
cpucycles observed persecond 921888888...1445142858 with 4096 loops 8 microseconds
cpucycles observed persecond 1029625000...1320642858 with 8192 loops 15 microseconds
cpucycles observed persecond 1137034482...1284481482 with 16384 loops 28 microseconds
cpucycles observed persecond 1155701754...1227454546 with 32768 loops 56 microseconds
cpucycles observed persecond 1177464285...1217163637 with 65536 loops 111 microseconds
cpucycles observed persecond 1188018099...1207858448 with 131072 loops 220 microseconds
cpucycles observed persecond 1189925170...1200519363 with 262144 loops 440 microseconds
cpucycles observed persecond 1193962457...1199117446 with 524288 loops 878 microseconds
cpucycles observed persecond 1194051324...1196780111 with 1048576 loops 1811 microseconds
```

`cfarm94`,
SiFive StarFive JH7110,
Alpine 3.22.0_alpha20250108,
Linux kernel 6.12.19-2-lts:
```
cpucycles version 20260105
cpucycles tracesetup 0 riscv64-perfrdcycle precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 riscv64-rdcycle precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 default-perfevent precision 2076 scaling 1.000000 only32 0
cpucycles tracesetup 3 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-monotonic precision 1060 scaling 1.500000 only32 0
cpucycles tracesetup 5 default-gettimeofday precision 1820 scaling 1500.000000 only32 0
cpucycles tracesetup 6 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 1500000000
cpucycles implementation default-monotonic
cpucycles iqm 1125 +0-375+0+0+0-375+0+0+0+0-375+0+0+0-375+0+0+0+0-375+0+0+0-375+0+0+0+0-375+0+0+0-375+0+1+0+0-375+0+0+0-375+0+0+0+0-375+0+0+0-375+0+0+0+0-375+0+0+0+0-375+0+0+0
cpucycles observed persecond 750000000...3562500000 with 1024 loops 3 microseconds
cpucycles observed persecond 875333333...2344250000 with 2048 loops 5 microseconds
cpucycles observed persecond 1041666666...1928571429 with 4096 loops 8 microseconds
cpucycles observed persecond 1232142857...1781250000 with 8192 loops 13 microseconds
cpucycles observed persecond 1365000000...1646739131 with 16384 loops 24 microseconds
cpucycles observed persecond 1451108695...1610818182 with 32768 loops 45 microseconds
cpucycles observed persecond 1464494736...1540344087 with 65536 loops 94 microseconds
cpucycles observed persecond 1481876344...1520402174 with 131072 loops 185 microseconds
cpucycles observed persecond 1492559659...1512874286 with 262144 loops 351 microseconds
cpucycles observed persecond 1494745428...1505308886 with 524288 loops 710 microseconds
cpucycles observed persecond 1497092198...1502148438 with 1048576 loops 1409 microseconds
```

`cfarm95`,
SpacemiT X60,
Debian trixie/sid,
Linux kernel 6.6.36-cfarm #1:
```
cpucycles version 20260105
cpucycles tracesetup 0 riscv64-perfrdcycle precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 riscv64-rdcycle precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 default-perfevent precision 1882 scaling 1.000000 only32 0
cpucycles tracesetup 3 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-monotonic precision 375 scaling 1.600000 only32 0
cpucycles tracesetup 5 default-gettimeofday precision 1920 scaling 1600.000000 only32 0
cpucycles tracesetup 6 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 1600000000
cpucycles implementation default-monotonic
cpucycles iqm 133 +67+0-1+2-68+2+0-1+2+0+1+267+0+0+1+0+0+1+0+267+0+1+0+0+1+0-66+333+1+0+0-66+0+1+0+267+0+1+0+0+1+0+0+267+1+0+0+1+0+0+1+267+0+0+1+0-66+0+0+1+0+0+1+0
cpucycles observed persecond 1066750000...2366500000 with 2048 loops 3 microseconds
cpucycles observed persecond 1400000000...2416750000 with 4096 loops 5 microseconds
cpucycles observed persecond 1503000000...1896333334 with 8192 loops 10 microseconds
cpucycles observed persecond 1497000000...1673350000 with 16384 loops 21 microseconds
cpucycles observed persecond 1565095238...1656700000 with 32768 loops 41 microseconds
cpucycles observed persecond 1580746987...1626370371 with 65536 loops 82 microseconds
cpucycles observed persecond 1589527272...1612300614 with 131072 loops 164 microseconds
cpucycles observed persecond 1596176291...1607776759 with 262144 loops 328 microseconds
cpucycles observed persecond 1596570259...1603129959 with 524288 loops 732 microseconds
cpucycles observed persecond 1597691546...1600790237 with 1048576 loops 1312 microseconds
```

`cfarm103`,
Apple M1 (Icestorm-M1 + Firestorm-M1),
Debian trixie/sid,
Linux kernel 6.5.0-asahi-00780-g62806c2c6f29:
```
cpucycles version 20260105
cpucycles tracesetup 0 arm64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 arm64-pmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 arm64-vct precision 186 scaling 86.000000 only32 0
cpucycles tracesetup 3 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 5 default-monotonic precision 395 scaling 2.064000 only32 0
cpucycles tracesetup 6 default-gettimeofday precision 2384 scaling 2064.000000 only32 0
cpucycles tracesetup 7 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2064000000
cpucycles implementation arm64-vct
cpucycles iqm 0 +86+0+0+0+86+0+0+0+172+0+0+86+0+0+0+0+0+0+0+86+0+0+0+0+0+0+0+0+86+0+0+0+0+0+0+0+0+86+0+0+0+0+0+0+0+0+86+0+0+0+0+0+0+0+0+86+0+0+0+0+0+0+0+86
cpucycles observed persecond 1651200000...2809333334 with 8192 loops 4 microseconds
cpucycles observed persecond 1741500000...2107000000 with 16384 loops 11 microseconds
cpucycles observed persecond 1994380952...2213368422 with 32768 loops 20 microseconds
cpucycles observed persecond 2045121951...2154410257 with 65536 loops 40 microseconds
cpucycles observed persecond 2047875000...2116709678 with 131072 loops 63 microseconds
cpucycles observed persecond 2055030674...2082161491 with 262144 loops 162 microseconds
cpucycles observed persecond 2056917647...2073857708 with 524288 loops 254 microseconds
cpucycles observed persecond 2060674868...2070218695 with 1048576 loops 568 microseconds
```

`cfarm104`,
Apple M1 (Icestorm-M1 + Firestorm-M1),
MacOSX 12.6 21.6.0:
```
cpucycles version 20251226
cpucycles tracesetup 0 arm64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 arm64-pmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 arm64-vct precision 200 scaling 100.000000 only32 0
cpucycles tracesetup 3 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 4 default-mach precision 200 scaling 100.000000 only32 0
cpucycles tracesetup 5 default-monotonic precision 2699 scaling 2.399988 only32 0
cpucycles tracesetup 6 default-gettimeofday precision 2699 scaling 2399.987654 only32 0
cpucycles tracesetup 7 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2399987654
cpucycles implementation arm64-vct
cpucycles iqm 0 +4500+0+0+0+0+0+100+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+100+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+100+0+0+0+0+0+0+0
cpucycles observed persecond 1725000000...3450000000 with 8192 loops 3 microseconds
cpucycles observed persecond 1885714285...2640000000 with 16384 loops 6 microseconds
cpucycles observed persecond 2300000000...2533333334 with 32768 loops 22 microseconds
cpucycles observed persecond 2339285714...2523076924 with 65536 loops 27 microseconds
cpucycles observed persecond 2348529411...2421212122 with 131072 loops 67 microseconds
cpucycles observed persecond 2377678571...2420909091 with 262144 loops 111 microseconds
cpucycles observed persecond 2387214611...2410138249 with 524288 loops 218 microseconds
cpucycles observed persecond 2397814207...2411813187 with 1048576 loops 365 microseconds
```

`cfarm110` (`gcc1-power7`),
IBM POWER7,
CentOS 7.9 AltArch,
Linux kernel 3.10.0-1160.105.1.el7.ppc64:
```
cpucycles version 20260105
cpucycles tracesetup 0 ppc64-mftb precision 212 scaling 7.000000 only32 0
cpucycles tracesetup 1 default-perfevent precision 236 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 465 scaling 3.550000 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 3880 scaling 3550.000000 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3550000000
cpucycles implementation ppc64-mftb
cpucycles iqm 167 +36-34+36-34-13+15-20+22-27+29-41+43-48+36-41+36-34+22-20+15-13+8-6-6+8-13+15-20+22-34+36+15-13+8-6+1+1-13+15-20+22-34+36-48+36-41+8-6+36-34+22-20+15-13+8-6-6+8-20+22-27+29-41+36
cpucycles observed persecond 3159333333...4905250000 with 4096 loops 5 microseconds
cpucycles observed persecond 3386727272...4216333334 with 8192 loops 10 microseconds
cpucycles observed persecond 3348333333...3739842106 with 16384 loops 20 microseconds
cpucycles observed persecond 3534829268...3733153847 with 32768 loops 40 microseconds
cpucycles observed persecond 3473016129...3599166667 with 65536 loops 61 microseconds
cpucycles observed persecond 3541000000...3609900000 with 131072 loops 111 microseconds
cpucycles observed persecond 3571450450...3606177273 with 262144 loops 221 microseconds
cpucycles observed persecond 3578959367...3596333334 with 524288 loops 442 microseconds
cpucycles observed persecond 3578793453...3587492082 with 1048576 loops 885 microseconds
```

`cfarm112` (`gcc2-power8`),
IBM POWER8E,
CentOS 7.9 AltArch,
Linux kernel 3.10.0-1127.13.1.el7.ppc64le:
```
cpucycles version 20260105
cpucycles tracesetup 0 ppc64-mftb precision 259 scaling 7.250000 only32 0
cpucycles tracesetup 1 default-perfevent precision 307 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 660 scaling 3.690000 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 4020 scaling 3690.000000 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3690000000
cpucycles implementation ppc64-mftb
cpucycles iqm 196 +14-8+0+0+7+7+0+7-8-7-1+0+0-8+0+0+7+0+7-1-7+0-1+0+0-8+0+0+0-1+0+0-8+7+0+0+0-1+7-7-8+15-8+0+7-8+0+7+7-7-1+0-7+7-1+0-22+22-15+7-8+0+7+14
cpucycles observed persecond 2936250000...9610000000 with 1024 loops 3 microseconds
cpucycles observed persecond 2954857142...4331200000 with 2048 loops 6 microseconds
cpucycles observed persecond 3493181818...4349222223 with 4096 loops 10 microseconds
cpucycles observed persecond 3601909090...4005950000 with 8192 loops 21 microseconds
cpucycles observed persecond 3655128205...3876594595 with 16384 loops 38 microseconds
cpucycles observed persecond 3644683333...3781000000 with 32768 loops 59 microseconds
cpucycles observed persecond 3673369747...3741435898 with 65536 loops 118 microseconds
cpucycles observed persecond 3687219409...3721314894 with 131072 loops 236 microseconds
cpucycles observed persecond 3710044585...3727196163 with 262144 loops 470 microseconds
cpucycles observed persecond 3709852441...3718409575 with 524288 loops 941 microseconds
cpucycles observed persecond 3709862869...3714579726 with 1048576 loops 1895 microseconds
```

`cfarm120`,
IBM POWER10,
AlmaLinux 9.7,
Linux kernel 5.14.0-427.31.1.el9_4.ppc64le:
```
cpucycles version 20260105
cpucycles tracesetup 0 ppc64-mftb precision 123 scaling 5.750000 only32 0
cpucycles tracesetup 1 default-perfevent precision 216 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 346 scaling 2.950000 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 3280 scaling 2950.000000 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2950000000
cpucycles implementation ppc64-mftb
cpucycles iqm 60 +15-37+15+14-37+21-31+14+9-31+15-37+15+20-31+9+15-32+9-31+21-37+14+15-37+15-31+14+9-31+15+20-37+15-31+9+15-32+21-37+15+14-37+15-31+9+20-31+9-31+15+9-32+21-31+9+14-31+9-31+15+14-31+15
cpucycles observed persecond 2386200000...4069333334 with 4096 loops 4 microseconds
cpucycles observed persecond 2290333333...2982571429 with 8192 loops 8 microseconds
cpucycles observed persecond 2794500000...3208142858 with 16384 loops 15 microseconds
cpucycles observed persecond 2895413793...3117555556 with 32768 loops 28 microseconds
cpucycles observed persecond 2849442307...2969420000 with 65536 loops 51 microseconds
cpucycles observed persecond 2900871428...2989911765 with 131072 loops 69 microseconds
cpucycles observed persecond 2919597122...2963941606 with 262144 loops 138 microseconds
cpucycles observed persecond 2931025270...2953010910 with 524288 loops 276 microseconds
cpucycles observed persecond 2940833333...2951965455 with 1048576 loops 551 microseconds
```

`cfarm185`,
Ampere eMAG 8180,
AlmaLinux 8.10,
Linux kernel 4.18.0-553.el8_10.aarch64:
```
cpucycles version 20260105
cpucycles tracesetup 0 arm64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 arm64-pmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 arm64-vct precision 175 scaling 75.000000 only32 0
cpucycles tracesetup 3 default-perfevent precision 285 scaling 1.000000 only32 0
cpucycles tracesetup 4 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 5 default-monotonic precision 395 scaling 3.000000 only32 0
cpucycles tracesetup 6 default-gettimeofday precision 3330 scaling 3000.000000 only32 0
cpucycles tracesetup 7 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3000000000
cpucycles implementation arm64-vct
cpucycles iqm 42 +33+33+33-42-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33-42+33+33-42+33-42+33-42+33-42+33-42+33-42+33
cpucycles observed persecond 2280000000...3950000000 with 4096 loops 4 microseconds
cpucycles observed persecond 2533333333...3300000000 with 8192 loops 8 microseconds
cpucycles observed persecond 2840625000...3267857143 with 16384 loops 15 microseconds
cpucycles observed persecond 2840625000...3040000000 with 32768 loops 31 microseconds
cpucycles observed persecond 2929838709...3031250000 with 65536 loops 61 microseconds
cpucycles observed persecond 2976024590...3027500000 with 131072 loops 121 microseconds
cpucycles observed persecond 2987654320...3013692947 with 262144 loops 242 microseconds
cpucycles observed persecond 2993659793...3006677019 with 524288 loops 484 microseconds
cpucycles observed persecond 2996671826...3003179938 with 1048576 loops 968 microseconds
```

`cfarm202`,
UltraSparc T5,
Debian forky/sid,
Linux kernel 6.17.0-rc5+:
```
cpucycles version 20260105
cpucycles tracesetup 0 sparc64-rdtick precision 65 scaling 1.000000 only32 0
cpucycles tracesetup 1 default-perfevent precision 598 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 535 scaling 3.599910 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 3929 scaling 3599.910000 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 3599910000
cpucycles implementation sparc64-rdtick
cpucycles iqm 73 +24+24+24+24+24+0+0+24+24+0+0+24+0+0+1+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0
cpucycles observed persecond 2899000000...5050666667 with 4096 loops 4 microseconds
cpucycles observed persecond 3196888888...4210714286 with 8192 loops 8 microseconds
cpucycles observed persecond 3191666666...3627500000 with 16384 loops 17 microseconds
cpucycles observed persecond 3478606060...3722064517 with 32768 loops 32 microseconds
cpucycles observed persecond 3530492307...3651936508 with 65536 loops 64 microseconds
cpucycles observed persecond 3557038759...3617251969 with 131072 loops 128 microseconds
cpucycles observed persecond 3584414062...3614846457 with 262144 loops 255 microseconds
cpucycles observed persecond 3591225048...3607400786 with 524288 loops 510 microseconds
cpucycles observed persecond 3593313111...3603112746 with 1048576 loops 1021 microseconds
```

`cfarm216`,
VM on SPARC-M8,
Solaris 11.4.88.207.1:
```
cpucycles version 20260105
cpucycles tracesetup 0 sparc64-rdtick precision 63 scaling 1.000000 only32 0
cpucycles tracesetup 1 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 614 scaling 5.067000 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 5387 scaling 5067.000000 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 5067000000
cpucycles implementation sparc64-rdtick
cpucycles iqm 69 +25+25+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+52780+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0
cpucycles observed persecond 4115600000...6985666667 with 8192 loops 4 microseconds
cpucycles observed persecond 4562000000...5932142858 with 16384 loops 8 microseconds
cpucycles observed persecond 4826411764...5501066667 with 32768 loops 16 microseconds
cpucycles observed persecond 4822647058...5138750000 with 65536 loops 33 microseconds
cpucycles observed persecond 4892223880...5049984616 with 131072 loops 66 microseconds
cpucycles observed persecond 4965643939...5045630770 with 262144 loops 131 microseconds
cpucycles observed persecond 4984129277...5024111112 with 524288 loops 262 microseconds
cpucycles observed persecond 4993422857...5013416827 with 1048576 loops 524 microseconds
```

`cfarm230`,
Cavium Octeon III V0.2,
Debian 10.13,
Linux kernel 4.9.79-UBNT_E300:
```
cpucycles version 20260105
cpucycles tracesetup 0 mips64-cc precision 323 scaling 1.000000 only32 1
cpucycles tracesetup 1 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 1543 scaling 2.399988 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 2719 scaling 2399.987654 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2399987654
cpucycles implementation mips64-cc
cpucycles iqm 405 +18+45+34+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0
cpucycles observed persecond 778900000...1621375000 with 1024 loops 9 microseconds
cpucycles observed persecond 924187500...1164857143 with 2048 loops 15 microseconds
cpucycles observed persecond 970133333...1090392858 with 4096 loops 29 microseconds
cpucycles observed persecond 962833333...1020155173 with 8192 loops 59 microseconds
cpucycles observed persecond 983846153...1013391305 with 16384 loops 116 microseconds
cpucycles observed persecond 990508620...1005204348 with 32768 loops 231 microseconds
cpucycles observed persecond 996039045...1003529412 with 65536 loops 460 microseconds
cpucycles observed persecond 998831338...1002535442 with 131072 loops 918 microseconds
cpucycles observed persecond 998601741...1000789216 with 262144 loops 1837 microseconds
cpucycles observed persecond 999574618...1000561853 with 524288 loops 3671 microseconds
cpucycles observed persecond 999717524...1000239919 with 1048576 loops 7366 microseconds
```

`cfarm400`,
Loongson-3C5000L-LL,
Debian trixie/sid,
Linux kernel 6.1.0-rc7+:
```
cpucycles version 20260105
cpucycles tracesetup 0 loong64-rdtime precision 124 scaling 24.000000 only32 0
cpucycles tracesetup 1 default-perfevent precision 180 scaling 1.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 416 scaling 2.399988 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 2729 scaling 2399.987654 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2399987654
cpucycles implementation loong64-rdtime
cpucycles iqm 48 +24+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+24+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+24+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+24+0+0+0+0+0+0+0
cpucycles observed persecond 1982400000...3408000000 with 4096 loops 4 microseconds
cpucycles observed persecond 1975200000...2514000000 with 8192 loops 9 microseconds
cpucycles observed persecond 2190666666...2482500000 with 16384 loops 17 microseconds
cpucycles observed persecond 2315294117...2469000000 with 32768 loops 33 microseconds
cpucycles observed persecond 2348776119...2425846154 with 65536 loops 66 microseconds
cpucycles observed persecond 2383636363...2422892308 with 131072 loops 131 microseconds
cpucycles observed persecond 2392425855...2411954023 with 262144 loops 262 microseconds
cpucycles observed persecond 2392334600...2402061069 with 524288 loops 525 microseconds
cpucycles observed persecond 2396822857...2401694657 with 1048576 loops 1049 microseconds
```

`cfarm425`,
Ampere Altra Max M128-30,
Debian forky,
Linux kernel 6.16.12+deb14+1-arm64:
```
cpucycles version 20260105
cpucycles tracesetup 0 arm64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 arm64-pmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 arm64-vct precision 196 scaling 96.000000 only32 0
cpucycles tracesetup 3 default-perfevent precision 183 scaling 1.000000 only32 0
cpucycles tracesetup 4 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 5 default-monotonic precision 512 scaling 2.399988 only32 0
cpucycles tracesetup 6 default-gettimeofday precision 2729 scaling 2399.987654 only32 0
cpucycles tracesetup 7 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2399987654
cpucycles implementation default-perfevent
cpucycles iqm 84 +13+10-2+0+0+0+0+0+10+0+0+0+0+0+0+0+0+0+0+0+11+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+0+230+0+0+0+0
cpucycles observed persecond 144875000...424833334 with 1024 loops 7 microseconds
cpucycles observed persecond 281000000...430500000 with 2048 loops 7 microseconds
cpucycles observed persecond 481500000...641250000 with 4096 loops 9 microseconds
cpucycles observed persecond 678142857...817416667 with 8192 loops 13 microseconds
cpucycles observed persecond 786041666...871772728 with 16384 loops 23 microseconds
cpucycles observed persecond 916658536...971717949 with 32768 loops 40 microseconds
cpucycles observed persecond 973724637...1007477612 with 65536 loops 68 microseconds
cpucycles observed persecond 1016033783...1032109590 with 131072 loops 147 microseconds
cpucycles observed persecond 966758503...975681507 with 262144 loops 293 microseconds
cpucycles observed persecond 1038860952...1043439771 with 524288 loops 524 microseconds
cpucycles observed persecond 1044148622...1046517752 with 1048576 loops 1015 microseconds
```

`cfarm430`,
AMD EPYC 7773X,
FreeBSD 16.0 VM:
```
cpucycles version 20251226
cpucycles tracesetup 0 amd64-perfpmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 1 amd64-pmc precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 amd64-tsc precision 122 scaling 1.000000 only32 0
cpucycles tracesetup 3 amd64-tscasm precision 122 scaling 1.000000 only32 0
cpucycles tracesetup 4 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 5 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 6 default-monotonic precision 468 scaling 2.399988 only32 0
cpucycles tracesetup 7 default-gettimeofday precision 2699 scaling 2399.987654 only32 0
cpucycles tracesetup 8 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 2399987654
cpucycles implementation amd64-tsc
cpucycles iqm 30 +14+14-8-8+14-8+14-8-8+630+14-8-8+14-8+14-8-8+14-8+14-8+14-8-8+14-8+14-8+14-8-8+14-8+14-8+14-8-8+14-8+14-8+14-8-8+14-8+14-8+14-8-8+14-8+14-8-8+14-8+14-8-8+14
cpucycles observed persecond 1557600000...2720666667 with 8192 loops 4 microseconds
cpucycles observed persecond 1949750000...2665666667 with 16384 loops 7 microseconds
cpucycles observed persecond 2078266666...2426769231 with 32768 loops 14 microseconds
cpucycles observed persecond 2078266666...2240071429 with 65536 loops 29 microseconds
cpucycles observed persecond 2132058823...2222666667 with 131072 loops 67 microseconds
cpucycles observed persecond 2168434782...2210318585 with 262144 loops 114 microseconds
cpucycles observed persecond 2196995594...2218284445 with 524288 loops 226 microseconds
cpucycles observed persecond 2196947136...2207446903 with 1048576 loops 453 microseconds
```

IBM z15:
```
cpucycles version 20230106
cpucycles tracesetup 0 s390x-stckf precision 250 scaling 1.269531 only32 0
cpucycles tracesetup 1 default-perfevent precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 2 default-mach precision 0 scaling 0.000000 only32 0
cpucycles tracesetup 3 default-monotonic precision 272 scaling 5.200000 only32 0
cpucycles tracesetup 4 default-gettimeofday precision 5400 scaling 5200.000000 only32 0
cpucycles tracesetup 5 default-zero precision 0 scaling 0.000000 only32 0
cpucycles persecond 5200000000
cpucycles implementation s390x-stckf
cpucycles median 48 +87+8+0-2+0+0+38-2+0+1-3+1+28+0+3-3+1+0+28+0-2+3+0-2+36+0+0+0+1+0+28+0-2+0+3-2+35+1+0-2+0+3+28+0-2+0+0-2+3+25+3+0-2+0+1+35+1+0+0-2+0+28+0
cpucycles observed persecond 4948941176...5627733334 with 8192 loops 16 microseconds
cpucycles observed persecond 4104125000...5515666667 with 16384 loops 7 microseconds
cpucycles observed persecond 5047076923...5987818182 with 32768 loops 12 microseconds
cpucycles observed persecond 5044846153...5475708334 with 65536 loops 25 microseconds
cpucycles observed persecond 5141313725...5357428572 with 131072 loops 50 microseconds
cpucycles observed persecond 5150892156...5257250000 with 262144 loops 101 microseconds
cpucycles observed persecond 5183421568...5236549505 with 524288 loops 203 microseconds
cpucycles observed persecond 5190282555...5216582717 with 1048576 loops 406 microseconds
```
