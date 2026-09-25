# Company website: GitHub Actions + JSM - demo

## Before starting

- This demo requires JSM Premium capabilities and the GitHub/Jira integration app connected to the intended site and repository.
- GitHub's native custom deployment protection rules are available for public repos on all plans. Private/internal repos require GitHub Enterprise. Do not change repository visibility or buy a plan without reviewing the implications.
- A missing integration/service association must be resolved separately; moving from Bitbucket to GitHub is not itself a verified repair.
- The fine-grained PAT used by local Git is for pushing files. It is NOT a JSM integration credential and must never be committed or pasted into either YAML file.

## Files and flow

- `.github/workflows/jsm-deployment.yml`: Build → Test → Staging → Production, started manually.
- `.jira/config.yml`: maps Production to the JSM service.
- `github-pipelines.yml` at the repository root is an older Bitbucket-format file. GitHub ignores it; it was left untouched. Do not try to enable GitHub Actions by renaming that file alone.

Build packages index.html. Test validates basic HTML structure and the artifact checksum. Staging serves it briefly on the runner for an HTTP smoke test. Production verifies the same artifact and prints a simulated-release message. Nothing publishes the site or provisions hosting.

## 1. Connect the app and service in JSM

1. Connect/install GitHub for Atlassian (the Jira app) using the official JSM GitHub connection flow. Give it access to this repository only, unless a broader scope is deliberately needed.
2. In the intended JSM ITSM project, open settings → Operations → Change management → Connect pipeline → GitHub.
3. Select/create the Company Website service and copy the exact Service ID supplied by the wizard.
4. Map the Production environment type to the chosen change request type (for example CICD change). Verify that request type's workflow actually requires the intended approval.
5. Configure and save allow/prevent status mappings using statuses from that workflow. Select a release status reached after approval, not one that bypasses approval.
6. Configure the change approver. If using service-based approval, confirm the created change contains the correct Affected services value and displays the expected approver.

## 2. Complete the repository configuration

Replace `REPLACE_WITH_JSM_SERVICE_ID` in `.jira/config.yml` with the exact Service ID from the GitHub connection wizard. It is not the cloud ID, repository UUID, project key, service name, or Assets object key.

The template deliberately stops at Build while that placeholder remains. This checks only that the template was edited; it does not verify the Service ID or the protection rule.

Review, commit, and push the new files to main when ready. There are no automatic push/PR triggers yet: adding the workflow does not start a deployment. Add those later after the gate has been rehearsed.

## 3. Enable Jira's GitHub protection rule

1. In GitHub, open Repository settings → Environments.
2. Create/open `Staging` and `Production` with exactly those names.
3. On Production, under Deployment protection rules, select Jira as a CUSTOM protection rule and save.
4. Restrict Production deployments to main. Where the control is available, disallow administrator bypass for an enforced approval demo.
5. Leave Staging without the Jira gate. Configure JSM tracking as intended; this YAML gates only Production.

The name Production must match across the GitHub environment, workflow `environment.name`, and `.jira/config.yml` environment list.

GitHub's Required reviewers option is not the same as Jira's protection rule. If Jira is missing, stop and check the app installation, repo access, plan eligibility, and service connection. Merely declaring an environment in YAML does not enforce approval.

## 4. Test

1. Configure the integration, Service ID, enforced approval, and environment rule first.
2. GitHub → Actions → Company website - JSM deployment demo → Run workflow → main.
3. Confirm Build, Test, and Staging succeed, then Production waits on the Jira app.
4. Confirm the expected JSM change appears, review it, approve, and reach the configured allow status.
5. Confirm Production proceeds afterward. Rehearse rejection separately.

The final release is simulated. Do not present the temporary local server or a completed job as proof that a public website was deployed. Five-minute timeouts apply to executing jobs, not to the pre-job environment-protection wait.

## Official references

- https://support.atlassian.com/jira-service-management-cloud/docs/connect-github-repositories-with-jira-service-management/
- https://support.atlassian.com/jira-service-management-cloud/docs/use-deployment-gating-with-github/
- https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/configure-custom-protection-rules
