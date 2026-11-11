# Denodo parity and Domino acceptance plan

Keep the established reporting population and calculations fixed while changing the source interface. Verify equivalent input records in a bounded read-only check. Validate hosting separately with synthetic inputs. The steps below define acceptance criteria for a future pilot.

## Confirm three prerequisites

1. **Runtime and client.** Identify the working application revision, the approved hosted Python environment and package versions, and the supported Denodo client and native library requirements.
2. **Source contract.** Confirm available views, logical fields, identifier and timestamp representations, binding rules and the supported way to read a consistent cohort across queries.
3. **Hosting and operations.** Confirm the listener, proxy, serving topology, persistent storage, viewer and data identities, and owners for preparation, review and run switching.

## Compare the same small cohort

1. Inspect the existing Oracle adapter for transaction setup, autocommit, rollback and connection cleanup, liveness SQL, named bind parameters, batched lookups, column aliases and returned Python types. Check each assumption with the supported Denodo client. A successfully bound query is one check. Establish whether the proposed read is coherent across all required views and linked lookups.
2. Choose one small cohort from the existing feedback reports. Use the same input versions, references, settings and agreed source cutoff for both routes. Follow each selected parent to all linked child records, including children outside the parent selection window. Retain unmatched feedback records. Establish an equivalent as-of snapshot or approved stable extraction window, accounting for view refresh and cache lag. Record parity as inconclusive when a common cutoff cannot be established.
3. Compare headers, types, nulls, timestamps and precision. Compare exact identities, row multiplicity, parent-child links, coverage by role and retained values after documented equivalent conversions. Check for unexpected or ambiguous parent matches. Record differences in optional coverage explicitly.
4. Pass both inputs through the same normalization and calculation version. Compare parent counts, original-currency amounts, converted totals, classifications and unresolved cases. Multiple children must conserve the parent count and amount. Review all differences and retain detailed comparisons within the approved environment.
5. Exercise missing fields, long and leading-zero identifiers, null versus zero, duplicate or unexpected matches, multiple children, chunk boundaries, unavailable optional sources and mid-read disconnection with synthetic cases. A required read failure stops publication. Retry the complete read against a consistent cutoff. Measure connection setup, execution and first fetch, remaining fetches, normalization and save time, and peak memory separately. Agree the acceptable resource envelope and workload with the source and platform owners.

## Test hosted behavior with a synthetic run

Preparation validates source inputs and saves a completed snapshot, integrity metadata and a reconciliation artifact. Serving reopens the selected snapshot and builds process-local indexes and caches. A browser session holds its own selections and applies them to the prepared data.

| Check | Acceptance criteria |
| --- | --- |
| Routing and proxy | Open the dashboard through its actual hosted URL. Navigate, reload deep links and verify local CSS, images and scripts. Exercise callbacks and rapid filter changes. A late response from a previous selection must leave the current selection intact. |
| Session isolation | Open two independent browser sessions with different filters and selections. Interact concurrently. Verify each session retains its own state and sees the intended run. |
| Startup and interaction | Measure reopen, integrity checks, index build and readiness separately from cold and warm callback time, response size, proxy/network delay and browser completion. Record memory per process and across the deployment. Each worker may hold its own snapshot, index and cache. Test against the agreed limits and workload. |
| Restart and freshness | Prepare a fictional run in the intended environment. Retain the files its freshness checks require. Restart the serving process and verify run identity and preparation time. Exercise missing or changed inputs, changed source code and corrupt artifacts. Check that a failed preparation leaves the prior accepted run available. |
| Explicit refresh | Prepare a new run, reconcile it, select it explicitly and verify its identity across all serving processes. Confirm that reopening and browser interaction use the selected snapshot. |
| Native PDF | Save the active filtered view from the supported browser. Check charts, page breaks and scope. Verify that saving or cancelling restores the interactive view and leaves other sessions unaffected. |
| Access and recovery | Verify allowed and denied access to pages, exports and status routes. Confirm effective identities for database reads, storage, viewing and publication. Restore the previous accepted revision, runtime, settings, references, inputs and run together, then verify the restored service. |

Proceed to real-data acceptance after the synthetic hosting checks and the bounded Denodo comparison. Use the original reporting definitions and have the source and business owners reconcile the representative results inside the approved environment.
