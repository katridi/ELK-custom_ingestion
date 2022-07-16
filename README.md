# Task description
## Create custom non-English analyzer for html based documents.
## Make sure that Roman numbers are replaced as is with it’s Arabic numbers variation.

<br></br>
## Execute analyzer

```
PUT /russian_example
{
  "settings": {
    "analysis": {
      "filter": {
        "russian_stop": {
          "type":       "stop",
          "stopwords":  "_russian_" 
        },
        "russian_keywords": {
          "type":       "keyword_marker",
          "keywords":   ["пример"] 
        }
      },
      "analyzer": {
        "custom_russian": {
          "type":"custom", 
          "char_filter":["html_strip"],
          "tokenizer":  "standard",
          "filter": [ "lowercase"]
        }
      }
    }
  },
  "mappings":{
    "properties":{
       "text": {
          "type":"text",
          "analyzer":"custom_russian"
      },
      "new_text": {
        "type":"text",
        "analyzer":"custom_russian"
    }
   }
}
}
```

## Check query 

``` console
POST russian_example/_analyze 
{
  "analyzer": "custom_russian", 
  "text":     "<br> XX век фокс ты ли это</br>?"
}
```


## Execute pipeline

``` console
PUT _ingest/pipeline/roman-pipeline
{
  "description": "My optional pipeline description",
  "processors": [
    {
        "script": {
          "description": "Convert roman to arabic numbers, save it in new_text field",
          "lang": "painless",
          "source": """
            Map map = [
            "I" : "1+",
            "IV" : "4+",
            "V" : "5+",
            "IX" : "9+",
            "X" : "10+",
            "XL" : "40+",
            "L" : "50+",
            "XC" : "90+",
            "C" : "100+",
            "CD" : "400+",
            "D" : "500+",
            "CM" : "900+",
            "M" : "1000+"
            ];

            String s = ctx['text'];
            List list = ["CM", "CD", "XC", "XL", "IX", "IV", "M", "D", "C", "L", "X", "V", "I"];
            for (value in list){ s = s.replace(value, map[value]); }

            def parts = s.splitOnToken(" ");
            String normalized = "";
            for(item in parts){
                if (item.contains("+")){
                    int sum = 0;
                    for (string_number in item.splitOnToken("+")) {
                        try {
                            sum += Integer.parseInt(string_number.toString());
                        }
                        catch(java.lang.NumberFormatException e) {}
                    } 
                    normalized += sum.toString();
                }
                else {
                    normalized += item;
                }
                normalized += " ";
            }
            ctx['new_text'] = normalized.trim();
          """
        }
      }
  ]
}
```

## Check pipeline

``` console
POST _ingest/pipeline/roman-pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "text": "<br> XIX век фокс </br>"
      }
    }
  ]
}
```

## Ingest data with pipeline

NB - pipeline first, then analyzer

``` console
POST russian_example/_doc?pipeline=roman-pipeline
{
        "text": "<br> XIX век фокс </br>"
}
```

## Check data
``` console
POST russian_example/_search
```