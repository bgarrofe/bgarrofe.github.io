# WCM Snowflake Naming Standard v1.3.11 | AI Instructions

Canonical file: `WCM_Snowflake_Naming_Contract_v1.3.11.json`  
Canonical SHA-256: `8985a706ef943097a35bee367c516accfb5e78f7580719072b8fd88b39162dd9`  
Status: PROPOSED_FOR_TEAM_REVIEW; effective date not yet set  
Date: 2026-11-19 | Supersedes: v1.3.10

**Source of truth:** The companion canonical JSON controls. This Markdown is its readable explanation. Names described as proposed are not deployed names. No object is renamed or moved by publishing this contract.

**v1.3.11 change:** New WCM Streamlit apps belong in `PROD.STREAMLIT` (and `DEV.STREAMLIT` for development equivalents), rather than `DATALAKE`. Truly app-private settings, audits, and procedures may live alongside the app. Shared crosswalk data remains in `MAPPINGS`; existing DATALAKE data and pipelines remain there. This is naming guidance only: no objects, grants, or references are moved automatically. All unrelated rules are unchanged.

## 1. Name construction
- **Fully qualified:** `<DATABASE>.<SCHEMA>.<OBJECT>`
- **Environment:** environment: DEV or PROD
- **Schemas:** Use owning business-domain schemas for data/processes; `MAPPINGS` for proposed shared crosswalks; `STREAMLIT` for new app definitions and strictly app-private support objects (`PROD.STREAMLIT`, optionally `DEV.STREAMLIT`); `BI` for consumer-facing interfaces. `DATALAKE` retains existing shared data and pipelines.
- **Data:** `<TYPE>_<PURPOSE>_<SUBJECT>`
- **Source data:** `<TYPE>_SRC_<ORIGIN>[_<CHANNEL>]_<SUBJECT>`
- **Process:** `<TYPE>_<ACTION>_<SUBJECT>`
- **Support:** `<TYPE>_<SUBJECT>`
- **Account-level:** `<TYPE>_<SUBJECT>[_<ENV>]`
- **Client view:** `<DATABASE>.BI.<BUSINESS_NAME>`
- **Case:** UPPER_SNAKE_CASE; unquoted identifiers; A-Z, 0-9 and single separators only (no leading, trailing or repeated underscores)
- **Identifier length:** Snowflake allows 255 characters per identifier; no smaller WCM length cap. The three-part qualified name has no extra combined 255-character cap.

## 2. Physical type prefixes
| Prefix | Type | Class |
| --- | --- | --- |
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
| --- | --- |
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
| --- | --- |
| `LOAD` | bring data in from outside Snowflake |
| `REFRESH` | recompute a result from its sources; safe to rerun |
| `VALIDATE` | compare or reconcile; writes evidence, not business data |
| `PUBLISH` | promote an approved result into its FACT or DIM object |
| `MONITOR` | inspect health or state and record findings |
| `NOTIFY` | send an alert or message |
| `ARCHIVE` | move data to long-term storage |
| `PRUNE` | delete data that is past retention |
| `RUN` | coordinate or execute a workflow consisting of multiple dependent steps |

New action verbs require a distinct justified definition and team agreement; do not silently invent synonyms. For a new one-to-one task/procedure pair, use the same action and subject. Use `RUN` when coordinating dependent steps. Existing deployed pairs are not silently renamed.

## 5. Shared mappings, Streamlit applications, and pluralization
**Schema rule:** `MAPPINGS` is the proposed shared business crosswalk schema, not the app schema. New WCM-owned Streamlit apps, including the mappings app, belong in `<DATABASE>.STREAMLIT`. Only strictly app-specific configuration, audit objects, and procedures belong alongside the app. Shared data, cross-app services, and pipelines remain in their owning domain or DATALAKE schemas. No separate `MAPPINGS_APP` schema is proposed, and no existing objects are moved by adopting this standard.

**Shared crosswalk:** `<DATABASE>.MAPPINGS.<TYPE>_MAP_<PLURAL_SUBJECT>`, for example `PROD.MAPPINGS.TBL_MAP_MODEL_ACCOUNTS`.

**Domain-only crosswalk:** `<DATABASE>.<DOMAIN_SCHEMA>.<TYPE>_MAP_<SUBJECT>`, for example `PROD.AUM.TBL_MAP_WRAP_OVERRIDES`.

