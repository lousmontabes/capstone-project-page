## Geolocalisation of DNA chimpanzee samples using machine learning

### 1. Background
As humans we have always been fascinated by our closest related species, the chimpanzees. At the same time, human activities have led to a drastic decline in the population size of this species, mainly caused by the illegal wildlife trade, habitat destruction, poaching for local consumption and human linked disease outbreaks, among many others. In this regard, genetic information could be useful to infer the populations of origin of confiscated individuals from illegal trade, detect poaching hotspots, and guide repatriation planning. Taking advantage of the most complete chimpanzee genetic map ever generated, here we implement a machine-learning geolocalisation approach, assessing its precision and robustness in determining the origin of confiscated chimpanzees.

<p align="center"> Summary image of the project</p>

<p align="center">
<img width="400" height="400" src="https://ars.els-cdn.com/content/image/1-s2.0-S2666979X22000623-fx1_lrg.jpg">
</p>
<p align="center"> <sub>Left panel: Dataset generated within the PanAf project. Right top panel: Study of chimpanzee history through population genomics.<br/> Right bottom panel: Geolocalisation of confiscated chimpanzees. Graphical abstract from Fontsere et al. 2022 designed by Marina Alvarez-Estape.</sub></p>

### 2. Objectives
To take advantage of an extensive dataset on genomic variation in georeferenced chimpanzees to develop a machine-learning-based tool for the geolocation of chimpanzee samples which can be further implemented in other species.

