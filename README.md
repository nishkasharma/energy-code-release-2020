# About
This repo contains the code and instructions for producing the results in:
Rode, A., Carleton, T., Delgado, M. et al. Estimating a social cost of carbon for global energy consumption. Nature 598, 308–314 (2021). https://doi.org/10.1038/s41586-021-03883-8

# Requirements For Using Code In This Repo

1. You need to have `python`, `Stata`, and `R` programming capabilities, or at least environments to run code in these languages, on your computer. 

2. We use `conda` to manage `python` enrivonments, so we recommend installing `conda` if you haven't already done so following [these instructions](https://docs.conda.io/projects/conda/en/latest/user-guide/install/macos.html). 

# Setup 

1. Clone the following repos to a chosen directory, which we'll call `yourREPO` from now onwards, with the following commands: 
```
cd <yourREPO>
git clone https://github.com/ClimateImpactLab/energy-code-release-2020.git
```

2. Install the `conda` environment included in this repo by running the following commands under the root of this repo:

```
cd <yourREPO>/energy-code-release-2020
conda env create -f energy_env_py3.yaml
```

Try activating the environment:
```
conda activate energy_env_py3

```
Please remember that you will need to activate this environment whenever you run python scripts in this repo, including the `pip install -e .` commands in the following section.

Also, you need to install Jupyter for the scc calculation code
```
conda install -c conda-forge jupyterlab
 
```

