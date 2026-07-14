# s3 bucket upload policy

- `Principal: "*"` means the rule applies to anyone
- `Resource: "arn:aws:s3:::bucket-name/*"` targets all objects inside the specified bucket.
- If no valid policy is attached, the upload will fail with `AccessDenied`.

## Bucket policy example
```bash
# s3 buckets are by default private
# create a upload policy for bucket

{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "PublicUpload",
			"Principal": "*",
			"Effect": "Allow",
			"Action": [
				"s3:PutObject"
			],
			"Resource": [
				"arn:aws:s3:::bucket-name/*" # * for all resources of bucket
			]
		}
	]
}
```
## Test upload and download with curl

```bash
# without upload policy it will give access denied
curl -X PUT   --upload-file test.png   https://bucket-name.s3.amazonaws.com/test.png

curl https://bucker-name.s3.amazonaws.com/test.png --output test-downloaded.png
```

## IAM user example
```bash
# attach AmazonS3FullAccess policy to IAM user
# iam user will get all s3 access operation
aws s3 ls

# upload an image
aws s3 cp cover.gif s3://bucket-name
```