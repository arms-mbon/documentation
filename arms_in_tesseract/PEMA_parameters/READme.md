On this repository, you can find information about the PEMA parameters, how to choose them and which ones have been previously used on ARMS data.

**PEMA parameters**

The parameter file is a critical component that governs the behavior of the PEMA pipeline, as it contains the necessary input parameters for the pipeline to work.

It is a tab-separated values file containing explanations for each parameter, followed by specific cells on where to set the parameters ("parameter-value pair"). Therefore, after each variable name, you have to leave one tab and fill in the parameter.
Depending on the marker gene used, specific algorithms and other parameters will have to be set. This is explained in detail in the file itself.

NB. It is known that the choice of parameters affects the output of each analysis; therefore, it is expected that different user choices might distort the derived outputs. For this reason, each parameter file that is used has to be kept and well identified (date, gene type, chosen algorithm) for comparability and reproducibility.

To see with parameter files have been previously used on ARMS data, go to: LINK TO DETAILED FILE

**IMPORTANT**

Make sure to have the correct set-up for your specific genetic marker (COI, 18S or ITS). 

Always remember to have matching versions between the parameter file and PEMA itself. The current version of PEMA in the ARMS workflow is v.2.1.4. 

**LINKS**

The parameter file (v.2.1.5) can be downloaded from the PEMA’s GitHub repository [here](https://github.com/hariszaf/pema/blob/master/pema_docker_image/sanity_check/COI/parameters.tsv).

The previous version of the parameter file that was used in earlier stages of the ARMS project can be found [here](https://github.com/hariszaf/pema/blob/master/analysis_directory/parameters.tsv).

Additional information on the parameters can be found in the [PEMA article](https://academic.oup.com/gigascience/article/9/3/giaa022/5803335).

