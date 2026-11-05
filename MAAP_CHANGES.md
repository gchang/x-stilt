# Changes from upstream

Upstream: https://github.com/uataq/X-STILT, `master` at commit 81e95e5890b2d6586f943308bdee202fe3c76cf4 (2026-08-20, "Handle normalized TROPOMI CO averaging kernels (#19)"), by Dien Wu and the X-STILT contributors. The `stilt` submodule stays at uataq/stilt commit 0f9ff18b7fbba7e4bde00c679676cd3a35a61024, as upstream pins it (clone with `git clone --recursive`). Upstream has no license file; none was added.

This copy starts from upstream's file tree at 81e95e5 without its git history. Only `run_xstilt.r` changed, as below. Without the new settings it behaves as before, so `source('run_xstilt.r')` in an R session, after editing it, still works the same way.

## 2026-11-05: run_xstilt.r takes site, coordinates and folders as key=value words

The script could only be run by editing it and sourcing it in an R session: it set the working folder, input, output and met folders to paths on the author's cluster (`/central/home/dienwu`, `/central/groups/POW`), read the site name from standard input, and looked the site up with Google geocoding, which needs an API key in `insert_ggAPI.csv`.

It now also reads optional `key=value` words from the command line: `xstilt_wd`, `site`, `site_lon`, `site_lat`, `input_path`, `store_path`, `recp_fn` and `met_path`. A setting that is not given keeps its old value. When `site` is given there is no prompt, and when `site_lon` and `site_lat` are given the coordinates go to `get.lon.lat()` as `site.loc` (the route its own comment describes), so no Google key is needed. For example, the ideal-mode demo:

```bash
Rscript run_xstilt.r xstilt_wd=$PWD site=Toronto site_lon=-79.38318 site_lat=43.65323 \
  store_path=$PWD/out met_path=/path/to/gfs0p25 slurm=F n_cores=4
```

Results do not change.

## 2026-11-05: run_xstilt.r runs on the machine's own cores when SLURM is off

`slurm = T` was fixed in the script, and `slurm = F` forced one core (`n_nodes = n_cores = 1`), so without SLURM the receptors ran one after another. `slurm=F` on the command line now runs them on this machine, and `n_cores=` sets how many worker processes `xstilt_apply()` forks for them (its existing single-node branch). SLURM stays the default, and `slurm = F` without `n_cores` still means one process. Results do not change.

## Check

On 2026-11-05, Linux x86-64, R 4.4.3 with conda-forge packages (dplyr 1.2.1, ncdf4 1.24, raster 3.6.32, ggmap 4.0.2, rworldmap 1.3.8, lutz 0.3.2, rslurm 0.6.2, R.utils 2.13.0, proj4 1.0.15, gfortran for STILT's `permute.so`), with no network, GFS 0.25 degree met for 2024-09-11 and 2024-09-12 downloaded beforehand from `ftp://arlftp.arlhq.noaa.gov/archives/gfs0p25/` (2.9 GB each), and a receptor file holding the first receptor of `receptor_demo.csv` (Toronto, 2024-09-12 00 UTC):

```bash
Rscript run_xstilt.r xstilt_wd=$PWD site=Toronto site_lon=-79.38318 site_lat=43.65323 \
  store_path=$PWD/out recp_fn=recp1.csv met_path=$PWD/gfs0p25 slurm=F n_cores=2
```

The first run set up STILT (`stilt/setup`), then ran the 3000-particle column release 12 hours back and wrote the trajectories and the time-integrated 0.1 degree column footprint (`out/out_2024091200_gfs0p25_ideal/by-id/202409120000_-79.38318_43.65323_X/`). Exit 0, 2 min 54 s, 2.0 GB peak memory. No prompt, no Google key and no `sbatch` were needed. The footprint is finite and non-negative, sums to 0.099 ppm per (umol m-2 s-1) and peaks in the cell at the receptor.
