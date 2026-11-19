# WCM Snowflake Naming Standard v1.3.9 | AI Instructions
Canonical file: `WCM_Snowflake_Naming_Contract_v1.3.9.json`  
Canonical SHA-256: `a50a25a32b78ecc6077bff4caae078bf9ac07c3d2df29587f56d96fea8ce4637`  
Status: PROPOSED_FOR_TEAM_REVIEW; effective date not yet set

Attach the canonical JSON to ChatGPT, Claude, Grok or an IDE coding assistant and use it as the source of truth. This file is a readable explanation derived from that JSON. Proposed names are not deployed names.

## 1. Name construction
- **Fully Qualified:** `<DATABASE>.<SCHEMA>.<OBJECT>`
- **Data:** `<TYPE>_<PURPOSE>_<SUBJECT>`
- **Source Data:** `<TYPE>_SRC_<ORIGIN>[_<CHANNEL>]_<SUBJECT>`
- **Process:** `<TYPE>_<ACTION>_<SUBJECT>`
- **Support:** `<TYPE>_<SUBJECT>`
- **Client View:** `<DATABASE>.BI.<BUSINESS_NAME>`
- **Case:** `UPPER_SNAKE_CASE; unquoted identifiers; A-Z, 0-9 and single separators only (no leading, trailing or repeated underscores)`
- **Identifier Length:** `Snowflake allows 255 characters per identifier; no smaller WCM length cap. The three-part qualified name has no extra combined 255-character cap.`

## 2. Physical type prefixes
| Prefix | Type | Class |
|---|---|---|
| `TBL_` | table | data |
| `VW_` | view | data |
| `DYN_` | dynamic table | data |
| `MVW_` | materialized view | data |
| `SP_` | stored procedure | process |
| `TASK_` | task | process |
| `PIPE_` | pipe | process |
| `TMP_` | temporary table | support |
| `FN_` | function / UDF | support |
| `STREAM_` | stream | support |
| `STAGE_` | stage | support |
| `FF_` | file format | support |
| `SEQ_` | sequence | support |
| `ALERT_` | alert | support |
| `TAG_` | governance tag | support |
| `POLICY_` | masking, row access or other schema-level policy | support |
| `ROLE_` | role | account |
| `WH_` | warehouse | account |
| `INTEG_` | integration | account |

## 3. Business-purpose prefixes
| Purpose | Meaning |
|---|---|
| `SRC_` | incoming data or a normalized source-facing view of it; always followed by the origin token |
| `CALC_` | intermediate business logic or its stored result; not the governed consumer contract |
| `FACT_` | approved consumer-facing measurement at a documented grain, even if computed or covering many years |
| `HIST_` | intentionally preserved earlier versions, calculation states, or provenance |
| `DIM_` | descriptive business entity with analytical attributes |
| `LOOKUP_` | small approved list that decodes a code into a label, type, or status |
| `MAP_` | crosswalk that links one identifier to another |
| `CFG_` | settings and parameters that change how a process runs |
| `AUDIT_` | run history, validation results, and reconciliation evidence |

## 4. Approved process actions
| Action | Meaning |
|---|---|
| `LOAD` | bring data in from outside Snowflake |
| `REFRESH` | recompute a result from its sources; safe to rerun |
| `VALIDATE` | compare or reconcile; writes evidence, not business data |
| `PUBLISH` | promote an approved result into its FACT or DIM object |
| `MONITOR` | inspect health or state and record findings |
| `NOTIFY` | send an alert or message |
| `ARCHIVE` | move data to long-term storage |
| `PRUNE` | delete data that is past retention |
| `RUN` | coordinate or execute a workflow consisting of multiple dependent steps |

