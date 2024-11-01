Assignment_B1
================
IP
2024-10-22

### Background: abundance data

In my field I work a lot with bacterial abundance data, which is
generated as a datatype called a **phyloseq object**. This object is
complex but with a highly conserved structure and functionality, and can
be converted to a dataframe using the phyloseq::psmelt() function.

The resulting dataframe should always contain columns called “Sample”
and “Abundance”, alongside columns containing more taxonomic information
and metadata (depending on what was originally included by the
researcher). Frequently, I want to calculate the *relative* abundance of
each bacteria within its sample, which means dividing that bacteria’s
abundance by the sum of all abundances within that Sample.

Thus I can define a function, calc_relative_abunance(), which takes in a
phyloseq object and returns a dataframe containing the psmelt’ed
phyloseq object, with relative abundance calculated. As this is a
dataframe, this is then very easily used for dplyr manipulation and
graphing in ggplot.

The function first checks that my input is a phyloseq object. It then
uses psmelt() to convert the phyloseq object into a dataframe, and
checks the resulting dataframe contains columns “Sample” and “Abundance”
(which the psmelt() function should produce by default).

Once this is confirmed, we filter our data to remove NA Abundance values
before grouping by sample and using mutate() to calculate relative
abundance.

### Defining the calc_relative_abunance() function

``` r
# Relative abundances in phyloseq object
# 
# Starting from a phyloseq object, this applies the psmelt function to convert data into an R dataframe and subsequently calculate relative abundance of each taxonomic unit within its sample. 

#' Phyloseq relative abundance
#'
#' @param ps a whole phyloseq object of taxonomic produced by the phyloseq::phyloseq() function.
#'        ps is named thus as this is common shorthand for a phyloseq object in the field.
#'
#' @return an R dataframe of microbiome composition data containing sample ID and all other 
#' provided metadata, taxonomic information, abundance values, and relative abundance values. 
#' Abundance values of NA are dropped.
#'
#' @examples
#' calc_relative_abunance(esophagus)
#' calc_relative_abunance(GlobalPatterns)
#' calc_relative_abunance(soilrep)
calc_relative_abunance <- function(ps) {
  
  if(class(ps)!="phyloseq") {
     stop("Input is not a phyloseq object, but rather " , class(ps))}
  
  data <- ps %>%
    psmelt() #convert phyloseq object to an R dataframe
  
  if (!"Sample" %in% colnames(data)) {
    stop("The 'Sample' column is missing from the data.")}
  if (!"Abundance" %in% colnames(data)) {
    stop("The 'Abundance' column is missing from the data.")}
  
   data <- data %>%  
    filter(!is.na(Abundance)) %>%
    group_by(Sample) %>% 
    mutate(Relative_Abundance = Abundance / sum(Abundance)) %>%
    ungroup()
  
  return(data)
}
```

### Examples

We are using the soilrep phyloseq object that comes with the phyloseq
package. This is passed through our calc_relative_abunance() function,
the results of which are piped through head() for easy visualisation as
the original dataset is very large.

We can see that a column has been created called “Relative_Abundance”.

``` r
data(soilrep)
calc_relative_abunance(soilrep) %>% head()
```

    ## Warning in psmelt(.): The sample variables: 
    ## Sample
    ##  have been renamed to: 
    ## sample_Sample
    ## to avoid conflicts with special phyloseq plot attribute names.

    ## # A tibble: 6 × 8
    ##   Sample OTU       Abundance Treatment warmed clipped sample_Sample
    ##   <chr>  <chr>         <int> <fct>     <fct>  <fct>   <fct>        
    ## 1 a_C081 OTU_R246         96 UC        no     yes     3CC          
    ## 2 a_C116 OTU_R4862        92 UC        no     yes     3CC          
    ## 3 a_C116 OTU_R246         67 UC        no     yes     3CC          
    ## 4 a_C074 OTU_R264         66 UU        no     no      2UC          
    ## 5 a_C074 OTU_R277         63 UU        no     no      2UC          
    ## 6 a_C082 OTU_R246         51 UC        no     yes     1CC          
    ## # ℹ 1 more variable: Relative_Abundance <dbl>

In the same way, we can use the GlobalPatterns phyloseq object that also
comes with the phyloseq package. This phyloseq object contained much
more metadata (eg. sample types, barcodes) and comprehensive taxonomic
information (kingdom, phyla, etc.) which is reflected in more columns.
However, regardless of the composition of the phyloseq object we still
produce the Relative Abundance values!

This demonstrates that our function can take in a phyloseq object of
varying complexity and reliably perform the calculation we desire.

``` r
data(GlobalPatterns)
calc_relative_abunance(GlobalPatterns) %>% head()
```

    ## # A tibble: 6 × 18
    ##   OTU    Sample Abundance X.SampleID Primer Final_Barcode Barcode_truncated_pl…¹
    ##   <chr>  <chr>      <dbl> <fct>      <fct>  <fct>         <fct>                 
    ## 1 549656 AQC4cm   1177685 AQC4cm     ILBC_… ACAGCT        AGCTGT                
    ## 2 279599 LMEpi…    914209 LMEpi24M   ILBC_… ACACTG        CAGTGT                
    ## 3 549656 AQC7cm    711043 AQC7cm     ILBC_… ACAGTG        CACTGT                
    ## 4 549656 AQC1cm    554198 AQC1cm     ILBC_… ACAGCA        TGCTGT                
    ## 5 360229 M31To…    540850 M31Tong    ILBC_… ACACGA        TCGTGT                
    ## 6 331820 M11Fc…    452219 M11Fcsw    ILBC_… AAGCTG        CAGCTT                
    ## # ℹ abbreviated name: ¹​Barcode_truncated_plus_T
    ## # ℹ 11 more variables: Barcode_full_length <fct>, SampleType <fct>,
    ## #   Description <fct>, Kingdom <chr>, Phylum <chr>, Class <chr>, Order <chr>,
    ## #   Family <chr>, Genus <chr>, Species <chr>, Relative_Abundance <dbl>

