---
title: Building Permits
description: >
  Issued building permits including construction type, location, contractor,
  estimated valuation, and current status. Updated weekly.
organization: Planning Department
category: Planning & Zoning
license: Public Domain
maintainer: Carlos Rivera
maintainer_email: crivera@example.gov
updated: 2026-02-22
created: 2019-01-01
version: "2024"
tags:
  - permits
  - construction
  - planning
  - zoning
preview: building-permits
resources:
  - name: Building Permits CSV
    url: https://raw.githubusercontent.com/YOUR_ORG/YOUR_REPO/main/datasets/building-permits.csv
    format: CSV
    description: Full permit dataset in comma-separated values format
---

The Building Permits dataset records all permits issued by the Planning Department
for construction, renovation, and demolition projects within city limits.

## Notes

- `status` values: Issued, Under Review, Expired, Finalled
- Valuations are contractor-reported estimates at time of application
- Data refreshed weekly every Monday at 06:00 AM
