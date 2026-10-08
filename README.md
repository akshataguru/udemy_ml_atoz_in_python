Root Cause Analysis – S1 Data Features / Insights Incident
1. Executive Summary
An S1 production incident was identified following the rollout of Data Features used to support customer insights.
The Data Features dataset is produced by the Data Features team and subsequently consumed through multiple downstream components and teams, including PnI, Wealth Engagement/Arion, Investment Funding, Data Hub, Insights and the UI layer.
Important RCA Position
The issues identified during the S1 investigation should not all be classified as Data Features defects.
Specifically:
•	Issue 1 was driven by a business-requirement gap around expected behaviour for accounts with no debit transactions.
•	Issue 2 was driven by a mismatch between the originally agreed account-level data model and a subsequently clarified ECI-level business expectation.
•	Issue 4 was caused by characteristics of the upstream/test dataset, where cloned profiles resulted in duplicated transactions.
•	Issue 5 remains under investigation and currently represents a reconciliation/methodology gap rather than a confirmed Data Features defect.
These issues represent business requirement, data-design, source-data and reconciliation assumptions that were discovered after implementation, rather than coding defects introduced independently by the Data Features team.
Issue 3 did include a Technology implementation/regression component and is acknowledged separately in this RCA.
This distinction is important because the remediation is not limited to code changes. The S1 exposed gaps in:
•	Requirement elaboration
•	Data-grain definition
•	Source-data validation
•	Cross-team data contracts
•	End-to-end UAT
•	Reconciliation methodology
•	Cross-team release ownership
The incident therefore should be viewed as an end-to-end delivery and validation failure across the broader solution, rather than solely as a failure of the Data Features implementation.
A significant contributing factor was that the complete end-to-end use cases were not sufficiently validated in UAT before production deployment.
During initial investigation, it was communicated that the relevant scenarios could not be validated in UAT. Subsequently, the same scenarios were successfully exercised in UAT, demonstrating that an opportunity for pre-production validation existed.
________________________________________
2. Issue Classification
Issue	Classification	Data Features Defect?	Primary Nature
Issue 1 – NULL Insight	Requirement gap	No	Expected behaviour for zero-debit accounts was not fully defined
Issue 2 – Credit Card Balance	Data-model / requirement gap	No	Account-level implementation vs ECI-level expectation
Issue 3 – Insight Not Populated	Integration + implementation issue	Partially / Yes	Account-number casing inconsistency plus cross-team contract gap
Issue 4 – Inflated Monthly Spend	Upstream data-quality issue	No	Cloned profiles resulted in duplicated transactions
Issue 5 – Remaining Spend Difference	Reconciliation gap / Under Investigation	Not established	Comparison methodology with Spend Calculator is not yet understood
Key Conclusion from Classification
Four of the five investigation items are either confirmed or potentially attributable to requirement, architecture, source-data or reconciliation gaps rather than defects in the underlying Data Features calculation.
The RCA should therefore not characterize the incident as five Data Features production defects.
________________________________________
3. System / Data Flow
The high-level flow is:
Data Sources → Data Features → PnI / Data Hub → Wealth Engagement / Arion → Investment Funding / Insights → UI
The Data Features team owns generation of the agreed Data Features according to the requirements and data model provided.
The final customer experience depends on multiple downstream transformations, APIs, lookups and consumers.
Successful validation of the Data Features dataset by itself therefore does not constitute successful end-to-end validation.
The production rollout required validation across:
1.	Source data
2.	Data Features output
3.	PnI/Data Hub processing
4.	Insights processing
5.	Downstream API behaviour
6.	UI rendering
7.	Business-level values
8.	End-to-end customer outcomes
________________________________________
4. Primary Root Cause
Primary Root Cause
The primary root cause was insufficient end-to-end requirement validation and UAT across the complete Data Features-to-UI solution before production rollout.
The incident was not caused by a single code defect.
Several important requirements, data behaviours and integration assumptions were identified only after production deployment, including:
•	Expected behaviour when no debit transactions exist during the 365-day lookback.
•	Account-level versus ECI-level data expectations.
•	Account-number case-sensitivity expectations across downstream systems.
•	Behaviour of cloned employee profiles and duplicated transactions.
•	Expected reconciliation methodology for Monthly Spend.
Root Cause Statement
The S1 occurred because the complete end-to-end business scenarios, data assumptions and integration contracts were not sufficiently defined, validated and signed off before production release.
As a result, behaviours that were consistent with the documented implementation requirements were subsequently found not to match the final business or downstream expectations.
Therefore, Issues 1, 2 and 4 should be treated primarily as requirement/data/design gaps, while Issue 5 remains a reconciliation investigation. They should not be represented simply as Data Features coding defects.
________________________________________
5. Contributing Factors
5.1 Requirement Validation Gap
Some business scenarios were not fully defined during requirement elaboration and became clear only after production behaviour was reviewed.
5.2 End-to-End Testing Gap
Testing was not sufficiently executed across the complete customer journey before production rollout.
5.3 Cross-Team Data Contract Gap
There was no universally agreed and enforced contract for attributes such as account-number casing across PnI, Data Hub and Insights.
5.4 Source/Test Data Gap
The employee test population contained cloned profiles and cloned transactions. This data characteristic was not sufficiently understood before the dataset was used for Monthly Spend calculations.
5.5 Reconciliation Gap
Monthly Spend was compared with another application without first confirming that both systems used equivalent data sources, calculation rules and aggregation logic.
5.6 Ownership Gap
No single accountable owner ensured that business requirements, source data, Data Features, downstream processing and final UI behaviour were collectively validated before release.
________________________________________
6. Issue 1 – Insight Returned as NULL
Observation
For certain accounts, the UI displayed no insight and the corresponding insight lookup returned NULL.
Existing Requirement
The agreed requirement stated that if an account had no debit transactions during the 365-day lookback period, that account should be removed from the final output.
The Data Features implementation followed this requirement.
Once the account was removed from the dataset, a downstream lookup against that account naturally returned NULL.
RCA Classification
Requirement Gap – Not a Data Features Coding Defect
The implementation behaved consistently with the documented requirement.
The gap was that the requirement did not define the downstream/customer-experience expectation for such accounts.
The business expectation was subsequently clarified to require the account to remain in the output with a value of zero.
Root Cause
The zero-debit scenario and its downstream impact were not identified during requirement elaboration or end-to-end UAT.
Resolution
The requirement was enhanced.
Instead of removing the account:
•	The account is retained.
•	Applicable balance/Monthly Spend is populated as 0.
Action Items
•	Product to document explicit behaviour for zero-transaction and no-activity scenarios.
•	Product and Technology to review edge cases during requirement elaboration before development starts.
•	Add zero-debit, zero-credit, dormant-account and missing-history scenarios to UAT acceptance criteria.
•	Ensure downstream impact is reviewed whenever records are intentionally filtered from a dataset.
________________________________________
7. Issue 2 – Credit Card Balance at Account Level vs ECI Level
Observation
For a credit-card account associated with two ECIs, the subsequent expectation was:
•	Each ECI should have its own Monthly Spend.
•	Both ECIs should display the overall credit-card account balance.
Existing Requirement / Design
The Data Features implementation was designed at account level, consistent with the original requirement.
An account-level Data Feature cannot inherently provide separate customer/ECI-specific context unless ECI is explicitly introduced as part of the data grain.
RCA Classification
Requirement / Data-Model Gap – Not a Data Features Coding Defect
The implementation reflected the originally agreed account-level grain.
The subsequent expected behaviour introduced a combination of:
•	Account-level balance
•	ECI-level Monthly Spend
That requirement was not represented in the original data model.
Root Cause
The required data grain was not fully defined during design.
The difference between account-level and ECI-level behaviour was discovered after implementation.
Resolution
Product requested that credit-card balance be removed from the Data Features dataset.
Technology proposed sourcing the balance through the existing API used by PnI and combining it with Data Features at the appropriate downstream layer.
Action Items
•	Product and Architecture to explicitly define the grain of every new Data Feature before implementation.
•	Requirements must state whether a field is at ECI, account, ECI-account, household or transaction level.
•	Architecture to validate whether requested fields logically belong in Data Features or should be sourced from an existing API/service.
•	Product to complete data-model review before finalizing acceptance criteria.
________________________________________
8. Issue 3 – Insight Not Populated After Changes
Observation
Following implementation of the preceding changes, insights continued to be unavailable for a small subset of accounts.
Approximately:
•	98% of account numbers were uppercase.
•	~2% were lowercase.
Investigation
The Data Features team had received guidance that account numbers should be represented in uppercase.
During investigation:
•	PnI Data Hub indicated account numbers should be uppercase.
•	Insights expected account numbers in lowercase.
A recent change also resulted in a small portion of records being emitted with inconsistent casing.
RCA Classification
Technology / Integration Issue
Unlike Issues 1, 2 and 4, this issue includes an implementation/regression component and is acknowledged as such.
However, the impact was also enabled by the absence of a consistent cross-team data contract.
Root Cause
Two factors contributed:
1.	Account casing was not consistently defined across PnI/Data Hub/Insights.
2.	A small subset of records was emitted with inconsistent casing during the change.
Resolution
The Data Features, PnI, Insights and Data Hub teams jointly investigated.
PnI/Data Hub subsequently requested that the flow standardize account numbers to lowercase.
The implementation was updated accordingly.
Action Items
•	PnI, Data Hub and Insights to agree on and document a single account-number format.
•	Shared attributes used for joins/lookups must be covered by a formal interface contract.
•	Data Features team to add casing validation to regression testing.
•	Urgent changes must include targeted regression checks for affected fields before release.
•	Peer downstream teams should internally align on interface expectations before communicating requirements to upstream producers.
________________________________________
9. Issue 4 – Inflated Monthly Spend
Observation
Monthly Spend appeared materially higher than expected for certain employee profiles.
Investigation
The employee testing population was generated by cloning customer profiles.
When the profiles were cloned, underlying transactions were also cloned.
For joint accounts, this could result in transactions being represented multiple times.
The Monthly Spend calculation therefore processed duplicated transaction records that existed in the source data.
RCA Classification
Upstream / Test Data Quality Gap – Not a Data Features Calculation Defect
The Data Features calculation processed the records supplied by the selected dataset.
The inflation occurred because duplicated transactions existed upstream as a consequence of the test-profile cloning process.
Root Cause
The characteristics of the employee/test dataset were not sufficiently analyzed before it was selected as a source for this business calculation.
Resolution
The Data Features logic was enhanced to treat cloned profiles appropriately and avoid duplicated spend for individual and joint-account scenarios.
Architectural Observation
Although mitigation was implemented within Data Features, the preferred solution is to eliminate duplication at the dataset/source layer.
Downstream applications should ideally consume canonical data instead of each implementing compensating logic for known source-data duplication.
Action Items
•	Product and Data Owners to validate source-data characteristics before selecting a dataset for a feature.
•	Data Owners to document cloning, duplication and joint-account behaviour.
•	Investigate eliminating duplicated records at the source/dataset layer.
•	Add duplicate-profile and joint-account cases to UAT.
•	Architecture to determine whether compensating logic should remain in Data Features or be removed once the source is corrected.
________________________________________
10. Issue 5 – Remaining Difference in Monthly Spend
Observation
After correcting duplicated-profile behaviour, Monthly Spend values became significantly closer to expectations, but smaller differences remain.
Investigation
The Data Features results are being compared against an existing Spend Calculator application.
However, the Spend Calculator methodology has not yet been fully established.
Differences may arise from:
•	Transaction source
•	Lookback period
•	Transaction categorization
•	Debit filters
•	Refund handling
•	Transfers
•	Joint-account treatment
•	Exclusion rules
•	Aggregation methodology
RCA Classification
Under Investigation – Not Currently Established as a Data Features Defect
Until the Spend Calculator logic is understood and a like-for-like reconciliation is completed, the remaining variance cannot legitimately be classified as a defect in Data Features.
A difference between two applications does not by itself establish that either implementation is incorrect.
Resolution / Next Step
Product is engaging the Spend Calculator team to establish:
1.	Source data
2.	Business rules
3.	Calculation methodology
4.	Inclusion/exclusion logic
5.	Lookback period
6.	Joint-account treatment
7.	Duplicate handling
8.	Reconciliation tolerance
A like-for-like comparison will then be performed.
Action Items
•	Product to obtain the complete Spend Calculator calculation methodology.
•	Establish a documented like-for-like reconciliation process before concluding that a variance is a defect.
•	Define an acceptable reconciliation tolerance.
•	Product and Architecture to assess whether the existing Spend Calculator value can be reused instead of maintaining a second calculation.
•	Complete root-cause classification only after both calculation methodologies have been compared.
________________________________________
11. Why UAT Did Not Identify These Issues
A key question from this incident is why these conditions were discovered after production release.
The primary factors were:
1.	There was no single owner accountable for the complete Data Features-to-UI UAT outcome.
2.	Individual teams primarily validated their respective components.
3.	Some scenarios were absent from the original requirements and acceptance criteria.
4.	Data-grain expectations were not completely established.
5.	Shared data contracts were not fully documented.
6.	Representative edge-case/test data was insufficiently validated.
7.	Benchmark/reconciliation methodology was not agreed in advance.
8.	Some scenarios initially considered unavailable for UAT testing were later successfully tested in UAT.
Key UAT Finding
Had the final business scenarios been exercised end-to-end in UAT, several of the requirement, data and integration assumptions identified during the S1 investigation could likely have been discovered before production.
Accordingly, future releases should require explicit end-to-end UAT evidence rather than relying solely on component-level testing.
________________________________________
12. Consolidated Action Items
#	Action Item	Owner	Priority	Target Date	Status
1	Assign a single accountable owner for end-to-end UAT	Product / Program	High	TBD	Open
2	Create a mandatory cross-team E2E UAT checklist	Product + Technology	High	TBD	Open
3	Require documented UAT evidence before production sign-off	Release Owners	High	TBD	Open
4	Define edge cases during requirement elaboration	Product + Technology	High	TBD	Open
5	Explicitly document the data grain of every feature	Product + Architecture	High	TBD	Open
6	Document shared producer/consumer data contracts	Data Features + PnI + Insights	High	TBD	Open
7	Standardize account-number casing across the flow	PnI / Data Hub / Insights	High	TBD	Open
8	Validate source datasets with Data Owners before development	Product + Data Owner	High	TBD	Open
9	Add cloned-profile, duplicate and joint-account scenarios to UAT	QA / Data Owner	Medium	TBD	Open
10	Add targeted regression validation for urgent code changes	Technology	High	TBD	Open
11	Establish Spend Calculator reconciliation methodology	Product	High	TBD	Open
12	Define reconciliation tolerance and expected variance	Product + Technology	Medium	TBD	Open
13	Evaluate reuse of existing authoritative calculations/services	Architecture + Product	Medium	TBD	Open
14	Review whether duplicated data can be corrected at source	Data Owner	High	TBD	Open
________________________________________
13. Overall RCA Conclusion
The S1 should not be characterized as a collection of Data Features defects.
The investigation demonstrates that:
•	Issue 1 was a requirement gap.
•	Issue 2 was a requirement/data-model gap.
•	Issue 3 contained a Technology implementation issue combined with a cross-team data-contract gap.
•	Issue 4 was an upstream/test-data quality issue.
•	Issue 5 remains a reconciliation investigation and has not been established as a Data Features defect.
Therefore, Issues 1, 2, 4 and potentially 5 represent requirement, data, architecture or validation assumptions that were discovered after implementation rather than independent defects introduced by the Data Features team.
The broader S1 was caused by weaknesses across the end-to-end delivery lifecycle:
Requirement Definition → Data Source Selection → Data Design → Development → Integration → UAT → Reconciliation → Production
The corrective actions identified in this RCA are intended to strengthen this complete lifecycle rather than addressing only individual code changes.
Future releases should not proceed to production until:
•	Business scenarios and edge cases are documented.
•	Data grain is agreed.
•	Source data is validated.
•	Data contracts are agreed by producers and consumers.
•	End-to-end integration is tested.
•	Benchmark/reconciliation methodology is established.
•	UAT evidence is available.
•	Product and Technology jointly approve the final customer outcome.

