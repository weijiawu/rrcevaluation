# Implementing your own docker
As working with docker, you can opt for your preferred programming language. The calculations but must be implemented on the same machine, you can’t call any external service for that purpose.

With the provided utility, an evaluation docker with Python it's runing on port 9020 (folder dockers/python3.9).

Your docker must implement a REST service with docker implementing the following 2 methods:

## Submition validation [POST] /validate
This method must validate the submition file/s and verify that all files have the correct format (all required fields are present and have to correct type) and the sample IDs match the Ground Truth ones.

INPUT:

| Parameter | Type | Description |
| --- | --- | --- |
| gt | String/required | Internal path of the Ground Truth. If not specified the Ground Truth has to be inside the docker. (/var/www/gt/test.json in the example) |
| results | String | Internal path with the results file. (* Required if resultsFile param is not specified.) RRC will call it with a value starting with /var/www/submits (the docker mounted folder) |
| resultsFile | File | File with the results |
| methodParams | String/optional | Method params in JSON |


OUTPUT:
A JSON string with the following Dict:

| Key | Type | Description |
| --- | --- | --- |
| result | Boolean | The results file is valid or not. |
| msg | String | If the results file is not valid, should return information here. |




## Results calculation [POST] /evaluate
This method evaluates the submition and calculates the results. If you want to show per sample information, the method has to generate a ZIP file with the file ‘method.json’ containing the method metrics results and also add individual sample information, adding the file {Sample ID}.json for each sample. Also, if your visualization requires extra information, you can add more files to the ZIP file so you can use it in your visualization.


INPUT:
| Parameter | Type | Description |
| --- | --- | --- |
| gt | String/required | Internal path of the Ground Truth. If not specified the Ground Truth has to be inside the docker. (/var/www/gt/test.json in the example) |
| results | String | Internal path with the results file. (* Required if resultsFile param is not specified.) RRC will call it with a value starting with /var/www/submits (the docker mounted folder) |
| resultsFile | File | File with the results |
| methodParams | String/optional | Method params in JSON |

OUTPUT:
A JSON string with the following Dict:
| Key | Type | Description |
| --- | --- | --- |
| result | Boolean | The evaluation has been completed succesfully. |
| msg | String | Information about the error on the evaluation. |
| method | dict | Results for the whole method. Metric and score. |
| samplesUrl | String * | URL to download the results ZIP file with samples information. *Required if you want to show samples information. |


## Configuration file
You have to provide a configuration JSON file about the task and the metrics expected for the evaluation.
This file it's generated by the utility provided.

| Key | Type | Description |
| --- | --- | --- |
| title | String | The task title |
| gt_path | String | The ground truth file path (located on the gt folder) |
| res_ext | String | Expected extension for the submitions (json,zip,jsonl,txt) |
| uploadInstructions | String | Describe the internal format of the submition file. (If it's zip file describe the format of the entries) |
| script | String | Script filename (without .py extension) of your evaluation. (placed in the /scripts/ folder) |
| scriptRequirements | Dict | PIP requirements of your script. Specify an exact version for each requirement.  |
| userParameters | Dict | Define parameters that user will have to manually select when uploading the results. These values will update the methodParameters param. Format: [{"paramKey":}] |
| methodParameters | Dict | Dict of parameters (key=>value) to send to the validation and evaluation API functions. |
| methodMetrics | JSON Dict | A JSON dictionary (metric: properties) defining all the metrics expected for the method result. On the following table you have the properties for each metric.  |
| samples | Boolean | Indicates if the evaluation gives results per sample.  |
| samples_path | String | The samples file path (.zip) (located on the gt folder)  |
| sampleMetrics | JSON Dict | A JSON dictionary (metric: properties) defining all the metrics expected for the method samples results. On the following table you have the properties for each metric.  |
| visualization | String | A visualization for the samples results. (default,TL_iou,Segm,E2E,E2E_video) Each visualization requires diferent metrics or properties on the results file generated by the evaluation. |


Metric properties
| Key | Type | Description |
| --- | --- | --- |
| long_name | String | The metric name |
| type | String | The metric type (integer|double|string) |
| order | String | This field is used to rank the methods in Ascending or Descending order (asc|desc). Only one metric for grafic is allowed. |
| show | Integer |  Enter the method ranking grafic number showing this metric. There must be at least one metric with the value '1' (graphic 1).  |
| format | String | (values: ,perc) Normal / percentage. If set to percentage, the results will be showed in percent. (Result of that field should be float values from 0-1)  |
| groupby | String | (values: method,property) Decides if the results graphic are grouped by methods or properties  |
| separation | String | (vaues: left,right) None / Left / Right. Shows a vertical separator on the results table  |
| table_group_name | String | Used to group similar properties (Repeat the same title on each property). If defined, it appears on a row over the property tile.  |
| graphic_name | String | If defined, this title will be shown on the header of graphic |
