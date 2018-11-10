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

TBD

## Introduction

Developed at NASA's Jet Propulsion Laboratory, the core of HySDS (Hybrid Cloud Science Data System) has been running on AWS (Amazon Web Services) for the past 7 years generating science data products for projects such as ARIA (Advanced Rapid Imaging and Analysis) [@aria], GRFN (Getting Ready for NISAR) [@grfn], and WVCC (Water Vapor Cloud Climatology) [@wvcc]. To mitigate the risk of vendor lock-in and increase the capabilities of the system, HySDS now needs to fully function on other private and public cloud vendors such as Google Cloud Platform (GCP), Microsoft Azure, FutureSystems, ChameleonCloud, High Performance Computing/High End Computing and other XSEDE resources. Multiple efforts are currently in progress to add support for Microsoft Azure, Google Cloud Platform and the Pleiades Supercomputer however there is currently no plan to add support for NSF-funded research-based cloud resources such as those provided by XSEDE, ChameleonCloud and FutureSystems.

## Proposal
For my project, I propose to extend the capabilities of the open-source HySDS core components to support running on a Kubernetes cluster provisioned on Indiana University’s Jetstream Cloud, a compute resource provided through the XSEDE (Extreme Science and Engineering Discovery Environment) program. We will leverage the Cloudmesh utility and the Cloudmesh Kubernetes plugin to integrate Kubernetes cluster provisioning on Jetstream [4]. This requires that the HySDS system must be adapted to run compute workers on a Kubernetes cluster. We will also adapt the ingestion, processing, and cataloguing of products generated through a scientific data pipeline running on the HySDS cluster to utilize cloud-specific but highly scalable features of the XSEDE resources. It will also need to automatically scale the amount of workers based on the number of jobs submitted into the system. Lastly, the system must be easily deployable and documented for open sourcing. Once the HySDS Core system is completely functional on XSEDE, research and flight mission adaptations can then be integrated and applied to provide real-world production use-cases. One such adaption is the GRFN SAR processing. The adapted HySDS/XSEDE system would ingest the Synthetic-Aperture Radar data (SLCs, Orbits, and Calibrations) coming from Sentinel 1A and 1B through a scientific data pipeline. The system must be able to detect any new and changed data coming from the Sentinel 1A and 1B and ingest them through the pipeline and catalog its provenance. The data would then be populated in FacetSearch where the data could be selected to produce interferograms. Use cases for running HySDS on XSEDE include forward processing of level 2 SAR interferometry products as well as generating real-time urgent response products to aid in disaster response to earthquakes, hurricanes, volcanic eruptions and flooding. After the core system and its adaptation has been established, testing will be conducted to test the stability, efficiency, and capabilities of the HySDS/XSEDE system. A fleet of different machine types from XSEDE will be compared and benchmarked to a similar fleet of machine types from AWS. A large number of jobs will be submitted to the system for a given amount of time. Afterwards, the results and observations will be documented to understand which machine type achieved the best performance. The combined software, documentation, and performance report will be submitted as deliverables for my project.

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
