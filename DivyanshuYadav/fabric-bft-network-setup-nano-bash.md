
# Hyperledger Fabric Network Setup Guide (Nano Bash with BFT)

This document outlines a systematic procedure for the establishment and initiation of a Hyperledger Fabric network, leveraging the **test-network-nano-bash** scripts in conjunction with the **BFT** consensus mechanism.

-----

## 1\. Prerequisites

Prior to the commencement of the setup process, it is imperative to ensure the installation of the following requisite components on the host system:

  * **Git**: A distributed version control system, essential for the cloning of the `fabric-samples` repository.
  * **Go (1.20+)**: A programming language, a fundamental requirement for the proper functioning of certain Fabric utilities.
  * **Docker and Docker Compose**: Containerization platforms, which are indispensable for the execution of Fabric images. The operational status of Docker Desktop must be confirmed.
  * **yq (version 4+)**: A portable YAML processor, utilized for the programmatic modification of configuration files. Its installation is a prerequisite if not already present; exemplary installation methodologies include `sudo snap install yq` or `go install github.com/mikefarah/yq/v4@latest`.

-----

## 2\. Installation of Fabric Binaries and Retrieval of Docker Images

This section delineates the procedures for establishing the necessary Fabric binaries and acquiring the requisite Docker images.

The `fabric-samples` project repository shall be cloned:

```bash
git clone https://github.com/hyperledger/fabric-samples.git
cd fabric-samples
```

The Fabric tools (version 3.1.1(latest for now), compiled for Linux AMD64 architectures) shall be downloaded and subsequently extracted:

```bash
wget https://github.com/hyperledger/fabric/releases/download/v3.1.1/hyperledger-fabric-linux-amd64-3.1.1.tar.gz
tar -xvzf hyperledger-fabric-linux-amd64-3.1.1.tar.gz
```

The Fabric binaries shall be appended to the system's `PATH` environment variable; this action facilitates the direct invocation of command-line utilities such as `peer` and `configtxgen` from any terminal session.

```bash
export PATH=$PATH:$(pwd)/bin
```

The versions of the installed Fabric binaries shall be verified:

```bash
peer version
configtxgen --version
```

(Confirmation that these utilities report version **3.1.1** or a compatible iteration is advised.)

All necessary Docker images shall be retrieved from their respective repositories:

```bash
docker pull hyperledger/fabric-peer:3.1.1
docker pull hyperledger/fabric-orderer:3.1.1
docker pull hyperledger/fabric-tools:3.1.1
docker pull hyperledger/fabric-ccenv:3.1.1
docker pull hyperledger/fabric-baseos:3.1.1
docker pull hyperledger/fabric-ca:1.5.7
```

-----

## 3\. Configuration of External Builders (Chaincode As A Service - CaaS)

This section details the configuration of the primary `core.yaml` file to enable external chaincode builders, specifically for Chaincode As A Service (CaaS). This configuration is an integral component of the network's foundational setup, irrespective of immediate chaincode deployment requirements.

It is essential to ensure that the current working directory is the root directory of `fabric-samples`:

```bash
cd ~/go/src/github.com/Scapesfear/fabric-samples
```

The `config/core.yaml` file shall be modified utilizing the `yq` utility:

```bash
yq -y -i 'del(.chaincode.externalBuilders) | .chaincode.externalBuilders[0].name = "ccaas_builder" | .chaincode.externalBuilders[0].path = env(PWD) + "/builders/ccaas" | .chaincode.externalBuilders[0].propagateEnvironment[0] = "CHAINCODE_AS_A_SERVICE_BUILDER_CONFIG"' config/core.yaml
```

This command facilitates the incorporation of the `ccaas_builder` configuration into the `core.yaml` file, thereby enabling peers to interact with external chaincode services.

This command specifically alters the `core.yaml` file located within the `fabric-samples/config` directory.

The external builders specific to the Nano Bash environment shall be configured:
Initially, navigation into the `test-network-nano-bash` directory is required, followed by the execution of the `configureExternalBuilders.sh` script. This script is designed to replicate the modified `core.yaml` into the Nano Bash operational environment and subsequently refine its configuration for the execution of non-Docker chaincode (encompassing both Go and Node.js builders).

```bash
# Navigate into the nano-bash directory
cd test-network-nano-bash

# Open the file:
# test-network-nano-bash/external_builders/core_yaml_change.yaml in a text editor.

# Locate the externalBuilders section.

# Select the code lines.

# Press Shift + Tab twice to correct the indentation.

# Revert to the main fabric-samples directory to execute the script
cd ..

# Execute the script to configure external builders for the nano-bash environment
./configureExternalBuilders.sh
```

This script is designed to either create or modify the `test-network-nano-bash/config/core.yaml` file.

-----

## 4\. Network Setup (Nano Bash with BFT Consensus)

This section provides a detailed exposition on the initiation of a Hyperledger Fabric network, utilizing the `test-network-nano-bash` scripts in conjunction with the BFT consensus protocol.

The primary terminal instance (designated as **Terminal 1 - Network Orchestrator**) shall be activated.
Navigation to the `test-network-nano-bash` directory is required:

```bash
# (TERMINAL 1)
cd ~/go/src/github.com/Scapesfear/fabric-samples/test-network-nano-bash
```

Cryptographic materials and channel artifacts shall be generated:

```bash
# (TERMINAL 1)
./generate_artifacts.sh BFT -ca
```

The network shall be commenced with BFT Consensus:

```bash
# (TERMINAL 1)
./network.sh start -o BFT
```

The successful commencement of the network is to be awaited. Observance of output indicating the operational status of orderers and peers is anticipated.

![alt text](<Screenshot 2025-07-20 190217.jpg>)

-----

## 6\. Troubleshooting Tips

Observations of "timed out waiting for txid on all peers," "connection refused," or "TLS handshake errors" are indicative of potential issues:

  * **Verification of all network terminals (Terminal 1) is mandated**: Assurance of the continuous operation of orderers and peers, devoid of error indications, is required. Their re-initiation is to be considered if deemed necessary.

  * **Ports Availability**: Make sure the ports are free to used by orderers and peers, if the orderers or peers are waiting then the ports are most probably not free and are used by phantom orderers and peers.

  * **Comprehensive Network Reinitialization**: Should persistent issues be observed, the cessation of all terminal processes is required, followed by the execution of `./network.sh down` and `./network.sh clean` within the `test-network-nano-bash` directory. This process culminates in the re-commencement of the entire network from its foundational state.