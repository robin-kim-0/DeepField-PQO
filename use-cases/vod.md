# Use case B : S3 → DeepField-PQO → MediaConvert

This use case describes a workflow where video files stored in AWS S3 are retrieved and processed with DeepField-PQO as a pre-processing step to intelligently optimize the video content, and then transcoded into HLS/DASH formats via AWS Elemental MediaConvert for Video on Demand (VOD) distribution.

## Architecture

<br/>
<img src="../images/vod/vod_architecture.png" width="90%" style="display: block; margin: auto;">
<br/>

## Prerequisites

Before you can use AWS Elemental MediaConvert and AWS S3, you need an AWS account and the appropriate permissions to access, view, and edit. For detailed information, see [Getting started with MediaConvert](https://docs.aws.amazon.com/ko_kr/mediaconvert/latest/ug/getting-started.html).

## Workflow

### Step 1. Download source file from AWS S3

Download the original low-quality video from designated AWS S3 input bucket.

```bash
aws s3 cp s3://<s3-input-bucket-name>/original_input.mp4 ./
download: s3://<s3-input-bucket-name>/original_input.mp4 to ./original_input.mp4
```

### Step 2. Run DeepField-PQO Processing

From Lambda or Step Functions, send the processing command to the AMI using SSM Run Command.

```bash
ffmpeg -i original_input.mp4 -vf bdpqo_cpu2cpu=pqo_model=pqoex13 -c:v h264_nvenc output.mp4
```

### Step 3. Upload Processed Output

Save the restored/upscaled video to the AWS S3 output bucket.

```bash
aws s3 cp output.mp4 s3://<s3-upscaled-bucket-name>/output.mp4
upload: ./output.mp4 to s3://<s3-upscaled-bucket-name>/output.mp4
```

### Step 4. Create MediaConvert Job

1. Go to the [Jobs](https://console.aws.amazon.com/mediaconvert/home#/jobs/list) page in the MediaConvert console.
2. Choose **Create job**.
3. Select **Input 1** pane, choose the desired file from the AWS S3 bucket.
   <br/>
   <br/>
   <img src="../images/vod/vod_create_job_1.png" width="90%" style="display: block; margin: auto;">
   <br/>
4. After specifying the input, create an output group by selecting **Add** in the **Output groups** section. Then choose either **Apple HLS** or **DASH ISO**.
   <br/>
   <br/>
   <img src="../images/vod/vod_create_job_2.png" width="90%" style="display: block; margin: auto;">
   <br/>
5. For **Destination**, specify the URI for the Amazon S3 location where the transcoding service will store your output files. You can specify the URI directly or choose **Browse** to select from your Amazon S3 buckets.
   <br/>
   <br/>
   <img src="../images/vod/vod_create_job_3.png" width="90%" style="display: block; margin: auto;">
   <br/>
6. In **Job settings** section, choose **AWS integration**.  
   For **IAM role**, choose an IAM role that has permissions to access the Amazon S3 buckets that hold your input and output files.
   <br/>
   <br/>
   <img src="../images/vod/vod_create_job_4.png" width="90%" style="display: block; margin: auto;">
   <br/>
7. Choose **Create** to start the job.

For detailed information on creating a MediaConvert job, see [Tutorial: Configuring job settings](https://docs.aws.amazon.com/mediaconvert/latest/ug/setting-up-a-job.html).

### Step 5. Check results

When the MediaConvert job is complete :

- Check the S3 output path for the `.m3u8`(HLS) or `.mpd` (DASH) file and the corresponding segments.
- Enter the playback URL into an HLS/DASH player (e.g., [hls.js demo](https://hlsjs.video-dev.org/demo)) to test video quality and seeking.
