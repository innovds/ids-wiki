Console récupérer des champs en format csv avec requete sql
```
POST _sql?format=csv
{
  "query": """SELECT
        "@timestamp", "@service", method, uri, params, elapsed, status, "@trace_id", "@span_id", remote_ip
    FROM "http-access-*"
    WHERE "@service" IN ('skill-ms', 'project-ms') AND uri NOT LIKE '/actuator/%'
    ORDER BY "@timestamp" ASC
  """
}
```
Console récupérer des champs en format csv avec api _search
```
GET http-access-*/_search
{
  "_source": [
    "@timestamp",
    "@service",
    "method",
    "uri",
    "params",
    "elapsed",
    "status",
    "@trace_id",
    "@span_id",
    "remote_ip"
  ],
  "size": 10000,
  "sort": [
    {
      "@timestamp": {
        "order": "asc"
      }
    }
  ], 
  "query": {
    "bool": {
      "filter": [
        {
          "terms": {
            "@service.keyword": [
              "skill-ms",
              "project-ms"
            ]
          }
        }
      ],
      "must_not": [
        {
          "wildcard": {
            "uri": "/actuator/*"
          }
        }
      ]
    }
  }
}
```

requête  Kibana count API
GET /http-access-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "@timestamp": {
              "gte": "2024-07-22T00:00:00",
              "lte": "2024-07-26T23:59:59"
            }
          }
        },
        {
          "bool": {
            "must_not": [
              {
                "wildcard": {
                  "uri.keyword": "/actuator/*"
                }
              },
              {
                "wildcard": {
                  "uri.keyword": "/websocket/*"
                }
              }
            ]
          }
        },
        {
          "bool": {
            "should": [
              { "match_phrase": { "@service.keyword": "skill-ms" }},
              { "match_phrase": { "@service.keyword": "project-ms" }},
              {
                "bool": {
                  "must": [
                    { "match_phrase": { "@service.keyword": "mission-ms" }},
                    { "terms": { "uri.keyword": [ "/api/v1/missions", "/api/v1/engineers-follow-up" ] }}
                  ]
                }
              },
              {
                "bool": {
                  "must": [
                    { "match_phrase": { "@service.keyword": "hr-ms" }},
                    { "terms": { "uri.keyword": [ "/api/v1/company-config", "/api/v1/human-resources", "/api/v1/hr-by-ids-as-basic", "/api/v1/hr-by-fullname", "/api/v1/sites-by-code", "/api/v1/user-settings-by-user", "/api/v1/user-accounts/me" ] }}
                  ]
                }
              },
              {
                "bool": {
                  "must": [
                    { "match_phrase": { "@service.keyword": "td-ms" }},
                    { "terms": { "uri.keyword": [ "/api/v1/experience", "/api/v1/technical-documents", "/api/v1/technical-documents/document", "/api/v1/technical-documents/download-cv-by-hr-id" ] }}
                  ]
                }
              },
              {
                "bool": {
                  "must": [
                    { "match_phrase": { "@service.keyword": "assessment-ms" }},
                    { "terms": { "uri.keyword": [ "/api/eval/v1/evaluations-by-hr" ] }}
                  ]
                }
              },
              {
                "bool": {
                  "must": [
                    { "match_phrase": { "@service.keyword": "recruitment-ms" }},
                    { "terms": { "uri.keyword": [ "/api/v1/recruitment-workflows" ] }}
                  ]
                }
              }
            ],
            "minimum_should_match": 1
          }
        }
      ]
    }
  },
    "aggs": {
    "data_over_time_custom": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "30m",
        "min_doc_count": 0
      },
        "aggs": {
        "uri_terms_custom": {
        "terms": {
          "script": {
            "source": "\r\n                def uri = doc['uri.keyword'].value;\r\n                def m = /\\/\\d{2,}/.matcher(uri);\r\n                return m.replaceAll('/:id')\r\n              "
          },
        "size": 100
        }
      }
    }}
  }
}