3. (You can skip this step if you do not want to run the impact projections on your own. This does not affect your ability of running other scripts, like regression or plotting code.) Clone `impact-calculations`, `open-estimate` and `impact-common` into `yourREPO` and install using the following commands (Note: You'll need to activate the `energy_env_py3` environment to install the repos): 

```
cd <yourREPO>
git clone https://gitlab.com/ClimateImpactLab/Impacts/impact-calculations.git
cd impact-calculations
pip install -e .
cd ..

git clone https://github.com/ClimateImpactLab/open-estimate.git
cd open-estimate
pip install -e .
cd ..

git clone https://github.com/ClimateImpactLab/impact-common.git
cd impact-common
pip install -e .
cd ..
```


4. Install the R packages needed using the following command from the root of this repo: 
```
cd <yourREPO>/energy-code-release-2020
Rscript install_R_packages.R
```

5. Download data from `https://doi.org/10.5281/zenodo.5099834` and unzip it somewhere on your machine with 50GB+ space. Let's call this location `yourDATA`. 

6. Set up a few environmental variables so that all the code runs smoothly.

On Mac, you can do this by appending the following lines to your `~/.bash_profile`.

First, do:
```
nano ~/.bash_profile

```

Then, point the variable `DATA` in the `yourDATA` dierctory in the downloaded data, and do the same for `OUTPUT`. Point the `REPO` variable to `yourREPO` path used above containing this repo and other repos by adding the following lines to `.bash_profile`:

```
export REPO=<yourREPO>
export DATA=<yourDATA>/DATA
export OUTPUT=<yourDATA>/OUTPUT
export LOG=<yourDATA>/LOG

```
Save and exit. 
Then, run `source ~/.bash_profile` to load the changes we just made.

On Windows or other operation systems, please search how to add environmental variables. 

7. This repo is tested with Stata/SE, and currently using the `stata-se` command to run stata scripts. If you're using other versionsof stata, please mass replace all occurences of `stata-se -b` in this repo with `stata-mp -b` or `stata -b` according to the version of your stata. If you're prompted `command not found` when trying to run `stata` commands from the console, install `stata(console)` for your machine according to stata official documentation that is available online. 

One source that should work is section C.4.2 of https://www.stata.com/manuals16/gsm.pdf.


8. Setup for the whole repo is done, thanks for your patience! Now please follow the `README`s in each subdirectory to run each part of the analysis. In general, each directory will come with a `.sh` bash script which you can use `./path_to_bash_script.sh` to run scripts in that subdirectory. If you encounter the permission denied error, use `chmod +x path_to_bash_script.sh` to make it runnable.


# The Social Cost of Global Energy Consumption Due to Climate Change

The analysis in the paper proceeds in **five steps**. 

1. Historical data on energy consumption and climate are cleaned and merged, along with other covariates needed in our analysis (population and income). 
2. Econometric analysis is conducted to establish the energy-temperature empirical relationship. 
3. This relationship is used to project future impacts of climate change using an ensemble of climate models 
    * Note: this step is exceptionally computationally intensive, and while we share all the code needed for producing all outputs in ther paper, releasing all the data is currently not possible due to the large size of the data, so you will not be able to run everything.
4. These impacts are translated into empirical “damage functions” relating monetized damages to warming 
5. Damage functions are used to compute an energy-only partial social cost of carbon. 

This master readme outlines the process for each step, and each analysis step has it’s own readme and set of scripts in a subdirectory.


## Description of folders

`0_make_dataset` - Code for constructing the dataset used to estimate all models described and displayed in the paper

`1_analysis` - Code for estimating and plotting all econometric models present in the paper

`2_projection` - Code for projection the impact of climate change to 2300

`3_post_projection` - Code for producing figures for the paper

`4_misc` - Code for producing additional figures and numbers for the paper

`projection_inputs` - Files needed to run projections

<!-- 
`data` - Repository for storing data related to the `1_analysis` part of our paper. 
 -->
<!-- `figures` - Contains figures produced by codes in this analysis
 -->
<!-- 
`sters` - Contains regression output, saved as .ster files 
 -->

## Step 1 - Historical Energy Consumption and Climate Dataset Construction

Data construction is a multi-faceted process. We clean and merge data on energy consumption from the International Energy Agency's (IEA) World Energy Balances dataset, 
historical climate data from the Global Meterological Forcing Dataset (GMFD), and income and population data from the IEA.

In Part A, we construct data on final consumption of electricity and other fuels, covering 146 countries annually over the period 1971 to 2010.  
In Part B, we construct data on historical climate to align with the geographic and temporal definitions used in the energy final consumption dataset. 
In Part C, we clean data on population and income of each country-year in our data. 
In Part D, we clean and merge together data produced in each of the previous parts, and output an intermediate merged dataset.
In Part E, we prepare merged data for econometric analysis.

#### Part 1.A - Final Consumption Energy Data

* Data on energy consumption were obtained from the International Energy Agency's (IEA) World Energy Balances dataset. 
* This dataset is not public, and not provided in this repository. 
* From this raw data, we construct a country-year-product level panel dataset (where product is either electricity or other_energy). 
* Due to data quality concerns, we incorporate information on data consistency and quality from the IEA's documentation into this dataset. 
    * Details of this can be found in Appendix Section A.1, and in the [0_make_dataset/coded_issues](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/0_make_dataset/coded_issues) folder of this repo.
    * This allows us to determine which data should be dropped, to contruct a set of fixed effects and FGLS weights to help deal with data quality concerns, and assemble climate data that reflects the geographic and temporal definitions used to report energy consumption data.

#### Part 1.B - Historical Climate Data

* We take Historical Climata Data on daily average temperature and precipitation from the Global Meteorological Forcing Dataset v1 (GMFD) dataset.
* The raw GMFD data are at the 0.25 x 0.25 degree gridded resolution. We link climate and energy con-
sumption data by aggregating gridded daily temperature data to the country-year level
using a procedure detailed in Appendix A.2.4 that preserves nonlinearity in the energy
consumption-temperature relationship.
    * This step is highly computationally intensive, and the code for this step is not currently provided in this repo.
* In addition to temperature and precipitation measures, we also calculate other climate measures, such as cooling and heating degree days.
* We then clean these data, so that they match the observations present in our energy consumption data. 
    * More documentation of the cleaning process can be found in [0_make_dataset/climate](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/0_make_dataset/climate)

#### Part 1.C - Population and income data

* We obtain historical values of country-level annual income per capita from within
the International Energy Agency's World Energy Balances dataset, which in turn sources
these data from the World Bank. 
* Cleaning steps undertaken on these variables can be found in [0_make_dataset/pop_and_income](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/0_make_dataset/pop_and_income)

#### Part 1.D - Merge energy final consumption, historical climate, population and income data

* As the final part of our dataset construction, we merge all of the data together. 
* Codes used in this step can be found in [0_make_dataset/merged](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/0_make_dataset/merged)

#### Part 1.E - Clean merged data for econometric analysis 

* Motivated by [tests](https://github.com/ClimateImpactLab/energy-code-release-2020/blob/master/0_make_dataset/3_unit_root_test_and_plot.do) 
that showed that our outcome variable has a unit root, we construct first differenced versions of our 
variables for use in later econometric analysis. 
* To nonlinearly model income heterogeneity in the energy-temperature response, we construct an income spline variable. We decide on knot location based on in sample income deciles.
* Lastly to plot 3 x 3 arrays displaying the interacted model (*Methods* Equation 1), we save a dataset with information on average income and climate in different terciles of the in sample data.

### Outputs of Step 1 

* Step 1 produces datasets ready to run regressions on, and datasets used in later plotting analysis. These can be found in `yourDATA`. Specifically,
    * Part 1.D produces `<yourDATA>/DATA/regression/IEA_Merged_long_GMFD.do` -- an intermediate dataset used to construct the final analysis dataset
    * Part 1.E produces: 
        * `<yourDATA>/regression/GMFD_*_regsort.dta` -- the analysis dataset used in `Step 2`
        * `<yourDATA>/regression/break_data_*.dta` -- a dataset used for plotting 3 x 3 arrays
* Within Step 1, we produce two figures that are used in the paper:
    * ***Figure Appendix A.1***: `fig_Appendix-A1_ITA_other_fuels_time_series_regimes.pdf`
        * This figure is used to visualise the persistent shocks present in energy consumption. 
    * ***Figure Appendix A.2***: `fig_Appendix-A2_Unit_Root_Tests_p_val_hists_*.pdf`
        * These figures are used to visualise the distribution of p-values for unit root tests for within regime energy consumption.

## Step 2 - Econometric Analysis to Establish Energy-Temperature Empirical Relationship

This step implements analysis to recover the emprical relationship between temperature and energy consumption.
In this step, we take the cleaned data produced in step 1, run regressions and then plot the resulting outputs. 

We run three kinds of regressions in this section: 
1. Uninteracted global regressions (*Appendix* Equation C.1)
    * These regressions recover the global population weighted average response of energy consumption to temperature variation. 
    * Code for these regressions can be found in [1_analysis/uninteracted_regression](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/1_analysis/uninteracted_regression)
    * This code outputs: 
        * ster files for both the first stage and the FGLS regression: `FD_global_TINV_clim.ster` and `FD_FGLS_global_TINV_clim.ster`, respectively  
        * ***Appendix Figure C1***: `fig_Appendix-B1_product_overlay_TINV_clim_global.pdf`
2. Decile regressions (*Appendix* Equation C.3)
    * These regressions are run in order to understand how the sensitivity of energy consumption to climate change modulates with income levels. 
    * Code for these regressions can be found in [1_analysis/decile_regression](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/1_analysis/decile_regression)
    * This code outputs:
        * ster files for both the first stage and the FGLS regression: `FD_FGLS_income_decile_TINV_clim.ster` and `FD_FGLS_income_decile_TINV_clim.ster`, respectively
        * ***Figure 1A***: `fig_1A_product_overlay_income_decile_TINV_clim.pdf` 
3. Interacted regressions (*Appendix* Equations C.4, I.1)
    * In these regressions, we model heterogeneity in the energy-climate relationship, by interacting our models with income and climate covariates.
    * Code for these regressions can be found in [1_analysis/interacted_regression](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/1_analysis/interacted_regression)
    * This code outputs: 
        * Ster files for the first stage and the FGLS regression for the main (***TINV_clim***), the excluding imputed data (***TINV_clim_EX***), and temporal trend (***TINV_clim_lininter***) models:
            * `FD_inter_*.ster` and `FD_FGLS_inter_*.ster`
        * The following paper and appendix figures:
            * ***Figures 1C***: `fig_1C_*_interacted_TINV_clim*.pdf`
            * ***Appendix Figures I2***: `fig_Appendix-G2_*_interacted_main_model_TINV_clim_overlay_model_EX*.pdf`
            * ***Appendix Figures I3A***: `fig_Appendix-G3A_ME_time_TINV_clim_lininter_*.pdf`
            * ***Appendix Figures I3B***: `fig_Appendix-G3B_*_interacted_main_model_TINV_clim_overlay_model_lininter.pdf`

## Step 3 - Project Future Impacts of Climate Change 

In this stage of our analysis, we take the coefficients identified in Step 2, 
and use them to project future impacts on energy consumption due to climate change. 

Running code for in this step is highly computationally intensive. Therefore, we are including the inputs from our analysis that would allow a user to run this step and code to run an example projection for one climate and socileconomic scenario, but we are also providing the outputs of this step that are required for further analysis as stand-alone `.csv` files.


## Step 4 - Estimate Empirical Damage Function

In this stage, we take the projected future impacts found in step 3, and use them to construct an empirical damage function. 

1. As part of this process, we first plot a selection of the projection system outputs, in order to help us understand the spatial and temporal patterns implied by our projections. Codes for this part of our analysis are contained in [3_post_projection/1_visualise_impacts](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/3_post_projection/1_visualise_impacts). 
      * These codes output take in outputs extracted from the projection system outputs. See readme in Step 3 for more details. 
      * They produce: 
         * ***Figure 2A***: `fig_2A_*_impacts_map.png`. These maps show the spatial distribution of the projected impacts of climate change on energy consumption by fuel types, in the year 2099. We also show the response functions associated with selected impact regions, both historically and in our 2099 projection (`fig_2A_city_response_functions_2015_and_2099.pdf`). 
         * ***Figure 2B***: `fig_2B_*_consumption_compared_to_2099_impact_bars.pdf`, which shows how projected impacts for certain countries compare to their current consumption. 
         * ***Figure 2C***: `fig_2C_*_time_series.pdf`, which shows aggregated global time series of our projected impacts by fuel type and RCP, with uncertainty. 
         * ***Figure 3*** `/fig_3/.`: Figures in this folder present visualisations of monetized damages, combined across fuel types. We present a map of the damages, to highlight the spatial distribution, with visualisations of uncertainty for selected impact regions (Figure 3A). We also show an aggregated time series showing total projected damages by year as percent of global gdp (Figure 3B). The damage functions in Figure 3C are produced by code in damage function estimation.  
         * Appendix figures including 
            * ***Appendix Figure C3***: `/fig_Appendix-C3_sample_overlap_present_future/.`
            * ***Appendix Figure D1***: `fig_Extended_Data_fig_5_global_total_energy_timeseries_all-prices-rcp*.pdf`
            * ***Appendix Figure H1***: `fig_Appendix-H1_SSP3-high_rcp85-total-energy-price014-damages_by_inc_decile.pdf`
            * ***Appendix Figure I1***: `fig_Appendix-G1_Slow_adapt-global_*_timeseries_impact-pc_CCSM4-SSP3-high.pdf`
            * ***Appendix Figure I3.C***: `fig_Appendix-G3_lininter-global_*_timeseries_impact-pc_SSP3-high-rcp85.pdf`
                                                

2. We then use the global damages implied by our projections to construct damage functions. Code for estimating these damage functions is contained in [3_post_projection/2_damage_function_estimation](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/3_post_projection/2_damage_function_estimation). 
      * This code plots visualisations of our damage functions in the year 2099 for electricity, other fuels, and total energy priced using our price014 price scenario, that are shown in the paper as: 
           * ***Figure 3C***: `fig_3C_damage_function_*_2099_SSP3.pdf` 
           * The code also outputs damage function coefficients for each price scenario, and quantile regression coefficients. 
           * ***Appendix Figure E1***: `fig_Appendix-E1_total_energy_damage_function_evolution_SSP3-price014.pdf`

## Step 5 - Compute Energy-Only Partial Social Cost of Carbon

In the final step of the analysis, we use the empirically derived damage function to calculate an energy-only, partial social cost of carbon.

Code for this step is contained in [3_post_projection/3_SCC](https://github.com/ClimateImpactLab/energy-code-release-2020/tree/master/3_post_projection/3_SCC). This code takes in the damage function coefficients generated in Step 4, and outputs SCC values.  

This code outputs all SCC values used in the tables in the paper. 

We are planning to add code that computes the uncertainty of these SCC values, but that is not currently in this repo. 
