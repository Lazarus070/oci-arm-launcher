# OCI ARM VM Auto-Launcher

A GitHub Actions workflow that automatically provisions an Oracle Cloud Always Free ARM (Ampere A1) VM in `eu-frankfurt-1`.

Oracle's free ARM capacity is frequently exhausted, so clicking "Create instance" in the console rarely succeeds. This workflow retries every 15 minutes until a slot opens, then disables itself.

## How it works

1. Installs the OCI CLI and authenticates using repository secrets.
2. Verifies the credentials, so a bad key fails loudly instead of looking like "no capacity".
3. Checks whether the VM already exists, to avoid duplicates.
4. Tries to launch a `VM.Standard.A1.Flex` instance (4 OCPUs, 24 GB RAM, 50 GB boot volume) in each availability domain in turn.
5. Treats "out of capacity" and throttling as expected and retries on the next run. Any other error fails the run visibly.
6. Disables its own schedule after a successful launch.

## Setup

1. Create an OCI API key for your user and note its fingerprint.
2. Add these under **Settings → Secrets and variables → Actions**:

| Secret | Value |
|---|---|
| `OCI_USER_OCID` | Your user OCID |
| `OCI_TENANCY_OCID` | Your tenancy OCID |
| `OCI_FINGERPRINT` | API key fingerprint |
| `OCI_PRIVATE_KEY` | Full private key, including BEGIN/END lines |
| `OCI_REGION` | `eu-frankfurt-1` |
| `OCI_COMPARTMENT_OCID` | Compartment to create the VM in |
| `OCI_SUBNET_OCID` | Subnet that allows public IPs |
| `OCI_IMAGE_OCID` | An aarch64 image OCID for your region |
| `SSH_PUBLIC_KEY` | Contents of your `.pub` file |

3. Open the **Actions** tab, select the workflow and click **Run workflow** to test it.

## Notes

- Scheduled runs are best-effort, so timing can drift.
- GitHub pauses scheduled workflows after 60 days without repo activity.
- Capacity in Frankfurt can take days to appear, so "no capacity" is a normal result.
- Always Free ARM resources can only be created in the tenancy's home region.
- Stays within the Always Free limits (4 OCPUs, 24 GB RAM, 200 GB storage).

## Tech

GitHub Actions · Oracle Cloud Infrastructure CLI · Bash
