# Occurrence processing

## Rationale

This repository contains the functionality to process occurrence data and create aggregated **occurrence data cubes**. An occurrence data cube is a multi-dimensional array of values. In our context we have three dimensions, ``N = 3``:

1. taxonomic (taxon)
2. temporal (year)
3. spatial (cell code)

For each triplet the stored value represent the number of occurrences found in [GBIF](www.gbif.org/occurrence).

As the data cubes are used as input for modeling and risk assessment, we store the lowest geographic coordinate uncertainty of the occurrences assigned to a certain cell code as value as well. The occurrences are first reassigned randomly within their uncertainty circle before assigning them to the cell they belong to. If uncertainty is not available, a default 1000 m radius is assigned. Due, to the random assignment, same occurrence data could result in different data cubes if iterated. This aspect will be one of the subjects of study [here](https://github.com/trias-project/aoo-paper).

Using a tabular structure (typical of R data.frames), an occurrence data cube would look like this:

taxon | year | cell code | number of occurrences | minimal coordinate uncertainty
--- | --- | --- | --- | ---
2366634 | 2002 | 1kmE3872N3101 | 8 | 250
2382155 | 2002 | 1kmE3872N3101 | 3 | 250
2498252 | 2002 | 1kmE3872N3149 | 2 | 1000
5232437 | 2002 | 1kmE3872N3149 | 4 | 1000

where number of columns is equal to number of dimensions, ``N``, plus number of values. In our case we have three dimensions and two values.

### Occurrence data cube of alien species in Belgium

One of the main output of TrIAS project is delivering a global, unified checklist of alien species in Belgium. This checklist is published on GBIF as [_Global Register of Introduced and Invasive Species - Belgium_](https://www.gbif.org/dataset/6d9e952f-948c-4483-9807-575348147c7e). More information can be found [here](https://trias-project.github.io/unified-checklist/). In this repository we produce an occurrence data cube of all taxa of the unified checklist linked to at least whose at least one occurrence have been found in Belgium. As it is impossible to retrieve occurrences of thousands of taxa from GBIF at once (query is limited to 12000 characters), we harvest all occurrences in Belgium. This first step is documented in [/src/belgium/download.Rmd](./src/belgium/download.Rmd). At the moment of writing, the GBIF download counts almost 20 millions of occurrences. In order to handle such huge amount of data, we decided to work using a [SQLite](https://sqlite.org/index.html) file, see second step: [/src/belgium/create_db.Rmd](./src/belgium/create_db.Rmd). Handling occurrence geographic uncertainty and assigning the correspondent cell codes to occurrences is the third step, documented in [/src/belgium/assign_grid.Rmd](./src/belgium/assign_grid.Rmd). Finally, we select the alien species belonging to the [_Global Register of Introduced and Invasive Species - Belgium_](https://www.gbif.org/dataset/6d9e952f-948c-4483-9807-575348147c7e) and aggregate the occurrences by year, cell code and taxon as described in the fourth and last pipeline: [/src/belgium/aggregate.Rmd](./src/belgium/aggregate.Rmd). The resulting occurrence data cube is saved in file [`/data/output/cube_belgium.tsv`](./data/output/cube_belgium.tsv). We also produce a data cube at kingdom level, [`/data/belgium/cube_baseline.tsv`](/data/belgium/cube_baseline.tsv), which can be used as baseline to tackle the research bias effort while calculating occurrence-based indicators.

### Occurrence data cube of selected modeling species in Europe

The TrIAS project aims to assess risk of invasion by applying distribution modeling and other modeling techniques for a subset of taxa of the unified checklist. The list is saved in file [`/references/modelling_species.tsv`](./references/modelling_species.tsv). For these species we build a specific occurrence data cube which takes into account all occurrences in Europe. The region we define as Europe is described by the European Environmental Agency, see image [here](https://github.com/trias-project/occ-processing/blob/master/references/Europe.png). Similarly to the _Belgian cube_, we first download the occurrences of these species following the workflow described in [`/src/europe/download.Rmd`](./src/europe/download.Rmd). Then, we assign the occurrences randomly within their uncertainty circles in order to calculate the 1km x 1km cell they belong to, see [`/src/europe/assign_grid.Rmd`](./src/europe/assign_grid.Rmd) and finally we aggregate as described in [/src/europe/aggregate.Rmd](./src/europe/aggregate.Rmd) in order to produce the final occurrence data cube at European level.

### Occurrences of accepeted taxa, synonyms or infraspecific taxa

If a taxon has taxonomic status `ACCEPTED`  or  `DOUBTFUL`, i.e. it's not a synonym, then GBIF returns not only the occurrences linked directly to it, but also the occurrences linked to its synonyms, its infraspecific taxa and their synonyms as well.

As example, consider the species [_Reynoutria japonica Houtt._`](https://www.gbif.org/species/2889173). If you search for its occurrences worldwide you will get all the occurrences from the synonyms and infraspecies too.

taxonKey | scientificName | numberOfOccurrences | taxonRank | taxonomicStatus
--- | --- | --- | --- | ---
5652243 | Fallopia japonica f. colorans (Makino) Yonek. 41 | FORM | SYNONYM
5652241 | Fallopia japonica var. compacta (Hook.fil.) J.P.Bailey | 52 | VARIETY | SYNONYM
2889173 | Reynoutria japonica Houtt. | 39576 | SPECIES | ACCEPTED
4038356 | Reynoutria japonica var. compacta (Hook.fil.) Buchheim | 19 | VARIETY | SYNONYM
4033014 | Tiniaria japonica (Houtt.) Hedberg | 28 | SPECIES | SYNONYM
5652236 | Fallopia japonica var. uzenensis (Honda) K.Yonekura & Hiroyoshi Ohashi | 212 | VARIETY | SYNONYM
5334352 | Polygonum cuspidatum Sieb. & Zucc. | 1570 | SPECIES | SYNONYM
7291566 | Polygonum japonicum (Houttuyn) S.L.Welsh | 2 | SPECIES | SYNONYM
5334357 | Fallopia japonica (Houtt.) Ronse Decraene | 110742 | SPECIES | SYNONYM
7291912 | Reynoutria japonica var. japonica | 2199 | VARIETY | ACCEPTED
6709291 | Reynoutria compacta (Hook.fil.) Nakai | 1 | SPECIES | SYNONYM
7413860 | Reynoutria japonica var. terminalis (Honda) Kitag. | 13 | VARIETY | SYNONYM
8170870 | Reynoutria japonica var. uzenensis Honda | 32 | VARIETY | SYNONYM
7128523 | Fallopia japonica var. japonica | 1560 | VARIETY | DOUBTFUL
5651605 | Polygonum compactum Hook.fil. | 28 | SPECIES | SYNONYM
5334355 | Pleuropterus zuccarinii Small | 1 | SPECIES | SYNONYM
4038371 | Reynoutria henryi Nakai | 14 | SPECIES | SYNONYM
8361333 | Fallopia compacta (Hook.fil.) G.H.Loos & P.Keil | 24 | SPECIES | SYNONYM
7291673 | Polygonum reynoutria (Houtt.) Makino | 3 | SPECIES | SYNONYM

See https://doi.org/10.15468/dl.rej1cz for more details.

By aggregating we would loose this information, so we provide aside the final cubes, [`cube_belgium.tsv`](./data/processed/cube_belgium.tsv) and [`cube_europe.tsv`](./data/processed/cube_europe.tsv), a taxonomic compendium, [`cube_belgium_taxa.tsv`](./data/processed/cube_belgium_taxa.tsv) and [`cube_europe_taxa.tsv`](./data/processed/cube_europe_taxa.tsv) respectively. For each taxon of the cubes the compendiums include all taxa (synonyms and/or infraspecies) whose occurrences contribute to the total count. This information is saved in last column called `includes`. Here below the taxon *Reynoutria japonica Houtt.* as example:

taxonKey | scientificName | rank| taxonomicStatus | includes
--- | --- | --- | --- | ---
2889173 | Reynoutria japonica Houtt. | SPECIES | ACCEPTED | 5334357: Fallopia japonica (Houtt.) Ronse Decraene \| 7128523: Fallopia japonica var. japonica \| 8361333: Fallopia compacta (Hook.fil.) G.H.Loos & P.Keil \| 2889173: Reynoutria japonica Houtt. \| 5334352: Polygonum cuspidatum Sieb. & Zucc.

Another example from `cube_belgium_taxa.tsv`: _Aedes japonicus (Theobald, 1901)_ is an accepted species present in the Belgian cube and its occurrences include occurrences linked to the following taxa:

taxonKey | scientificName | rank| taxonomicStatus | includes
--- | --- | --- | --- | ---
1652212 | Aedes japonicus (Theobald, 1901) | SPECIES | ACCEPTED | 4519733: Ochlerotatus japonicus (Theobald, 1901) \| 7346173: Aedes japonicus japonicus \| 1652212: Aedes japonicus (Theobald, 1901)

## Repo structure

The repository structure is based on [Cookiecutter Data Science](http://drivendata.github.io/cookiecutter-data-science/). Files and directories indicated with `GENERATED` should not be edited manually.

```
├── README.md            : Description of this repository
├── LICENSE              : Repository license
├── occ-processing.Rproj : RStudio project file
├── .gitignore           : Files and directories to be ignored by git
│
├── references
│   ├── Europe.png       : Map of Europe
│   ├── modelling_species.tsv: List of species whos occurrences are queried from GBIF at European level
│
├── data
│   ├── raw              : Occurrence data as downloaded from GBIF GENERATED
│   ├── interim          : big sqlite and intermediate text file, stored locally  GENERATED
│   └── processed        : occurrence data cubes and correspondent taxonomic compendiums GENERATED
│
└── src
    ├── belgium
        ├── download.Rmd    : Script to trigger a download of occurrences in Belgium
        ├── create_db.Rmd   : Script to genereate a sqlite file and perform basic filtering
        ├── assign_grid.Rmd : Script to assign cell code to occurrences
        ├── aggregate.Rmd   : Script to aggregate data and make the Belgian data cube
    ├── europe
        ├── download.Rmd    : Script to trigger a download of occurrences in Belgium
        ├── assign_grid.Rmd : Script to perform basic filtering and assign cell code to occurrences
        ├── aggregate.Rmd   : Script to aggregate data and make the Belgian data cube    
```

## Workflow

Clone this repository to your computer and open the RStudio project file,  `occ-processing.Rproj`.

### Generate occurrence data cube for Belgium

You can generate the Belgian occurrence data cube by running the [R Markdown files](https://rmarkdown.rstudio.com/) in `src/belgium` following the order shown here below:

1. `download.Rmd`: trigger a GBIF download and add it to the list of triggered downloads
2. `create_db.Rmd`: create a sqlite database and perform basic data cleaning
3. `assign_grid.Rmd`: assign geographic cell code to occurrence data
4. `aggregate.Rmd`: aggregate occurrences per taxon, year and cell code to create the _Belgian occurrence data cube_, its taxonomic compendium and data cube at kingdom level (baseline)

The output files, `cube_belgium.tsv`, `cube_belgium_taxa.tsv` and `cube_baseline.tsv` are automatically generated in  folder `/data/output/`.

### Generate occurrence data cube for Europe

At European level we are interested in occurrences of a list of taxa, which will be used for modelling and risk assessment. This list is maintained in file `modelling_species.tsv` in folder `references`. 

You can generate the European occurrence data cube by running the [R Markdown files](https://rmarkdown.rstudio.com/) in `src/europe` following the order shown here below:

1. `download.Rmd`: trigger a GBIF download and adding it to the list of triggered downloads
3. `assign_grid.Rmd`: assign geographic cell code to occurrence data
4. `aggregate.Rmd`: aggregate occurrences per taxa, year and cell code, the European _occurrence data cube_.

The output files, `cube_europe.tsv` and `cube_europe_taxa.tsv` are automatically generated in  folder `/data/output/`.

## Contributors

[List of contributors](https://github.com/trias-project/unified-checklist/contributors)

## License

[MIT License](https://github.com/trias-project/unified-checklist/blob/master/LICENSE) for the code and documentation in this repository.
