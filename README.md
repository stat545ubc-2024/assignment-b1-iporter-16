## calc_relative_abundance()

In microbiology we frequently work with **phyloseq** objects generated in R using the [phyloseq package](https://joey711.github.io/phyloseq/) by joey711. These objects are complex, and often simplified into dataframes using the phyloseq::psmelt() function.

In this project, I define a function **calc_relative_abundance()** which takes in a phyloseq object. This object is passed through **psmelt()** to convert it into a dataframe, after which relative abundances are calculated for each taxonomic unit within a sample.

The expected output is a dataframe as produced by psmelt(), with an additional column called Relative_Abundance which contains these calculations. This can be used downstream for visualisation of the microbiome data in base R or ggplot() calls.

### Requirements

This function requires the phyloseq and tidyverse packages. 

The function tests additionally require the testthat package.

[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/s4oIzs8K)
