We will be using AWS EKS (kubernetes service)
 - why not GCP GKS?

argoCD is a continuos deployment tool (no more jenkins)
- cleaner UI
- App + infra viz
- new CLI
- faster deploys and rollbacks (under 3 mins)
- no more ☕️ breaks while Jenkins deploys

Simplified networking and scaling + secure traffic internally within VPC (Virtual Private Cloud)

where does the argoCD instance sit - does that need scaling?

Everything declarative!
- env vars secrets and config
- scaling min/max

Teleport tool
- audited prod access without VPN using Okta creds
- moderated shell
- DB proxy access
- Temp access delegation

Canary deploys? 
- one datadog query to bail out of a bad deploy and rollback

Automatic branch env w/ e2e tests
- ephemeral dev envs based on a branch
	- can we get our own choice of IDE - neovim/vs code etc.

Image builds are run on Circle CI
- separate workflows for build/deploy processes
- heavier reliance on wave CLI for triggering dev deploys/jobs
- prod apps will have autodeploy and automigrations enabled by default
	- what could be some side-effects of this?
