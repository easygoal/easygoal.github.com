---
layout: post
title: Veloce Basics
category: tutorial
tagline: ""
tags: ["veloce", "emulation", "tech"]
keywords: veloce, compile 
ref: veloce-compile-flow
lang: en
---

## Compilation Flow 

1. Setup environment variable
SSH to one dlinux server
```
sshnew
source /pub/emu/veloce/strato_v1709.csh
```
2. Velanalyze
Navigate to compilation folder 
```
cd /pub/emu/veloce/SharkL5_DE_102
```
open Make.file, if the compilation hasn’t run before or the RTL code hasn’t been modified, you need to analyze RTL code first, “make lib analyze”, elsewise you can omit this step.

3. Velcomp
Execute ``./run``, the content is ``bsub –P SharkL5 –q analog –Is –XF "velcomp"``, it will submit to LSF group. If you have modified some veloce related script files, like ``veloce.clk/veloce.cnffi/veloce.tnfi/veloce.cnffi/veloce.tfi/veloce.force``, you need to restart to run ``velcomp``

4. Force signals
If you want to add some signals to force file, and modify the value during runtime, you need to add the path to veloce.force, and rerun velcomp. The path format is like below.
``NET proj_top_th/chip_top/EXT_RST_B``
Please note the path should be full path, ‘*/?’ are supported.

5. Trigger signals.
You need to add signals to veloce.tfi before using them as trigger signals, format is like:
``NET proj_top_th/chip_top/dut/u_digital_top/u_sys_pub/*_cpu_emu*``
Registers could be used as trigger signals without adding to veloce.tfi file, so if you want to trig some signals but find they are not added to veloce.tfi file and don’t want to recompile, you could find the drivers of the signals, adding the drivers which are FF output to trigger conditions.


## Runtime Flow
1. When the compilation is finished, run ``make report`` or ``./report.sh``, it will print some information, like how many boards are used and the performance, like 500 KHz, please copy the value to variable EMU_DB_FREQ of do.file
2. SSH to veloce02 server, ``ssh –XY veloce02``, and navigate to path, like ``/proj/emu/veloce/SharkL5/SharkL5_DE_102``, please note start from /proj not /pub, and source environment variable, ``source /proj/emu/veloce/strato_v1709.csh``
3. Check whether the resource is available, “whoison –emul $Emulator”, if there are enough resources, you could use it.
4. Execute ``make run`` or ``velrun –gui –do do.file &``, it will start to run with GUI. You may use ``run –stop`` or ``stop`` to stop current running, and ``reinitialize`` to restart a new session without quit the current connect, then copy and paste the do.file content.
5. You may modify the do.file content, like memory download/reg force/velclockgen, etc.
6. You may use trigger edit tool to write the trigger file, and download it. If the trigger signals are not available (not FF out or on the veloce.tfi list), it will report error.
7. You may upload the RAW trace data to server within velrun GUI, like ``hwtrace upload –tracedir jtag_ok –noreplay``, then generate the waveform in dlinux* servers with command ``velwavegen –tracedir veloce.wave/jtag_ok.stw``.
8. Also, you could generate FSDB format waveform with command ``ecfwave –tracedir veloce.wave/jtag_ok.stw –siglist fsdb.list –fsdb –o jtag_ok.fsdb``, the name could be changed, you may edit the content of fsdb.list as below.
```
proj_top_th/*
proj_top_th/* -r
```

The reference manual could be find from server path ``$VMW_HOME/doc/pdfdocs/``, the useful files are veloce_ug.pdf and veloce_ref.pdf.
