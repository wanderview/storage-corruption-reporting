# Storage Corruption Reporting

[Tag Review](https://github.com/w3ctag/design-reviews/issues/419)

## Introduction

When implementing storage APIs, the browser must deal with the possibility of corrupt content on disk.  This can happen for a number of different reasons:

1.  Hardware failures.
1.  External software modifying disk content.
1.  Browser bugs that cause content to be written in a way that prevents it from being read properly.

Currently the various storage specs largely ignore this issue.  As a result, browser implementations largely design and implement their own strategies for dealing with corruption.  These strategies can range from "don’t touch the disk when it's corrupted" to "wipe the entire origin storage partition".

For their part, sites are often left in a difficult position.  First, they have little insight into the problem.  While an individual storage operation may throw, it's not clear if its due to a site software error, a bug in the browser, or corrupt storage.  Also, if the browser chooses to wipe storage, then the site may not have any direct indication that there is a problem at all.  The user on that device would just appear to be effectively gone.

This explainer will propose adding a mechanism for observing when storage corruption happens on a site.  The goal of this is to improve transparency and increase developer trust of the web platform.

## Goals

*   Notify sites when storage corruption has been detected.
*   Notify sites if an action was taken in response to the storage corruption.

## Non-Goals

*   This effort will not formalize the exact actions taken by the browser when storage corruption is detected.  Ultimately this needs to be done, but it may be difficult to agree until we can give sites finer grained policy control with something like storage "buckets" or "boxes".

## API Sketch

We propose to use the existing Reporting API [[0]] to expose corruption events to sites.  The main extension would be to define a new storage corruption report type:


```
    {
    	"type": "storage",
    	"reason": "corruption",
    	"subsystem": "indexeddb",
    	"action_taken": "wiped all",
    	"debug": <browser specific info>
    }
```

Report fields are defined as follows:

 * The "type” value is the main Reporting API key used to separate this kind of report from other kinds of reports.
 * The "reason" value indicates why the report and action were taken.  The only value would be "corruption" for now, but in the future could include values to indicate deletion due to quota eviction or clear-site-data.
 * The "subsystem" value is also an enumeration defining the storage subsystem where the corruption was detected; e.g. "indexeddb", "cache", "localstorage".
 * The "action_taken" value contains an action and a possible list of subsystems affected; e.g. "none", "wiped indexeddb,cache", and "wiped all".  These would indicate if no action was taken, if a particular set of storage subsystems were wiped, or the entire origin was wiped respectively.
 * The "debug" value would be allowed to contain browser-specific information about what failed.  For example, chromium browsers might report that the problem was in LevelDB checksums, etc.  The intent is that sites could copy this information and paste it into a bug for the offending browser engine.

These reports would be available via both the server mechanism defined in the Reporting API [[1]] and the Reporting Observer [[2]].  The Reporting Observer would let sets programmatically process the corruption report on the client.  There would be no timing guarantees about when the report is issued.

The proposal does not include a new header type.  Instead storage reports would go to the "default" report group similar to intervention, deprecation, and crash report types defined in the Reporting API spec.

## Considered Alternatives

### Thrown Exceptions

An alternative would be to try to define a type of exception that must be thrown from various API calls.  This is difficult, though, for a number of reasons:

1.  There may be a large number of methods and call-sites that would need to be modified to process these new exceptions.
1.  Storage corruption can be detected by the browser without a method being called.  For example, during quota size calculation [[3]].  In these cases the browser may wish to wipe the origin immediately.
1.  Some types of corruption may affect multiple storage subsystems.  This would be confusing if it's reported via an exception thrown from a particular subsystem method call.

For these reasons it would be preferable to use an out-of-band mechanism like the Reporting API.

## Privacy & Security Considerations

From a security perspective this explainer depends on the Reporting API’s design.  Reporting endpoints are associated with an origin which aligns well with origin storage APIs.

In terms of privacy, this API only introduces a small additional amount of stored state for the storage report registration data.  Again, we rely on the privacy characteristics of Reporting API since all reporting categories will be faced with this.  While its possible this could be used as a cookie, a browser could treat pages with partitioned storage as also having partitioned reporting API registration state as well.

This API should also be careful not to expose the removal of storage data when the user may wish to keep that action private.  For example, if the user manually deletes the storage for a site it’s a strong signal that the user does not want to interact with that origin further.  Similarly, storage deletion at the end of a browser session should probably also not be reported.

It's unclear to me if things like quota eviction or clear-site-data fall in this privacy-sensitive category or not.  For example, maybe there is some fingerprinting that could be done on users that encounter frequent quota eviction.  For now they are excluded from the effort.

Browser implementations would have to be careful not to include user identifying information in the free-form "debug" field.

## Stakeholder Feedback / Opposition

TBD after stakeholder review.

## References & Acknowledgements

Reviewed by:

*   Darwin Huang
*   Olivier Yiptong
*   Chase Phillips

References:

* \[0]: https://w3c.github.io/reporting
* \[1]: https://w3c.github.io/reporting/#endpoint-delivery
* \[2]: https://w3c.github.io/reporting/#observers
* \[3]: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Browser_storage_limits_and_eviction_criteria

[0]: https://w3c.github.io/reporting
[1]: https://w3c.github.io/reporting/#endpoint-delivery
[2]: https://w3c.github.io/reporting/#observers
[3]: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Browser_storage_limits_and_eviction_criteria
