# 📝 Create a swagger Docs:

## 1️⃣ Packages:
```json
    "swagger-jsdoc": "^6.2.8",
    "swagger-ui-express": "^5.0.1"
```

## 2️⃣ swagger options in ./config:
``swagger-options.js:``
```js
const swaggerOptions = {
    validatorUrl: false,
    defaultModelsExpandDepth: -1,
    filter: true,
}

module.exports = swaggerOptions;
```

## 3️⃣ the Output Docs:
``swagger_output.json``
```json
{
    "openapi": "3.0.2",
    "info": {
      "title": "Mail Dashboard Api",
      "description": "Mail Dashboard API Documentation",
      "version": "1.0.0"
    },
    "servers": [
        {
        "url": "http://localhost:5000/api/v2" 
        }
    ],
    "paths": {
        "/auth/login": {
            "post": {
                "tags": [
  
  .............code...........
```

## 4️⃣ index.js:
```js
const swaggerUi = require('swagger-ui-express');
const swaggerOptions = require('./src/config/swagger-options')
const swaggerFile = require('./src/public/swagger_output.json');

app.use('/doc', swaggerUi.serve, swaggerUi.setup(swaggerFile, {
    swaggerOptions: swaggerOptions
}))
```

--- 
# A complete code of swagger_output.json:
```json
{
    "openapi": "3.0.2",
    "info": {
      "title": "Mail Dashboard Api",
      "description": "Mail Dashboard API Documentation",
      "version": "1.0.0"
    },
    "servers": [
        {
        "url": "http://localhost:5000/api/v2" 
        }
    ],
    "paths": {
        "/auth/login": {
            "post": {
                "tags": [
                    "Auth Endpoints"
                ],
                "summary": "Login endpoint",
                "requestBody":{
                    "required":true,
                    "content":{
                        "application/json":{
                            "schema":{
                                "type":"object",
                                "properties":{
                                    "email":{
                                        "type":"string",
                                        "example":"super@admin.dad"
                                    },
                                    "password":{
                                        "type":"string",
                                        "example":"Root@1234@"
                                    },
                                    "rememberMe":{
                                        "type":"boolean",
                                        "example":true
                                    }
                                },
                                "required":["email", "password"]
                            }
                        }
                    }
                },
                "responses": {
                    "200": {
                    "description": "OK"
                    },
                    "401": {
                    "description": "Unauthorized"
                    },
                    "403": {
                    "description": "Forbidden"
                    }
                }
            }
        },
        "/auth/profile": {
            "get": {
                "tags": [
                    "Auth Endpoints"
                ],
                "description": "Getting logged user profile endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses": {
                    "200": {
                        "description": "OK"
                    },
                    "400": {
                        "description": "Bad Request"
                    },
                    "403": {
                        "description": "Forbidden"
                    },
                    "500": {
                        "description": "Internal Server Error"
                    }
                }
            }
            },
        "/user/all":{
            "get":{
                "tags": [
                    "Users Endpoints"
                ],
                "description": "Get all users endpoint",
                "security":[
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/user/add": {
            "post": {
                "tags": [
                    "Users Endpoints"
                ],
                "description": "",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "requestBody":{
                    "required":true,
                    "content":{
                        "application/json":{
                            "schema":{
                                "type":"object",
                                "properties":{
                                    "firstname":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "lastname":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "email":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "pseudo":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "role":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "password":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "blocked":{
                                        "type":"string",
                                        "example":false
                                    }
                                }
                            }
                        }
                    }
                },
                "responses": {
                    "200": {
                        "description": "OK"
                    },
                    "400": {
                        "description": "Bad Request"
                    },
                    "403": {
                        "description": "Forbidden"
                    },
                    "409": {
                        "description": "Conflict"
                    },
                    "500": {
                        "description": "Internal Server Error"
                    }
                }
            }
        },
        "/user/{id}":{
            "put":{
                "tags":[
                    "Users Endpoints"
                ],
                "description":"",
                "security":[
                    {
                        "BearerAuth":[]
                    }
                ],
                "parameters":[
                    {
                        "name": "id",
                        "in": "path",
                        "required": true,
                        "schema":{
                            "type": "string"
                        }
                    }
                ],
                "requestBody":{
                    "required":true,
                    "content":{
                        "application/json":{
                            "schema":{
                                "type":"object",
                                "properties":{
                                    "firstname":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "lastname":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "email":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "pseudo":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "role":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "oldPassword":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "password":{
                                        "type":"string",
                                        "example":"any"
                                    },
                                    "blocked":{
                                        "type":"string",
                                        "example":false
                                    }
                                }
                            }
                        }
                    }
                },
                "responses": {
                    "200": {
                        "description": "OK"
                    },
                    "400": {
                        "description": "Bad Request"
                    },
                    "403": {
                        "description": "Forbidden"
                    },
                    "409": {
                        "description": "Conflict"
                    },
                    "500": {
                        "description": "Internal Server Error"
                    }
                }
            },
            "delete": {
                "tags": [
                    "Users Endpoints"
                ],
                "description": "",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "parameters": [
                    {
                        "name": "id",
                        "in": "path",
                        "required": true,
                        "type": "string"
                }
                ],
                "responses": {
                    "201": {
                        "description": "Created"
                    },
                    "403": {
                        "description": "Forbidden"
                    },
                    "500": {
                        "description": "Internal Server Error"
                    }
                }
            }
        },
        "/uploader/upload": {
            "post": {
                "summary": "Upload 10 file max",
                "tags": ["Uploader"],
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "requestBody": {
                    "required": true,
                    "content": {
                    "multipart/form-data": {
                        "schema": {
                        "type": "object",
                        "properties": {
                            "files": {
                            "type": "array",
                            "items":{
                                "type": "string",
                                "format": "binary"
                            },
                            "description": "File to upload"
                            }
                        }
                        }
                    }
                    }
                },
                "responses": {
                    "200": {
                    "description": "File uploaded successfully"
                    },
                    "400": {
                    "description": "Bad request"
                    },
                    "500": {
                    "description": "Internal server error"
                    }
                }
            }
        },
        "/info/mail-data":{
            "get":{
                "tags": [
                    "Mails info Endpoints"
                ],
                "description": "Getting all Mails endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/info/mail-data/summary":{
            "get":{
                "tags": [
                    "Mails info Endpoints"
                ],
                "description": "Getting Mails without body endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/info/mail-data/by-ip/{ip}":{
            "get":{
                "tags": [
                    "Mails info Endpoints"
                ],
                "description": "Getting Mail by IP endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "parameters":[
                    {
                        "name": "ip",
                        "in": "path",
                        "required": true,
                        "type": "string"
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/info/last-upload":{
            "get":{
                "tags": [
                    "Mails info Endpoints"
                ],
                "description": "Getting the last upload endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/counter/count":{
            "get":{
                "tags": [
                    "Mails Counts Endpoints (Aggregation)"
                ],
                "description": "Getting the number of mails endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/counter/file-counts":{
            "get":{
                "tags": [
                    "Mails Counts Endpoints (Aggregation)"
                ],
                "description": "Getting the daily and montly mails count endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/counter/from-counts":{
            "get":{
                "tags": [
                    "Mails Counts Endpoints (Aggregation)"
                ],
                "description": "Aggregation of companies with reported IP mails count endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/counter/ip-counts":{
            "get":{
                "tags": [
                    "Mails Counts Endpoints (Aggregation)"
                ],
                "description": "Aggregation of IP and how many times was detected counts endpoint",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/counter/company-counts":{
            "get":{
                "tags": [
                    "Mails Counts Endpoints (Aggregation)"
                ],
                "description": "Aggregation for the number of report per company",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/book/tb-book":{
            "get":{
                "tags": [
                    "ThreatBook API Data Endpoints"
                ],
                "description": "Aggregation for the number of report per company",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        },
        "/book/tb-book/{ip}":{
            "get":{
                "tags": [
                    "ThreatBook API Data Endpoints"
                ],
                "description": "Aggregation for the number of report per company",
                "security": [
                    {
                        "BearerAuth": []
                    }
                ],
                "parameters":[{
                    "name": "ip",
                    "in": "path",
                    "required": true,
                    "type": "string"
                }],
                "responses":{
                    "200":{
                        "description": "OK"
                    },
                    "403": {
                    "description": "Forbidden"
                    },
                    "500": {
                    "description": "Internal Server Error"
                    }
                }
            }
        }
    },
    "components": {
      "securitySchemes": {
        "BearerAuth": {
          "type": "http",
          "scheme": "bearer",
          "bearerFormat": "JWT"
        }
      }
    }
  }
  
```