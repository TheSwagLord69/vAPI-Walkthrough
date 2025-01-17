# Setup vAPI
```
cd /home/kali/Desktop/lab
sudo git clone https://github.com/roottusk/vapi.git
cd vapi
sudo docker-compose up -d
```

Once vAPI is running you can navigate to the the vAPI home page:
```
http://127.0.0.1/vapi
```

Note that vAPI comes with a prebuilt Postman collection and environment. 
You can access these in the vAPI/postman folder:
```
cd /home/kali/Desktop/lab/vapi/postman
ls
```

You can import these into Postman to be prepared for testing for future assessments. Simply open Postman, select the Import button (top right), and select the two vAPI JSON documents (see above image). Finally, confirm the import and select the Import button.

One more thing to note about vAPI is that the Resources folder contains secrets that will be necessary to complete certain attacks. The resources folder can be found at:
```
cd /home/kali/Desktop/lab/vapi/Resources
ls
```

# Docker Commands

Lists containers for a Compose project, with current status and exposed ports.
```
sudo docker-compose ps
```

Starts existing containers for a service
```
sudo docker-compose start
```

Stops running containers without removing them. They can be started again with docker compose start
```
sudo docker-compose stop
```

# API documentation

Download and view thevAPI (1.1) OpenAPI specification
```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "vAPI",
    "description": "vAPI is Vulnerable Adversely Programmed Interface which is Self-Hostable API that mimics OWASP API Top 10 scenarios in the means of Exercises.\n\n# Authentication\n\n<!-- ReDoc-Inject: <security-definitions> -->",
    "version": 1.1
  },
  "servers": [
    {
      "url": "http://{{host}}"
    }
  ],
  "components": {
    "securitySchemes": {
      "noauthAuth": {
        "type": "http",
        "scheme": "noauth"
      }
    }
  },
  "tags": [
    {
      "name": "API1",
      "description": "Broken Object Level Authorization\r\n\r\nYou can register yourself as a User , Thats it ....or is there something more?"
    },
    {
      "name": "API2",
      "description": "Broken Authentication\r\n\r\nWe don't seem to have credentials for this , How do we login? (There's something in the Resources Folder given to you )"
    },
    {
      "name": "API3",
      "description": "Excessive Data Exposure\r\n\r\nWe have all been there , right? Giving away too much data and the Dev showing it . Try the Android App in the Resources folder"
    },
    {
      "name": "API4",
      "description": "Lack of Resources & Rate Limiting\r\n\r\nWe believe OTPs are a great way of authenticating users and secure too if implemented correctly!"
    },
    {
      "name": "API5",
      "description": "Broken Function Level Authorization\r\n\r\nYou can register yourself as a User. Thats it or is there something more? (I heard admin logins often but uses different route)"
    },
    {
      "name": "API6",
      "description": "Mass Assignment\r\n\r\nWelcome to our store , We will give you credits if you behave nicely. Our credit management is super secure"
    },
    {
      "name": "API7",
      "description": "Security Misconfiguration\r\n\r\nHey , its an API right? so we ARE expecting Cross Origin Requests . We just hope it works fine."
    },
    {
      "name": "API8",
      "description": "Injection\r\n\r\nI think you won't get credentials for this.You can try to login though."
    },
    {
      "name": "API9",
      "description": "Improper Assets Management\r\n\r\nHey Good News!!!!! We just launched our v2 API :)"
    },
    {
      "name": "API9 > v2"
    },
    {
      "name": "API10",
      "description": "Nothing has been logged or monitored , You caught us :( !"
    }
  ],
  "paths": {
    "/vapi/api1/user": {
      "post": {
        "tags": [
          "API1"
        ],
        "summary": "Create User",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "username": "",
                  "name": "",
                  "course": "",
                  "password": ""
                }
              }
            }
          }
        },
        "security": [
          {
            "noauthAuth": []
          }
        ],
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          },
          {
            "name": "Accept",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api1/user/{api1_id}": {
      "get": {
        "tags": [
          "API1"
        ],
        "summary": "Get User",
        "parameters": [
          {
            "name": "Authorization-Token",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "{{api1_auth}}"
          },
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          },
          {
            "name": "api1_id",
            "in": "path",
            "schema": {
              "type": "string"
            },
            "required": true
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      },
      "put": {
        "tags": [
          "API1"
        ],
        "summary": "Update User",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "username": "",
                  "name": "",
                  "course": "",
                  "password": ""
                }
              }
            }
          }
        },
        "security": [
          {
            "noauthAuth": []
          }
        ],
        "parameters": [
          {
            "name": "Authorization-Token",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "{{api1_auth}}"
          },
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          },
          {
            "name": "api1_id",
            "in": "path",
            "schema": {
              "type": "string"
            },
            "required": true
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api2/user/login": {
      "post": {
        "tags": [
          "API2"
        ],
        "summary": "User Login",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "email": "",
                  "password": ""
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api2/user/details": {
      "get": {
        "tags": [
          "API2"
        ],
        "summary": "Get Details",
        "parameters": [
          {
            "name": "Authorization-Token",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "{{api2_auth}}"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api3/user": {
      "post": {
        "tags": [
          "API3"
        ],
        "summary": "Create User",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "username": "",
                  "password": "",
                  "name": ""
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api4/login": {
      "post": {
        "tags": [
          "API4"
        ],
        "summary": "Mobile Login",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "mobileno": "8000000535"
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api4/otp/verify": {
      "post": {
        "tags": [
          "API4"
        ],
        "summary": "Verify OTP",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "otp": "9999"
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api4/user": {
      "get": {
        "tags": [
          "API4"
        ],
        "summary": "Get Details",
        "parameters": [
          {
            "name": "Authorization-Token",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "{{api4_key}}"
          },
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api5/user": {
      "post": {
        "tags": [
          "API5"
        ],
        "summary": "Create User",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "username": "testuser2",
                  "password": "test123",
                  "name": "Test User",
                  "address": "ABC",
                  "mobileno": "888888888"
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api5/user/{api5_id}": {
      "get": {
        "tags": [
          "API5"
        ],
        "summary": "Get User",
        "parameters": [
          {
            "name": "Authorization-Token",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "{{api5_auth}}"
          },
          {
            "name": "api5_id",
            "in": "path",
            "schema": {
              "type": "string"
            },
            "required": true
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api6/user": {
      "post": {
        "tags": [
          "API6"
        ],
        "summary": "Create User",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "name": "",
                  "username": "",
                  "password": ""
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api6/user/me": {
      "get": {
        "tags": [
          "API6"
        ],
        "summary": "Get User",
        "parameters": [
          {
            "name": "Authorization-Token",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "{{api6_auth}}"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api7/user": {
      "post": {
        "tags": [
          "API7"
        ],
        "summary": "Create User",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "username": "",
                  "password": ""
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api7/user/login": {
      "get": {
        "tags": [
          "API7"
        ],
        "summary": "User Login",
        "parameters": [
          {
            "name": "Authorization-Token",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "{{api7_auth}}"
          },
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api7/user/key": {
      "get": {
        "tags": [
          "API7"
        ],
        "summary": "Get Key",
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api7/user/logout": {
      "get": {
        "tags": [
          "API7"
        ],
        "summary": "User Logout",
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api8/user/login": {
      "post": {
        "tags": [
          "API8"
        ],
        "summary": "User Login",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "username": "",
                  "password": ""
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api8/user/secret": {
      "get": {
        "tags": [
          "API8"
        ],
        "summary": "Get Secret",
        "parameters": [
          {
            "name": "Authorization-Token",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "{{api8_auth}}"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api9/v2/user/login": {
      "post": {
        "tags": [
          "API9 > v2"
        ],
        "summary": "Login",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "example": {
                  "username": "richardbranson",
                  "pin": "****"
                }
              }
            }
          }
        },
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    },
    "/vapi/api10/user/flag": {
      "get": {
        "tags": [
          "API10"
        ],
        "summary": "Get Flag",
        "description": "I am not kidding!",
        "parameters": [
          {
            "name": "Content-Type",
            "in": "header",
            "schema": {
              "type": "string"
            },
            "example": "application/json"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {}
            }
          }
        }
      }
    }
  }
}
```

View a prettified version with the online swagger editor
```
https://editor.swagger.io/
```

# Import Postman Collection and Postman Environment

Navigate to the file location
```bash
cd /home/kali/Desktop/labs/vapi/postman
```

Create backups of the files
```bash
sudo cp vAPI.postman_collection.json vAPI.postman_collection.json.backup
sudo cp vAPI_ENV.postman_environment.json vAPI_ENV.postman_environment.json.backup
```

Start up Postman
```bash
/opt/Postman/Postman
```

Import Postman Collection and Postman Environment
1. File > Import... > Upload Files 
2. Select the Postman Collection file and the Postman Environment file
3. Click Import

References:
https://zerodayhacker.com/vapi-walkthrough/
https://owasp.org/API-Security/editions/2023/en/0x00-header/
https://owasp.org/API-Security/editions/2019/en/0x00-header/