## 5. Shared schemas and natural pluralization
**MAPPINGS:** MAPPINGS is the proposed shared business crosswalk schema, MAPPINGS_APP is the separate application operations schema. Crosswalk datasets use MAP and plural subjects where applicable. Existing objects are not moved merely by adopting a naming standard.
**Shared crosswalk pattern:** `<DATABASE>.MAPPINGS.<TYPE>_MAP_<PLURAL_SUBJECT>, e.g. PROD.MAPPINGS.TBL_MAP_MODEL_ACCOUNTS`
**MAPPINGS_APP:** `<DATABASE>.MAPPINGS_APP. Examples: PROD.MAPPINGS_APP.TBL_CFG_SETTINGS, DEV.MAPPINGS_APP.SP_RUN_MAPPING_BATCH.`
**Deployed legacy mapping app:** PROD.MAPPINGS contains archived legacy objects in the supplied live catalog. DEV.MAPPING_APP exists and remains unchanged until an intentional migration. The audit does not verify an active PROD.MAPPINGS_APP schema.
**Pluralization:** Datasets and collections take a plural subject (ACCOUNTS, MAPPINGS, VALUATIONS, MARKET_VALUES, STATUSES). Business measures and aggregate metrics remain singular (AUM, MARKET_VALUE when naming a single metric). Process actions remain singular. Use the explicit approved subject dictionary below for established exceptions.
### Approved subject dictionary
| Subject | Fixed meaning |
|---|---|
| `APEX_DAILY_VALUE` | Approved established business/source term for PROD.DATALAKE.APEX_DT, even though the view returns many rows. Do not alternate with APEX_DAILY_VALUES. |
| `APEX_VALUATIONS` | Multi-column valuation source view for the legacy CORE APEX valuation logic; different object from APEX_DT. |
| `ETF_MARKET_VALUES` | Dataset with multiple ETF market-value observations; use plural VALUES. |
| `ETF_ACCOUNTS` | Crosswalk of ETF identifiers to WCM accounts; use plural ACCOUNTS. |
| `MODEL_ACCOUNTS` | Uploaded Model account dataset; use plural ACCOUNTS. |
| `WRAP_ACCOUNTS` | Uploaded WRAP account dataset; use plural ACCOUNTS. |
| `MODEL_AUM` | Business AUM measure remains singular. |
| `MONTH_END_AUM` | Governed AUM measurement remains singular. |

### Naming subject guidance
- For a dataset or collection, use plural subjects: MODEL_ACCOUNTS, ETF_MARKET_VALUES, APEX_VALUATIONS, ACCOUNT_STATUSES.
- For a business measure or aggregate, retain singular terms: MODEL_AUM, MONTH_END_AUM. APEX_DAILY_VALUE is a specifically approved source-term exception.
- Preserve established acronyms AUM, ETF, SEI, APEX, WRAP. Use full ACCOUNT rather than ACCT in new names.
- Do not repeat type, purpose, environment, database or schema tokens in the subject. Do not pluralize action verbs.

## 6. BI schema
Use BI for consumer-facing business interfaces, normally plain-name views. Existing BI tables, dynamic tables and procedures remain as deployed. New internal pipelines do not read BI.
New WCM internal calculations, refresh procedures, tasks and transformation SQL must read the owning CORE/DATALAKE/domain objects, not PROD.BI or DEV.BI consumer-facing interfaces. Existing dependencies are not silently rewritten.
Example: `PROD.BI.MONTH_END_AUM`

## 7. Original source versus ingestion channel
ORIGIN represents the named producing source system or accountable data supplier. CHANNEL is an optional separately identified ingestion path. File forwarding or uploading alone does not establish the original producer.
If the origin or original producer is unknown, do not mark a specific origin as confirmed. Label any candidate naming PROVISIONAL_ORIGIN_UNVERIFIED and verify before executable new production DDL.
Approved channels: WCMUPLOADER, SFTP, API, REPLICATION.
Quarterly Model AUM source: WCM transmitted the quarterly workbook in April 2026. June AUM Jira and November uploader activity identify the file family, not the original compiler.
FactSet ETF lineage: PROD.FACTSET.TASK_REFRESH_MARKET_VALUE_ETF -> PROD.DATALAKE.SP_REFRESH_MARKET_VALUE_ETF() -> PROD.DATALAKE.FACTSET_SECURITY_ATTRIBUTES + ETF account reference mapping -> PROD.DATALAKE.MARKET_VALUE_ETF.
FactSet ETF source and mapping: Documents establish FactSet lineage and reference-mapping responsibility; these are naming proposals, not evidence of new deployed objects or authority to alter the pipeline.

