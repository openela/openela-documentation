<!--
SPDX-FileCopyrightText: 2023,2024 Oracle and/or its affiliates.
SPDX-License-Identifier: CC-BY-SA-4.0
-->
# Setting Up TLS/SSL With OpenSSL

## Creating Key Pairs

## Creating Certificate Signing Requests With OpenSSL

## Signing Certificates With OpenSSL

### Creating Self-Signed Certificates for Testing and Development

### Creating a Private Certification Authority

#### Create the CA Root

##### Create a CA Directory Structure

##### Create a CA Root Configuration File

##### Create and Verify the CA Root Key Pair

#### Create an Intermediary CA

##### Create a CA Directory Structure

##### Create the Intermediary CA Configuration

##### Create a CSR for the Intermediary CA

##### Create a Signed Certificate for the Intermediary CA

##### Create a Certificate Chain File

#### Process CSRs and Sign Certificates

#### Manage a Certificate Revocation List

##### Generate the CRL

##### Revoke a Certificate

#### Configure and Run an OCSP Server

## Debugging and Testing Certificates With OpenSSL

### Examining Certificates

### Check That a Private Key Matches a Certificate

### Changing Key or Certificate Format

### Check Certificate Consistency and Validity

### Decrypting Keys and Adding or Removing Passphrases

### Using OpenSSL to Test SSL/TLS Configured Services

## Using OpenSSL for File Encryption and Validation

## More Information About OpenSSL

