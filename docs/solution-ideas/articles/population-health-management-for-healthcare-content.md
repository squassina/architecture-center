[!INCLUDE [header_file](../../../includes/sol-idea-header.md)]

Population Health Management is an important tool that is increasingly being used by health care providers to manage and control the escalating costs. The crux of Population Health Management is to use data to improve health outcomes. Tracking, monitoring, and bench marking are the three bastions of Population Health Management, aimed at improving clinical and health outcomes while managing and reducing cost.

In this solution, we will use the clinical and socioeconomic in-patient data generated by hospitals for population health reporting. As an example of a machine learning application with population health management, a model is used to predict length of hospital stay. It's geared towards hospitals and health care providers to manage and control the health care expenditure through disease prevention and management. You can learn about the data used and the length of hospital stay model in the manual deployment guide for the solution. Hospitals can use these results to optimize care management systems and focus their clinical resources on patients with more urgent need. Understanding the communities they serve through population health reporting can help hospitals transition from fee-for-service payments to value-based care while reducing costs and providing better care.

Examples

* Patient monitoring

* Clinical Trials

* Smart Clinics

## Architecture

![Architecture diagram](../media/population-health-management-for-healthcare.png)
*Download an [SVG](../media/population-health-management-for-healthcare.svg) of this architecture.*

### Data flow