## 8. Lifecycle and migration
- **_V<n>:** Side-by-side development or evaluation candidate. May exist temporarily in DEV or PROD while evaluated; once promoted as canonical, the PROD name must be unversioned.
- **Z_ARCHIVE_<ORIGINAL_NAME>_<YYYY_MM_DD>:** Preferred format for new dated archive objects; matches the predominant WCM practice, e.g. Z_ARCHIVE_WEBSITE_RETURNS_DAY_VW_2026_11_10.
- At approved cutover, the canonical production object must use its unversioned name. A production _V2 candidate is not automatically a violation simply because it exists.
- Existing deployed and third-party object names are not automatically noncompliant or subject to rename. Reference their current names in executable SQL. New names are examples until an explicitly planned migration.
- Inventory current objects and dependencies, update consumers and grants as needed, verify behavior, and cut over deliberately. No automatic DROP or rename is authorized by this standard.
**Physical type changes:** For an explicitly approved physical type change, deploy the correctly prefixed replacement, update all references to the new name, then drop the obsolete object after cutover. Never leave an incorrectly prefixed object as the final implementation.
**Compatibility exception:** A temporary, read-only pass-through view retaining the old interface may be used only when an existing external consumer requires it; name its retirement point. It is an exception, not the final desired design.
**Task/procedure names:** For a new single-purpose task that calls one procedure, use TASK_<ACTION>_<SUBJECT> and SP_<ACTION>_<SUBJECT> with the same ACTION and SUBJECT; example: TASK_REFRESH_MODEL_AUM calls SP_REFRESH_MODEL_AUM.
**Orchestrators:** When the task and procedure both coordinate several steps, use TASK_RUN_<WORKFLOW> and SP_RUN_<WORKFLOW>. This is a naming preference for new matched pairs, not a requirement to rename existing working tasks.
**Already deployed task pair:** DEV.AUM.TASK_REFRESH_AUM calls DEV.AUM.SP_REFRESH_MODEL_AUM; their deployed names remain unchanged.

