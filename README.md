# Grant Application Completeness Assistant

A structured specification for building a Grant Application Completeness Assistant that helps reviewers assess grant applications against supplied grant guidelines.

## Overview

The system is designed as an evidence-review and completeness tool.

It focuses on:

- Extracting requirements from supplied grant guidelines
- Mapping application evidence to requirements
- Identifying satisfied, missing, weak, and ambiguous evidence
- Detecting unsupported claims
- Tracking required supporting documents
- Generating clarification questions
- Supporting human review of AI-generated suggestions
- Maintaining audit history and version information
- Calculating completeness deterministically

The system must not make authoritative legal, regulatory, compliance, or funding-eligibility decisions.

## Core Workflow

```text
Grant Guideline
       ↓
Requirement Extraction
       ↓
Application Evidence Extraction
       ↓
Requirement Mapping
       ↓
Unsupported Claim Detection
       ↓
Clarification Questions
       ↓
Human Review
       ↓
Deterministic Completeness
       ↓
Reviewed Report
