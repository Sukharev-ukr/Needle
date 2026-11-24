# ADR-0000: Azure region

- Status: proposed (fill in after the day-1 check)
- Date: YYYY-MM-DD
- Deciders: team

## Context

All data and processing stay in the EU. Functions run on the Flex Consumption plan, which is only offered in some regions. Static Web Apps, Storage, SQL and the k6 VM must be in the same region as the Function App.

## Options

1. West Europe (NL), if Flex Consumption is offered there.
2. North Europe (IE).
3. Sweden Central.

## Decision

Run the check below and use the first region in the list above that passes. Write the result here.

```bash
az functionapp list-flexconsumption-locations --query "sort_by(@, &name)[].name" -o tsv
```

Also confirm in that region: we may create Azure SQL databases and a small VM (quota), and the free-offer SQL region is fixed per subscription by the first free database.

Result: _region = ..., checked on ..., by ..._

## Consequences

If the region is not West Europe, the report says "EU region" and not "NL".

## Evidence

Output of the command above, saved in `docs/architecture/region-check.txt`.
