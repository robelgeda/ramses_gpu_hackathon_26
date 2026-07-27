# ramses_gpu_hackathon_26

GPU port and profiling work on RAMSES for the 2026 Princeton/NVIDIA Open Hackathon. This repo tracks the test problems, build instructions, and kernel fixes worked on during the event. See the [Wiki](https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki) for full testing results, profiling notes, and fix write-ups.

## Related repos

- Last year's hackathon notes: [ramses_gpu_hackathon_25](https://github.com/robelgeda/ramses_gpu_hackathon_25)
- GPU version ("mini-ramses"): [bitbucket.org/rteyssie/mini-ramses](https://bitbucket.org/rteyssie/mini-ramses/src/develop/)
- Main RAMSES repo: [ramses-organisation/ramses](https://github.com/ramses-organisation/ramses)

## Cluster access

For the Princeton/NVIDIA Open Hackathon, reserved GPU nodes on Della and Stellar are available starting May 26, 2026. Request them with the Slurm reservation flag:

`SBATCH --reservation=openhack`

Background on Princeton GPU jobs:

- [gpu_programming_intro](https://github.com/PrincetonUniversity/gpu_programming_intro/tree/master/03_your_first_gpu_job)
- [Research Computing Slurm/GPU docs](https://researchcomputing.princeton.edu/support/knowledge-base/slurm#gpus)


## Wiki index

Fixes and profiling write-ups from the hackathon:

- [(Sedov3d AMR) Fix 1 — remove unnecessary DtoH copy in Hilbert sort](<https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/(Sedov3d-AMR)-Fix-1:-in-gpu_refine,-remove-unnecessary-DtoH-copy-in-Hilbert-sort>)
- [(Sedov3d AMR) Fix 2 — change kernel call for gather, replace scatter with memcpy(DtoD)](<https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/(Sedov3d-AMR)-Fix-2:-in-gpu_refine,-change-kernel-call-for-gather-and-replace-scatter-by-memcpy(DtoD).>)
- [(Sedov3d AMR) Fix 3 — fused kernels, keep variables in device memory](<https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/(Sedov3d-AMR)-Fix-3:-fused-2-kernels-and-keep-as-much-variables-as-possible-in-device-memory>)
- [(Sedov3d Unigrid) Nsight Compute Copilot Summary](<https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/(Sedov3d-Unigrid)-NVIDIA-Nsight-Compute-Copilot-Summary>)
- [(Sedov3d Unigrid) Troels Hydro Kernel Profiling](<https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/(Sedov3d-Unigrid)-Troels-Hydro-Kernel-Profiling>)
- [(Coeur AMR) Nsight Compute](<https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/(Coeur-AMR)-Nsight-Compute>)
- [How to Run DMO Cosmo Test Sim on GPU](https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/How-to-Run-DMO-Cosmo-Test-Sim-on-GPU)

## Build

Load the NVHPC module before compiling:

```bash
module load nvhpc/25.5
```

Compile targets:

```bash
# sedov3d (hydro only)
make NDIM=3 NPRE=4 HYDRO=1 COMPILER=NVHPC

# coeur (hydro + self-gravity)
make NDIM=3 NPRE=4 HYDRO=1 GRAV=1 COMPILER=NVHPC UNITS=COEUR INIT=COEUR
```

`NPRE=4` sets single precision. If `NPRE` is left unset, double precision is assumed, and the default `1024^3` grid will not fit in memory. For rapid testing, drop `levelmax` to 8 (`256^3`) and shift `levelmin` down by the same amount (e.g. `levelmin=6` for `sedov3d_amr`).

## Running

Once built, run any of the four test problems with the namelist file matching the problem (AMR vs unigrid, resolution are set there):

```bash
#!/bin/bash -l
#SBATCH --job-name=sedov3d
#SBATCH --time=01:00:00
#SBATCH --mem-per-cpu=100G
#SBATCH --gres=gpu:1
#SBATCH --constraint=gpu80

module purge
module load nvhpc/25.5

./ramses3d sedov3d_amr.nml > run.log
```

## Test problems

| Namelist | Tests |
|---|---|
| `sedov3d_amr` | hydro + AMR |
| `sedov3d_unigrid` | hydro |
| `coeur_amr` | hydro + AMR + self-gravity |
| `coeur_unigrid` | hydro + self-gravity |
| DMO cosmo run | dark-matter-only N-body cosmological sim, periodic BCs — see [How to Run DMO Cosmo Test Sim on GPU](https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/How-to-Run-DMO-Cosmo-Test-Sim-on-GPU) |

Namelist files and full parameter blocks for each are in the [Ramses Hackathon Wiki](https://github.com/robelgeda/ramses_gpu_hackathon_26/wiki/Ramses-Hackathon-Wiki#test-problems) page.

## Profiling tools

- **NVIDIA Nsight Systems** — quick look at how long each kernel takes
- **NVIDIA Nsight Compute** — deep dive on specific kernels (slows the code down a lot, runs each kernel ~10x to profile it)
- **NVIDIA Nsight Copilot** — LLM tool inside Nsight Compute that helps read the profiling output
