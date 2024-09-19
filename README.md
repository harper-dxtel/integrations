### Summary: Leveraging S3 PUT Endpoint with Presigned URLs

This document outlines the process of uploading files to our S3 bucket using presigned URLs and provides details on how to handle upload confirmation. The third-party client will use the provided `GetPresignedURL` endpoint to obtain a temporary URL for direct file uploads.

#### 1. **Generating a Presigned URL**

Before uploading files, the client must first obtain a presigned URL using our `GetPresignedURL` API endpoint:

- **Endpoint**: `GET /integrations/direct/presigned-url?filename=<filename>`
- **Headers**: Include the API key in the request header:
  ```
  Authorization: ******
  ```
- **Query Parameters**:
  - `filename`: The name of the file you intend to upload, including its extension (e.g., `example.csv`). Currently, only `.csv` and `.zip` files will be accepted (all other file types will throw an error).

##### Example Request:
```bash
curl -X GET "https://your-api.com/integrations/direct/presigned-url?filename=example.csv" \
    -H "Authorization: ******"
```

##### Example Response:
The API returns a JSON response with the presigned URL:
```json
{
    "presignedURL": "https://directworx-addresses-dev.s3.amazonaws.com/example.csv?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...&X-Amz-Date=...&X-Amz-Expires=...&X-Amz-SignedHeaders=host&X-Amz-Signature=..."
}
```

#### 2. **Uploading a File to S3 Using the Presigned URL**

Use the returned presigned URL to upload the file directly to S3. Note that this upload must occur within the URL's expiration time (typically 15 minutes).

- **Method**: `PUT`
- **Headers**: Set the `Content-Type` header based on the file type (e.g., `text/csv` for CSV files).
- **URL**: Use the `presignedURL` provided in the response from the previous step.

##### Example `PUT` Request:
```bash
curl -X PUT "https://directworx-addresses-dev.s3.amazonaws.com/example.csv?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=...&X-Amz-Date=...&X-Amz-Expires=...&X-Amz-SignedHeaders=host&X-Amz-Signature=..." \
    -H "Content-Type: text/csv" \
    --upload-file /path/to/your/example.csv
```

- **Success**: If the file is uploaded successfully, S3 will return an HTTP status code of `200 OK` or `204 No Content`.

#### 3. **Confirming the Upload**

TBD

#### 4. **Error Handling**

- If the `PUT` request to the presigned URL fails (e.g., due to an expired URL or insufficient permissions), S3 will return an appropriate error response, such as `403 Forbidden` or `400 Bad Request`.
- If there is an issue obtaining the presigned URL or confirming the upload, the API will return an error message with details.

#### 5. **Important Notes**

- **Expiration**: The presigned URL is only valid for a limited time (15 minutes). Ensure that the file upload is completed within this timeframe.
- **File Overwriting**: Using the same filename in subsequent uploads will overwrite the previous file. If you want to avoid overwriting, include unique identifiers (e.g., timestamps) in the filenames.
- **Security**: Keep the API key (`******`) secure and do not expose it in client-side code.

By following this workflow, you can securely and efficiently upload files to our S3 bucket and receive confirmation that the upload was processed. If you have any questions or encounter issues, please contact our support team.

