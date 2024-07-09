# VEX Hub

## Overview

VEX Hub is a centralized repository that collects and manages [Vulnerability Exploitability eXchange (VEX)][vex] documents from various open-source software projects.
It serves as a comprehensive resource for vulnerability information, helping users and security tools to efficiently access and utilize VEX data across multiple projects and ecosystems.

VEX Hub automatically retrieves VEX documents from source repositories of registered projects.
By identifying the source repository from the registered [Package URL (PURL)][purl], VEX Hub copies and organizes the VEX documents, making them easily accessible to the wider community.

## Quick Start: Registering VEX Documents in VEX Hub

To register VEX documents in VEX Hub, follow these simple steps:

1. Save your VEX file (in [OpenVEX][openvex] or [CSAF][csaf] format) in the `.vex/` directory at the root of your source repository.
2. Register the PURL of your package in VEX Hub.

That's it!
VEX Hub will automatically retrieve and process your VEX documents.

## List of PURLs
VEX Hub maintains a list of PURLs for discovering VEX documents.
The PURL definition file format is as follows:

```yaml
pkg:
  npm:
    - namespace: "@angular"
      name: animations
  golang:
    - namespace: github.com/aquasecurity
      name: trivy
  pypi:
      - name: django
  maven:
    - namespace: org.junit.jupiter
      name: junit-jupiter-api
  oci:
    - name: trivy
      qualifiers:
        - key: repository_url
          value: ghcr.io/aquasecurity/trivy
```

When specifying PURLs, the following components are required:

* type
* namespace
* name

The `version` must be omitted.
The `namespace`, `qualifiers` and `subpath` may be necessary for certain ecosystems (such as `oci`).
For detailed information about PURL composition, please refer to the PURL [specification](https://github.com/package-url/purl-spec/blob/b33dda1cf4515efa8eabbbe8e9b140950805f845/PURL-SPECIFICATION.rst).

The list of PURLs can be updated by anyone through Pull Requests. If VEX documents are already stored in the source repository of an open-source project, individuals other than the project's maintainers are welcome to register the PURL in VEX Hub.

## Identifying Source Repositories
The method for identifying source repositories varies by ecosystem. For detailed information on how source repositories are identified and crawled for different package types, please refer to the [vex-crawler documentation](https://github.com/aquasecurity/vex-crawler).

## Discovery of VEX Documents
VEX Hub automatically discovers and retrieves VEX documents from the source repositories of registered projects. The discovery process involves the following key points:

- VEX documents are searched for in the `.vex/` directory at the root of the repository.
- Supported VEX formats are OpenVEX and CSAF.
- File names matching specific patterns (e.g., *.vex.json, *.csaf.json) are considered.
- PURLs must be used for products and subcomponents in the VEX documents.
- VEX documents not matching the registered PURL are ignored.

For detailed information on the discovery process, supported ecosystems, and specific requirements, please refer to the [vex-crawler documentation](https://github.com/aquasecurity/vex-crawler).

## Directory Structure

Directories are created based on the Package URL (PURL), excluding `version`, `qualifiers` and `subpath`.
The structure adheres to the following rules:

1. The directory hierarchy is derived from the PURL's scheme, type, namespace (if present), and name components.
2. Version and qualifiers are omitted from the directory structure. These elements must be specified within the `products` field of the VEX documents.
3. URL encoding is applied to special characters in the PURL components.

Directory structure formation examples:

- `pkg:npm/express` → `/npm/express/`
- `pkg:golang/github.com/gorilla/mux` → `/golang/github.com/gorilla/mux/`
- `pkg:maven/org.apache.xmlgraphics/batik-anim` → `/maven/org.apache.xmlgraphics/batik-anim/`

Example with special character encoding:

- `pkg:npm/@angular/core` → `/npm/%40angular/core/`

## Distribution
To distribute VEX documents, we will add vex_repository.json and index.json files to conform to [the VEX Repository (VEXR) Specification][vexr-spec].

### Multiple VEX Documents per PURL

VEX Hub has the capability to store multiple VEX documents for a single PURL, as it copies all matching VEX documents from the source repository.
However, the VEX Repository Specification allows only one VEX document per PURL.
To comply with this specification, VEX Hub implements the following distribution strategy:

- When multiple VEX files exist for a single PURL, VEX Hub selects the first file in lexicographic order.
- Write the selected file to index.json for distribution.

### VEX Document Creation for VEX Hub

To prevent confusion and ensure consistency, we recommend creating only one VEX file per PURL.
Note that it is common and acceptable for a single source repository to publish multiple VEX files for different PURLs.

Example scenario:
Consider a source repository `https://github.com/org/repo` that publishes two VEX files:

- `golang.vex`: For the Go module, with PURL `pkg:golang/github.com/org/repo`
- `oci.vex`: For the OCI image, with PURL `pkg:oci/repo?repository_url=docker.io/org/repo`

The following structure is not recommended:

- `v1.vex`: For the Go module, with PURL `pkg:golang/github.com/org/repo@1.0.0`
- `v2.vex`: For the Go module, with PURL `pkg:golang/github.com/org/repo@2.0.0`

In this example, `v1.vex` is selected for distribution, and `v2.vex` is ignored.
They should be combined into a single VEX file, `golang.vex`, with multiple versions and qualifiers.

## Troubleshooting
If VEX documents in the source repository of a registered PURL are not being updated properly in VEX Hub, please check the following:

1. Ensure that the VEX documents are placed in the correct location (`.vex/` directory at the root of the repository).
2. Verify that the VEX documents use one of the supported file naming conventions.
3. Confirm that the PURL in the VEX document matches the registered PURL in VEX Hub.
4. Review the [vex-crawler documentation](https://github.com/aquasecurity/vex-crawler) to ensure all specifications for crawling are met.

If issues persist after confirming these points, please open an issue in the VEX Hub repository for further assistance.

## Contributing
Contributions to improve VEX Hub are welcome. Please submit issues and pull requests to the GitHub repository.

[vex]: https://www.ntia.gov/files/ntia/publications/vex_one-page_summary.pdf
[purl]: https://github.com/package-url/purl-spec
[vexr-spec]: https://github.com/aquasecurity/vexr-spec
[openvex]: https://github.com/openvex/spec
[csaf]: https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html