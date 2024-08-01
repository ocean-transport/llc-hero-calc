# llc-hero-calc

Notebooks and scripts for vorticity-strain-divergence histogram calculations on LLC4320 data.

## LLC4320

LLC4320 is a very high-resolution simulation of global ocean circulation circulation, done by Dmitri Menemelis et al. on the NASA Pleiades supercomputer using the [MIT general circulation model](https://github.com/MITgcm/MITgcm).

See [these slides](https://online.kitp.ucsb.edu/online/blayers18/menemenlis/pdf/Menemenlis_BLayers18_KITP.pdf) for more details about the simulation dataset.

## Vorticity-strain histograms

Vorticity-strain-divergence histograms are a way of diagnosing general features of turbulent ocean flow. In particular [Balwada et al. (2021)](https://journals.ametsoc.org/view/journals/phoc/51/9/JPO-D-21-0016.1.xml) showed that they can give information about ocean ventilation.

They generally look like this

![image](https://user-images.githubusercontent.com/35968931/221258445-0ae6267b-ff00-45ce-8e88-c17217931130.png)

## Aim

We aim to compute the full vorticity-strain-divergence histogram for all locations and all times in the LLC4320 dataset. We will divide the whole global ocean up into many small regions (similar to the approach used for spectral analysis in Torres et al. 2018) and create one histogram for each region, averaged over an intermediate time period.

Because this analysis requires processing 16 Terabytes of velocity data (that's just for the surface - the full dataset with all vertical depth levels would be Petabytes!) this is a significant computational challenge. Doing this in python with the Pangeo tools would be a powerful demonstration of their utility on a real science problem, which is why we have been referring to this analysis task as the "hero calculation".

## Computation

For this calculation we have to perform several steps:
1) Compute the vorticity, strain, and divergence from the x- and y-components of the flow velocities, correctly accounting for the fact the variables lie on an Arakawa grid staggered relative to one another;
2) Split the resulting spatial fields up into a number of smaller regions;
3) Compute the 3-dimensional vorticity-strain-divergence histogram for each region, without also counting over the time dimension;
4) Average the resulting histograms over some time period (e.g. a few days),
5) Save the new regional histogram dataset to an intermediate zarr store,
6) Extract useful information from the resulting histograms.

For (1) we used [xGCM](https://github.com/xgcm/xgcm) - see also [these AMS talk slides](https://speakerdeck.com/tomnicholas/xgcm-staggered-grids-topologies-and-ufuncs-in-python). To compute arbitary operations like vorticity we refactored xGCM to be able to apply "grid ufuncs". The [documentation on grid ufuncs is here](https://xgcm.readthedocs.io/en/latest/grid_ufuncs.html), but a blog post is also planned.

For (3) we used [xhistogram](https://github.com/xgcm/xhistogram) - see also [this slide (and the next) from my Scipy 2022 talk](https://speakerdeck.com/tomnicholas/scipy-2022-can-we-analyse-the-largest-ocean-simulation-ever?slide=15).

For (2) and (4) we use [xarray](https://xarray.dev/)'s built-in [`.coarsen` feature](https://docs.xarray.dev/en/stable/user-guide/reshaping.html#reshaping-via-coarsen).

All these steps have to be done on a very large amount of data, which we handled using dask. We found that dask's distributed scheduler needed to be improved before it could handle this workload - see this [pangeo blog post](https://medium.com/pangeo/dask-distributed-and-pangeo-better-performance-for-everyone-thanks-to-science-software-63f85310a36b) and this [Coiled blog post](https://www.coiled.io/blog/reducing-dask-memory-usage).

For (5) you can see the code used for the computation in the `compute` directory, specifically [this notebook](https://github.com/ocean-transport/llc-hero-calc/blob/main/compute/coarsen-nan-padding.ipynb). The actual dataset was written out to a zarr store in this [bucket](gs://leap-persistent/tomnicholas/hero-calc/compute/llc4320/vort_strain_div_histogram_coarsen_nan_padding.zarr).

## Results

We are in the progress of doing (6). See notebooks in this repository in the `analysis` directory.

## Data availability

Currently available as a Zarr store sat in a persistent Google cloud storage [bucket](gs://leap-persistent/tomnicholas/hero-calc/compute/llc4320/vort_strain_div_histogram_coarsen_nan_padding.zarr) on the [LEAP-Pangeo hub](https://leap-stc.github.io/intro.html) but we plan to put the output histogram data in a permanent public bucket for other researchers to use.
