# HySDS on Kubernetes: ARIA InSAR Processing on XSEDE Jetstream :hand: fa18-516-14

| Gerald Manipon, Gregor von Laszewski, Hook Hua
| gmanipon@iu.edu, laszewski@gmail.com, hook.hua@jpl.nasa.gov
| Indiana University, Indiana University, NASA Jet Propulsion Laboratory
| hid: fa18-516-14
| github: [:cloud:](https://github.com/cloudmesh-community/fa18-516-14/blob/master/report.md)
| code: [:cloud:](https://github.com/pymonger/hysds-k8s/tree/grfn-jetstream-iu)

---

Keywords: E516, NASA, JPL, HySDS, Hybrid Cloud Science Data System, InSAR, Interferogram, Radar Interferometry, Sentinel-1, ISCE, InSAR Scientific Computing Environment, Kubernetes, OpenStack, Jetstream, Python, Terraform

---

## Abstract

Developed at NASA's Jet Propulsion Laboratory, the core of HySDS (Hybrid Cloud Science Data System) has been running on AWS (Amazon Web Services) since 2011 generating science data products for projects such as ARIA (Advanced Rapid Imaging and Analysis) [@aria], GRFN (Getting Ready for NISAR) [@grfn], and WVCC (Water Vapor Cloud Climatology) [@wvcc]. The upcoming NASA missions, SWOT (Surface Water and Ocean Topography) [@swot] and NISAR (NASA-ISRO SAR Mission) [@nisar], are slated to launch in September 2021 and January 2022, respectively, and will use HySDS in AWS as the baseline science data system. To mitigate the risk of vendor lock-in and increase the capabilities of the system, HySDS now needs to fully function on the IaaS (Infrastructure as a Service) services provided by other private and public cloud vendors such as Google Cloud Platform (GCP), Microsoft Azure, FutureSystems, Jetstream, ChameleonCloud, High Performance Computing/High End Computing and other XSEDE resources. Multiple funded efforts are currently in progress at JPL to add support for Microsoft Azure, Google Cloud Platform and the Pleiades Supercomputer however there is currently no plan to add support for NSF-funded research-based cloud resources such as those provided by XSEDE: Jetstream, ChameleonCloud and FutureSystems. In addition, HySDS was developed to operate at the IaaS layer however Kubernetes, a container orchestration framework open-sourced by Google, provides an abstraction layer to the vendor-specific IaaS services and can be utilized as a PaaS (Platform as a Service) to improve the portability of HySDS to other vendors. In this project for the Fall 2018 E516 course at Indiana University at Bloomington, we pathfind a potential avenue for utilizing these NSF-funded cloud resources by prototyping a real-life science use case, ARIA InSAR processing of Sentinel-1 SLC data, running on HySDS on a Kubernetes cluster on IU's Jetstream Cloud. We found that by generalizing the way HySDS handles product generation executables (PGEs) and their inputs, HySDS can successfully leverage the IaaS abstraction provided by Kubernetes and can thus be provisioned onto any private or public cloud vendor capable of running Kubernetes. Further work however is needed to assess the additional overhead and complexity that Kubernetes adds to the end-to-end science data system as well as how well such a system performs and scales in a production environment.

## Introduction

HySDS was originally developed as the framework used to create the science data system (SDS) that *"scalably powers the ingestion, metadata extraction, cataloging, high-volume data processing, and publication of the geodetic data products for the Advanced Rapid Imaging & Analysis (ARIA), Getting Ready for NISAR (GRFN), and Water Vapor Cloud Climatology (WVCC) projects at JPL"* [@hysds]. The role of the SDS is to process the raw level0 (L0) data downloaded by the ground data system (GDS) from the satellites to higher-order level1 (L1), level2 (L2) and level3 (L3) products which are more usable to end-users. The SDS then delivers these products to a distributed active archive center (DAAC) for the purpose of providing public distribution, access, and discovery of these products. +@fig:large-data-flow depicts the role of the SDS in the end-to-end system for the upcoming NISAR mission.

![Large Data Flow](images/large-data-flow.png){#fig:large-data-flow}

### Architecture

The SDS itself is comprised of various components to orchestrate and facilitate the processing of the raw L0 satellite data. +@fig:sds-functional-architecture depicts the functional architecture of the SDS in the context of running in AWS.

![Functional Architecture](images/sds-functional-architecture.png){#fig:sds-functional-architecture}

The GDS system will stage L0 data to the incoming bucket of the HySDS cluster. When an L0 dataset file is deposited, an event trigger submits an ingestion job to the resource manager which is then pulled by an autoscaling group of workers specifically provisioned for the purpose of dataset/product ingestion. During the ingestion job, the L0 data file is verified, metadata is extracted and browse images are generated. They are then copied to the SDS dataset bucket and the metadata/location of the L0 dataset is indexed into the dataset catalog. Upon indexing of the dataset into the catalog, trigger rules are evaluated and if their conditions are met, further downstream processing jobs are submitted to the resource manager. For example, the use-case we will be prototyping is the ARIA InSAR processing of Sentinel-1 SLCs into L2 interferograms. In the ARIA SDS, ingestion of an SLC triggers the Sentinel-1 interferogram workflow. This workflow, which runs on a different autoscaling group, determines if all the conditions have been met (e.g. pertinent ancillary orbit files, calibration files, digital elevation maps exist) for processing an interferogram job. If so, it proceeds to generate the interferogram dataset which in turn gets ingested into the dataset bucket and indexed into the dataset catalog. Finally, the ingestion of these higher-order products trigger an evaluation of a delivery rule which runs on yet a different autoscaling group to deliver the interferogram to the DAAC's bucket or to notify them that it can be picked up from the SDS's dataset bucket.

### Processing Algorithm Library

Note also in the above figure that HySDS provides a processing algorithm library which serves as the catalog of all PGEs (product generation executors) that can be used by the worker fleet. Processing algorithms are usually developed by subject matter experts (e.g. radar scientists) and integrated into HySDS by installing them in container images (Docker, Singularity, rkt) along with interface documents needed by the rest of the system to submit and run jobs.

### Pedigree

The SDS architecture above is *NOT* novel and is very much aligned with the architectural design patterns of most other cloud data processing systems out there (e.g. AWS Batch, GCP Dataflow, Azure Batch). In fact, other data processing systems take a more cloud-native, all-in approach to their implementation. The goal of HySDS from day-one was to be as *vendor-neutral* and *cross-cloud interoperable* as possible all the while taking advantage of any performance gains by utilizing vendor-specific cloud-native features. The value-added that HySDS provides is the experience, pedigree and lessons-learned from operating the ARIA science data system since 2011. One such example is the suite of HySDS watchdogs built into the system. These watchdog subcomponents of the resource manager are dispatched to monitor and mitigate any potential issues. For example, take an SDS that has 100 jobs submitted to the resource manager. There are 5 compute workers running and executing jobs. If one of those workers had an issue, for example the root disk volume was mounted as read-only on boot-up, the job immediately fails. The compute worker then pulls the next job but that too immediately fails. Within seconds, the rogue compute worker has drained the job queue resulting in what is known as “job drain”. A highly-resilient data processing system would employ a job drain watchdog to detect this situation, determine that the rogue compute worker was not healthy, and shut down that worker immediately before more damage can be done.

## Proposal

For my project, I propose to extend the capabilities of the open-source HySDS core components to support running on a Kubernetes cluster provisioned on Indiana University’s Jetstream Cloud,jkkkkk a compute resource provided through the XSEDE (Extreme Science and Engineering Discovery Environment) program. We will leverage the Cloudmesh utility and the Cloudmesh Kubernetes plugin to integrate Kubernetes cluster provisioning on Jetstream [4]. This requires that the HySDS system must be adapted to run compute workers on a Kubernetes cluster. We will also adapt the ingestion, processing, and cataloguing of products generated through a scientific data pipeline running on the HySDS cluster to utilize cloud-specific but highly scalable features of the XSEDE resources. It will also need to automatically scale the amount of workers based on the number of jobs submitted into the system. Lastly, the system must be easily deployable and documented for open sourcing. Once the HySDS Core system is completely functional on XSEDE, research and flight mission adaptations can then be integrated and applied to provide real-world production use-cases. One such adaption is the GRFN SAR processing. The adapted HySDS/XSEDE system would ingest the Synthetic-Aperture Radar data (SLCs, Orbits, and Calibrations) coming from Sentinel 1A and 1B through a scientific data pipeline. The system must be able to detect any new and changed data coming from the Sentinel 1A and 1B and ingest them through the pipeline and catalog its provenance. The data would then be populated in FacetSearch where the data could be selected to produce interferograms. Use cases for running HySDS on XSEDE include forward processing of level 2 SAR interferometry products as well as generating real-time urgent response products to aid in disaster response to earthquakes, hurricanes, volcanic eruptions and flooding. After the core system and its adaptation has been established, testing will be conducted to test the stability, efficiency, and capabilities of the HySDS/XSEDE system. A fleet of different machine types from XSEDE will be compared and benchmarked to a similar fleet of machine types from AWS. A large number of jobs will be submitted to the system for a given amount of time. Afterwards, the results and observations will be documented to understand which machine type achieved the best performance. The combined software, documentation, and performance report will be submitted as deliverables for my project.

## Progress report

As of November 5, 2018, we have successfully ported HySDS core software components to be able to run on Kubernetes on Jetstream. We are currently adding the adaptation layer so that our HySDS cluster on K8s can produce Sentinel-1 interferograms. The memory (30 GB RAM) and CPU (8 vCPUs) requirements of ISCE (the software used for running interferogram generation) and the disk space requirements of the inputs (60GB): Sentinel-1 SLCs, precise orbits, and calibration files have only allowed us to scale up to 1 kubernetes worker for production. We’d like to scale the number of workers to 10 or 20 to further test the performance of HySDS on K8s.

## Requirements

All images must be referred to in the text. The words bellow and above
must not be used in your paper for images, tables, and code.

+@fig:fromonetotheorther shows a nice figure exported from Powerpoint
to png. If you like you can use this as a basis for your drawings.

![A simple flow chart](images/from-one-to-the-other.png){#fig:fromonetotheorther}

Figures must not be cited with an explicit number, but automated
numbering must be used. Here is how we did it for this paper:

```
+@fig:fromonetotheorther shows a nice
figure exported from Powerpoint to png.
If you like you can use this as a basis
for your drawings.

![A simple flow chart](images/from-one-to-the-other.png){#fig:fromonetotheorther}
```

## Design

## Architecture

## Dataset

## Implementation

## Benchmark

## Conclusion

## Acknowledgement

"We thank [consultant name(s)] for [his/her/their] assistance with [describe tasks such as porting, optimization, visualization, etc.], which was made possible through the XSEDE Extended Collaborative Support Service (ECSS) program."

## Workbreakdown

Only needed if you work in a group.