1. Real-time data generating devices (IoMT) transfer data to a streaming data ingestion sink with device authentication such as IoT Hub.  This sink could be a standalone Azure IoT Hub or it could be included in a fully managed application platform like [Azure IOT Central](/azure/iot-fundamentals/iot-services-and-technologies#azure-iot-central) with solution accelerators such as a [continuous patient monitoring template](/azure/iot-central/healthcare/overview-iot-central-healthcare#what-is-continuous-patient-monitoring-template).

2. The device data is then received into IoMT FHIR Connector for Azure, where it's normalized, grouped, transformed, and persisted into the Azure API for FHIR.

3. Data sources such as Electronic Medical Record systems, patient administration systems, or lab systems may generate other message formats such as HL7 messages that are [converted](https://github.com/microsoft/health-architectures/tree/master/HL7Conversion) via an HL7 ingest and conversion workflow.  The HL7 ingest platform consumes HL7 Messages via MLLP and securely transfers them to Azure via HL7overHTTPS. The data lands in blob storage, which produces an event on Azure Service Bus for processing. The HL7 convert is an Azure Logic App based workflow that performs orderly conversion from HL7 to FHIR via the FHIR Converter, persists the message into an Azure API for FHIR Server Instance

4. Data is exported from the Azure FHIR Service to Azure Data Lake Gen2 using the [Bulk Export](/azure/healthcare-apis/export-data) feature.  Sensitive data can be [anonymized](https://github.com/microsoft/FHIR-Tools-for-Anonymization) as part of the export function.

5. Azure Data Factory jobs are scheduled to copy other data sources from on-premises or alternate sources to Azure Data Lake Gen 2.

6. Use Azure Databricks to clean and transform the structureless datasets and combine them with structured data from operational databases or data warehouses.  Use scalable machine learning/deep learning techniques, to derive deeper insights from this data using Python, R, or Scala, with inbuilt notebook experiences in Azure Databricks.  In this solution, we use Databricks to bring together related, but disparate datasets for use in the patient length of stay model.

7. Experimentation and model development occurs in Azure Databricks.  Integration with Azure ML through [mlflow](/azure/machine-learning/how-to-use-mlflow-azure-databricks) allows for rapid model experimentation with tracking, model repository, and deployment. 

8. Publish trained models using Azure Machine Learning service for batch scoring through [Azure Databricks endpoints](/azure/machine-learning/how-to-use-mlflow-azure-databricks#deploy-models-to-adb-endpoints-for-batch-scoring), or as a real-time endpoint using an [Azure Container Instance](/azure/machine-learning/how-to-deploy-mlflow-models#deploy-to-azure-container-instance-aci) or [Azure Kubernetes Service](/azure/machine-learning/how-to-deploy-mlflow-models#deploy-to-azure-kubernetes-service-aks).  

## Components

* [Azure IoT Connector for FHIR](/azure/healthcare-apis/iot-data-flow) is an optional feature of Azure API for FHIR that provides the capability to ingest data from Internet of Medical Things (IoMT) devices.  Alternatively, anyone wishing to have more control and flexibility with the IoT Connector, the [IoMT FHIR Connector for Azure](https://github.com/Microsoft/iomt-fhir) is an open-source project for ingesting data from IoMT devices and persisting the data in a FHIR® server.  A simplified deployment template is available [here](https://github.com/microsoft/iomt-fhir/blob/master/docs/ARMInstallation.md).

* [Azure Data Factory](https://azure.microsoft.com/services/data-factory) is a hybrid data integration service that allows you to create, schedule, and orchestrate your ETL/ELT workflows.

* [Azure API for FHIR](/azure/healthcare-apis/) is a fully managed, enterprise-grade service for health data in the FHIR format.

* [Azure Data Lake Storage](/azure/storage/blobs/data-lake-storage-introduction) is massively scalable, secure data lake functionality built on Azure Blob Storage.  

* [Azure Databricks](/azure/databricks/scenarios/what-is-azure-databricks) is a fast, easy, and collaborative Apache Spark-based data analytics platform.  

* [Azure Machine Learning](/azure/machine-learning/service/overview-what-is-azure-ml) is a cloud service for training, scoring, deploying, and managing machine learning models at scale. This architecture uses the Azure Machine Learning service's native support for MLflow to log experiments, store models, and deploy models.

* [Power BI](https://powerbi.microsoft.com) is a suite of business analytics tools that deliver insights throughout your organization. Connect to hundreds of data sources, simplify data prep, and drive interactive analysis. Produce beautiful reports, then publish them for your organization to consume on the web and across mobile devices.

## Description

Two sample projects are detailed here that can be imported into Azure Databricks.  Note: that Standard Cluster Mode must be used on the Predicting Length of State notebooks due to the use of R code.

1. [Live Population Health Report with Length of Stay predictions](https://github.com/Azure/cortana-intelligence-population-health-management/tree/master/Azure%20Data%20Lake/ManualDeploymentGuide/Model) trains a model using encounter-level records for a million or so patients. The schema for data matches the State Inpatient Databases (SID) data from the [Healthcare Cost and Utilization Project](https://www.hcup-us.ahrq.gov/)(HCUP) to facilitate the solution's use with real HCUP data. It is suitable for use on similar patient populations, though we recommend that hospitals retrain the model using their own historical patient data for best results. The solution simulates 610 clinical and demographic features, including age, gender, zipcode, diagnoses, procedures, charges, etc. for about a million patients across 23 hospitals. To be applied to newly admitted patients, the model must be trained using only features that are available for each patient at the time of their admission.

2. [Patient-specific Readmission Prediction and Intervention for Health Care](https://github.com/Azure/cortana-intelligence-population-health-management/blob/master/Spark/Manual%20Deployment%20Guide/HDInsight%20Spark/1_Data_Preparation.ipynb) uses a [diabetes dataset](https://archive.ics.uci.edu/ml/datasets/Diabetes) originally produced for the 1994 AAI Spring Symposium on Artificial Intelligence in Medicine, now generously shared by Dr. Michael Kahn on the [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/).

## Next steps

* [Azure Health Architectures](https://github.com/microsoft/health-architectures) from the Microsoft Health Cloud & Data Architectural Engineering team, includes many reference architectures obtained by working closely customers, partners, and coworkers in the Health domain.
* [Continuous patient monitoring](/azure/iot-central/healthcare/concept-continuous-patient-monitoring-architecture) provides an app template that can build a continuous patient monitoring solution.
* [Medical Imaging Server for DICOM](https://github.com/microsoft/dicom-server) is a .NET Core implementation of DICOMweb™ that can be run in Azure.
* [OpenHack for FHIR](https://github.com/microsoft/OpenHack-FHIR) is a collection of OpenHack based tutorials that can be used to learn about the FHIR-related services in Azure.

## Related resources

* [Artificial intelligence (AI) - Architectural overview](../../data-guide/big-data/ai-overview.md)
* [Business Process Management](./business-process-management.yml)
* [Predict Length of Stay and Patient Flow](./predict-length-of-stay-and-patient-flow-with-healthcare-analytics.yml)
* [Remote Patient Monitoring Solutions](./remote-patient-monitoring.yml)