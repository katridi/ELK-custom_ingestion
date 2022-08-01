# Task description
## Create custom non-English analyzer for html based documents.
## Make sure that Roman numbers are replaced as is with it’s Arabic numbers variation.

<br></br>

## Create pipeline: see [file](./pipeline.json) 

## Check pipeline

``` console
POST _ingest/pipeline/roman-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "text": "<br> XIX ВЕК фОкс </br>"
      }
    }
  ]
}
```

## Ingest data with pipeline


``` console
POST russian_example/_doc?pipeline=roman-pipeline
{
       "text": "<br> X <b>ВЕК</b> фОкс </br>"
}
```

## Check data
``` console
POST russian_example/_search
```