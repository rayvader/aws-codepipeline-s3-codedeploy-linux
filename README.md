# aws-codepipeline-s3-aws-codedeploy_linux

> **About this fork:** a personal copy of [aws-samples/aws-codepipeline-s3-codedeploy-linux](https://github.com/aws-samples/aws-codepipeline-s3-codedeploy-linux), used to try the CodePipeline walkthrough. The only change from upstream is the `<h2>` text in `index.html`, edited to check that a source change flows through the pipeline.

Use this sample when creating a simple pipeline in AWS CodePipeline while following the Simple Pipeline Walkthrough tutorial. http://docs.aws.amazon.com/codepipeline/latest/userguide/getting-started-w.html

## What it does

AWS CodeDeploy copies a static `index.html` page to Apache (`httpd`) on Amazon Linux EC2 instances. CodePipeline pulls the source bundle from an Amazon S3 bucket and hands it to CodeDeploy.

## Tech stack

- AWS CodePipeline (S3 source), AWS CodeDeploy, Amazon EC2
- Apache `httpd`, installed with `yum` (Amazon Linux / RHEL family)
- Bash lifecycle scripts

## Project structure

```
.
├── appspec.yml              # CodeDeploy app spec: files and lifecycle hooks
├── index.html               # The page that gets deployed
├── scripts/
│   ├── install_dependencies # yum install -y httpd
│   ├── start_server         # service httpd start
│   └── stop_server          # stops httpd if it is running
├── dist/
│   └── aws-codepipeline-s3-aws-codedeploy_linux.zip  # Ready-made upstream bundle to upload to S3
└── LICENSE
```

`dist/` holds the original upstream bundle. It does not include this fork's `index.html` change.

## How the deployment runs

`appspec.yml` (`os: linux`):

| Step | Action |
|---|---|
| `ApplicationStop` | `scripts/stop_server` (root) |
| `BeforeInstall` | `scripts/install_dependencies`, then `scripts/start_server` (root) |
| Install | Copy `/index.html` to `/var/www/html/` |

Each hook has a 300-second timeout.

## Prerequisites

- An AWS account with CodePipeline, CodeDeploy, S3 and EC2 access
- Amazon Linux EC2 instances running the CodeDeploy agent, with an instance profile that can read the S3 bucket
- A CodeDeploy application and deployment group that target those instances
- An S3 bucket with versioning enabled (CodePipeline S3 sources need it)

## Deploy

1. Build the bundle from the repository root (or use the zip in `dist/`):
   ```bash
   zip -r aws-codepipeline-s3-aws-codedeploy_linux.zip appspec.yml index.html scripts LICENSE README.md
   ```
2. Upload it to your versioned bucket:
   ```bash
   aws s3 cp aws-codepipeline-s3-aws-codedeploy_linux.zip s3://<your-bucket>/
   ```
3. Create a pipeline with an Amazon S3 source (bucket and object key above) and an AWS CodeDeploy deploy stage, as in the walkthrough linked above.
4. Open an instance's public DNS name in a browser to see the page.

Upload a new version of the zip to trigger the pipeline again.

## Configuration

There are no environment variables or config files. Change `index.html` to change the page, and `appspec.yml` to change the hooks or target path.

## Testing

There are no automated tests.

## License

Apache License 2.0. See [LICENSE](LICENSE).