In comparison, calling the function with a non-phyloseq input calls our
first error message.

``` r
calc_relative_abunance(1:5) 
```

    ## Error in calc_relative_abunance(1:5): Input is not a phyloseq object, but rather integer

``` r
calc_relative_abunance(c("This is a character!")) 
```

    ## Error in calc_relative_abunance(c("This is a character!")): Input is not a phyloseq object, but rather character

### Testing the calc_relative_abunance() function

#### Variable specificity

First let’s demonstrate that the code calls an error if we don’t use a
phyloseq object. I have defined a simple dataframe, called df, as part
of this test. We expect that any non-phyloseq input returns an error,
which is what we see below!

``` r
df <- data_frame("Num"=1:5,"Let"=c("a","b","c","d","e"))
```

    ## Warning: `data_frame()` was deprecated in tibble 1.1.0.
    ## ℹ Please use `tibble()` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

``` r
test_that("Testing variable class specificity", {
  expect_error(calc_relative_abunance(2:10))
  expect_error(calc_relative_abunance("This is a character!"))
  expect_error(calc_relative_abunance(df))
  })
```

    ## Test passed 🥇

#### Correct output

This is a bit harder, as it requires me to put in a phyloseq object for
which I already know the desired Relative Abundances. I’ll do that in
the first chunk, bear with!

``` r
# Create OTU table (matrix of abundance counts)
otu_matrix <- matrix(c(10, 1, 0, 2, 10, 22), nrow=3, byrow=TRUE)
rownames(otu_matrix) <- c("OTU1", "OTU2", "OTU3")
colnames(otu_matrix) <- c("Sample1", "Sample2")
otu_table <- otu_table(otu_matrix, taxa_are_rows=TRUE)

# Create sample data (sample metadata, ie. samples and treatments)
sample_data_frame <- data.frame(SampleID = c("Sample1", "Sample2"),
                                Condition = c("Control", "Treated"))
rownames(sample_data_frame) <- sample_data_frame$SampleID
sample_data <- sample_data(sample_data_frame)

# Create taxonomy table (matrix of taxonomic classification)
taxonomy_matrix <- matrix(c("Bacteria", "Firmicutes", "Clostridia", 
                            "Bacteria", "Proteobacteria", "Gammaproteobacteria",
                            "Bacteria", "Actinobacteria", "Actinobacteria"), 
                          nrow=3, byrow=TRUE)
rownames(taxonomy_matrix) <- c("OTU1", "OTU2", "OTU3")
colnames(taxonomy_matrix) <- c("Domain", "Phylum", "Class")
tax_table <- tax_table(taxonomy_matrix)

# Combine all components into a phyloseq object
sample_physeq <- phyloseq(otu_table, sample_data, tax_table)

# Check structure is corrects
sample_physeq
```

    ## phyloseq-class experiment-level object
    ## otu_table()   OTU Table:         [ 3 taxa and 2 samples ]
    ## sample_data() Sample Data:       [ 2 samples by 2 sample variables ]
    ## tax_table()   Taxonomy Table:    [ 3 taxa by 3 taxonomic ranks ]

In this sample_physeq we have three OTUs (basically individual taxa) and
two Samples. I have set the Abundance values as below, so that we know
what the relative abundances should be. I also know psmelt() arranges
rows by descending abundance, so this is taken into account when
designing our expected test outputs. I have included alongside the
expected relative abundance, both the expected abundance and OTUs to
make sure that our values are also assigned to the correct OTUs.

| Sample   | OTU  | Abundance | Relative Abundance |
|:---------|:----:|:----------|--------------------|
| Sample 1 | OTU1 | 10        | 10/20 = 0.5        |
| Sample 1 | OTU2 | 0         | 0                  |
| Sample 1 | OTU3 | 10        | 10/20 = 0.5        |
| Sample 2 | OTU1 | 1         | 1/25 = 0.04        |
| Sample 2 | OTU2 | 2         | 2/25 = 0.08        |
| Sample 2 | OTU3 | 22        | 22/25 = 0.88       |

We can now run tests on our function to ensure these values are met!

``` r
expected_rel_abundance <- c(0.88,0.5,0.5,0.08,0.04,0)
expected_abundance <- c(22,10,10,2,1,0)
expected_OTU <- c("OTU3","OTU1","OTU3","OTU2","OTU1","OTU2")
test_that("Testing output accuracy", {
  expect_equal(calc_relative_abunance(sample_physeq)$Relative_Abundance,
               expected_rel_abundance)
  expect_equal(calc_relative_abunance(sample_physeq)$Abundance,
               expected_abundance)
  expect_equal(calc_relative_abunance(sample_physeq)$OTU,
               expected_OTU)
  })
```

    ## Test passed 🌈