## 9. Structurally valid names (examples, not existence claims)
| Object name | Type | Note |
|---|---|---|
| `DEV.AUM.VW_SRC_SEI_WCMUPLOADER_WRAP_ACCOUNTS` | view | SEI-supplied WRAP account records normalized after upload through the WCM application; proposed name. |
| `PROD.AUM.VW_SRC_INDATA_APEX_DAILY_VALUE` | view | Proposed view over InData-derived APEX daily values; PROD.DATALAKE.APEX_DT is verified to be a view. |
| `PROD.AUM.TBL_MAP_ETF_ACCOUNTS` | table | Example crosswalk linking ETF identifiers to accounts. |
| `DEV.AUM.TMP_FACT_REFRESH` | temporary table | Session-scoped work table for refreshing a fact. |
| `DEV.AUM.VW_CALC_MODEL_AUM` | view | Intermediate Model AUM logic; AUM is a business measure and remains singular. |
| `DEV.AUM.DYN_CALC_MODEL_AUM` | dynamic table | Dynamic-table version of an intermediate Model AUM result. |
| `DEV.AUM.TBL_HIST_MODEL_AUM` | table | Preserved history and calculation provenance for Model AUM. |
| `DEV.AUM.TBL_FACT_MONTH_END_AUM` | table | Month-end AUM measurements at a defined reporting grain. |
| `DEV.AUM.TBL_DIM_ACCOUNTS` | table | Descriptive entity rows for accounts. |
| `DEV.AUM.TBL_LOOKUP_ACCOUNT_STATUSES` | table | Controlled account-status codes and labels. |
| `PROD.MAPPINGS.TBL_MAP_MODEL_ACCOUNTS` | table | Shared mapping records for Model accounts; proposed name, not a deployed table claim. |
| `PROD.MAPPINGS.TBL_MAP_STRATEGIES` | table | Shared strategy identifier crosswalk, collection name is plural. |
| `PROD.AUM.TBL_MAP_WRAP_OVERRIDES` | table | AUM-only override mappings. |
| `PROD.MAPPINGS_APP.TBL_CFG_SETTINGS` | table | Mapping application settings, separate from shared business crosswalks. |
| `DEV.MAPPINGS_APP.TBL_AUDIT_MAPPING_RUNS` | table | Mapping application run history and diagnostics. |
| `DEV.AUM.TBL_CFG_AUM_REFRESH` | table | AUM refresh configuration; AUM remains singular. |
| `DEV.AUM.TBL_AUDIT_AUM_REFRESH_RUNS` | table | History of AUM refresh runs. |
| `DEV.AUM.SP_REFRESH_MODEL_AUM` | stored procedure | Refreshes Model AUM; already exists in DEV.AUM. |
| `DEV.AUM.TASK_REFRESH_AUM` | task | Scheduled task to refresh AUM; already exists in DEV.AUM. |
| `DEV.AUM.SP_RUN_AUM_WORKFLOW` | stored procedure | Orchestrates a multi-step AUM workflow. |
| `DEV.MAPPINGS_APP.SP_RUN_MAPPING_BATCH` | stored procedure | Runs mapping application batch processing; operational rather than business crosswalk data. |
| `DEV.AUM.PIPE_LOAD_MODEL_FILES` | pipe | Loads Model files into an ingest path. |
| `DEV.AUM.FN_LAST_WEEKDAY_OF_MONTH` | function / UDF | Returns the last weekday of a month; functional name stays singular. |
| `DEV.AUM.STREAM_SRC_SEI_WCMUPLOADER_WRAP_ACCOUNTS` | stream | Stream follows a single source dataset. |
| `DEV.AUM.STAGE_MODEL_UPLOADS` | stage | Stage containing Model upload files. |
| `DEV.AUM.FF_MODEL_CSV` | file format | CSV file format definition for Model files. |
| `DEV.AUM.ALERT_AUM_REFRESH_FAILURE` | alert | Alert for failed AUM refresh. |
| `PROD.BI.MONTH_END_AUM` | view | Existing unprefixed client-facing BI view. |
| `DEV.AUM.TBL_FACT_MONTH_END_AUM_V2` | table | Temporary side-by-side candidate during evaluation, not canonical production name. |
| `DEV.AUM.Z_ARCHIVE_TBL_FACT_MONTH_END_AUM_2026_11_18` | table | Archive naming matches the prevalent WCM YYYY_MM_DD style. |
| `PROD.AUM.TBL_SRC_FACTSET_ETF_MARKET_VALUES` | table | Proposed FactSet-derived ETF daily market-values dataset; FactSet origin documented in ETF refresh chain. |
| `DEV.AUM.VW_SRC_FACTSET_ETF_MARKET_VALUES` | view | Proposed DEV view over the FactSet-derived ETF market-values dataset. |
| `DEV.AUM.VW_MAP_ETF_ACCOUNTS` | view | Proposed mapping view over the ETF reference crosswalk. |
| `DEV.AUM.TASK_REFRESH_MODEL_AUM` | task | Illustrative new task paired by suffix with SP_REFRESH_MODEL_AUM; not asserted deployed. |

## 10. Conditional examples (not approved without source verification)
| Candidate name | Why conditional |
|---|---|
| `DEV.AUM.VW_SRC_WCM_WCMUPLOADER_MODEL_ACCOUNTS` | WCM was observed transmitting the quarterly Model workbook, but original producer is unverified; name remains provisional. [ORIGIN_UNVERIFIED] |

