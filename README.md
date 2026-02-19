# UCL-Shimadzu Pipeline Documentation

Documentation for using the UCL-Shimadzu pipeline for analysing output files from Shimadzu LabNIRS. All analysis scripts are written in MatLab and can be run independently. This document is currently maintained by **Jia Zhang - <jia.zhang.25@ucl.ac.uk>**

## Installation

> - To use the analysis script, simply download the specific ```.m``` script or download the entire repo and unzip it. Your ```working directory``` should be the folder containing your analysis scripts.
> - The analysis requires the [HOMER2](https://homer-fnirs.org/) analysis suite. The software is currently **deprecated and no longer maintained**. [Click here](https://www.nitrc.org/projects/homer2) for download.
> - [SPM12](https://www.fil.ion.ucl.ac.uk/spm/software/spm12/) and [SPM for fNIRS](https://www.nitrc.org/projects/spm_fnirs/) are also required for analysis and preprocessing. 
> - ***A ToolBox repo containing all the analysis packages mentioned above can be found [here](https://github.com/CogNIRS/Toolboxes).***

#### Configuring Matlab

> Apart from adding all of the Scripts in this pipeline to the path of MatLab several toolboxes need to be installed through the MatLab **Add-On Manager**. 
> - [Wavelet Toolbox](https://uk.mathworks.com/products/wavelet.html?s_tid=AO_PR_info)
> - [Statistics and Machine Learning Toolbox ](https://uk.mathworks.com/products/statistics.html?s_tid=AO_PR_info) 
> - [Signal Processing Toolbox ](https://uk.mathworks.com/products/signal.html?s_tid=AO_PR_info)


## Pipeline Overview

![PipelineFlowChart](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/flowchart-general.png)

The main analysis component is based on the [HOMER2 toolbox](https://homer-fnirs.org/)

### [```Shimadzu2Homer.m```](https://github.com/CogNIRS/GeneralPipeline/blob/main/Shimadzu2Homer.m)
The output format of LabNIRS is ```.txt```. It must be transformed into ```.nirs``` format to be analysed. See [here](#shimadzu2homerm-1) for more detail.

### [```Run_Preprocessing.m```](https://github.com/CogNIRS/GeneralPipeline/blob/main/Run_Preprocessing.m)
Preprocess the data and produce **PSD plots** which can be used to quickly evaluate the data quality of each channel. This can be used to decide which channels to exlude from further analysis. See [here](#run_preprocessingm-1) for more detail.

### [```Run_GLM.m```](https://github.com/CogNIRS/GeneralPipeline/blob/main/Run_GLM.m)
GLM analysis up until the computation of the design matrix and the esitmation of the *beta-values* for each regressor. **Note files and folders have to be prepared in advance to use this script**. See [here](#run_glmm-1) for more detail.

### [```Run_Contrasts.m```](https://github.com/CogNIRS/GeneralPipeline/blob/main/Run_Contrasts.m)
Creates ```xCon.mat``` for experiments with multiple conditions. **Note files and folders have to be prepared in advance to use this script**. See [here](#xconmat) for more detail.

## Scripts Instructions

All the scripts are written in MatLab, so a version of MatLab must be installed to edit and run the scripts. 

### ```Shimadzu2Homer.m```
Since LabNIRS outputs a ```.txt``` file, it is important to convert the file format to ```.nirs``` allowing existing toolboxes to process the data recorded. While the pipline uses HOMER2, the ```.nirs``` format is compatible with a variety of libraries [see here](); this can be used in other pipelines.

#### Input Variables

Input variables that need to be changed are between ```line 11``` to ```line 38```. The rest of the Script does not need to be editted.
> - ```line 11```: Change the ```AnalysisDir``` to the path (folder) containing your scriptfiles. If you have extracte the folder from a direct zip download, your path should look something like: ```'C:\GeneralPipeline-main\GeneralPipeline-main'```.
> - ```line 15```: Change ```nCh``` to the total number of **channels** of your setup.
> - ```line 17```: Edit ```SD.Lamda``` accordingly, this usually remain as ```[780, 805, 830]```.
> - ```line 18```: Change ```SD.nSrcs``` to match the number of **sources** or **transmitters**. 
> - ```line 19```: Change ```SD.nDets`` to the number of **detectors** or **receivers**

#### ```SD.SrcPos``` and ```SD.DetPos```

![HomerGrid](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/HomerGrid.png)

It should be noted that this is completely arbitrary, each source/ detector can be labelled however you want. This is simply to indicate the relative positions of each optode in the next step.

> The coordinates present the *X, Y, Z* coordinates on your setup in **cm**. The data structures of ```SD.SrcPos``` and ```SD.DetPos``` is: ```=[Optode1; Optode2; Optode3 ...; OptodeN]```
>
>You can choose to use the above as a reference. The first source  - **S1**'s coordiante would be ```0 3 0```; the second source - **S2**'s coordinate will be ```3 0 0```. Therefore, ```line 22``` should look like: ```SD.SrcPos=[0 3 0; 3 0 0; ...; 21 0 0]``` in the setup above. Similarly, ```line 23``` should look like ```SD.SetPos=[0 0 0; 3 3 0; ...; 21 3 0]```. The *Z coordinate* should remain zero unless your setup requires it.


#### Channels

>The ```Channels``` data structure is: ```=[Channel1Source Channel1Detector; Channel2Source Channel2Detector; ...; ChannelNSource ChannelNDetector]``` in the order of your optode configuration.
>
>In the example from above, *Channel 1* is made between *S1* and *D1* so the first element in ```Channels``` should be ```[1 1; ...]```, corresponding to the first element in ```SD.SrcPos``` and the fist element in ```SD.DetPos```.

A channel map can be found in the **Logical CH** tab in the **Channel Editor** of the Shimadzu fNIRS software. Each channel should correspond to the configuration shown here.

![ChannelMap](images\ChannelMap.png)

#### Running the Script

With all setups complete, simply click on run in MatLab or type ```Shimadzu2Homer``` in the console and press ```Enter```. A file selection window up popup, simply select the ```.txt``` file from *LabNIRS* and press ```Open```.

![FileSelect](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/fileselect.png)

After running the script, a ```.nirs``` file will be written in the same directory as the ```.txt``` file. This can then be used for preprocessing or other pipelines. 

### ```Run_Preprocessing.m```

This script carries out preprocessing on the data collected. This will produce PSD plots that can be used to identify channels that are excluded in the final analysis. This script requires the [*Signal Processing Toolbox*](https://uk.mathworks.com/products/signal.html) which can be installed using the *MatLab Add-Ons* tool. After running the script, the pre-processed data will automatically overwrite the original ```.nirs``` file.

>Other than changing ```Homerdir``` on ```line 17``` and ```AnalysisDir``` on ```line 18``` to the corredponding folder paths. The input variables should not need to be changed for standard preporcessing. *An official guide for the input variables is still in progress*.
>
>The script will produce several figures containing visualisation of data quality for each channel. There will be 3 graphs for each channel: **Intensity**, **PSD**, and **OD**. For mannually assessing channel quality, **Intensity** and **PSD** will be used. 

#### Output

At this stage, the preprocessing output can be checked by looking at the MatLab workspace. These 7 data structure should be produced:

![structs](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/fields-preproc.png)

The ```Data``` structure being the most important one, containing the HbO, HbR, and CBSI signals.

![data](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/data-preproc.png)

> If the output does not contain any of the above data structures, it is possible that the toolboxes have not been configured correctly.

#### Intensity

Channels are split into 3 grades:. *Good (2)*, *Acceptable (1)* and *Exclude (0)*. As illustrated below, this assessment is based on 3 parameters: 1). Whether the data lines are well-separated 2). Whether the data lines are thin. 3). Whether the range of intensity is high ( usually between 0.5 - 2.5). 

>These are examples of Intensity graphs that are rated *Good (2)*
>
>![GoodIntensity](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/GoodIntensity.png)


>These are examples of Intensity graphs that are rated *Acceptable (1)*
>
>![AcceptIntensity](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/AcceptIntensity.png)

>These are examples of Intensity graphs that are rated *Exclude (0)*
>
>![BadIntensity](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/BadIntensity.png)

#### PSD

Channels are either *Accepted (2)* or *Excluded (0)*. As illustrated, this assessment is based on whether the graph shows a characterisitc *c.a.* 1 Hz peak for *Heart Beat*. 

>These are examples of Intensity graphs that are *Included (2)*
>
>![GoodPSD](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/GoodPSD.png)

>These are examples of Intensity graphs that are *Excluded (0)*
>
>![GoodPSD](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/BadPSD.png)

#### Channel Assessment Spreadsheet

There is no official way to record channel qualities, but the suggestion is to use the numbering system mentioned above to create a spread sheet. Below is an example spreadsheet for a **60-Channel** setup for **6 participants**.

>![ChannelChart](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/ChannelInclusionChart.png)

With this, an initial assessment of the data quality can be visualised. In the above example, participants 07 and 08 did not produce data of good quality.

#### Channel Exclusion File

Create a ```Channels2Exlude.mat``` file using your own script. This should be a ```Number of Subjects x 2 ``` matlab cell variable with each row named after the participant (e.g. ```'P01'```, ```'P02'``` etc). If no channels are excluded for a participant, leave the row blank. See below for an example.

![channels2exlcude](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/channels2exclude.png)

### ```Run_GLM.m```

Runs GLM analysis up until the design matrix and the estimation of the beta values for each regressor. Follow the steps below to set up the required files and folders to run the script. 

#### Folder Organisation

```ROOT``` contains the Pipeline Scripts. Create a new folder called ```Data``` and create a folder for each partiicpant; this will contain the files that need to be created by following the steps below. 

``` 
ROOT
├─── Data
|   ├─── P01
|   |   ├─── Digitizer
|   |   |   |   ch_config.txt
|   |   |   |   P01_Optodes.csv
|   |   |   |   p01_Reference.csv
|   |   |   NIRS.mat
|   |   |   P01_onsets.mat
|   |   |   P01_Regressors.mat
|   ├─── P02
|
|   · · ·
|
|   ├─── P#
|   |   xCon.mat
|   Channels2Exclude.mat

```

> - The ```Digitizer```  folder includes the Polhemus data for reference coordinates; see [here](#digitizer-files) for how to create them.
> - ```P#_onsets.mat``` contains the onsets of the experimental conditions; see [here](#onsets-file) for how to create the file.
> - ```P#_Regressors.mat``` contains the additional regressors to include in the design matrix; see [here](#regressors-file) for how to create the file.
>- ```NIRS.mat``` will be created by running ```Run_GLM.m```; see [here](#nirs-file) for more detail.
> - ```xCon.mat``` is created by running the ```CreateContrasts.m``` Script; see [here](#xconmat) for more detail.
> - ```Channels2Exclude.mat``` contains wich channels to exclude from the analysis. This is the result of the reprocessing above; see [here](#channels2excludemat).

#### Digitizer Files

Manually create the 2 ```.csv``` and ```.txt``` files with the **same format** as the sample data; these can be found in ```TestData > P# > Digitizer```. While this can be completed using a MATLAB script, the script in question will be different for different ways of digitization.

- ```P#_Reference.csv```: contains at least 4 reference points. The names of each reference should be shared across different digitization processes. Make sure that the headings in the csv file is **exactly as follows**:

> |Reference|X|Y|Z| 
> |:---:|:---:|:---:|:---:|
> |NzHS|40.399|12.738|-3.115|
> |IzHS|19.848|14.135|-3.76|
> |ARHS|30.193|10.975|3.794|
> |ALHS|30.726|10.943|-10.558|
> |Fp1HS||||
> ...
> |F8HS||||
> |CzHS|29.585|24.242|-4.053|
> ...

- ```P#_Optodes.csv```: contains the coordiantes of sources and detectors. These should correspond to the channel map in ***LabNIRS***, and in the same order of the digitization process. Make sure that the headings in the CSV file is **exactly as follows**:

> |Optode|X|Y|Z| 
> |:---:|:---:|:---:|:---:|
> |D1|35.183|18.887|2.435|
> |S1|37.654|19.178|0.77|
> |D2|39.138|19.301|-1.66|
> |S2|39.094|19.199|-4.477|
> ...
> |D13|35.901|15.823|-9.425|
> |S13|38.548|16.035|-7.671|

- ```Ch_config.txt``` defines the optodes corresponding to each channel. Make sure that the headings in the CSV file is **exactly as follows**:

> |Ch|Source|Detector|
> |:---:|:---:|:---:|
> |1|4|1|
> |2|1|1|
> |3|4|4|
> |4|1|4|
> ...
> |16|3|6|



#### Onsets File
Create a ```P#_onsets.mat``` file for each participant. THis will contain **4** variables that details the experimental design: `onsets`, `durations`, `names`, and `Params`. The following examples will decribe a block-design experiment consisting of **5** blocks with **2** conditions. The first condition is presented for **15s** and the second condition is presented for **30s** followed by a **10s** rest. Depending on the analysis, the *rest* block could be included or omitted; the example below **includes** the rest block.

- ```onesets```: a ```cell``` variable of size ```[1 x number of conditions]```. ```onsets{1,1}``` contains, in seconds, the presentation of each condition. 

> |1| 
> |:---:|
> |0, 55, 110, 165, 220|
> |15, 70, 125, 180, 235|
> |45, 100, 155, 210, 265|
  
- ```durations```: a ```cell``` variable of size ```[1 x number of conditions]```. This will detail the duration of each block.

> |1|
> |:---:|
> |15, 15, 15, 15, 15|
> |30, 30, 30, 30, 30|
> |10, 10, 10, 10, 10|

- ```names```: a ```cell``` variable of size ```[1 x number of conditions]```. This will contain the arbitary names of each condition.

> |1|
> |:---:|
> |condition 1|
> |condition 2|
> |rest|

- ```Params```: a ```structure``` variable of size ```[1x number of conditions]``` with **3** ```fields``` that includes information about parametric or non-parametric regressors. This variable could be initialised as:

``` m
Params.Pname = 'none'
Params.h = 0
Params.P = zeros(1,C) %where C is the number of blocks
Params = repmat(Params, 1,N); %where N is the number of conditions
```

> For parametric regressors: ```Params(1,*N).Pname = 'parametric'```; ```Params(1,*N).h = 1```; ```Params(1,*N).P = coeffiencients```.
>
> For non-parametric regressors: ```Params(1,*N).Pname = 'none'```; ```Params(1,*N).h = 0```; ```Params(1,*N).P = []```.

#### Regressors file

Create a ```P#_Regressors.mat``` for each participant. This details additional regressors to be included in the design matrix for GLM. This should contain **2** variables, ```Reggressors_names``` and ```Regressors```.

- `Regressors_names`: a `[1 x number of additional regressors]` cell variable that includes the names of each additional regressor. This is arbitary.
- `Regressors`: a `[number of samples x number of additional regressors]` matrix. Each column corresponds to the additional signal to be included in the design matrix (e.g. the *ECG signal*).

Additional regressors must have the same number of samples as the fNIRS data and also sampled at the same frequency.

#### NIRS file

This file is automatically created by running ```Run_GLM.m```. It contains several variables that detail different data and parameters. Specfically:

- ```Y``` is a ```1 x 1 struct``` with **4** fields: `od`, `hbo`, `hbr`, and `hbt`. These are the preprocessed fNIRS data.


#### ```xCon.mat```

This will be a ```.mat``` file containing the contrast vectors. This can be created using the ```CreateContrast.m``` script or be manually written. To use ```CreateContrast.m```, first create an Excel file (```.xlsx```) with different contrasts (number of contrasts x conditions + a column of zeros for the  constant in the design matrix).

![contrastsExcel](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/contrasts.png)

Change the ```root``` path on ```line 6``` to the path to the newly created ```Data``` folder. Change the path for ```xlsread()``` on ```line 8``` to the ```.xlsx``` file you have just created. Change the ```contrnum``` variable on ```line 10``` to match the number of contrasts.

Running the file will output a ```struct``` variable with one contrast per row. You will find that ```xCon.c``` includes the cntrast vector defined in your ```.xslx``` file.

![contrastsMat](https://github.com/jcjiazhang/UCLShimadzuPipelineREADME/blob/main/images/contrasts2.png)

#### ```Channels2Exclude.mat```

Create the ```Channels2Exclude.mat``` file. This will detail the channels that are deemed too low quality to be included in the analysis. This should simply be a cell variable of size ```[number of subjects x 2]```. The first column should be the folder name for each participant (e.g. ```'P1'```). The second column should contain the list of the channels to exclude for each participant.

> |1|2|
> |:---:|:---:|
> |'P01'|[2;12;47;51]|
> |'P02'|[15;12;36]|
> |'P03'|[3;16;21;27;31]|
> ...
> |'P24'|[3;14;52]|






