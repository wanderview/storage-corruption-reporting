# TAG Security & Privacy Questionnaire Answers #

* **Author:** wanderview@google.com
* **Questionnaire Source:** https://www.w3.org/TR/security-privacy-questionnaire/#questions

## Questions ##

* **What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?**
  * This API exposes when the browser encounters unrecoverable corruption in storage APIs.  It also reports whether the browser wiped storage or left it in its broken state.  Finally, the reporting also includes a free form field where the browser can include debugging information to be pasted into bug reports on the browser engine; e.g. schema version mismatches, checksum failures, etc.
* **Is this specification exposing the minimum amount of information necessary to power the feature?**
  * I believe so.  Knowing where corruption occurred and whether data has been wiped is important for sites to be able to manage state on client devices.  Exposing debug information is important for browser to actually be able to fix corruption issues in the field.
* **How does this specification deal with personal information or personally-identifiable information or information derived thereof?**
  * There shold not be able PII involved in this API.  Browsers must be careful not to include this kind of information by accident in the free form debug field.
* **How does this specification deal with sensitive information?**
  * This API does not handle or report sensitive information.
* **Does this specification introduce new state for an origin that persists across browsing sessions?**
  * The reporting API endpoint is persisted in the [reporting cache](https://w3c.github.io/reporting/#reporting-cache).  This is existing in the reporting API spec and not newly introduced in this proposal.  The reporting API also specifies the reporting cache must be cleared if the user clear's the origin's storage/cookies.  The reporting API spec also requires browsers to provide a mechanism to opt-out of reporting.
* **What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?**
  * In the common case none.  If corruption occurs, then the debug free form field could consider information regarding the integrity of storage managed by the browser.  There is no mandated level of granularity in the debug field.
* **Does this specification allow an origin access to sensors on a user’s device**
  * No.
* **What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.**
  * This proposal exposes the corruption detection via events to the origins using the ReportingObserver API.  This is information that could likely already be observed through trying to use the storage subsystem and receving an exceptional failure.
* **Does this specification enable new script execution/loading mechanisms?**
  * No.
* **Does this specification allow an origin to access other devices?**
  * No.
* **Does this specification allow an origin some measure of control over a user agent’s native UI?**
  * No.
* **What temporary identifiers might this this specification create or expose to the web?**
  * The proposal does not include any new temporary identifiers.
* **How does this specification distinguish between behavior in first-party and third-party contexts?**
  * No.  The behavior should follow the behavior of Reporting API in first-party and third-party contexts which does not distinguish between these contexts currently.
* **How does this specification work in the context of a user agent’s Private \ Browsing or "incognito" mode?**
  * In theory there is no difference in private browser modes.  Given most private browsing implementations implement storage in memory, however, it much less likely any corruption would actually occur.
* **Does this specification have a "Security Considerations" and "Privacy Considerations" section?**
  * Yes.
* **Does this specification allow downgrading default security characteristics?**
  * No.