**Streamlit application schema:** `<DATABASE>.STREAMLIT`, using `PROD.STREAMLIT` and `DEV.STREAMLIT` when a development equivalent is appropriate. Proposed app: `PROD.STREAMLIT.MAPPINGS_UPLOADER` (illustrative name only). App-private settings: `PROD.STREAMLIT.TBL_CFG_MAPPING_SETTINGS`; app-private batch procedure: `DEV.STREAMLIT.SP_RUN_MAPPING_BATCH`. These examples are naming proposals, not deployed names. Streamlit app objects use descriptive `UPPER_SNAKE_CASE` names without an extra `STREAMLIT_` prefix or mandatory data/process prefix. Supporting tables/procedures retain normal type and purpose/action prefixes.

**Existing deployed names:** `PROD.MAPPINGS` contains archived legacy objects in the supplied live catalog, not proof of active shared crosswalks. `DEV.MAPPING_APP` is an observed legacy schema; its current names and references remain unchanged without a separately approved migration. Existing DATALAKE data/process objects remain where deployed. This document does not verify where every Streamlit app is currently deployed or whether `PROD.STREAMLIT` already exists.

**Migration and permissions:** A naming contract is not a deployment plan. Before moving any app, inventory its current object, app source files/stages, owner role, `CREATE STREAMLIT` and schema permissions, `USAGE` grants, dependency references, app URLs, shared data access, and workflows. Create and validate the replacement, test owner-rights behavior, then cut over only with explicit engineering approval. Do not grant broad data rights just to move an app.

**Shared mapping metadata:** owner, business keys, conflict handling, downstream use, permissions/grants.

**Pluralization:** Datasets and collections take a plural subject (ACCOUNTS, MAPPINGS, VALUATIONS, MARKET_VALUES, STATUSES). Business measures and aggregate metrics remain singular (AUM, MARKET_VALUE when naming a single metric). Process actions remain singular. Use the explicit approved subject dictionary below for established exceptions.

### Approved subject dictionary
| Subject | Fixed meaning |
| --- | --- |
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

**Example:** `PROD.BI.MONTH_END_AUM`.

## 7. Source origin versus ingestion channel
ORIGIN represents the named producing source system or accountable data supplier. CHANNEL is an optional separately identified ingestion path. File forwarding or uploading alone does not establish the original producer.

**Unverified origin:** If the origin or original producer is unknown, do not mark a specific origin as confirmed. Label any candidate naming PROVISIONAL_ORIGIN_UNVERIFIED and verify before executable new production DDL.

**Approved channel tokens:** `WCMUPLOADER`, `SFTP`, `API`, `REPLICATION`.

**Model AUM evidence:** WCM transmitted the quarterly workbook in April 2026. June AUM Jira and November uploader activity identify the file family, not the original compiler.

**FactSet ETF verified refresh chain:** `PROD.FACTSET.TASK_REFRESH_MARKET_VALUE_ETF -> PROD.DATALAKE.SP_REFRESH_MARKET_VALUE_ETF() -> PROD.DATALAKE.FACTSET_SECURITY_ATTRIBUTES + ETF account reference mapping -> PROD.DATALAKE.MARKET_VALUE_ETF.`

**FactSet candidate-name caution:** Documents establish FactSet lineage and reference-mapping responsibility; these are naming proposals, not evidence of new deployed objects or authority to alter the pipeline.

## 8. Lifecycle, environment, and migration
- **DEV / PROD:** The same schema and object name is used in DEV and PROD. The environment appears only in the database name.
- **Account-level objects:** Roles, warehouses and integrations have no database. Add the environment as the last token only when the object is environment-specific (WH_AUM_PROCESSING_DEV). Existing roles and warehouses keep their names.
- **_V<n>:** Side-by-side development or evaluation candidate. May exist temporarily in DEV or PROD while evaluated; once promoted as canonical, the PROD name must be unversioned.
- **Z_ARCHIVE_<ORIGINAL_NAME>_<YYYY_MM_DD>:** Preferred format for new dated archive objects; matches the predominant WCM practice, e.g. Z_ARCHIVE_WEBSITE_RETURNS_DAY_VW_2026_11_10.
- **Forbidden permanent suffixes:** Avoid permanent NEW/FINAL/FIX/OLD/TEST/TMP/COPY/BAK suffixes on canonical objects. Preserve legacy naming where no controlled migration is planned.
- **Approved production cutover:** At approved cutover, the canonical production object must use its unversioned name. A production _V2 candidate is not automatically a violation simply because it exists.
- **Existing objects:** Existing deployed and third-party object names are not automatically noncompliant or subject to rename. Reference their current names in executable SQL. New names are examples until an explicitly planned migration.
- **Migration:** Inventory current objects and dependencies, update consumers and grants as needed, verify behavior, and cut over deliberately. No automatic DROP or rename is authorized by this standard.
- **Type changes:** For an explicitly approved physical type change, deploy the correctly prefixed replacement, update all references to the new name, then drop the obsolete object after cutover. Never leave an incorrectly prefixed object as the final implementation.
- **Compatibility:** A temporary, read-only pass-through view retaining the old interface may be used only when an existing external consumer requires it; name its retirement point. It is an exception, not the final desired design.

