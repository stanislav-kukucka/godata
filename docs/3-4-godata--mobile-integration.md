---
layout: default
title: 4. Mobile App to Go.Data Integration
parent: Interoperability Examples
nav_order: 4
permalink: /3-4-godata--mobile-integration/
---
# Mobile App to Go.Data Integration
Mobile apps and survey tools are widely used by Go.Data implementers and other health initiatives for on-the-ground data collection, response, basic monitoring. Many of these tools are open-source and developed on similar frameworks (e.g., ODK, Kobo Toolbox, ONA, CommCare), so the available integration options are very similar. 

![use-case-4](../assets/use-case-4.png)

---
**Use Case:**
**#4.** _As a MOH employee, I would like to integrate case data collected via my mobile survey tool so that I can monitor new case registrations in real-time and quickly follow up with new contacts._

---
## Solution Overview
In this reference implementation, we integrated a COVID-19 data collection form configured on [Kobo Toolbox](https://docs.openfn.org/) with Go.Data to deliver real-time case registration as mobile data is captured. See diagram below for the data flow. 
- Kobo Toolbox is a free mobile data collection app with offline support that is widely used in humanitarian response contexts. 
- Here we leverage the [OpenFn integration platform](https://docs.openfn.org/) to automate the data integration flow & quickly map Kobo data elements to Go.Data. OpenFn offers [open-source](https://openfn.github.io/microservice/readme.html) and hosted implementation options. 

![dataflow-4](../assets/io-use-case-4.png)

## Implementation Steps
1. Consider how to extract data from your mobile application ([see this section](https://worldhealthorganization.github.io/godata/topics/#8-integrating-with-mobile-data-collection-apps) for an overview of common options). In this implementation we configured a REST Service on Kobo Toolbox ([see instructions](https://docs.openfn.org/kobo-toolbox.html)) to automatically forward new Kobo Toolbox form submissions collected via mobile to a specified endpoint–our demo OpenFn integration layer. 

![kobo-rest-service1](../assets/kobo-rest-1.png)
![kobo-rest-service2](../assets/kobo-rest-2.png)

See sample [JSON output](https://github.com/WorldHealthOrganization/godata/blob/master/interoperability-jobs/sampleData/koboForm.json) of a Kobo form submission once forwarded via the REST Service: 
```.json
"form": "covid19-registration",
  "body": {
    "Age_in_year": "32",
    "Covid_19_suspected_criteria/HF_visited": "no",
    "Sample_Classification": "n_id",
    "patient_address/teknaf_Camp": "camp_23",
    "Covid_19_suspected_criteria/Symptoms": "difficulty_breathing",
    "Patient_name": "Jane Doe",
    ...
```

2. We then mapped relevant data elements from the Kobo Toolbox survey form to Go.Data `Cases`. See example [mapping specification](https://drive.google.com/drive/folders/1qL3el6F2obdmtu2QKgcWYoXWsqBkhtII). In this step, it was important to determine the unique identifier that could be used to look-up existing `Case` records in Go.Data to ensure no duplicates were created (see the [Unique Identifiers](http://worldhealthorganization.github.io/godata/topics/1-unique-identifier-schemes) section for more on this design topic). 
3. We then drafted an OpenFn integration script (or ["job"](https://docs.openfn.org/documentation.html#jobs)) to automate the data integration mapping whenever a new Kobo form submission is forwarded to the OpenFn project inbox via the Kobo REST Service. 

In [L21](https://github.com/WorldHealthOrganization/godata/blob/docs-toolkit/interoperability-jobs/4-upsertCases.js#L21) we perform an "upsert" operation to (1) check if the case record exists, and then (2) insert/update the record in Go.Data accorndingly. We chose to check for existing Go.Data records using the `visualId` as we are also capturing the external `caseId` (e.g., `C19-930020123`) in Kobo Toolbox and can using this unique identifier for matching resources. 
```.js
upsertCase( //checks for existing cases & then sends PUT/POST request to Go.Data API Cases endpoint
    '3b5554d7-2c19-41d0-b9af-475ad25a382b', // outbreakId in Go.Data
    'visualId', //caseId - shared unique identifier 
    { data } //data mappings
  )
```
Check out the job on OpenFn.org or explore the OpenFn Inbox to see other example `JSON` payloads received from Kobo Toolbox. 
![openfn-4](../assets/openfn-4.png)

**→ See the full [job script](https://github.com/WorldHealthOrganization/godata/blob/docs-toolkit/interoperability-jobs/4-upsertCases.js).** This job leverages the [language-godata](https://openfn.github.io/language-godata/) API adaptor, which offers helper functions like `upsertCase(...)` for quicker integration setup. 

## Explore the Implementation
1. [See this video](https://drive.google.com/drive/folders/1Rf9TXCXkn8_XnjH4FcRsIGqDZ-UkVvdC) of the demo solution configured to demonstrate this use case #4.  
2. Kobo Toolbox: Explore the demo Kobo project at [https://kobo.humanitarianresponse.info/](https://kobo.humanitarianresponse.info/) & REST Service configured using the login details: `godata_demo`; pw: `interoperability`. 
3. Integration: See [example integration scripts](https://github.com/WorldHealthOrganization/godata/tree/master/interoperability-jobs) implemented on the OpenFn integration platform for automated data exchange. Explore the solution at [OpenFn.org](https://www.openfn.org/login) using the login details: `demo@godata.org`; pw: `interop!2021`. 
4. Go.Data API Wrapper: See the open-source OpenFn adaptor [language-godata](https://openfn.github.io/language-godata/). 
5. See the solution [design documentation](https://drive.google.com/drive/folders/1qL3el6F2obdmtu2QKgcWYoXWsqBkhtII).