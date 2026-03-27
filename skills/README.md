# CIDOC-CRM & LINCS Skills

Claude Code skills for working with CIDOC-CRM ontology and LINCS (Linked Infrastructure for Networked Cultural Scholarship) application profiles.

## Skills

| Skill | Slash Command | Description |
|-------|---------------|-------------|
| [cidoc-crm.md](cidoc-crm.md) | `/cidoc-crm` | CIDOC-CRM v7.3.1 ontology reference — classes, properties, CRMgeo/CRMdig extensions, modeling decision guide, anti-patterns |
| [lincs-profile.md](lincs-profile.md) | `/lincs-profile` | LINCS application profile — 8 reusable patterns, person/organization/government/document templates from all 3 History of Canada datasets |
| [lincs-validate.md](lincs-validate.md) | `/lincs-validate` | Model validator — 9-category checklist producing PASS/WARN/FAIL report against LINCS requirements |
| [cidoc-to-rdf.md](cidoc-to-rdf.md) | `/cidoc-to-rdf` | Neo4j → RDF/Turtle converter — 8 translation rules for namespace mapping, relationship property reification, authority URI insertion |
| [lincs-sparql.md](lincs-sparql.md) | `/lincs-sparql` | SPARQL query builder — 10 templates for the LINCS Fuseki endpoint plus local census data queries and LINCS↔census bridging queries |

## Usage

### As Claude Code slash commands
```
/cidoc-crm E93_Presence
/lincs-profile how to model an occupation
/lincs-validate my-model.ttl
/cidoc-to-rdf Westmeath 1871 census data
/lincs-sparql find all people born in Saskatchewan
```

### As reference documents
Each file is self-contained markdown with tables, Turtle examples, and decision guides. Read them directly in any editor or browser.

## Sources

Built from analysis of:
- [LINCS Historical Canadian Persons](https://lincsproject.ca/docs/explore-lod/project-datasets/hist-cdns/)
- [LINCS Historical Indian Affairs Agents](https://lincsproject.ca/docs/explore-lod/project-datasets/ind-affairs/)
- [LINCS Cabinet Conclusions](https://lincsproject.ca/docs/explore-lod/project-datasets/cabinet-conclusions/)
- CIDOC-CRM v7.3.1 specification (ISO 21127)
- CRMgeo geospatial extension
- CRMdig digital extension
- Canadian Historical GIS (CHGIS) Temporal Census Polygons project
