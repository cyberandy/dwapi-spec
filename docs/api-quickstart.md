Using this API users are able to easily access data and manage their data projects regardless of language or tool of preference.

## Creating Datasets

To create a dataset, use the #endpoint:93LqHoGaE2snXCCpu endpoint. For example, you can create a `PRIVATE` dataset called `API sandbox`, with the following command:

```bash
curl --request POST \
  --url "https://api.data.world/v0/datasets/${DW_USERNAME}" \
  --header "Authorization: Bearer ${DW_API_TOKEN}" \
  --header "Content-type: application/json" \
  --data '{"title":"API Sandbox","visibility":"PRIVATE"}'
```

Where the following must be replaced:

* `${DW_USERNAME}`: your data.world username
* `${DW_API_TOKEN}`: your API authentication token

As a result, the server response should look like this:

```json
{
  "message": "Dataset has been successfully created.",
  "uri": "https://data.world/[USERNAME]/api-sandbox"
}
```

Once a dataset is created, other endpoints can be used to update it, including its set of files. Check out #endpoint:asFRaPWvQM5eDqeKt and #endpoint:XdcQKTziah5iiEsYp, for additional details.

Now, let's see how files can be added to datasets.

## Uploading files

Files can be added to datasets by direct upload (#endpoint:Q9ro4unMQB6rJfM46), from URLs (#endpoint:ujqwuHZYZP8yRKgcz) or via streams (#endpoint:8k5qZPvcfGxXuMF6n).

### Direct Uploads

The #endpoint:Q9ro4unMQB6rJfM46 endpoint allows direct upload os files from the local file system.  
It expects requests of type `multipart/form-data` and you can include one or more parts named `file`, each containing a different file to be uploaded.

For example, assuming that you want to upload two files in the current directory named `file1.csv` and `file2.csv` to the `API Sandbox` dataset you just created, this is what the cURL command would look like.

```bash
curl \
  --header "Authorization: Bearer ${DW_API_TOKEN}" \
  -F "file=@file1.csv" \
  -F "file=@file2.csv" \
  --url https://api.data.world/v0/uploads/${DW_USERNAME}/api-sandbox/files
```

### Add from URL

Alternatively, if you have the URL for a file hosted publicly on the web, you can skip the download/upload steps altogether.

For example, you can get a complete list of NYC subway stations from data.cityofnewyork.us in CSV format at <https://data.cityofnewyork.us/api/views/kk4q-3rt2/rows.csv?accessType=DOWNLOAD>.  
You can add that file to our `API Sandbox` dataset using the #endpoint:ujqwuHZYZP8yRKgcz endpoint with the following command:

```bash
curl --request POST \
  --url "https://api.data.world/v0/datasets/${DW_USERNAME}/api-sandbox/files" \
  --header "Authorization: Bearer ${DW_API_TOKEN}" \
  --header "Content-type: application/json" \
  --data '{"files":[{"name":"nyc-subways.csv","source":{"url":"https://data.cityofnewyork.us/api/views/kk4q-3rt2/rows.csv?accessType=DOWNLOAD"},"description":"List of NYC subway stations","labels":["raw data"]}]}'
```

For clarity, this is the JSON data included in the above request:
```json
{
  "files": [
    {
      "name": "nyc-subways.csv",
      "source": {
        "url": "https://data.cityofnewyork.us/api/views/kk4q-3rt2/rows.csv?accessType=DOWNLOAD"
      },
      "description": "List of NYC subway stations",
      "labels": [
        "raw data"
      ]
    }
  ]
}
```

As a result, the server response should look like this:
```json
{
  "message": "Dataset has been successfully updated with new sources. Sync in progress."
}
```

### Streaming

Data can be streamed to data.world one record at a time using the #endpoint:8k5qZPvcfGxXuMF6n endpoint. data.world will accumulate stream data and periodically (at least once a day) process it, at which point the data becomes consumable with the related dataset.  
A stream named `temperatures`, for example, will appear in the associated dataset as `temperatures.jsonl` once processed.

As needed, users can force stream data to be processed, by invoking the #endpoint:Lqji3375Xx5W3K2Pm endpoint.

To create a stream, simply append the first row of data to a stream that didn't exist before. For example:
```bash
curl --request POST \
  --url "https://api.data.world/v0/streams/${DW_USERNAME}/api-sandbox/temperatures" \
  --header "Authorization: Bearer ${DW_API_TOKEN}" \
  --header "Content-type: application/json" \
  --data '{"temperature": 34, "location": "New York", "measurement_time": "2017-10-24T01:00:00Z"}'
```

It is also possible to append multiple records at once, so long as the content type in the request is set to `application/json-l` ([JSON lines](http://jsonlines.org/)). For example:
```bash
curl --request POST \
  --url "https://api.data.world/v0/streams/${DW_USERNAME}/api-sandbox/temperatures" \
  --header "Authorization: Bearer ${DW_API_TOKEN}" \
  --header "Content-type: application/json-l" \
  --data '{"temperature": 37, "location": "New York", "measurement_time": "2017-10-24T02:00:00Z"}
  {"temperature": 40, "location": "New York", "measurement_time": "2017-10-24T03:00:00Z"}'
```

Processing of files is asyncrhonous and finishes quickly. However, large files can take a few minutes to process. Once the dataset is in `LOADED` status, the data is ready to be consumed.

Let's look at how you can check the status of a dataset.

## Retrieving dataset info

To retrieve dataset info, use the #endpoint:asFRaPWvQM5eDqeKt endpoint. For example:
```bash
curl --request GET \
  --url "https://api.data.world/v0/datasets/${DW_USERNAME}/api-sandbox" \
  --header "Authorization: Bearer ${DW_API_TOKEN}" \
  --header "Content-type: application/json"
```

As a result, the server response should look like this:
```json
{
  "owner": "rflprr",
  "id": "api-sandbox",
  "title": "API Sandbox",
  "visibility": "PRIVATE",
  "files": [
    {
      "name": "nyc-subways.csv",
      "sizeInBytes": 64188,
      "source": {
        "url": "https://data.cityofnewyork.us/api/views/kk4q-3rt2/rows.csv?accessType=DOWNLOAD",
        "expandArchive": false,
        "syncStatus": "OK",
        "lastSyncStart": "2017-10-24T21:27:15.406Z",
        "lastSyncSuccess": "2017-10-24T21:27:04.197Z"
      },
      "created": "2017-10-24T21:27:04.618Z",
      "updated": "2017-10-24T21:27:04.618Z",
      "description": "List of NYC subway stations",
      "labels": [
        "raw data"
      ]
    },
    {
      "name": "stream-temperatures.jsonl",
      "sizeInBytes": 178,
      "source": {
        "url": "https://data.world/rflprr/d/api-sandbox/stream-temperatures",
        "syncStatus": "OK",
        "lastSyncStart": "2017-10-24T21:27:15.406Z",
        "lastSyncSuccess": "2017-10-24T21:27:15.424Z"
      },
      "created": "2017-10-24T21:27:15.683Z",
      "updated": "2017-10-24T21:27:15.683Z"
    }
  ],
  "status": "LOADED",
  "created": "2017-10-24T21:26:49.543Z",
  "updated": "2017-10-24T21:27:18.658Z",
  "accessLevel": "ADMIN",
  "isProject": false
}
```

Note that `syncStatus` is `OK` and `status` is `LOADED`. That indicates that the file was synchronized correctly and that the dataset is ready to use.

Below is what the same dataset looks like on data.world. Observe how each of the dataset attributes returned previously are presented.

!["API Sandbox Dataset"](https://cdn.filepicker.io/api/file/kf4U3SJCTlqk249ICZB3 "API Sandbox Dataset")

## Querying with SQL

Now that the dataset is ready to be used, you can, for example, discover which NYC subway stations the `7 Express` train stops at using the #endpoint:AYbqu2jrzNYYr9xsW endpoint. For example:

```bash
curl --request POST \
  --url "https://api.data.world/v0/sql/${DW_USERNAME}/api-sandbox" \
  --header "Authorization: Bearer ${DW_API_TOKEN}" \
  --header "Accept: text/csv" \
  --data-urlencode 'query=SELECT name, line FROM nyc_subways WHERE line LIKE "%7 Express%"'
```

As a result, the server response should look like this:
```
NAME,LINE
Vernon Blvd - Jackson Ave,7-7 Express
Queensboro Plz,7-7 Express-N-W
Times Sq - 42nd St,7-7 Express
Grand Central - 42nd St,7-7 Express
Mets - Willets Point,7-7 Express
Junction Blvd,7-7 Express
Flushing - Main St,7-7 Express
5th Ave - Bryant Pk,7-7 Express
34th St - Hudson Yards,7-7 Express
Woodside - 61st St,7-7 Express
Court Sq,7-7 Express
Hunters Point Ave,7-7 Express
```

### Additional query options

Query results can be obtained in a variety of different formats, including:

- `text/csv`: Comma-separated values
- `text/tab-separated-values`: Tab-separated values
- `application/json`: JSON array
- `application/x-ndjson` and `application/json-l`: [JSON lines](http://jsonlines.org/)

When requesting a JSON format, you can obtain schema information alongside the data.
To do that, add `includeTableSchema=true` in the query string.
The schema information will be the first element or row in the response.

Here is what the previous request would look like if modified to produce JSON lines including schema information:
```bash
curl --request POST \
  --url "https://api.data.world/v0/sql/${DW_USERNAME}/api-sandbox?includeTableSchema=true" \
  --header "Authorization: Bearer ${DW_API_TOKEN}" \
  --header "Accept: application/json-l" \
  --data-urlencode 'query=SELECT name, line FROM nyc_subways WHERE line LIKE "%7 Express%"'
```

In this case, this is what the response will look like:
```
{"fields":[{"name":"name","type":"string","rdfType":"http://www.w3.org/2001/XMLSchema#string"},{"name":"line","type":"string","rdfType":"http://www.w3.org/2001/XMLSchema#string"}]}
{"name":"Vernon Blvd - Jackson Ave","line":"7-7 Express"}
{"name":"Queensboro Plz","line":"7-7 Express-N-W"}
{"name":"Times Sq - 42nd St","line":"7-7 Express"}
{"name":"Grand Central - 42nd St","line":"7-7 Express"}
{"name":"Mets - Willets Point","line":"7-7 Express"}
{"name":"Junction Blvd","line":"7-7 Express"}
{"name":"Flushing - Main St","line":"7-7 Express"}
{"name":"5th Ave - Bryant Pk","line":"7-7 Express"}
{"name":"34th St - Hudson Yards","line":"7-7 Express"}
{"name":"Woodside - 61st St","line":"7-7 Express"}
{"name":"Court Sq","line":"7-7 Express"}
{"name":"Hunters Point Ave","line":"7-7 Express"}
```

Notice that the first line is the [table schema](https://specs.frictionlessdata.io/table-schema/), as expected:
```json
{
   "fields":[
      {
         "name":"name",
         "type":"string",
         "rdfType":"http://www.w3.org/2001/XMLSchema#string"
      },
      {
         "name":"line",
         "type":"string",
         "rdfType":"http://www.w3.org/2001/XMLSchema#string"
      }
   ]
}
```

## Next steps

1. Take some time to read the "Conventions" section below and to explore more endpoints and their documentation.
2. Check out existing [integrations](https://data.world/integrations) to see if an integration for your favorite tools already exists, and skip using this API altogether. *Pro tip:* come back often as we are constantly implementing new integrations.
3. Learn how easily you can generate #docTextSection:8NpYGToZm2LsA9A9J for the language of your choice.
4. Browse all endpoints and use the **Try it out** feature to interact with them in your browser.
5. Let us know what cool things you create and make sure to reach out if you need support. We love to hear from you, so connect with us through our [help portal](https://help.data.world/hc/en-us/requests/new) or on [Slack](https://slack.data.world). 

# Conventions

## Authentication
All data.world API calls require an API token. After logging into data.world, find your token by navigating to your profile settings, under the Advanced tab or go to [https://data.world/settings/advanced](https://data.world/settings/advanced)

Authentication must be provided in API requests via the `Authorization` header. For example, for a user whose API token is `my_api_token`, the request header should be `Authorization: Bearer my_api_token` (note the `Bearer` prefix).

Those developing applications and integrations are encouraged to implement authentication directly within their apps via #docTextSection:QRKZGmkzDw89ibrYm.

## Content type  
By default, `application/json` is the content type used in request and response bodies. Exceptions are noted in respective endpoint documentation.

## HTTPS only  
Our APIs can only be accessed via HTTPS.
