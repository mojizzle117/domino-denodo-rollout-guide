# Run an existing GitLab dashboard on Domino

Start with the private GitLab repository that already runs from VS Code. Use its working revision as the baseline. The first milestone is a restricted Domino App that opens through the Domino URL and responds to a button click using fictional data. Then serve the actual dashboard with its synthetic demo, verify saved-run storage, and connect the approved reporting source.

The menu examples below are from Domino 6.2. Use the equivalent controls and the listener, proxy and storage settings confirmed for your installation. Commands marked `APP_CLI.py` use the launcher in your checkout. Replace that name with the actual filename and confirm its commands using `--help`. The [short hosting guide](README.md) and [acceptance plan](READINESS.md) cover the design and acceptance criteria.

| Tool | Role |
| --- | --- |
| VS Code | Edit and test the existing local checkout. |
| Internal GitLab | Store and review the application code and record the revision Domino should run. |
| Domino Project and Workspace | Connect the repository and check the checkout, Linux runtime and files. |
| Domino Job | Prepare a report and save a completed run when Jobs are the supported preparation route. |
| Domino App | Run the web server behind the Domino proxy and serve the selected report run. |

## 1. Record the working baseline in VS Code

1. Open the existing application folder in VS Code. Open **Terminal → New Terminal**. From the Git checkout run these commands.

   ```text
   git status --short --branch
   git rev-parse HEAD
   python --version
   python APP_CLI.py --help
   ```

   If the launcher sits in a subfolder, change to that folder before the Python commands. Use the interpreter that already runs the dashboard. If the CLI offers `support`, run `python APP_CLI.py support` with the same settings selection you use locally. Record the app version, commit, branch, Python version and required packages in your internal notes. `support` reads the selected settings in the inspected launcher.