## 11. DEV.AUM inventory from the live report
14 data objects (11 views, 3 base tables), submitted in the November 18, 2026 checker result. These types are observed; proposed names are not deployed.
| Existing | Type | Candidate or decision |
|---|---|---|
| `DEV.AUM.CALC_COMBINED_AUM` | VIEW | `VW_CALC_COMBINED_AUM` |
| `DEV.AUM.CALC_MODEL_AUM` | VIEW | `VW_CALC_MODEL_AUM` |
| `DEV.AUM.CALC_MONTH_END_AUM` | VIEW | `VW_CALC_MONTH_END_AUM` |
| `DEV.AUM.CALC_WRAP_AUM` | VIEW | `VW_CALC_WRAP_AUM` |
| `DEV.AUM.FACT_MONTH_END_AUM` | BASE TABLE | `TBL_FACT_MONTH_END_AUM` |
| `DEV.AUM.FACT_MONTH_END_AUM_PARITY_R26` | BASE TABLE | VALIDATION_ARTIFACT |
| `DEV.AUM.HIST_MODEL_AUM` | BASE TABLE | `TBL_HIST_MODEL_AUM` |
| `DEV.AUM.SRC_EXTERNAL_APEX_CANDIDATE` | VIEW | EVALUATION_ONLY |
| `DEV.AUM.SRC_EXTERNAL_APEX_REPORTING` | VIEW | `VW_SRC_INDATA_APEX_DAILY_VALUE` |
| `DEV.AUM.SRC_EXTERNAL_APEX_VALUATIONS` | VIEW | `VW_SRC_INDATA_APEX_VALUATIONS` |
| `DEV.AUM.SRC_EXTERNAL_ETF` | VIEW | `VW_SRC_FACTSET_ETF_MARKET_VALUES` |
| `DEV.AUM.SRC_EXTERNAL_ETF_REFERENCE` | VIEW | `VW_MAP_ETF_ACCOUNTS` |
| `DEV.AUM.SRC_UPLOAD_MODEL` | VIEW | ORIGIN_TO_VERIFY |
| `DEV.AUM.SRC_UPLOAD_WRAP` | VIEW | `VW_SRC_SEI_WCMUPLOADER_WRAP_ACCOUNTS` |

## 12. Live findings relevant to this standard
- APEX_DT is a VIEW (IS_DYNAMIC=NO), therefore a proposed view name uses VW_.
- Archived names predominantly use _YYYY_MM_DD; use that format for new archives.
- BI currently contains tables, dynamic tables and a procedure. Do not report those as automatic failures.
- RIDGELINE and SF2SF have independent naming patterns and are outside the default AUM/BI checker scope.
- The canonical shared schemas are MAPPINGS and MAPPINGS_APP; the current DEV.MAPPING_APP remains as deployed.

## 13. AI assistant behavior
For a proposed name, return its fully qualified name, physical type, purpose or action, a short reason, and verification status. Avoid claiming a proposed object exists. Preserve deployed names unless a migration is explicitly requested.
- Do not claim that a proposed name exists in Snowflake.
- Do not alter deployed objects without explicitly authorized deployment.
- Do not assert the original Model workbook compiler or producer from a forwarding email.
- Do not treat a PROD _V2 evaluation candidate as an automatic violation; after cutover the canonical production name is unversioned.
- Do not use consumer-facing BI objects as inputs to newly authored internal SQL.

## 14. Advisory SQL review
Use `WCM_Snowflake_Naming_Conformance_Check_v1.3.9.sql` only for read-only, scoped review. Edit the `params.schemas` array to expand beyond AUM and BI. Review new object naming in the context of proposed DDL; the catalog alone cannot establish a policy adoption date.
The checker does not perform automatic renaming, deletion or enforcement. v1.3.9 SQL is not yet live-compiled; the previous v1.3.7 was reported to compile live.

## 15. Still unverified
- MODEL_ORIGIN: Confirm which organization/system originally produces and calculates Model AUM Quarterly Summary; WCM sending the workbook is insufficient proof.
- MAPPINGS_APP_DEPLOYMENT: The target application schema name is MAPPINGS_APP. The observed DEV.MAPPING_APP remains as deployed until intentionally migrated.

The JSON is authoritative. Do not invent sources, presume file provenance, or perform production changes without explicit authorization.
