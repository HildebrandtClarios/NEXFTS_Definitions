# NEXFTS - Neutral Exchange Format for Battery Test Sequences

## What is NEXFTS?

**NEXFTS** (Neutral Exchange Format for Battery Test Sequences) is a YAML-based format for the complete, machine-readable description of battery test sequences.

A **NEXFTS** file defines all test steps (discharge, charge, rest phases, loops,
jumps) including temperatures, currents, voltages and abort criteria.

It can be used as Basis for official documentation in international standards
and for automated generation of Device-specific test implementation by converter scripts

**NEXFTS** replaces error-prone manual transcription from tables in international standards.

**NEXFTS** ensures that test procedures can be carried out correctly and in accordance with standards

**NEXFTS** enables the release of official versions via an open-source platform, such as GitHub

## NEXFTS uses YAML files for test descriptions

A YAML file is a human-readable text file used to store structured data (YAML stands for: YAML Ain’t Markup Language)
* Readable: uses indentation instead of brackets
* Hierarchical: represents nested data
* Minimal syntax: easy to write and review

## NEXFTS comes with a schema-file

A schema file defines the rules and structure that a YAML file must follow.

* It is describing what fields are allowed, their types, and constraints
* It is used to validate the syntax of the NEXFTS-YAML file.
* It uses JSON (JavaScript Object Notation) for rules definition
* It ensures NEXFTS-Files without formal errors

