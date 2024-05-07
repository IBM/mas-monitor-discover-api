# Discover & play with MAS Monitor API v2
**Last Updated:** 08 May 2024 **Author:** Christophe Lucas <br>
**Products Used**: <a href="https://www.ibm.com/docs/en/maximo-monitor/continuous-delivery" target="_blank">IBM Maximo Monitor</a><br>
**Disclaimer:** This code is delivered as-is and is NOT formal IBM documentation in any way
<a id='toc'></a>
## Table of Contents
- [**Introduction & Overview**](#overview)
- [**Prerequisites**](#prereq)
- [**Get Ready !**](#ready)
    - [Get Monitor {X-api-key, X-api-token, tenantId, mam_user_email} & Server URL](#credentials)   
    - [Create sample Robot Device Type, Devices & Alerts to test your API calls](#robot)
- [**Overview of MAS Monitor API v2 & Anatomy of an API call**](#apioverview)
    -  [Overview of the API v2 structure & definitions](#structure)
    -  [Anatomy of an API Call](#anatomy)
- [**Discover & Try Monitor API v2**](#discover)
    -  [Try Me - Using Monitor Swagger API Web UI](#monitorui)
    -  [Try Me - Using POSTMAN](#postman)
-  [**Play with 12 API calls using MAS Cloud Pak for Data**](#cp4d)
    -  [Setup Project, Import Notebook](#setupnotebook)
    -  [How to use the Notebook](#cp4dnotebook)

<a id='overview'></a>
# Introduction & Overview
In this Tutorial, you will:
1. **GET READY** to use Monitor API v2 by finding your MAS Monitor Environment required info & by creating sample Monitor Robot Devices and Alerts to test the calls.
2. **UNDERSTAND** the Monitor API v2 structure and the anatomy of an API v2 call & 𝗗𝗜𝗦𝗖𝗢𝗩𝗘𝗥 how to execute any call with Swagger & Postman.
3. **PLAY** with 12 API v2 ready-to-use calls using a provided Jupyter Notebook, on your MAS Cloud Pak for Data instance.

![image](/images/tutorial-overview.jpg)

<a id='prereq'></a>
# Prerequisites
To complete this tutorial, you need:
- an ID with Administrator Access to the Monitor Application of a MAS 811 instance
- Postman or an equivalent tool for running APIs. Postman can be downloaded here: <a href="https://www.postman.com/downloads/" target="_blank">Download Postman</a>.
- Access to a Cloud Pak for Data Instance to run the [Play with 12 API calls using MAS Cloud Pak for Data](#cp4d) section.

This tutorial was built using a **MAS 8.11.10** instance with **Monitor 8.11.6** and **IoT 8.8.7**.

<a id='ready'></a>
# Get Ready !

<a id='credentials'></a>
## Get Monitor {X-api-key, X-api-token, tenantId, mam_user_email} & Server URL
As per <a href="https://www.ibm.com/docs/en/maximo-monitor/continuous-delivery?topic=reference-apis" target="_blank">Monitor APIs</a> documentation, in order to run Monitor API v2 calls, you will need:
- For **GET** calls: `X-api-key`, `X-api-token` values.<br> 
- For **POST**, **PUT**, or **DELETE** calls: `X-api-key`, `X-api-token` + `tenantId` and `mam_user_email` values.<br>
- Your MAS `Monitor Server URL`.<br>

Follow the below steps to get all those required values:

**Get X-api-key & X-api-token values**<br>
There are 2 options/ways to get the `X-api-key`, `X-api-token` values:
- **Option 1 - Using Monitor as_apikey & as_apitoken (via MAS - Openshift Container Platform)**
  - From the OCP console, click `Project`. In the `Search by name` field, start typing `mas-monitor` and locate e.g. the `mas-***-monitor` Project. Open the Project.
  - On the `Inventory` card, click `Secrets`. Search for and open `monitor-api`.
  - Copy and save `as_apikey` and `as_token` - those are your required `X-api-key` & `X-api-token` values.

- **Option 2 - Using Watson IoT Platform API Key & Authentication Token**
  - From the Monitor Home menu, click `Open the IoT tool` to access the Watson IoT Platform. Click `Apps` menu.
  - Click `Generate API Key`, Next and click expiration `Off`. Select `Standard Application` role . Click Generate Key. 
  - Copy and save `API key` and `Authentication token` - those are your required `X-api-key` & `X-api-token` values.

**Get tenantId, mam_user_email**<br>
- To get your `tenantId`, look at your Monitor homepage URL, it should like this `https://tenantId.monitor.domain.com/`. Example 1: `https://mygeo.monitor.mygeomas.gtm-pat.com/` where `mygeomas.gtm-pat` is the `domain` and `mygeo` is the `tenantId`. Example 2: `https://main.monitor.customer-poc.suite.maximo.com/` where `customer-poc.suite.maximo` is the `domain` and `main` is the `tenantId`. Another simple way to get the value of your `tenantId` is to click the `Open the IoT tool` from Monitor home page, then copy (top-right) the value of your Watson IoT Platform ID.
- To get your `mam_user_email`, jut click on `Manage your Profile` (top-right) and copy the value of your primary email - that will be your `mam_user_email` value.

**Get Monitor Server URL**<br>
- Access the Monitor v2 API by clicking Monitor top-right `Help - API` menu. This will open a screen where you will note 4 distinct API sets. 
- Click any of these API sets, and notice top-left the `Server` box where the `Generated server url` will appear in plain sight. Copy it and use this as the `your-mas-monitor-server-url` in all below API v2 Postman (or CloudPakForData) calls.

![image](/images/get-api-values.jpg)

<a id='robot'></a>
## Create sample Robot Device Type, Devices & Alerts to test your API calls
The purpose of this section is to create standard ootb Monitor `Robot` Device Type, Devices & Alerts that you will use to test all the API calls. Creating a `Robot` Device Type will automatically create 5 Devices named `73000`, `73001`, `73002`, `73003` and `73004`, and data will keep flowing to them. All following sections will be based on a `XY-Robot` Device Type - where `XY` will be your initials (e.g. `CL-Robot` for Christophe Lucas).

**Create Sample Robot Device Type**
1. On Monitor Home screen, click `Create a device type` then select `Sample robot type template`, name your Device Type `XY-Robot` (where`XY` are your initials), click `Create`.
2. Go to the `Setup` menu and select `XY-Robot`.
Notice how Monitor automatically created Devices `73000`, `73001`, `73002`, `73003` and `73004` as well as data for those Devices in the past. It will keep creating new data in the future.

**Create 2 Alert Types** <br>
1. Click on the + `Batch data metric (calculated)` link on the left, select `AlerthHighValue`. Select `speed` in `input_items`, set `upper_threshold` to `7`, `Severity` to `High` and `Status` to `New`. Click Next, call the output_items `alert_speed_7`. Unclick `Auto schedule` and enter `5 Minutes` in the `Executing every` field, and `7 Days` in the `Calculating the last` field, click `Create`.
2. Repeat 1. for `alert_torque_16` (with `upper_threshold` = `16`, `Severity` = `Medium` and `Status` = `Validated`).
3. Wait minimum 5 minutes (take a longer break !) for the Analytics jobs to run. Observe how `alert_torque_16` and `alert_speed_7` alerts are regularly created at an acceptable frequency (i.e. from a couple to a dozen to sometimes more per day).

The picture below highlights the steps you just completed.
![image](/images/cls-mas-monitor-api-image002.jpg)


<a id='apioverview'></a>
# Overview of MAS Monitor API v2 & Anatomy of an API call

<a id='structure'></a>
## Overview of the API v2 structure & definitions
The Monitor API v2 is very rich and contains dozens and dozens of calls. The best way to discover it is to first zig-zag through its content:
1. To access the API, click Monitor top-right `Help - API` menu. This will open a screen where you will note 4 distinct API sets: `Manage core resources`, `Query data`, `Query data for dashboards`, `Manage images`(we will disregard `Manage Data Store Connector / FactoryTalk Integration` in this tutorial).
2. Click on each API set, and spend minimum 10 minutes looking at the descriptions of the calls.
3. For `Manage core resources`, `Query data` specifically, notice how the API set contains multiple Definitons (top-right drop-down list). Click one after the other and notice how each contains many different APIs available for `DeviceType`, `DeviceType -> Device`, `DeviceType V2`, `Organization`, `Site`,`Site -> Asset`, `Site -> Location` etc.<br>

![image](/images/apis-overview.jpg)

<a id='anatomy'></a>
## Anatomy of an API Call
The following image highlights the constitutive elements that must be considered to run any API call:
1. **Parameters**: these are the INPUT parameters, either mandatory or optional.
2. **Call Type**: each API call will have a GET, POST, PUT, DELETE type. 
3. **Call URL**: contains the URL of your Monitor instance required to execute the API call.
4. **Request Body**: either mandatory or optional.
5. **Ouptut**: the result (i.e. OUTPUT) of your API call execution.

[below images are taken from either the Swagger, the Postman or the Cloud Pak for Data interface that you will use to run the calls]
![image](/images/anatomy.jpg)
<a id='discover'></a>
# Discover & Try Monitor API v2

<a id='monitorui'></a>
### Try Me - Using Monitor Swagger API Web UI
First, let's use the Monitor API Swagger Web UI which is available directly from within the Monitor application. We will use a simple `Manage core resources DeviceType - Device Type Management` API call, which returns the list of all *DeviceTypes* in your Monitor environment:
1. Click Monitor top-right `Help - API` menu. Open the `Manage core resources` API, select the `DeviceType` definition (top right). On right-side of the screen, click `Authorize`.
2. In the opened `Available authorizations` window, enter the value of `X-api-key`, click `Authorize`, then repeat the same, one after the other, for `X-api-token`, `tenantId` and `mam_user_email`that your retrieved in the [Get Monitor {X-api-key, X-api-token, tenant, mam_user_email} & Server URL](#credentials) section.
3. Go to the `Device Type Management` section and open the `POST /api/v2/core/deviceTypes/search - Get Device types, filter by status, supports pagination` call. 
4. Click `Try it out` and empty the whole `Request body` content (that box should be blank).
5. Click `Execute` and observe the succesfull `200` Server response code.
6. Observe the returned json in the `Response Body`. It should list all the Monitor `Device Types` within your Monitor system, including the `XY-Robot` that you created in [Create sample Robot Device Type, Devices & Alerts to test your API calls](#robot) section. Find-copy the `uuid` of your `XY-Robot` that we will use in the next section.

![image](/images/cls-mas-monitor-api-image004.jpg)

<a id='postman'></a>
### Try Me - Using POSTMAN
We will use a `Query data - DeviceType definition -- Device Type Alert Management` API call to test an API call using Postman. This call returns the list of *Alerts* for a given *DeviceType*.
1. Launch Postman desktop. Click the `+` button to create a new Collection, name it `My Monitor REST API v2 Collection`. Try to be well structured, so under that Collection, create folders and sub-folders (e.g. `Query data` - `DeviceType` - `Device Type Alert Management`) that reflect the API structure as you observed it in [Overview of the API v2 structure & definitions](#structure).
2. Under your `Query data` - `DeviceType` - `Device Type Alert Management` sub folder, click on the right `...` and `Add request`. 
3. Make sure your request is a `POST`. Name it `api/v2/datalake/deviceTypes/{deviceTypeId}/alerts/search`. In the `POST` field, enter `https://your-mas-monitor-server-url/api/v2/datalake/deviceTypes/{deviceType}/alerts/search` where (1) `your-mas-monitor-server-url` is the value you retrieved in [Get Monitor {X-api-key, X-api-token, tenantId, mam_user_email} & Server URL](#credentials) and (2) `{deviceType}` is the `uuid` value of your `XY-Robot` as returned in the previous section.
4. Go to the `Headers` tab and add 1 after the other 4 Keys: `X-API-KEY`, `X-API-TOKEN`, `tenantId` & `mam_user_email`, with the values you retrieved in the [Get Monitor {X-api-key, X-api-token, tenantId, mam_user_email} & Server URL](#credentials) section.
5. Go to the `Body` tab, and the first time you run the call, leave it empty - this will return all the Alerts of your `XY-Robot`. The second time you run the call, fill the `Body` section with this snippet where you update `"tsBegin"` and `"tsEnd"` values to the time range you want to list the Alerts for:
```
{
  "search": "alert_speed_7",
  "tsBegin": "2024-04-29T07:31:57.893Z",
  "tsEnd": "2024-05-01T07:31:57.893Z"
}
```

![image](/images/monitor-postman.jpg)

<a id='cp4d'></a>
## Play with 12 API calls using MAS Cloud Pak for Data
In this section, you will download and re-use a sample Jupyter Notebook which contains 12 sample API calls. Each has a varying level of 'complexity' - try them all in the order set in the Notebook.
<a id='setupnotebook'></a>
### Setup Project, Import Notebook
1. Donwload the <a href="https://github.com/IBM/mas-monitor-discover-api/blob/main/notebooks/cl-mas-monitor-discover-api-cp4d-template.ipynb" target="_blank">cl-mas-monitor-discover-api-cp4d-template.ipynb</a> Jupyter Notebook provided with this tutorial.
2. From your MAS Cloud Pak for Data instance home page, click `All Projects` from the left menu. Click `New Project`.
3. Click `Create an empty project`, name it `xy-mas-monitor-discover-api-cp4d` (where `xy` are your initials). Click `Create`.
4. Once the project is created, click `New asset`, select `Jupyter notebook editor`. Select the default `Python 3.10` and `Runtime 22.2 on Python 3.10`.
5. Go to the `From File` tab of the project. In the `Drag and drop files here or upload` box, upload the Notebook you just downloaded. Name it `xy-mas-monitor-discover-api-cp4d`. Click `Create`.

![image](/images/cloudpakfordata.jpg)

<a id='cp4dnotebook'></a>
### How to use the Notebook
Execute the cells in the just-imported `cl-mas-monitor-discover-api-cp4d-template.ipynb` Notebook one after the other.
1. In the `GET SET - Get & Set your MAS Monitor ENV Details` Notebook initial section, replace `X-api-key`, `X-api-token`, `tenantId`, `mam_user_email` as you found them in [Get Monitor {X-api-key, X-api-token, tenantId, mam_user_email} & Server URL](#credentials).
2. Run all subsequent API calls in order, 1 by 1, by following the instructions which each include these details:<br>
```
What this call does: Short explanation
Your action: What you need to do
Note: where applicable
Back to Table of Contents
```