## 9. Structurally valid names (examples, not existence claims)
| Proposed / observed name | Type | Purpose or note |
| --- | --- | --- |
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
| `PROD.STREAMLIT.TBL_CFG_MAPPING_SETTINGS` | table | Illustrative configuration private to the mappings Streamlit app; not a deployed object or shared crosswalk. |
| `DEV.STREAMLIT.TBL_AUDIT_MAPPING_RUNS` | table | Illustrative DEV audit history private to a Streamlit app; no existing object is moved. |
| `DEV.AUM.TBL_CFG_AUM_REFRESH` | table | AUM refresh configuration; AUM remains singular. |
| `DEV.AUM.TBL_AUDIT_AUM_REFRESH_RUNS` | table | History of AUM refresh runs. |
| `DEV.AUM.SP_REFRESH_MODEL_AUM` | stored procedure | Refreshes Model AUM; already exists in DEV.AUM. |
| `DEV.AUM.TASK_REFRESH_AUM` | task | Scheduled task to refresh AUM; already exists in DEV.AUM. |
| `DEV.AUM.SP_RUN_AUM_WORKFLOW` | stored procedure | Orchestrates a multi-step AUM workflow. |
| `DEV.STREAMLIT.SP_RUN_MAPPING_BATCH` | stored procedure | Illustrative app-private mapping procedure, not a shared data pipeline or deployed-name claim. |
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

## 10. Corrections
| Avoid for new proposals | Use instead | Reason |
| --- | --- | --- |
| `FACT_MONTH_END_AUM` | `TBL_FACT_MONTH_END_AUM` | Tables use a physical TBL_ prefix. |
| `VW_MODEL_AUM` | `VW_CALC_MODEL_AUM` | Data objects need one meaningful business purpose. |
| `TBL_FACT_CALC_MODEL_AUM` | `TBL_FACT_MONTH_END_AUM or TBL_CALC_MODEL_AUM` | Choose one purpose based on responsibility. |
| `PROD.MAPPINGS.TBL_MODEL_ACCOUNTS` | `PROD.MAPPINGS.TBL_MAP_MODEL_ACCOUNTS` | MAP identifies a crosswalk; MAPPINGS identifies the shared namespace. |
| `PROD.MAPPINGS_APP.TBL_CFG_SETTINGS proposed as a new application schema` | `PROD.STREAMLIT.TBL_CFG_MAPPING_SETTINGS` | A new Streamlit app and strictly app-private configuration belong in `STREAMLIT`, while shared crosswalks stay in `MAPPINGS` and shared pipelines remain in their existing schemas. No automatic migration of `DEV.MAPPING_APP` or other deployed objects. |
| `TBL_MAP_SEI_ACCT_MAPPING` | `TBL_MAP_SEI_ACCOUNTS` | Use full ACCOUNT and avoid repeating MAP or MAPPING. |
| `VW_SRC_SFTP_WRAP_ACCOUNTS` | `VW_SRC_SEI_SFTP_WRAP_ACCOUNTS` | Identify the supplier before the transfer channel. |
| `VW_SRC_STREAMLIT_WRAP_ACCOUNTS` | `VW_SRC_SEI_WCMUPLOADER_WRAP_ACCOUNTS` | WCMUPLOADER identifies the application, not its current UI framework. |
| `PROD.BI.VW_FACT_MONTH_END_AUM` | `PROD.BI.MONTH_END_AUM` | Consumer-facing BI views use plain business names. |
| `TBL_FACT_MONTH_END_AUM_NEW` | `DEV.AUM.TBL_FACT_MONTH_END_AUM_V2 for evaluation; PROD.AUM.TBL_FACT_MONTH_END_AUM after approved cutover` | Use an evaluation version temporarily and remove it from the canonical production name. |
| `SP_MODEL_AUM` | `SP_REFRESH_MODEL_AUM` | Process objects need an action. |
| `VW_CALC_MODEL_AUM when deployed as dynamic table` | `DYN_CALC_MODEL_AUM` | The physical prefix must match the actual object type. |

