
# 3rd Party Software Security Management

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Technologies Used](#technologies-used)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [File Structure](#file-structure)
- [API Endpoints](#api-endpoints)
  - [Analyzing SBOM for Vulnerabilities](#analyzing-sbom-for-vulnerabilities)
  - [Assessing Vulnerabilities by CVE ID](#assessing-vulnerabilities-by-cve-id)


## Overview

This system is designed to demonstrate the following workflows:

SBOM Request and Delivery Vulnerability Remediation Request and Response Continuous Monitoring and Risk Assessment The system is implemented using Python and leverages open-source libraries and frameworks such as FastAPI, Hugging Face Transformers, and LangChain.

## Features

- Generate SBOMs for packages and Docker images.
- Analyze SBOM artifacts for vulnerabilities using the NVD API.
- Manage vendors, products, vulnerabilities, and fixes via API.
- Support for PostgreSQL database to store SBOM-related data.

## Technologies Used

- **FastAPI**: For building APIs for Agents.
- **SQLAlchemy**: ORM for database interactions.
- **PostgreSQL**: Relational database for data storage.
- **NVD API**: National Vulnerability Database API for querying vulnerabilities.
- **Pydantic**: For data validation and parsing in FastAPI.
- **Syft**: For SBOM generation and analysis.
- **Docker**: For containerization.
- **PGAdmin**: For database management.
- **pgvector**: For database vector embeddings of vulnerabilities knowlegebase.

## Prerequisites

1. **Python 3.8+**
2. **PostgreSQL** installed and running in Docker container.
3. **Docker** For creating container for every Agent.
4. **Syft**: Installed for SBOM generation.
5. **pip** for package Installations.

## Installation

1. **Clone the repository:**
    ```bash
    git clone https://github.com/yourusername/your-repo.git
    cd your-repo
    ```



1. **Setup pgadmin:**

```
docker pull dpage/pgadmin4

docker run --name pgadmin-container -p 5050:80 -e PGADMIN_DEFAULT_EMAIL=user@domain.com -e PGADMIN_DEFAULT_PASSWORD=password -d dpage/pgadmin4
```
2. **Setup pgvector:**

```
docker pull pgvector/pgvector:pg16

docker volume create pgvector-data

docker run --name pgvector-container -e POSTGRES_PASSWORD=password -p 5432:5432 -v pgvector-data:/var/lib/postgresql/data -d pgvector/pgvector:pg16
```

## File Structure

```plaintext
Here's a detailed file structure of the project:


├── BuyerAgent/
│   ├── BuyerAgent.py
    ├── create_pgsqltables.py
│   ├── Dockerfile
│   └── requirements.txt
├── fixAgent/
│   ├── fixagent.py
│   ├── dockerfile
│   └── pycache/
├── integrationAgent/
│   ├── fapi_integrationAgent.py
│   ├── dockerfile
│   ├── requirements.txt
│   ├── readme.md
│   ├── .dockerignore
│   └── pycache/
├── SecurityAgent/
│   ├── securityAgent.py
│   ├── dockerfile
│   ├── requirements.txt
│   ├── .dockerignore
│   └── pycache/
└── VendorAgent/
    ├── VendorAgent.py
    ├── dockerfile
    ├── docker-compose.yaml
    ├── requirements.txt
    ├── kotlin-stdlib-1.4.21.jar
    ├── log4j-core-3.0.0-beta2.jar
    ├── log4j-nosql-2.3.2-javadoc.jar
    ├── log4j-web-2.3.2-javadoc.jar
    ├── openssl-1_1_1.jar
    └── poi-5.3.0.jar

└── README.md
```

**how to run the code**




## BuyerAgent

- **Request SBOM: BuyerAgent**
  - Buyers send a request with a product_id to the /request_sbom/ endpoint.
  - The API forwards this request to the Integration Agent.
  - The Integration Agent retrieves the SBOM from the Vendor API.
  - The SBOM is then sent back to the buyer.

- *Assess SBOM Risk:*
  - Buyers send SBOM data to the /assess_sbom_risk/ endpoint.
  - The API forwards the SBOM data to the Integration Agent.
  - The Integration Agent uses the Security Agent API to evaluate the risk.
  - The risk assessment results are returned to the buyer.

## IntegrationAgent

- *Check API Status:*
  - *Endpoint:* /
  - *Function:* Returns a simple message greet to confirm the API is working.

- *Get SBOM:*
  - *Endpoint:* /get_sbom
  - *Function:* Receives a request with a product_id and forwards it to the Vendor API to retrieve the SBOM. Returns the response from the Vendor API.

- *Access SBOM:*
  - *Endpoint:* /acess_sbom
  - *Function:* Receives SBOM JSON data from the buyer and forwards it to the Security API for vulnerability analysis. Returns the analysis results from the Security API.


## VendorAgent

Here's a breakdown of how the FastAPI application works:

- *Generate SBOM:*
  - *Endpoint:* /generate-sbom/
  - *Function:* Generates a Software Bill of Materials (SBOM) for a specific JAR file based on the product_id provided.
  - *Process:*
    - Receives a product_id in the request.
    - Maps product_id to a specific JAR file path.
    - Checks if the file exists at the specified location.
    - Calls the generate_sbom() function to generate the SBOM using the syft tool.
    - Returns the SBOM data in JSON format.

- *Acknowledge Fix Request:*
  - *Endpoint:* /acknowledge-fix-request/
  - *Function:* Acknowledges a request to fix vulnerabilities for a specific product.
  - *Process:*
    - Receives product_id and vulnerability_ids in the request.
    - Logs the fix request (simulates interaction with a database or external service).
    - Returns a confirmation message indicating successful acknowledgment.

- *Update Product Status:*
  - *Endpoint:* /update-product-status/
  - *Function:* Updates the status of a product in an in-memory dictionary.
  - *Process:*
    - Receives product_id and status in the request.
    - Updates the product status in an in-memory dictionary (product_statuses).
    - Logs the status update (simulates interaction with a database or external service).
    - Returns a confirmation message indicating the status update.

- **Utility Function - generate_sbom():**
  - *Function:* Executes a syft command to generate an SBOM for a given package or container image.
  - *Process:*
    - Builds and runs a syft command with the specified package path and output format.
    - Captures and parses the command output.
    - Returns the SBOM data as a dictionary.

## SecurityAgent

- *Analyze SBOM Vulnerabilities:*
  - *Endpoint:* /analyze_sbom_vulneribilitys/
  - *Function:* Analyzes an SBOM by querying the NVD for vulnerabilities associated with CPE names found in the SBOM. Returns a list of vulnerabilities for each CPE.

- *Assess Vulnerability:*
  - *Endpoint:* /assess_vulnerability
  - *Function:* Receives a specific CVE ID, queries the NVD for details about that vulnerability, and returns a detailed assessment including CVSS score and severity.

- *Utility Functions:*
  - **get_vulnerabilities_from_nvd(cpe_name):** Queries the NVD API for vulnerabilities related to a specific CPE name.
  - **check_vulnerabilities(cpe):** Checks SBOM dependencies for known vulnerabilities and formats the results.
  - **check_vulnerabilities_info(vulnerability, cveid):** Provides detailed information about a specific CVE, including CVSS scores and severity metrics.





## fixAgent
- Prioritize Fixes:
  - *Endpoint:* /prioritize_fixes
  - *Function:* Prioritizes vulnerabilities based on IDs and returns their priority.

- *Generate Fix Plan:* 
  - *Endpoint:* /generate_fix_plan
  - *Function:* Creates a fix plan with actions and timelines for each vulnerability.

- *Generate VEX Document:* 
  - *Endpoint:* /generate_vex
  - *Function:* Produces a VEX document listing vulnerabilities and their fix status.

- *Update SBOM:* 
  - *Endpoint:* /update_sbom
  - *Function:* Updates an SBOM with the applied fixes and returns the updated version.



*Additional Details:*
- *Error Handling:* The application raises HTTPException with appropriate status codes and error messages if there are issues during file generation, parsing, or other processes.
- *Logging:* Simulated logging is done using print() statements for demonstration purposes. In a production environment, proper logging should be used.
#   s b o m _ r a g  
 