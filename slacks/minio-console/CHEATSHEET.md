# MinIO / S3 Cheatsheet

Quick reference for MinIO, S3 operations, and the `mc` CLI.

## MinIO Console Basics

- **Buckets**: Containers for objects (like folders in a filesystem)
- **Objects**: Files stored in buckets (up to 5TB each)
- **Policies**: Control who can read/write/delete buckets and objects
- **Identity**: Manage users, groups, and service accounts

## Install `mc` (MinIO Client)

```bash
# macOS
brew install minio/stable/mc

# Linux (direct download)
curl -O https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/

# Verify
mc --version
```

## Configure `mc`

```bash
# Add a MinIO alias (alias = connection profile)
mc alias set myminio http://localhost:9000 minioadmin minioadmin

# List configured aliases
mc alias list

# Test connection
mc admin info myminio
```

## Bucket Operations

```bash
# Create a bucket
mc mb myminio/my-bucket

# Create a bucket in a specific location
mc mb --region us-east-1 myminio/my-bucket

# List buckets
mc ls myminio

# List objects in a bucket
mc ls myminio/my-bucket

# Recursively list all objects
mc ls -r myminio/my-bucket

# Remove a bucket (must be empty)
mc rb myminio/my-bucket

# Remove a bucket even if not empty (DANGER!)
mc rb --force myminio/my-bucket
```

## Object Operations

```bash
# Upload a file
mc cp ./myfile.txt myminio/my-bucket/

# Upload a directory
mc cp -r ./my-dir/ myminio/my-bucket/

# Download a file
mc cp myminio/my-bucket/myfile.txt ./downloaded.txt

# Download recursively
mc cp -r myminio/my-bucket/ ./local-dir/

# Copy between buckets
mc cp myminio/bucket1/file.txt myminio/bucket2/file.txt

# Move (copy + delete)
mc mv myminio/bucket1/file.txt myminio/bucket2/file.txt

# Remove an object
mc rm myminio/my-bucket/file.txt

# Generate a pre-signed URL (expires in 7 days)
mc share download myminio/my-bucket/file.txt --expire 7d

# Get file info
mc stat myminio/my-bucket/file.txt
```

## Bucket Policies

```bash
# Make bucket publicly readable
mc policy set public myminio/my-bucket

# Make bucket private (default)
mc policy set private myminio/my-bucket

# Upload to a public bucket without credentials
curl -T ./myfile.txt http://localhost:9000/my-bucket/myfile.txt
```

## Common S3 Patterns

### Static Website Hosting

```bash
# Set bucket policy for static hosting
mc policy set download myminio/static-site

# Upload HTML files
mc cp ./index.html myminio/static-site/
mc cp ./style.css myminio/static-site/

# Access via browser
http://localhost:9000/static-site/index.html
```

### File Sharing with Presigned URLs

```bash
# Generate a URL that expires in 1 hour
mc share upload --expire 1h myminio/my-bucket/sensitive.pdf

# Output: https://... (share this link)
```

### Versioning (keep object history)

```bash
# Enable versioning on a bucket
mc version suspend myminio/my-bucket    # disable
mc version enable myminio/my-bucket      # enable
mc version info myminio/my-bucket        # check status
```

## MinIO Console Tips

- **Create buckets**: Click "Buckets" → "Create Bucket"
- **Upload files**: Click a bucket → "Upload" button
- **Download files**: Hover over a file → click the download icon
- **Share files**: Hover over a file → click the share icon for presigned URLs
- **Manage users**: Go to "Identity" → "Users"
- **Set policies**: Go to "Access Policies" to create custom permissions

## API Endpoint Reference

```
# S3 API endpoint
http://localhost:9000

# Health check
http://localhost:9000/minio/health/live

# Console UI
http://localhost:9001
```

## Using with AWS SDK (Node.js example)

```javascript
import { S3Client } from "@aws-sdk/client-s3";

const s3 = new S3Client({
  endpoint: "http://localhost:9000",
  region: "us-east-1",
  credentials: {
    accessKeyId: "minioadmin",
    secretAccessKey: "minioadmin",
  },
  forcePathStyle: true, // required for MinIO
});
```