## 11. Conditional examples (not approved without verification)
| Candidate name | Reason | Verification |
| --- | --- | --- |
| `DEV.AUM.VW_SRC_WCM_WCMUPLOADER_MODEL_ACCOUNTS` | WCM was observed transmitting the quarterly Model workbook, but original producer is unverified; name remains provisional. | ORIGIN_UNVERIFIED |

## 12. DEV.AUM inventory from the November 18 live report
14 data objects (11 views and 3 base tables) were reported. Types are observed; proposed names are not deployed. The inventory does not constitute a new production scan.

| Existing | Type | Candidate or decision |
| --- | --- | --- |
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

## 13. Live findings relevant to the standard
- `PROD.DATALAKE.APEX_DT` is an observed `VIEW`; `VW_` is the proposed prefix.
- 249 of 290 archived objects in supplied report used _YYYY_MM_DD; existing variants remain unchanged.
- PROD.BI contains tables, dynamic tables and a procedure, not only views.
- Checker defaults to AUM and BI; other schemas including RIDGELINE and SF2SF are outside the default scope. The scoped checker does not require redundant exclusions.
- Observed mapping schemas: `PROD.MAPPINGS`, `DEV.MAPPING_APP`. Proposed app target: `PROD.STREAMLIT` (and `DEV.STREAMLIT` where appropriate). These schema targets are not verified deployed names, and no migration is implied.
- Original checker results: 1,573 observed catalog objects; results are not production defect counts.
- Documented WCM ETF migration specifies native FACTSET_SECURITY_ATTRIBUTES input, refresh procedure and mapping reference; accepted as established project lineage.

## 14. AI assistant behavior and advisory SQL review
**New-name response format:** `Fully qualified name | object type | business purpose or action | rationale | verification status`.

**Deployed SQL:** Keep deployed names unchanged unless migration was explicitly requested.

**Uncertainties:** Clearly label uncertain source owner, object type or provenance as UNVERIFIED; never guess a production prefix.

- Do not claim that a proposed name exists in Snowflake.
- Do not alter deployed objects without explicitly authorized deployment.
- Do not assert the original Model workbook compiler or producer from a forwarding email.
- Do not treat a PROD _V2 evaluation candidate as an automatic violation; after cutover the canonical production name is unversioned.
- Do not use consumer-facing BI objects as inputs to newly authored internal SQL.

**Conformance report:** The unchanged `WCM_Snowflake_Naming_Conformance_Check_v1.3.9.sql` is read-only; default scope is DEV/PROD AUM and BI. It is a prior-version supporting tool, not a rebuilt v1.3.11 checker. v1.3.7 checker ran live. v1.3.9 is modified offline and must be checked live to confirm its account-level behavior. The checker cannot authorize renames, deletion, or new DDL; it does not validate proposed Streamlit app placement. Expand scope only intentionally.

## 15. Still unverified
- **MODEL_ORIGIN:** Confirm which organization/system originally produces and calculates Model AUM Quarterly Summary; WCM sending the workbook is insufficient proof.
- **LEGACY_DEV_MAPPING_APP:** DEV.MAPPING_APP is observed as deployed but its active use and migration needs have not been verified by this document update; preserve deployed references unless separately reviewed.

## 16. Contract provenance and artifact status
The authoritative JSON is `WCM_Snowflake_Naming_Contract_v1.3.11.json` with SHA-256 `8985a706ef943097a35bee367c516accfb5e78f7580719072b8fd88b39162dd9`. This document is `WCM_Snowflake_Naming_AI_Instructions_v1.3.11.md`. The v1.3.9 SQL checker and PDF were not regenerated and should not be represented as v1.3.11 deliverables. No Snowflake changes, GitHub publication, or production verification have been performed.

**Final rule:** Do not invent sources, presume file provenance, use proposed names as though deployed, or perform production changes without explicit authorization.
