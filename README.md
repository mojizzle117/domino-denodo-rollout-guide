# GitLab and Domino hosting guide

This guide describes how to move a Python reporting dashboard into a managed Domino environment. Start with the release that already runs on the workstation. Keep that release and its accepted report runs available during the pilot. Use the [readiness and acceptance plan](READINESS.md) to check Denodo compatibility and test the hosted application with synthetic inputs.

## 1. Confirm the environment and ownership

Identify who owns the application, approves reporting results, supports the hosted service and can restore a previous release. Confirm the Python and package versions, time zone data, and supported database client in the actual hosted environment. Ask the platform owner about the launch interface, proxy path, serving topology, compute limits, persistent storage and viewer access.

## 2. Import a clean baseline into GitLab

Create the application repository in the organization's approved GitLab namespace with the visibility and review rules required for the application. Start from a clean copy of the tested release. Track reviewed source, generic settings templates, permitted assets, synthetic tests and operating instructions. Keep credentials, configured connection hooks, private settings, source extracts, completed runs and local environments outside Git history.

Set ignore rules before the first commit and inspect the staged file list. Record the application version and source revision. Use an approved runner for automated synthetic checks when available. A reviewer can run the same checks manually in the approved environment while runner access is pending.

## 3. Configure the hosted application

Use the installed platform's supported Git integration and pin the intended source revision. Confirm which identities sign in, read the source repository, query the database, prepare reports and view the results. Supply credentials through the platform's approved mechanism.

Agree where original inputs, reviewed references, run artifacts and exported reports live. Check that preparation and serving can access the files each needs and that accepted runs survive a restart. Set the listener, proxy path and serving command from the actual platform contract.

## 4. Test with synthetic inputs

Run the runtime preflight and targeted application tests in the hosted environment. Prepare a synthetic report, reopen it, and check navigation, direct links, callbacks, assets, session isolation and native PDF. Exercise both approved and denied access. Restart the service and verify that it reopens the expected run. Use the readiness plan for the detailed acceptance sequence.

## 5. Review, switch and recover

Assign an owner to prepare each new run and a reviewer to reconcile its coverage and results. Once accepted, explicitly select the run and verify that every serving process displays the intended version. Record the source revision, runtime, settings, input version and run identity together. Rehearse restoring that complete set after a failed preparation or deployment. Confirm the user experience and operating limits under an agreed representative workload before expanding access.
