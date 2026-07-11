```bash
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

```bash
# without upload policy it will give access denied
curl -X PUT   --upload-file test.png   https://bucket-name.s3.amazonaws.com/test.png

curl https://bucker-name.s3.amazonaws.com/test.png --output test-downloaded.png
```