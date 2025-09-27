# Integrate IBM Environmental Intelligence Suite (EIS) with SAP Business Technology Platform (BTP)

This repository is intended to demonstrate integrations between IBM Environmental Intelligence Suite (EIS) and SAP Business Technology Platform (BTP).

There are two tutorials in this repository:

1. [Access Weather APIs.](https://github.com/ibm-ecosystem-engineering/ibm-eis-sap-btp/tree/main/access-weather-api) This tutorial demonstrates how multiple lines-of-business (LoB) can share a single IBM EIS API key, while allowing administrators to track usage per LoB. This is accomplished using SAP Integration Suite on BTP. The tutorial further demonstrates a weather application built using the SAP Build App framework.
2. [Consume Weather Alerts.](https://github.com/ibm-ecosystem-engineering/ibm-eis-sap-btp/tree/main/alert-demo) This tutorial demonstrates how IBM EIS weather alerts can be published to SAP Event Mesh message queues, from which multiple applications can potentially consume the alerts.

In order to perform the tutorials you will need to have access to IBM EIS.

- IBMers can request EIS access [here](https://eistrialrequest.ideas.aha.io/portal_session/new).
- Non-IBMers can request a [30-day free trial](https://www.ibm.com/account/reg/us-en/signup?formid=urx-51911&_gl=1*9cen1r*_ga*NzczNTIyMDM3LjE2ODkxNzIwNjE.*_ga_FYECCCS21D*MTY4OTUzODI1My4yMi4xLjE2ODk1Mzg2MjEuMC4wLjA).

In addition, you will need access to the SAP Business Technology Platform (BTP). For the first tutorial you can use a [trial account](https://developers.sap.com/tutorials/hcp-create-trial-account.html). The second tutorial requires an [enterprise account](https://help.sap.com/docs/btp/sap-business-technology-platform/enterprise-accounts) in order to deploy SAP Event Mesh.

IBMers and registered Business Partners may also visit the associated [TechZone Collection](https://techzone.ibm.com/collection/64b059317f0fe20017d86683).

## Maintainer

Manoj Gali is a Senior SAP CPI Consultant with over 5 years of experience in SAP integration technologies. Specializing in SAP CPI, PI/PO, and hybrid integration landscapes, he focuses on designing and optimizing scalable integration solutions to ensure secure, high-performance data exchange across SAP and non-SAP systems.

For inquiries regarding these integration patterns or technical support, please contact:
- Email: manoj.gali695@gmail.com