2. In GitLab, open **Code → Commits** and confirm the working commit is present. If it depends on uncommitted changes, review them and establish a reproducible private baseline first. Review tracked and staged files for private settings, real inputs, outputs, credentials and local environments. Keep those in their approved locations.
3. From the working baseline create a pilot branch with `git switch -c domino-pilot`, or use the branch name your project requires. Plan to commit the synthetic probe and later the hosting adaptation here. Use the GitLab review workflow for changes intended for team use. [GitLab merge requests](https://docs.gitlab.com/user/project/merge_requests/creating_merge_requests/)

**Checkpoint:** you can identify the exact working GitLab revision and the application subfolder.

## 2. Connect the existing GitLab repository in Domino

1. Open the approved Domino deployment. Confirm the installed version, allowed data plane, project creation rights and the approved Git authentication method. Domino must be able to reach the internal GitLab host from the relevant execution environment. The local VS Code login does not configure this connection.
2. If required, set up the approved credential under **Account → Account Settings → Git Credentials**. Choose the credential type supported by the GitLab integration and enter the secret in Domino's credential UI. The project owner can confirm the scopes for repository discovery versus Git read access. Keep tokens and SSH material out of URLs, scripts and shell history. [Domino Git credentials](https://docs.dominodatalab.com/en/6.2/user_guide/314004/import-git-repositories/) · [GitLab token scopes](https://docs.gitlab.com/security/tokens/access_token_scopes/)
3. Go to **Develop → Projects → Create Project**. Choose restricted project visibility, select the Git provider and approved credential, then select the **existing** GitLab repository or enter its clone URL. If the project already exists in Domino, verify the repository and use that project. Select the pilot branch when it is available. [Domino 6.2 Git-based Projects](https://docs.dominodatalab.com/en/6.2/user_guide/eaab17/create-a-git-based-project/) · [Domino Cloud project creation](https://docs.domino.ai/cloud/platform-capabilities/core-concepts/projects/create-projects)
4. If the platform team uses a **Domino File System Project**, use its supported **Code → Git Repositories → Add a New Repository** flow. Confirm the imported directory, revision and how the App launch file reaches the imported checkout. Attached-repository paths and Git-based project paths differ. [Domino repository imports](https://docs.dominodatalab.com/en/6.2/user_guide/314004/import-git-repositories/)

**Checkpoint:** Domino lists the intended private repository or attachment and can open the selected revision. If authentication fails, check host reachability, TLS trust, repository visibility and the Domino credential with the platform owner.

## 3. Open a Workspace and check the runtime

1. In the Domino Project start a **Workspace** with the selected revision and an approved compute environment. Use a terminal inside that Workspace. Verify the actual checkout location and revision. `git rev-parse HEAD` works when the checkout includes Git metadata. Otherwise check the revision shown by Domino.

   ```sh
   pwd
   git rev-parse HEAD
   python --version
   python -m pip check
   ```

2. Change to the folder containing the application launcher and run `python APP_CLI.py --help`. Compare the Python and package versions with the application's requirements and working baseline. Check Dash and the app's other requirements, spreadsheet readers if used, and the reporting time zone. Record the Domino **environment revision**. An environment owner can supply missing packages through the approved environment build so Workspace, Job and App all receive the same dependencies.
3. If available in that revision, run `python APP_CLI.py selftest`. Check for actual tests run and read the first failure before moving on. For a local process check use the app's synthetic settings and references with `python APP_CLI.py demo --settings "DEMO_SETTINGS.json" --no-browser --port 8054`, substituting an available Workspace port. The demo is an in-memory snapshot. Stop the process after the check. A terminal listener check and a published App test exercise different paths.

**Checkpoint:** the correct revision imports and its synthetic checks pass in the intended environment. Keep the package/client checks with that environment revision.

## 4. Publish a small Dash probe before adapting the dashboard

The probe verifies the checkout, launch file, browser prefix, one static asset and one callback. It uses no database or report files. Domino 6.2 documents `0.0.0.0:8888` inside the App container and a root Dash server route in its example. Confirm the platform's listener and proxy rules first. `0.0.0.0` is the bind address inside the container. Open the URL supplied by Domino in the browser. Use this development-server probe only for a restricted synthetic check if the platform permits it. [Domino App lifecycle](https://docs.dominodatalab.com/en/6.2/user_guide/9d3c12/app-structure-and-lifecycle/) · [Domino Dash example](https://docs.dominodatalab.com/en/6.2/user_guide/a537c2/common-app-frameworks/) · [Werkzeug development server](https://werkzeug.palletsprojects.com/en/stable/serving/)

On the pilot branch create `domino_probe/probe.py`.

```python
import os

from dash import Dash, Input, Output, dcc, html


def pathname_prefix(value):
    path = value.strip("/")
    return f"/{path}/" if path else "/"


browser_prefix = pathname_prefix(
    os.environ.get("APP_BROWSER_PREFIX", os.environ.get("DOMINO_RUN_HOST_PATH", ""))
)
route_prefix = pathname_prefix(os.environ.get("APP_ROUTE_PREFIX", ""))
app = Dash(
    __name__,
    requests_pathname_prefix=browser_prefix,
    routes_pathname_prefix=route_prefix,
    serve_locally=True,
)
app.layout = html.Main(
    [
        html.H1("Domino Dash probe"),
        dcc.Link("Open probe subpage", href=app.get_relative_path("/probe")),
        html.Button("Test callback", id="probe-button", n_clicks=0),
        html.P("Clicks: 0", id="probe-result"),
    ]
)


@app.callback(Output("probe-result", "children"), Input("probe-button", "n_clicks"))
def show_clicks(clicks):
    return f"Clicks: {clicks or 0}"


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=int(os.environ.get("APP_PORT", "8888")), debug=False)
```

Create `domino_probe/assets/probe.css` so a visible change confirms local CSS is served.

```css
main { font-family: sans-serif; padding: 2rem; }
h1 { color: #185fa5; }
```

Create `domino_probe/app.sh` with LF line endings. This example assumes Domino executes the selected script file and `python` points to the approved environment. Select or adjust the launch path according to your project type.

```sh
#!/bin/sh
set -eu
cd "$(dirname "$0")"
exec python probe.py
```

Check the two prefix settings with your platform owner. Use the path portion of the published URL, with leading and trailing slashes. The example accepts the documented `DOMINO_RUN_HOST_PATH` as a default when it supplies the browser base path. Set `APP_BROWSER_PREFIX` explicitly if your deployment provides the browser base another way. `APP_ROUTE_PREFIX` defaults to `/` for a proxy that removes the browser prefix before forwarding. If the proxy keeps the prefix and no middleware strips it, set `APP_ROUTE_PREFIX` to the same prefix as the browser. With a root URL and root internal routes use `/` for both. [Domino base-path configuration](https://docs.dominodatalab.com/en/6.2/user_guide/cb9195/app-security-and-identity/) · [Dash pathname settings](https://dash.plotly.com/reference)

Commit and push only the three probe files to the existing **internal GitLab** remote. The `--chmod=+x` index change keeps the script executable even when committing from Windows. Continue to `git commit` once the staged list contains only these three files and the diff check passes. Substitute your remote or branch name if it differs from the example.

```text
git add domino_probe/probe.py domino_probe/assets/probe.css
git add --chmod=+x domino_probe/app.sh
git diff --cached --name-only
git diff --cached --check
git commit -m "Add synthetic Domino Dash probe"
git push -u origin domino-pilot
```

In Domino open the Project and go to **Deployments → Apps & Agents → Publish App**. Choose a clear pilot name, the probe revision, approved environment, hardware tier and `domino_probe/app.sh` as launch file. Select **Restricted** and check who has inherited access through project collaboration. Leave real data unattached. Enable deep links if available and needed for the test. Publish and open the App URL. Check for the blue heading, open the subpage and reload it, then click **Test callback** and confirm the number changes. In browser developer tools check that the CSS, Dash layout and callback requests stay within the App URL and complete successfully. [Domino 6.2 App publication and sharing](https://docs.dominodatalab.com/en/6.2/user_guide/cd0095/publish-and-share-an-app/)

**Checkpoint:** the restricted App serves the synthetic probe through the real proxy, with a working asset, callback and direct link. Keep the probe as a known comparison while adapting the dashboard.

## 5. Add a Domino entry point for the real dashboard

Make the following reviewed change on the pilot branch in the private repository. Check the exact working revision before editing. The desktop launcher pattern binds its server and port checks to `127.0.0.1`. `--port` changes the port. `--no-browser` suppresses the desktop browser. For the inspected launcher pattern, a separate hosted entry point is required to bind at the address Domino expects.

1. In VS Code search for `127.0.0.1`, `make_server`, `create_app` and `dcc.Location`. Locate the blocks for runtime checks, synthetic data construction, saved-run verification and Dash construction. Create a separate hosted entry point, for example `domino_dashboard.py`, that calls the existing functions for those stages. Keep the existing desktop startup command available.
2. Give the hosted entry point an explicit synthetic mode and an explicit saved-run mode. For example, read `APP_MODE=demo` or `APP_MODE=saved-run` and an `APP_SETTINGS_FILE` path supplied through the approved runtime configuration. For the synthetic mode, reuse the existing demo builder with neutral settings and references. For saved-run mode, require `APP_RUN_FOLDER` and use the same checks as the desktop launcher's explicit `--run` path. Verify integrity, input inventory and code compatibility, then construct the Dash app from the verified snapshot. Startup must fail with a safe stage message when a required selection is missing or invalid. Keep extraction in the preparation command or Job.
3. Pass the tested request and route prefixes when constructing Dash. Generate internal links with `app.get_relative_path()` and assets with `app.get_asset_url()`. Use `app.strip_relative_path()` before dispatching a `dcc.Location.pathname` callback. Review redirects, JavaScript route comparisons, custom status routes and the startup layout check against both local and hosted paths. Keep Dash assets served locally. [Dash routing and asset helpers](https://dash.plotly.com/reference)
4. Expose the Dash instance's Flask server as `server` in the hosted entry point. Use the approved WSGI server and listener in the foreground. For example, if Gunicorn is already approved and available in the Linux environment and the platform confirms `0.0.0.0` inside the App container, the launch script can run the following from the application folder. `APP_PORT` is the confirmed container port. Begin with the worker count agreed for the pilot because separate processes can each allocate the snapshot and indexes. [Flask WSGI deployment](https://flask.palletsprojects.com/en/stable/deploying/) · [Gunicorn invocation](https://flask.palletsprojects.com/en/stable/deploying/gunicorn/)

   ```sh
   exec gunicorn --bind "0.0.0.0:${APP_PORT:?Set the approved App port}" \
     --workers 1 domino_dashboard:server
   ```

   This command assumes `domino_dashboard.py` has already been implemented and reviewed and that one worker was selected. Keep debug and auto-reload off. Confirm the listener is reachable through the platform proxy within the App network boundary.

5. Push the reviewed hosting revision to GitLab. Publish a restricted **dashboard pilot** App from that revision, using the synthetic mode and the tested environment, launch file and proxy settings. Run the synthetic checks locally and in Domino. Open the real dashboard, navigate all views, reload a deep link, change filters quickly, test two simultaneous browser sessions, and save and cancel the native PDF print dialog.

**Checkpoint:** the actual dashboard runs with invented data through Domino. Browser sessions retain independent filters, and native PDF matches the active view.

## 6. Set up storage and prove reopen across executions

1. Ask the platform and data owners for the supported durable location visible to both the **preparation execution** and the **serving App**. Use a mounted Dataset or another approved filesystem service if it provides the required paths and rename behavior. Verify the mount in each execution type separately. Domino 6.2 documents that Apps do not automatically write files back to the project repository. [Domino App persistence](https://docs.dominodatalab.com/en/6.2/user_guide/b99d5d/persist-data-from-apps/)
2. Separate the original reports and reviewed references, settings and connection-hook file, and completed-run output. Grant the preparation identity the approved source read and run write access. Grant serving the file read access it needs to verify and open the run. Keep database credentials in the preparation environment's approved secret mechanism. A mounted hook file may need to remain available for fingerprint checks even if the serving process never uses its connection.
3. Use the **same stable absolute paths and bytes** in both executions for every file recorded in the manifest, with matching app code and release. Relative settings entries are resolved against the settings file's folder in the inspected launcher pattern. A run prepared on a Windows drive can fail verification after being copied to Linux because its recorded input paths differ. Prepare a new run inside Domino at the final approved mount paths. Keep previous accepted code and input versions available for rollback.
4. Prove persistence first with a fictional fixture that uses the normal prepare and reopen path and a fake source. Reuse the matching synthetic test fixture as the starting point for this exercise. The in-memory `demo` has no saved run to reopen. Verify that preparation writes the completed run and manifest, and that the App can reopen it after a real restart. Test a missing input and a changed input in a disposable fixture and confirm the launch fails with a clear freshness error. Keep the last accepted run available throughout.

**Checkpoint:** after stopping and restarting the App, the intended saved synthetic run still opens with its original preparation time and run identity.

## 7. Prepare and reconcile the first restricted reporting run

1. Confirm with the source owner which read-only route is approved from the preparation environment. If the existing source route is supported, test it there under the approved identity. If Domino requires Denodo, first validate the exact client, views, type and bind behavior, and consistent extraction cutoff using the [Denodo parity plan](READINESS.md). An Oracle connection hook can rely on Oracle transaction and query behavior beyond its connection parameters. Keep the published pilot synthetic until the applicable source check passes.
2. In the approved Workspace or Job, use the same reviewed revision and environment as serving. Put the approved settings and references at the agreed paths. Start with the agreed bounded feedback scope. Use the actual launcher and paths in place of the placeholders below. Run each command separately and stop on an error.

   ```sh
   python APP_CLI.py check --settings "SETTINGS_FILE"
   python APP_CLI.py prepare --settings "SETTINGS_FILE"
   python APP_CLI.py reconcile --settings "SETTINGS_FILE" --run "COMPLETED_RUN_FOLDER"
   ```

   `check` validates configured local inputs in the inspected launcher. `prepare` reads the approved source and saves a new run. Copy the completed folder path from its output into the last command. Use that path every time you open or reconcile this run. In the inspected launcher `start` or `reconcile` without `--run` can initiate preparation. The reconciliation command opens the saved run and identifies the review artifact. A reviewer compares coverage, report population, counts, amounts and representative linked cases internally.
3. In the restricted reporting App select the verified settings and **that exact completed run** using the reviewed hosted entry point. Pin the commit, environment, launch file and approved mounts. Confirm the App identity can read what it needs, and that an unauthorized account cannot access pages, exports or data routes. Confirm who can see all rows in the prepared snapshot before adding viewers. Check effective data access with the platform owner because viewer permissions and mounted-file access depend on configuration. [Domino App identity and mounted data](https://docs.dominodatalab.com/en/6.2/user_guide/5bef19/authentication-and-identity-for-apps/)

**Checkpoint:** the authorized App shows the intended accepted run with reconciled scope and results. Verify the user-visible report date and capture measured startup, memory and interaction times under the agreed workload before expanding access.

## 8. Refresh, diagnose and recover

For a refresh, prepare a new run, review it, select it explicitly in the App configuration and restart or republish according to the installed workflow. Verify the new run identity from separate browser sessions and across serving processes. A code push, new output folder or browser reload alone leaves the serving selection unchanged in this design. Keep an internal record of commit, environment, launch settings, input/reference versions, accepted run, reviewer and date.

For rollback, redeploy the previous known-good code revision and environment with its matching settings, input versions and completed run. Recheck the display and access. Domino 6.2 documents App version history for inspection. Follow the installed platform's redeploy workflow. [Domino App logs and versioning](https://docs.dominodatalab.com/en/6.2/user_guide/bb4b24/app-logs-and-versioning/)

| Symptom | First check |
| --- | --- |
| Repository missing or checkout fails | Confirm selected provider, internal clone URL, credential authorization, TLS trust and network access. |
| App exits or proxy shows a gateway error | Inspect App logs for the first failing stage. Confirm revision, launch path, script line endings, executable mode, imports, listener and port. |
| Page opens but layout, CSS or filters fail | Compare browser request paths with the configured prefixes. Inspect `/_dash-layout`, local asset and `/_dash-update-component` requests under the hosted path. |
| Direct link fails after reload | Check the platform deep-link option, the forwarded route and the app's route normalization. |
| Run fails freshness verification | Compare approved mount paths, settings, inputs, hook and code with the manifest. Reprepare with matching files if the inputs or code changed. |
| New figures do not appear | Check the selected run and App revision, then confirm every serving process restarted on the intended selection. |
| Real preparation returns unexpected coverage | Stop publication and review source cutoffs, scope, role coverage and data identity. Reconcile the discrepancy before selecting a new run. |
