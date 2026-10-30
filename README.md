# AWX and Ansible Automation Platform migration sample

A small, real example of moving from AWX or Ansible Automation Platform (AAP) to [Gardience](https://gardience.com):
three playbooks, and an AWX export file whose job templates, surveys, inventory groups, and schedules point at them.
Import the export into Gardience to see exactly what carries over before you try your own.

## What is here

| Path | What it is |
|---|---|
| `playbooks/deploy-web.yml` | A stand-in deploy. Writes the release and colour to `/opt/sample-web/RELEASE`, so you can see survey answers and extra variables arrive. |
| `playbooks/motd.yml` | Sets a login banner. |
| `playbooks/disk-report.yml` | Read-only. Fails if `/` is fuller than a threshold. |
| `awx/awx-export-sample.json` | The shape `awx export` produces, cut down to this project. |

## Try the import

1. In Gardience, open **Automation > Projects**.
2. Under **Import from AWX or Ansible Automation Platform**, choose `awx/awx-export-sample.json` and **Preview**.
3. Read the preview. Nothing is created until you choose **Import**.

| In the export | Becomes in Gardience |
|---|---|
| Project `Sample Playbooks` (this repository, branch `main`, update on launch) | Git project `sample-playbooks` |
| Job template `Deploy web` (limit `web-*`, tag `deploy`, skip tag `slow`, extra variable `release`, survey `Colour`) | Runbook with a Colour form field |
| Job template `Disk report` (check mode, survey `Warn above`) | Runbook that runs in check mode, with a number field |
| Inventory `Production` and groups `webservers`, `databases` | Node groups of the machines Gardience manages, matched by short name |
| Schedules `Patch Tuesday deploy` (2nd Tuesday) and `Nightly disk report` | Schedules, imported turned off |
| Machine credential `root ssh` | Not needed: Gardience reaches machines over Salt |
| Workflow `Release train` | Listed as not imported, with what to do instead |

Hosts are matched to your machines by name or short name, so `web-01.example.com` matches a machine named `web-01`.
Hosts Gardience does not manage are left out, and the preview says so.

## Export your own

On a machine that can reach AWX or AAP:

```bash
pip install awxkit
export CONTROLLER_HOST=https://awx.example.com CONTROLLER_USERNAME=you CONTROLLER_PASSWORD=...
awx export --projects --job_templates --inventory --schedules > awx.json
```

Repository tokens are not included in exports. Store yours in the Gardience vault and set it on the project after
the import. See [Ansible playbooks](https://gardience.com/docs/using/ansible-playbooks.html) for the full list of
what does and does not carry over.

## License

MIT. See [LICENSE](LICENSE).
