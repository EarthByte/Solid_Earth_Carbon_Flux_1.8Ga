# Solid_Earth_Carbon_Flux_1.8Ga

This repository is derived from:

https://github.com/EarthByte/Solid_Earth_carbon_degassing_sequestration_1Ga

and applied to the plate model by Cao et al. (2024) placed in a mantle reference frame using the model by Shirmard et al. (2025).

## Dependencies
The worfkflow requires the following Python packages:

- [pyGPlates](https://www.gplates.org/docs/pygplates/pygplates_getting_started.html#installation)
- [PlateTectonicTools](https://github.com/EarthByte/PlateTectonicTools)
- [GPlately](https://github.com/GPlates/gplately/tree/master)
- [Slab-Dip](https://github.com/brmather/Slab-Dip)
- [melt](https://github.com/brmather/melt)
- [joblib](https://joblib.readthedocs.io/en/stable/) _(these workflows use parallelisation with `joblib` and cannot be run on a single thread)._
- [cmcrameri](https://github.com/callumrollo/cmcrameri) _(for geoscience colourmaps)._

## Input files
This workflow requires a series of input files:

### Required input data 
The utility folder `./utils/` contains all required text files, spreadsheets, Python scripts and the Cao et al. 2024 plate reconstruction model required to run this workflow (in the directory `./utils/Cao_etal_2024_1.8_Ga_mantle_ref_frame`). The workflow is structured to access these files without any further action.

### Required netCDF grids

This workflow requires a number of input netCDF grids over the time range of the chosen plate reconstruction model. The following list describes these grids and the corresponding notebooks and workflows which can be run to prepare these grids:

1. **Pelagic carbonate sediment thickness grids (km) (spanning 170-0Ma)** - these can be produced from [EarthByte's CarbonateSedimentThickness workflow](https://github.com/EarthByte/CarbonateSedimentThickness).
2. **Contoured continental masks** - these should be contoured/buffered continental polygon masks which outline continental passive margin segments (continent-ocean boundary line segments) through time. These may be produced using [EarthByte's continent contouring workflow](https://github.com/EarthByte/continent-contouring), which also produces passive margins, trenches and contoured continent polygon gpmls. The passive margin gpml file is also a required input for this workflow.
3. **Crustal CO2 grids (wt%)** - These grids are calculated using [**/utils/Min-Mean-Max-Crustal-Carbon.ipynb**](./utils/Min-Mean-Max-Crustal-Carbon.ipynb) and contain area densities of crustal CO2 (wt%) as a function of both seafloor age and bottom water temperature, determined from a bilinear-log lookup table (all contained in the `/utils/` parent directory). This workflow assumes 100% of crustal CO2 lie in the top 450km of crust.
4. **Seafloor age grids (Myr)** - These grids can be made with [**/utils/A-SeafloorGrids.ipynb**](./utils/A-SeafloorGrids.ipynb) from the contoured continent masks (Point 2). These must be based on a full spreading rate in units of mm/yr.
5. **Seafloor spreading rate (Myr)** - as for seafloor age grids. Considers a full spreading rate in units of mm/yr.
6. **Total sediment thickness grids (m)** - Made using [EarthByte's predicting sediment thickness workflow](https://github.com/EarthByte/predicting-sediment-thickness), which calculates thickness using polynomials of ocean floor age and distance to passive margins.

### Accessing the input grids
The notebooks will reference these grids using the following directory structure:

```
    # Parent directory for input grids
    grid_directory = "./Grids/InputGrids/"
    # Spreading rate grid filename
    spreadrate_filename = grid_directory+"SpreadingRate/Cao2024_SPREADING_RATE_grid_{:.2f}Ma.nc"
    # Seafloor age grid filename
    agegrid_filename = grid_directory+"SeafloorAge/Cao2024_SEAFLOOR_AGE_grid_{:.2f}Ma.nc"
```

This can be changed to suit the directory made for the input grids above, or the input grids can be saved to the following directory structure for ease of running the notebooks (i.e. they have been designed to run with the following directory structure):

```plaintext
📁 Grids/
 └── 📁 InputGrids/
     ├── 📁 SeafloorAge/              # seafloor age grids
     ├── 📁 SpreadingRate/            # seafloor spreading rate grids
     ├── 📁 TotalSediment/            # total sediment thickness grids
     ├── 📁 ContinentalMasks/         # continental/ocean mask grids
     ├── 📁 CarbonateSediment/        # carbonate sediment thickness
     │    ├── 📁 min/
     │    ├── 📁 mean/
     │    └── 📁 max/
     └── 📁 CrustalCarbon/            # crustal CO2 (wt%) grids
          ├── 📁 min/
          ├── 📁 mean/
          └── 📁 max/
```

# Workflow

## Main Notebooks

Once the input files above have been generated, run the following notebooks in order:

1. [**01-Sources-of-Carbon**](./01-Sources-of-Carbon.ipynb) - Generates grids of carbon area density (t/m2) contained within 5 storage reservoirs: crust, lithosphere, pelagic carbonate sediments, serpentinite and organic sediments. 
2. [**02-Subducted-Carbon**](./02-Subducted-Carbon.ipynb) - Interpolates the grids from *01-Sources-of-Carbon* onto tessellated points of global subduction zones, quantifying the proportion of sequestered carbon that is subducted at trenches every 1 million years for each storage reservoir. The global distribution of cumulative subducted carbon is also analysed.
   - The auxiliary notebook [**./utils/0A-Cumulative-Subducted-Carbon**](./utils/0A-Cumulative-Subducted-Carbon.ipynb) (which is not required for the main workflow) calculates and exports grids of the area density of cumulative subducted carbon at each storage reservoir.
4. [**03-Carbon-Degassing**](./03-Carbon-Degassing.ipynb) - Computes the atmospheric degassing of carbon at mid ocean ridges (using a fitted polynomial by Keller et al. 2017) and at trenches (by subtracting the CO2 lost at depth from the subducted carbon from (Notebook 2)).

Notebooks from `./H2O/` consider pore water and chemically bound water within major volatile-carrying reservoirs. The following 3 notebooks calculate water subduction flux and water slab storage rate, which are needed to calculate the outflux of H2O from slabs. This is a required input to calculate the hydrous mantle melting rate as per Katz et al. (2003). The melting rate is taken to equal the carbonate platform degassing rate.

4. [**/H2O/01-Sources-of-Water**](./H2O/01-Sources-of-Water.ipynb) - Quantifies in-plate contained water in Mt/m2
/yr for four reservoirs: crust, lithosphere, sediments and serpentinite.
5. [**/H2O/02-Subducted-Water**](./H2O/02-Subducted-Water.ipynb) - Quantifies how much water is subducted from each reservoir, as well as water plate influx.
6. [**/H2O/06-Maps-of-water-storage.ipynb**](./H2O/06-Maps-of-water-storage.ipynb.ipynb) - Calculates water subduction flux and water slab storage rate.

7. [**04-Carbonate-Platform-Degassing**](./04-Carbonate-Platform-Degassing.ipynb) - Calculates the carbonate platform degassing rate from outputs of the H2O notebooks (Steps 4-6 above).
8. [**05-Atmospheric-Carbon**](./05-Atmospheric-Carbon.ipynb) - Calculates gross and net atmospheric influx of carbon through time from various sources like mid-ocean ridges, subduction zones, carbonate platforms, rifts and intraplate volcanism.
9. [**06-PlateTectonicStats**](./06-PlateTectonicStats.ipynb) - An optional notebook that calculates plate tectonic statistics through time, such as plate boundary (mid-ocean ridge and subduction zone) lengths, spreading and convergence rates, crustal production and destruction rates, age of subducted and global seafloor, passive margin and rift lengths, and continental subduction zone length. 

# Changes since *Solid Earth Carbon Degassing Sequestration to 1Ga*
Changes to the workflow since [Müller et al. (2024)](https://zenodo.org/records/13888368) in addition to applying the workflow to the Cao et al. (2024) model. are as follows:
1. **01-Sources-of-Carbon** - N/A
2. **02-Subducted-Carbon** - N/A
3. **03-Carbon-Degassing** - N/A
4. **/H2O/01-Sources-of-Water** - N/A
5. **/H2O/02-Subducted-Water** - N/A
6. **/H2O/06-Maps-of-water-storage.ipynb** - N/A

7. **04-Carbonate-Platform-Degassing** - The carbonate-platform-intersecting subduction zone outflux was scaled using a correction taken from *Stewart, E.M. and Penman, D.E., 2024. Enhanced metamorphic CO2 release on the Proterozoic Earth. Proceedings of the National Academy of Sciences, 121(40), p.e2401961121.* due to metamorphic carbon degassing along subduction zones being higher between 1.8 Ga and 500 Ma than afterwards. Moreover, carbonate platform degassing calculations within the plotting functions were amended as follows: Where melting rates are calculated along all tessellated trench points, Müller et al. (2024) took all non-masked melting rates pre-499Ma to be the degassing rate and masked all post-499Ma melting rates with a double-kd tree (trenches intersecting both continent polygons and carbonate platform polygons). Now, all pre-460Ma melting rates are masked such that only trenches intersecting continents contribute to the degassing, and post-460Ma are double-kd-tree masked. These changes only affected the calculations for plotting - the general carbonate platform outflux methodology for the main workflow remained the same. Moreover, We follow Muller et al. (2024) and use a global scaling parameter of 1 between model carbonate platform degassing from present-day melting rates and the equivalent observed present-day outflux estimated from a combination of diffuse degassing (Fisher and Aiuppa, 2020) and an estimated percentage of volcanic arc degassing (Bekaert et al., 2021) of 90% (+/- 10%). See detailed discussion in Muller et al. (2024).

8. **05-Atmospheric-Carbon** - Rift lengths were extended to 1.8Ga, taken to be equal to the average of median-filtered biased and unbiased rift lengths from 0-900Ma. Another plot was added to the end of the notebook for total outflux, total plate influx (which is equal to total outflux - net outflux) and net outflux, considering unbiased rift lengths and sediment storage from 90-0Ma. 

9. **06-PlateTectonicStats** - Crustal production and destruction rates (m2/yr) are now calculated using `gplately`’s `PlateReconstruction object`’s `crustal_production_destruction_rate()`` function which makes production approximately equal destruction at each timestep. As per Notebook 5, rift lengths for plate tectonic stats plot were extended to 1.8Ga, taken to be equal to the average of median-filtered biased and unbiased rift lengths from 0-900Ma. Moreover, plate tectonic stats panel plot now includes average global ocean floor age over time.


## Plotting Notebooks - can run once all main workflow outputs are obtained
The following notebooks in the `./utils/` folder can be run to generate *panel plots* of workflow outputs at snapshots through geological time, as well as animations of global plate tectonic reconstructions through time.
1. [**/utils/0C-Carbon-Panel-Plots**](./utils/0C-Carbon-Panel-Plots.ipynb) - Generates panel plots of workflow outputs such as the grids in Notebook 1 (grids of carbon area density (t/m2) contained within each of the storage reservoirs), seafloor age distribution, subduction zone slab dip magnitudes, global distributions of spreading and convergence rate, seafloor spreading rate, total sediment thickness and the convergence rates of continental arcs.

2. [**/utils/0D-Carbon-Panel-Plot-Videos**](./utils/0D-Carbon-Panel-Plot-Videos.ipynb) - Produces the video animation equivalents of the panel plots in Notebook 0C. Requires `moviepy`.