### 3. Overview of the data
The dataset used for this project was retrieved from [Fontsere et al. 2022](https://www.sciencedirect.com/science/article/pii/S2666979X22000623) and consists of a catalog of genomic diversity obtained by capturing chromosome 21 from 828 non-invasively collected samples at 48 sampling sites across Africa. 

##### Geographic distribution of chimpanzee subspecies and PanAf sampling locations

<p align="center">
<img src="https://user-images.githubusercontent.com/68989675/176246832-35da20f9-e285-4cf1-ab46-0118a27753e1.png">
</p>

<sub>The western chimpanzee is shown in blue, Nigeria-Cameroon in pink, central in green, and eastern in orange. The size of the dots represents the number of sequenced samples (n = 828) and color intensity represented the amount of chimpanzee genetic data generated (mega-base pairs of mapped sequence) from each sampling site. Source: Fontsere et al. 2022.</sub>

##### Genomic data

This dataset contains the genomic information along chromosome 21 derived from fecal samples. Due to the nature of non-invasive samples (feces and hair) retrieving good quality genomic information is challenging. For each position in the chromosome, two nucleotides (A, T, C or G) are present, each of which deriving from one of the parents. Genomic data is mapped and referenced to the genomic information of the so-called reference genome. A reference genome is a sequence of DNA used as the reference or gold-standard of genomic information of a certain species. These reference genomes only contain one copy of each chromosome (as opposed to the two copies a biological sample would have) and so, only one nucleotide is chosen to be the representative for each position (eg. position 1 is A, position 2 is T, position 3 is G, position 4 is T…). The genomic information of the chimpanzee sample is stored in relation to the one in the reference. To simplify, in our dataset if the sample has the same information as the one in the reference genome, it is denoted with a *0*, if it has one of the two nucleotides like the one in the reference and the other is different, it would be denoted with a *1*. Finally, if the two nucleotides are different from the one in the reference, this position would be denoted with a *2* in our dataset. Finally, positions for which no genomic information is available are denoted with a *9*. Since the data used in this project comes from feces, it is low quality, and so for many samples and positions there will not be genomic information, and so the value will be *9*. Each of these samples is georeferenced, which means that the exact coordinates of the place of collection of the feces are known. Because of this, for each sample, we have information about its GPS coordinates, sampling site, country or origin, and also the chimpanzee subspecies it belongs.

The following image summarises how genomic data is interpreted and has been stored in this project.
<p align="center">
<img src=https://user-images.githubusercontent.com/68989675/176548399-4794e9c1-7efc-4eeb-8960-87a5bc65394a.png>
</p>

##### More information

To know more about what it means to work with fecal samples, please have a look at the following [video](https://www.youtube.com/watch?v=Fv_LzqCeFoI&t=1457s) (in Catalan). It explains the methodology and main aim of geoposition (in the context of Gorillas instead of Chimpanzees) and it is less than 4 minutes-long.

### 4. Data processing

In order for our data to be suitable for our purposes, we had to first pre-process it to fit our needs. There were several obstacles to overcome:

#### Non-informative positions
Several columns in our data contained the same value accross all samples: either the information was missing or the genotype was the same in each row. Since these columns provided no information, we proceeded to remove them.

#### Missing data
##### Removing samples and positions with too many NaNs
Even though non-informative positions have been filtered out, there are still some positions and samples which remain non-informative: these are positions with a high number of NaN for most samples and also samples which have a high number of NaN for most positions. Since we are facing a problem of high degree of missing values, it is important to consider those samples and positions that do not add a lot of information for the modelling.

Leveraging the number of samples and positions that would result from this filtering, we ended up choosing to remove positions (columns in the dataframe) with more than 80% missingness (defined as those that did not have information for 80% or more of the samples, *rows*). We also filtered out the samples (rows in the dataframe) with more than 70% missingness (with NaN for 70% of their positions, *columns*).

##### Imputing missing values
For the the data that was still missing after our initial filtering, since we didn't want it to skew our models in any way, we opted to imput it with the global mean. 

### 5. Modelling
We used our data to train supervised models on three different variables: two which where classification problems: the **subspecies** of the chimpanzee the sample was taken from and **sampling site** it was obtained from; and one regression problem: the **approximate coordinates** of the sample's origin.

#### 5.1. Classification of subspecies

Considering the fact that we are working with genetic data of chimpanzees distributed accross a very large geographical range encompassing most of west and central Africa, the most obvious distinction we can hope to find in our data is the **subspecies** of chimpanzee the DNA corresponds to.

We can identify four distinct subspecies:

- Central
- Eastern
- Nigeria-Cameroon
- Western

We used the following models in this prediction:

1. Random Forest
2. Nearest Neighbor
3. Support Vector Machine
4. Naive Bayes

The code used in this section can be found in *Notebooks/Geolocalisation - Notebook 2.1. Modelling genomic data - SUBSPECIES.ipynb*

The modelling for this data is very straightforward, since we will use, in all cases, the default parameters for each model. This is because early on we found that we were obtaining very good results without the need to adjust parameters.

The results of predicting at subspecies level were very satisfactory, achieving perfect classifications accross three out of the four classifiers used.

![image](https://user-images.githubusercontent.com/7366498/176046539-025d8a9c-1e1b-424d-ba24-cb15a324cfd5.png)![image](https://user-images.githubusercontent.com/7366498/176046744-7c5ee3b2-226b-495f-b07a-4a07fd9e9703.png)
![image](https://user-images.githubusercontent.com/7366498/176046757-a310e948-c443-4ef2-9341-ad522b5a3995.png)![image](https://user-images.githubusercontent.com/7366498/176046763-045e8d5f-e5ea-4f40-9a1d-c475c71f6d4b.png)
_Confusion matrices of each model's predictions of subspecies of the test set_

We can see that **Categorical Naive Bayes** is the only model that didn't achieve a perfect f1-score.

#### 5.2. Classification of sampling site

Having obtained good results at subspecies level, we will proceed to attempt to classify our samples at a higher resolution: predicting the sampling site the sample was obtained from.

We have a total of 36 named sampling sites that we will serve as our labels.
Based on the performance of the different classifiers in Section 5.1, the classifiers that have been evaluated are:

1. Random Forest
2. Nearest Neighbor
3. Support Vector Machine

The code used in this section can be found in *Notebooks/Geolocalisation - Notebook 2.2. Modelling genomic data - SAMPLINGSITE.ipynb*

![image](https://user-images.githubusercontent.com/7366498/176051160-dca7b668-797c-45f7-8dbd-f55ff3a3baf3.png)
![image](https://user-images.githubusercontent.com/7366498/176051196-2b3a9fef-3e72-4a60-83e0-48d7a20d9bda.png)
![image](https://user-images.githubusercontent.com/7366498/176051226-efdcd42a-b847-4d0f-8269-5085f0b5c0c5.png)
_Confusion matrices of each model's predictions of sampling sites of the test set_

#### 5.3. Geographic prediction

As we have the information regarding the coordinates of each sample, we used it to create regression models and predict the relative area where a sample was taken from. We used the following models:
1. Linear Regression
2. K Nearest Neighbors Regression

##### KNearest Neighbors Regression
Based on the k-nearerst neighbors algorithm.
The target is predicted by performing a local interpolation of the nearest neighbors in the training set

Can be used in cases where the labels are continuous, the label assigned to each query point is computed based on the mean of the labels of its nearest neighbors

With the weights keywords you can swap between uniform weight or give more to the closer points, the closer distance approach gives slightly better results.
It uses different algorithms but it can automatically chose one with the information that is given.
You can change the number of nearest neighbors, gave similar results but the default (5) and 4 had the best values.
The leaf_size makes no changes.
The power parameter is better on default (euclidean distance).

This algorythm gave slightly better results than the linear regression, depending on the parameters it gives a mean error in prediction between 1.30 and 1.315 depending on the algorithm and a r2-score between 0.973 and 0.978.

![image](https://user-images.githubusercontent.com/25895127/176486370-dec18a36-ce50-4148-9248-80c4f685ed51.png)
![image](https://user-images.githubusercontent.com/25895127/176486495-0f7674b5-b752-4f63-8f78-614b618fc819.png)


The code used in this section can be found in *Notebooks/Geolocalisation - Notebook 2.3. Modelling genomic data - Geographical prediction
.ipynb*

#### 6. Limitations of the study and future advances
+ Require training vs test datasets to be equality represented and consider removing sampling sites with a small number of samples.
+ Stratify the perfomance metrics according to subspecies in order to account for differences in the distribution ranges of each subspecies.
+ Use lower quality samples in order to assess the performance of the classifiers.
+ Assess alternative imputation methods, eg. linkage disequilibrium.
+ Develop a metric to assess the perfomance of the predictors based on distance to true origin rather than success in prediction.

#### 7. Conclusions
+ It is possible to use classifiers to accurately predict the subspecies of a chimpanzee using genomic information.
+ Using a support-vector-machine-based classifier is a good alternative to geolocalise chimpanzee samples with considerably high accuracy (~0.8 f1-score).
