---
title: Using the Local Validator
metaTitle: Using the Local Validator
description: Learn how to setup a local development environment and use a local validator
# remember to update dates also in /components/guides/index.js
created: '08-9-2024'
updated: '08-9-2024'
---

## Overview

A **Local Validator** acts as your personal node, providing a local sandbox environment for testing applications without the need to connect to a live blockchain network. It operates a **fully customizable local test ledger**, which is a simplified version of the Solana ledger, equipped with all **native programs pre-installed** and various features enabled.

### Setup 

To start using the local validator, you'll need to install the **Solana Tools CLI** using the appropriate commands for your operating system.

{% dialect-switcher title="Installation Commands" %}

{% dialect title="MacOs & Linux" id="MacOs & Linux" %}

```
sh -c "$(curl -sSfL https://release.solana.com/v1.18.18/install)"
```

{% /dialect %}

{% dialect title="Windows" id="Windows" %}

```
cmd /c "curl https://release.solana.com/v1.18.18/solana-install-init-x86_64-pc-windows-msvc.exe --output C:\solana-install-tmp\solana-install-init.exe --create-dirs"
```

{% /dialect %}

{% /dialect-switcher %}

**Note**: The installation script references the `1.18.18` version of Solana. To install the latest version or discover different installation methods, refer to the official [Solana documentation](https://docs.solanalabs.com/cli/install).

### Usage

After installing the CLI, you can start your local validator by running a simple command.

```
solana-test-validator
```

Upon launch, the validator will be accessible at a **local URL(http://127.0.0.1:8899)**. You'll need to establish a connection by configuring your code with this URL.

```ts
import { createUmi } from '@metaplex-foundation/umi-bundle-defaults'

const umi = createUmi("http://127.0.0.1:8899")
```

The local validator will generate a directory named `test-ledger` in your user folder. This directory holds all data related to your validator, including accounts and programs. 

To reset your local validator, you can either:
- **Delete the `test-ledger` folder**, or
- Use a reset command to restart the validator.

Additionally, the `solana-logs` feature is extremely useful for monitoring program outputs during testing.

## Managing Programs and Accounts

The **Local Validator** doesn’t include specific programs and accounts found on mainnet. It only comes with **Native Programs** and the accounts you create during testing. If you need specific programs or accounts from mainnet, the **Solana CLI** allows you to download and load them onto your local validator.

### Downloading Accounts and Programs:

You can easily download accounts or programs from a source cluster to your local validator for testing purposes. This allows you to replicate the mainnet environment.

**For accounts:**
```
solana account -u <source cluster> --output <output format> --output-file <destination file name/path> <address of account to fetch>
```
**For Programs:**
```
solana program dump -u <source cluster> <address of account to fetch> <destination file name/path>
```

### Loading Accounts and Programs:

Once downloaded, these accounts and programs can be loaded into your local validator using the CLI. You can run commands to load specific accounts and programs into your local environment, ensuring they are ready for testing.

**For accounts:**
```
solana-test-validator --account <address to load the account to> <path to account file> --reset
```
**For programs**
```
solana-test-validator --bpf-program <address to load the program to> <path to program file> --reset
```

## Creating a "Metaplex" Local Validator

With the basics of the local validator setup and management, you can create and manage personalized local validators through **bash scripts**. 

For example, you can create a **`metaplex-test-validator`** that includes main **Metaplex programs** like `mpl-token-metadata`, `mpl-bubblegum`, and `mpl-core`.

### Setting Up Directories and Downloading Program Data

First, you'll create a directory within your path to store the necessary programs for your **local validator**.

```
mkdir ~/.local/share/metaplex-local-validator
```

Then, download the program data from specified addresses into this directory.

```
solana program dump -u m metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s ~/.local/share/metaplex-local-validator/mpl-token-metadata.so
```
```
solana program dump -u m BGUMAp9Gq7iTEuizy4pqaxsTyUCBK68MDfK752saRPUY ~/.local/share/metaplex-local-validator/mpl-bubblegum.so
```
```
solana program dump -u m CoREENxT6tW1HoK8ypY1SxRMZTcVPm7R94rH4PZNhX7d ~/.local/share/metaplex-local-validator/mpl-core.so
```

### Creating a Validator Script

Next, create a **validator script** that simplifies the process of running your local validator with all the required programs. By scripting the validator setup, you can easily start testing with your personalized environment, including all relevant Metaplex programs.

Start by opening a new script file using:

```
sudo nano /usr/local/bin/metaplex-local-validator
```

**Note**: If the /user/local/bin directory doesn’t exist, you can create it using `sudo mkdir -p -m 775 /usr/local/bin.`

Paste in the following code into the editor and save it:

```bash
#!/bin/bash

# Validator command
COMMAND="solana-test-validator -r --bpf-program metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s ~/.local/share/metaplex-local-validator/mpl-token-metadata.so --bpf-program BGUMAp9Gq7iTEuizy4pqaxsTyUCBK68MDfK752saRPUY ~/.local/share/metaplex-local-validator/mpl-bubblegum.so --bpf-program CoREENxT6tW1HoK8ypY1SxRMZTcVPm7R94rH4PZNhX7d ~/.local/share/metaplex-local-validator/mpl-core.so"

# Append any additional arguments passed to the script
for arg in "$@"
do
    COMMAND+=" $arg"
done

# Execute the command
eval $COMMAND
```

**Note**: To exit and save, use Ctrl + X, then Y to confirm, and Enter to save.

Once your script is ready, modify its permissions so it can be executed:

```
sudo chmod +x /usr/local/bin/metaplex-test-validator
```

Finally, test your new validator within your project folder:

```
metaplex-test-validator
```