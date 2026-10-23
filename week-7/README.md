# Week 7, Continuous Delivery

## What this does

- `.github/workflows/deploy-gate.yml` runs on every push to `master`. It checks the OpenTofu files and renders the Prometheus chart.
- `week-7/deploy.sh` checks for new commits on `master`. If the Deploy gate passed, it runs OpenTofu and updates Prometheus.
- Cron runs the deployment script every two minutes.
- My VM connects to GitHub instead of allowing GitHub to connect directly to the VM.
- The deploy script checks for the correct branch, uncommitted changes, and unexpected local commits before deploying.

## Requirements

- Kubernetes (k3s) running on the VM
- OpenTofu, Helm, kubectl, and GitHub CLI installed
- GitHub CLI logged in
- Cron installed and active
- The Week 5 web app and Week 6 monitoring stack

## Deploy

After merging a pull request, wait for the gate to pass and cron to deploy.

To run the deployment manually:

```bash
./week-7/deploy.sh
```

## Roll back

Find the merge commit to undo and revert it:

```bash
git revert -m 1 <merge-commit>
```

Open a pull request, check it, and merge it.

## Check the deployment

```bash
crontab -l
tail -n 20 ~/deploy.log
gh run list --workflow deploy-gate.yml --limit 3
```

## Rebuild on a new VM

1. Install k3s, OpenTofu, Helm, and GitHub CLI.
2. Log in using `gh auth login`.
3. Clone the GitHub repository.
4. Copy `week-5/terraform.tfvars.example` to `week-5/terraform.tfvars` and add the appropriate local values.
5. Import the Week 4 web image into k3s.
6. Run `./week-7/deploy.sh`.
7. Recreate the cron schedule:

```bash
( crontab -l 2>/dev/null; echo "*/2 * * * * $PWD/week-7/deploy.sh >> $HOME/deploy.log 2>&1" ) | crontab -
```

## Known limitations

The deployment uses a folder that I also work in, which can cause conflicts with local edits. The state and variable files are stored on one VM, so losing the VM means rebuilding that setup. The deployment gate also cannot catch every possible problem. A change could pass the automated tests but still cause issues in the actual application. There is also no manual approval before OpenTofu applies a change.
