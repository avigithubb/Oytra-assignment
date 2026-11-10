[Task_2_workflow_Abhishek_Napit.json](https://github.com/user-attachments/files/34624100/Task_2_workflow_Abhishek_Napit.json)
# Oytra-assignment

## Task 2 Explanation and completion:

- Used github API i.e. https://api.github.com/search/repositories to fetch the repositories from github using n8n's fetch action.
- Filtered top 5 records according to the stars.
- Fetched top repository Readme.md file along with its content.
- stored the top 5 repositories in a single object called repos.
- Implemented a check to know if the current repository have stars more than 1000 or not.
- If it does then formed a message including the details of the repository and the latest news.
- Sent the formed message to Gmail.
- If the repository doesn't have enough stars then it falls back to the failure action where we created a message for the failure mail and sent it to the email.

<img width="1500" height="435" alt="Screenshot 2026-11-10 151650" src="https://github.com/user-attachments/assets/7477b510-139b-4604-8ffa-47b36f300b9f" />
<img width="1450" height="566" alt="Screenshot 2026-11-10 151720" src="https://github.com/user-attachments/assets/5853c1e8-97cc-4470-8c06-da4183e96848" />

-----------------------------------------------------------------------------------------------------------------------------------------------------------------
JSON of workflow
-----------------------------------------------------------------------------------------------------------------------------------------------------------------
  "name": "GitHub Trending Morning Brief",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "hours"
            }
          ]
        }
      },
      "id": "3e2cc690-2f0d-424a-b5ba-ddf368ae1f52",
      "name": "Every Hour",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.4,
      "position": [
        0,
        96
      ]
    },
    {
      "parameters": {
        "url": "https://api.github.com/search/repositories",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "q",
              "value": "=topic:ai created:>{{ $now.minus({ days: 30 }).toFormat('yyyy-MM-dd') }}"
            },
            {
              "name": "sort",
              "value": "stars"
            },
            {
              "name": "order",
              "value": "desc"
            },
            {
              "name": "per_page",
              "value": "20"
            }
          ]
        },
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Accept",
              "value": "application/vnd.github+json"
            }
          ]
        },
        "options": {
          "timeout": 15000
        }
      },
      "id": "5d294b40-b121-4538-95cd-9d268ee2cd30",
      "name": "Fetch Trending Repos",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.5,
      "position": [
        224,
        96
      ],
      "onError": "continueErrorOutput"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "e1",
              "name": "status",
              "type": "string",
              "value": "failed"
            },
            {
              "id": "e2",
              "name": "failedNode",
              "type": "string",
              "value": "Fetch Trending Repos"
            },
            {
              "id": "e3",
              "name": "errorMessage",
              "type": "string",
              "value": "={{ String($input.item.json.error ? ($input.item.json.error.message || JSON.stringify($input.item.json.error)) : 'Unknown error').replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;') }}"
            },
            {
              "id": "e4",
              "name": "failedAt",
              "type": "string",
              "value": "={{ $now.toISO() }}"
            },
            {
              "id": "e5",
              "name": "executionId",
              "type": "string",
              "value": "={{ $execution.id }}"
            }
          ]
        },
        "options": {}
      },
      "id": "5f5c5928-7736-4df6-81da-0c8fbcfe1848",
      "name": "Log API Failure",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.5,
      "position": [
        496,
        320
      ]
    },
    {
      "parameters": {
        "sendTo": "avishek.npt@gmail.com",
        "subject": "=Morning Brief FAILED - {{ $json.failedAt }}",
        "message": "=<b>Morning Brief FAILED</b><br>Node: {{ $json.failedNode }}<br>Error: {{ $json.errorMessage }}<br>Time: {{ $json.failedAt }}<br>Execution: {{ $json.executionId }}",
        "options": {
          "appendAttribution": false
        }
      },
      "id": "3e0c9d21-251c-43ee-ad3c-61a30e7e850e",
      "name": "Email Failure Alert",
      "type": "n8n-nodes-base.gmail",
      "typeVersion": 2.2,
      "position": [
        720,
        320
      ],
      "webhookId": "54ccd204-04c9-4fbf-bb8a-ba9090bb093e",
      "credentials": {
        "gmailOAuth2": {
          "id": "UYbbh7aLaASVMXK6",
          "name": "Gmail account"
        }
      }
    },
    {
      "parameters": {
        "jsCode": "const esc = s => String(s || '').replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');\nconst raw = $input.first().json.items || [];\n\nconst repos = raw\n  .filter(r => !r.fork && !r.archived)\n  .map(r => ({\n    fullName: r.full_name,\n    nameHtml: esc(r.full_name),\n    url: r.html_url,\n    description: esc((r.description || 'No description').slice(0, 120)),\n    stars: Number(r.stargazers_count) || 0,\n    language: r.language || 'n/a',\n  }))\n  .sort((a, b) => b.stars - a.stars)\n  .slice(0, 5);\n\nreturn [{\n  json: {\n    repos,\n    topRepo: repos[0] || null,\n    repoCount: repos.length,\n    generatedAt: $now.toFormat('yyyy-MM-dd HH:mm'),\n  },\n}];"
      },
      "id": "17f5b8f1-5900-44e5-8f15-a7c2bd3f0476",
      "name": "Keep Top 5",
      "type": "n8n-nodes-base.code",
      "typeVersion": 2,
      "position": [
        496,
        128
      ]
    },
    {
      "parameters": {
        "url": "=https://api.github.com/repos/{{ $json.topRepo.fullName }}/readme",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Accept",
              "value": "application/vnd.github.raw"
            }
          ]
        },
        "options": {
          "response": {
            "response": {
              "responseFormat": "text",
              "outputPropertyName": "readme"
            }
          },
          "timeout": 15000
        }
      },
      "id": "eb15855b-d0e1-4164-bfc9-8e93b1fefdc7",
      "name": "Fetch Top Repo README",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.5,
      "position": [
        720,
        128
      ],
      "onError": "continueRegularOutput"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "a1",
              "name": "repos",
              "type": "array",
              "value": "={{ $('Keep Top 5').first().json.repos }}"
            },
            {
              "id": "a2",
              "name": "topRepo",
              "type": "object",
              "value": "={{ $('Keep Top 5').first().json.topRepo }}"
            },
            {
              "id": "a3",
              "name": "topStars",
              "type": "number",
              "value": "={{ $('Keep Top 5').first().json.topRepo ? $('Keep Top 5').first().json.topRepo.stars : 0 }}"
            },
            {
              "id": "a4",
              "name": "generatedAt",
              "type": "string",
              "value": "={{ $('Keep Top 5').first().json.generatedAt }}"
            },
            {
              "id": "a5",
              "name": "readmeExcerpt",
              "type": "string",
              "value": "={{ $json.readme ? String($json.readme).replace(/<[^>]*>/g, ' ').replace(/!\\[[^\\]]*\\]\\([^)]*\\)/g, ' ').replace(/[#*`>]/g, '').replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/\\s+/g, ' ').trim().slice(0, 300) + '...' : 'README unavailable (enrichment call failed: ' + ($json.error && $json.error.message ? $json.error.message : 'unknown error') + ')' }}"
            }
          ]
        },
        "options": {}
      },
      "id": "b7b92a87-c7f6-4a45-a16f-d54047f58739",
      "name": "Add README Excerpt",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.5,
      "position": [
        944,
        128
      ]
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "loose",
            "version": 2
          },
          "conditions": [
            {
              "id": "c1",
              "leftValue": "={{ $json.topStars }}",
              "rightValue": 1000,
              "operator": {
                "type": "number",
                "operation": "gt"
              }
            }
          ],
          "combinator": "and"
        },
        "looseTypeValidation": true,
        "options": {}
      },
      "id": "97be641e-f3fb-495b-9151-38b94820f0b1",
      "name": "Top Repo Over 1000 Stars?",
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.2,
      "position": [
        1024,
        224
      ]
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "h1",
              "name": "message",
              "type": "string",
              "value": "=<b>GitHub Morning Brief - {{ $json.generatedAt }}</b>\nHOT: the top new AI repo has passed 1,000 stars.\n\n<b>Spotlight: {{ $json.topRepo.nameHtml }}</b> ({{ $json.topStars }} stars)\n<i>{{ $json.readmeExcerpt }}</i>\n\n<b>Top 5 new AI repos (last 30 days)</b>\n{{ $json.repos.map((r, i) => (i + 1) + '. <a href=\"' + r.url + '\">' + r.nameHtml + '</a> (' + r.stars + ' stars, ' + r.language + ') - ' + r.description).join('\\n') }}"
            }
          ]
        },
        "options": {}
      },
      "id": "886c67d2-6848-456b-ac9a-529efa205984",
      "name": "Format Hot Digest",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.5,
      "position": [
        1248,
        128
      ]
    },
    {
      "parameters": {
        "sendTo": "avishek.npt@gmail.com",
        "subject": "=GitHub Morning Brief - {{ $(\"Add README Excerpt\").item.json.generatedAt }}",
        "message": "=<div style=\"font-family: Arial, sans-serif; font-size: 14px; line-height: 1.5;\">{{ $json.message.replace(/\\n/g, \"<br>\") }}</div>",
        "options": {
          "appendAttribution": false
        }
      },
      "id": "059df463-5b0c-4aae-be88-8707d13d5285",
      "name": "Email Morning Brief",
      "type": "n8n-nodes-base.gmail",
      "typeVersion": 2.2,
      "position": [
        1472,
        224
      ],
      "webhookId": "2e353989-90b3-4b1c-a27e-eb76613e0070",
      "credentials": {
        "gmailOAuth2": {
          "id": "UYbbh7aLaASVMXK6",
          "name": "Gmail account"
        }
      }
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "r1",
              "name": "message",
              "type": "string",
              "value": "=<b>GitHub Morning Brief - {{ $json.generatedAt }}</b>\nQuiet day: no new AI repo above 1,000 stars yet. Rising picks:\n\n<b>Watch: {{ $json.topRepo ? $json.topRepo.nameHtml : 'none found' }}</b> ({{ $json.topStars }} stars)\n<i>{{ $json.readmeExcerpt }}</i>\n\n<b>Top 5 new AI repos (last 30 days)</b>\n{{ $json.repos.map((r, i) => (i + 1) + '. <a href=\"' + r.url + '\">' + r.nameHtml + '</a> (' + r.stars + ' stars, ' + r.language + ') - ' + r.description).join('\\n') }}"
            }
          ]
        },
        "options": {}
      },
      "id": "e768a4b7-4e46-4a1f-b46b-7d3506586b4a",
      "name": "Format Rising Digest",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.5,
      "position": [
        1248,
        320
      ]
    }
  ],
  "pinData": {},
  "connections": {
    "Every Hour": {
      "main": [
        [
          {
            "node": "Fetch Trending Repos",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Fetch Trending Repos": {
      "main": [
        [
          {
            "node": "Keep Top 5",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Log API Failure",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Log API Failure": {
      "main": [
        [
          {
            "node": "Email Failure Alert",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Keep Top 5": {
      "main": [
        [
          {
            "node": "Fetch Top Repo README",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Fetch Top Repo README": {
      "main": [
        [
          {
            "node": "Add README Excerpt",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Add README Excerpt": {
      "main": [
        [
          {
            "node": "Top Repo Over 1000 Stars?",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Top Repo Over 1000 Stars?": {
      "main": [
        [
          {
            "node": "Format Hot Digest",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Format Rising Digest",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Format Hot Digest": {
      "main": [
        [
          {
            "node": "Email Morning Brief",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Format Rising Digest": {
      "main": [
        [
          {
            "node": "Email Morning Brief",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1",
    "binaryMode": "separate",
    "availableInMCP": true
  },
  "versionId": "12dba8fb-047a-43d4-a045-ee711352a513",
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "85ec284c3aa12ee1842873fc55c9f31a649c572f211a6f54ec30440a8061c808"
  },
  "nodeGroups": [
    {
      "id": "fbe98b2f-1f4d-40d9-a6a5-b1266f530bcb",
      "name": "Rank and enrich repos",
      "nodeIds": [
        "17f5b8f1-5900-44e5-8f15-a7c2bd3f0476",
        "eb15855b-d0e1-4164-bfc9-8e93b1fefdc7",
        "b7b92a87-c7f6-4a45-a16f-d54047f58739"
      ],
      "description": "Keeps the 5 most-starred new repos, then pulls the leader's README; a failed lookup falls back to a placeholder note"
    },
    {
      "id": "d2f26550-c5c1-485c-a978-62c5ae31cf3e",
      "name": "Deliver morning brief",
      "nodeIds": [
        "97be641e-f3fb-495b-9151-38b94820f0b1",
        "886c67d2-6848-456b-ac9a-529efa205984",
        "e768a4b7-4e46-4a1f-b46b-7d3506586b4a",
        "059df463-5b0c-4aae-be88-8707d13d5285"
      ],
      "description": "Picks a \"hot\" or \"rising\" tone based on a 1,000-star threshold and emails the digest to your inbox"
    },
    {
      "id": "19b223e4-1a9a-48ee-a180-bb0fd66a2ca6",
      "name": "Alert on fetch failure",
      "nodeIds": [
        "5f5c5928-7736-4df6-81da-0c8fbcfe1848",
        "3e0c9d21-251c-43ee-ad3c-61a30e7e850e"
      ],
      "description": "Runs only if the trending search fails: records the error details and emails you so nothing fails silently"
    }
  ],
  "id": "vXY1kJvpV6EM0cYU",
  "tags": []
}
