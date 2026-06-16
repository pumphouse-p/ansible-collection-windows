# README for `pumphouse_p.windows.update_report`

## Description

An Ansible Role that generates and publishes a Windows Update report. This role scans Windows systems for available updates, generates an HTML report, and publishes it to an AWS S3 bucket configured as a static website.

## Requirements

* `ansible.windows`
* `community.aws`
* Windows Server 2012 or later, or Windows 8/10/11
* AWS S3 bucket configured for static website hosting
* AWS credentials with S3 write permissions

## Role Variables

Available variables are described in the table below.

|        Variable Name         | Default | Required | Type |                Description                |
|:----------------------------:|:-------:|:--------:|:----:|:-----------------------------------------:|
| `aws_s3_website_bucket_name` | `None`  |   yes    | str  | Name of the S3 bucket to publish report to |

## AWS Configuration

This role requires:
* An S3 bucket configured for static website hosting
* AWS credentials configured on the Ansible control node (via environment variables, `~/.aws/credentials`, or IAM role)
* S3 permissions: `s3:PutObject`, `s3:PutObjectAcl`, `s3:DeleteObject`, `s3:ListBucket`

## How It Works

The role performs the following steps:
1. Queries each Windows host for available updates using `ansible.windows.win_updates`
2. Collects update information (title, KB number, category, severity, etc.)
3. Generates an HTML report from a Jinja2 template on the control node
4. Publishes the report to the specified S3 bucket
5. Outputs the URL where the report can be accessed

## Dependencies

None.

## Example Playbook

### Basic Usage

```yaml
---
- name: Generate Windows update report
  hosts: win_servers
  gather_facts: true
  become: true

  roles:
  - role: pumphouse_p.windows.update_report
    vars:
      aws_s3_website_bucket_name: "my-windows-updates-reports"
```

### Multiple Server Groups

```yaml
---
- name: Generate update reports for different environments
  hosts: windows_prod,windows_dev
  gather_facts: true
  become: true

  roles:
  - role: pumphouse_p.windows.update_report
    vars:
      aws_s3_website_bucket_name: "{{ environment }}-update-reports"
```

## Report Contents

The generated HTML report includes:
* Hostname of each scanned system
* Total number of updates available
* List of updates with details:
  * Update title
  * KB article number
  * Category (Security, Critical, Important, etc.)
  * Installation status
* Timestamp of when the report was generated

## Accessing the Report

After the role completes, the report URL will be displayed in the debug output:
```
Report URL: http://your-bucket-name.s3-website.us-east-2.amazonaws.com
```

**Note**: The S3 region in the URL may vary based on your bucket configuration.

## Notes

* This role only **scans** for updates; it does not install them
* The report is generated on the Ansible control node and published to S3
* Ensure your S3 bucket has appropriate permissions for public access if you want the report to be publicly viewable
* The working directory `/tmp/report_workdir` is created on the control node during report generation

## License

GPL

## Author Information

Devin Parrish ([GitHub](https://github.com/pumphouse-p))
