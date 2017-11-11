# Hasura Auth API Quickstart
> Get started with a free Hasura Project

This quickstart consists of a basic hasura project with a simple nodejs express app running on it. Once this project is deployed on to a hasura cluster, you will have the nodejs app will run at https://app.cluster-name.hasura-app.io

## Sections

* [Introduction](#introduction)
* [Quickstart](#quickstart)
* [Auth API](#auth-apis)
* [Data API](#data-apis)
* [Filestore API](#filestore-apis)
* [FAQ](#faq)
## Introduction

This quickstart project comes with the following by default:
1. A basic hasura project
2. Two tables `article` and `author` with some dummy data
3. A basic nodejs-express app which runs on the `app` subdomain.

## Quickstart

Follow this section to get this project working. Before you begin, ensure you have the latest version of hasura cli tool installed.

### Step 1: Getting the project

```sh
$ hasura quickstart hasura/auth-api-quickstart
$ cd auth-api-quickstart
```

The above command does the following:
1. Creates a new folder in the current working directory called `auth-api-quickstart`
2. Creates a new trial hasura cluster for you and sets that cluster as the default cluster for this project
3. Initializes `auth-api-quickstart` as a git repository and adds the necessary git remotes.

#### Getting cluster status

After the free cluster is installed, get brief information about the cluster via the following command,

```
$ hasura cluster status
```
This will give you your cluster status like so

```sh
INFO Status:
Cluster Name:       h34-excise98-stg
Cluster Alias:      hasura
Kube Context:       h34-excise98-stg
Platform Version:   v0.15.3
Cluster State:      Synced
```

Keep a note of your cluster name. Alternatively, you can also go to your [hasura dashboard](https://dashboard.hasura.io) and see the clusters you have.

#### Deploying on a hasura cluster

After you are done with quickstart, you will have a free cluster on Hasura, up and running. You need to **git push** your project to the cluster and apply the configuration. This step will apply migrations and deploy custom services that are applicable.

```
$ git add .
$ git commit -m "Hasura Auth API Quickstart"
$ git push hasura master
```

#### Accessing Console

Now that you have deployed the project on your cluster, you would want to manage the schema and explore APIs.

Access the **api-console** via the following command:

```
$ hasura api-console
```

This will open up Console UI on the browser. You can access it at [http://localhost:8080](http://localhost:8080)

## Usage

Using the **api-console**, you can explore different Hasura APIs.

The API console will open the API Explorer tab, where you can try out APIs (Data, Auth, Filestore and Notify) using the API Collections on the left.

### Auth APIs

Every app almost always requires some form of authentication. This is useful to identify a user and provide some sort of personalised experience to the user. Hasura provides various types of authentication (username/password, mobile/otp, email/password, Google, Facebook etc). Hasura Auth calls each authentication method a “provider”.

You can enable/disable providers in your auth configuration. Once a provider is enabled you can use them to signup your users. You can have multiple providers enabled at the same time.

You can also create your custom provider (i.e if you have any custom authentication logic) and configure it with Hasura Auth.

#### Signup
Lets look at an example for simple username/password signup.

* Signup using username and password:
POST `auth.cluster-name.hasura-app.io/v1/signup` HTTP/1.1
Content-Type: application/json
```json
{
  "provider" : "username",
  "data" : {
     "username": "johnsmith",
     "password": "somepass123"
  }
}
```

Typical response of a signup request looks like:
```json
{
  "auth_token": "b4b345f980ai4acua671ac7r1c37f285f8f62e29f5090306",
  "username": "johnsmith",
  "hasura_id": 79,
  "hasura_roles": [
      "user"
  ]
}
```

Once a user is registerd (or signed-up) on Hasura, it attaches a Hasura Identity or (hasura_id) to every user. A Hasura identity is an integer. You can use this value in your application to tie your application’s user to this identity.

#### Login
Now that the user is signed up, let's log in.

* Login using username and password:
POST `auth.cluster-name.hasura-app.io/v1/login` HTTP/1.1
Content-Type: application/json
```json
{
  "provider" : "username",
  "data" : {
     "username": "johnsmith",
     "password": "somepass123"
  }
}
```

After logging in successfully, auth_token is returned in the response. It is the authentication token of the user for the current session.

When a user is logged-in, a session is attached.

#### Understanding Sessions
A session is nothing but a unique, un-guessable identifier attached to that Hasura Auth user account (referred to as auth_token) for that session. This way a user can make subsequent requests without having to authenticate with credentials on every request. Instead, on every request the user can present the auth_token to identify themself.

Every microservice benefits from having the user’s information (id and roles) with each request. In the Hasura platform, every request goes through the API Gateway. So, the API Gateway integrates with the session store to act as a session middleware for all microservices.

When the API gateway receives a request, it looks for a session token in the Bearer token of Authorization header or in the cookie. It then retrieves the user’s id and roles attached to this session token from the session store. This information is sent as `X-Hasura-User-Id` and `X-Hasura-Role` headers to the upstream microservice.

You can make a get request to the following url - `https://app.<cluster-name>.hasura-app.io/whoami` to see the logged in user's information.

You can try out more of these APIs in the `API EXPLORER` tab of the `api console`. 

Additionally, to learn more, check out our [docs](https://docs.hasura-stg.hasura-app.io/0.15/manual/users/index.html)

### Data APIs

The Hasura Data API provides a ready-to-use HTTP/JSON API backed by a PostgreSQL database.

These APIs are designed to be used by any client capable of making HTTP requests, especially
and are carefully optimized for performance.

The Data API provides the following features:
* CRUD APIs on PostgreSQL tables with a MongoDB-esque JSON query syntax.
* Rich query syntax that supports complex queries using relationships.
* Role based access control to handle permissions at a row and column level.

 The url to be used to make these queries is always of the type: `https://data.cluster-name.hasura-app.io/v1/query` (in this case `https://data.h34-excise98-stg.hasura-app.io`)

As mentioned earlier, this quickstart app comes with two pre-created tables `author` and `article`.

```
**author**

column | type
--- | ---
id | integer NOT NULL *primary key*
name | text NOT NULL

**article**

column | type
--- | ---
id | serial NOT NULL *primary key*
title | text NOT NULL
content | text NOT NULL
rating | numeric NOT NULL
author_id | integer NOT NULL
```

Alternatively, you can also view the schema for these tables on the api console by heading over to the tab named `data`.

You can just paste the queries shown below into the json textbox in the API explorer and hit send to test them out.
(The following is a short set of examples to show the power of the Hasura Data APIs, check out our [documentation](https://docs.hasura.io/) for more when you're done here!)

Let's look at some sample queries to explore the Data APIs:

#### CRUD
Simple CRUD Operations are supported via an intuitive JSON query language.

* Select all entries in the article table, ordered by rating:
```json
{
    "type": "select",
    "args": {
        "table": "article",
        "columns": ["*"],
        "order_by": [
            {
                "column": "rating"
            }
        ]
    }
}
```

* Update a particular entry in the author table:
```json
{
    "type": "update",
    "args": {
        "table": "author",
        "where": {
            "name": {
                "$eq": "Adams"
            }
        }
    }
}
```

* The where clause on the Data API is a very expressive boolean expression, and can be arbitrarily complex. For example:
```json
{
    "type": "select",
    "args": {
        "table": "article",
        "columns": [
            "content"
        ],
        "where": {
            "$and": [
                {
                    "$or": [
                        {
                            "author_id": {
                                "$eq": "7"
                            }
                        },
                        {
                            "title": {
                                "$like": "Editorial%"
                            }
                        }
                    ]
                },
                {
                    "rating": {
                        "$gte": "3"
                    }
                }
            ]
        },
        "order_by": [
            {
                "column": "rating",
                "order": "asc"
            }
        ]
    }
}
```

  This query will select all the articles with ratings above 3, which were either written by an author with author_id 7 or, which have a title starting with "Editorial". This can be used to construct complex queries that feel very intuitive.

#### Role based access control
Permissions on the Data APIs are designed to restrict the operations that can be performed on the database by various users/roles. The Data APIs support setting permissions on various CRUD operations at a row/column granularity.  By default, the admin role has access to all operations.

This is accomplished through the session middleware that Hasura provides. This session middleware provides the Data API with the role and user id of the current user with every request, and this lets the Data service apply the permissions as appropriate.

* The permissions can be based on a user id check from the information provided by the session middleware:
```json
{
    "type" : "create_insert_permission",
    "args" : {
        "table" : "article",
        "role" : "user",
        "permission" : {
            "check" : {
                "author_id" : "REQ_USER_ID"
            }
        }
    }
}
```

  This query will set select permissions on the article table for the user role so that users will be able only insert entries into the article table with author_ids matching their user ids. This means that the database will not permit a user to write an article in another user's name.
  This sort of a constraint is a property of the data, and therefore should be accomplished in the database, and the permission layer provides the perfect tools for the job.
  Apart from create_insert_permissions, the Data API also provides other types of queries to create select/update and delete permissions. This way, permissions can be set on all CRUD operations.

* The permission object in the json query uses syntax very similar to a where clause in the select query, making it extremely expressive,  as shown here:
```json
{
    "type" : "create_update_permission",
    "args" : {
        "table" : "article",
        "role" : "user",
        "permission" : {
            "check" : {
                "author_id" : "REQ_USER_ID",
                "$or" : [
                    {
                        "category" : "editorial",
                        "is_reviewed" : false
                    },
                    {
                        "category" : { "$neq" : "editorial"}
                    }
                ]
            }
        }
    }
}
```

This query sets insert permissions on the article table for the user role so that users can only insert entries into the table if the author_id is the same as their user id, and if is_reviewed is false when the category is editorial.

* This permissions setup can be further improved by creating custom roles. For example, the above schema can be improved by having an author role that can be given permissions to edit only the article table, and nothing else.

This is very useful for a more complex schema, say a forum, with several types of users like admins, moderators, thread owners, and normal users.

Explore the Data APIs further using the learning-center in the API console!
Bring up the API console using
```bash
$ hasura api-console
```
And then navigate to the [ Learning center ](http://localhost:8080/learning-center) tab.

### File APIs

Sometimes, you would want to upload some files to the cloud. This can range from a profile pic for your user or images for things listed on your app. You can securely add, remove, manage, update files such as pictures, videos, documents using the Hasura filestore.

This is done via simple POST, GET and DELETE requests on a single endpoint.

Just like the Data service, the File API supports Role based access control to the files, along with custom authorization hooks. (Check out our [ documentation ](https://docs.hasura.io/) for more!)

#### Uploading files

Uploading a file requires you to generate a file_id and make a post request with the content of the file in the request body and the correct mime type as the content-type header.

```http
POST https://filestore.project-name.hasura-app.io/v1/file/05c40f1e-cdaf-4e29-8976-38c899 HTTP/1.1
Content-Type: image/png
Authorization: Bearer <token>

<content-of-file-as-body>
```

This is a very simple to use system, and lets you directly add an Upload button on your frontend, without spending time setting up the backend.

#### Downloading files
Downloading a file requires the unique file id that was used to upload it. This can be stored in the database and retrieved for download.

To download a particular file, what is required is a simple GET query.
```http
GET https://filestore.project-name.hasura-app.io/v1/file/05c40f1e-cdaf-4e29-8976-38c899 HTTP/1.1
Authorization: Bearer <token>
```

#### Permissions
By default, the File API provides three hooks to choose from

  1. Private: Only logged in users can upload/download.
  2. Public: Anyone can download, but only logged in users can upload.
  3. Read Only: Anyone can download, but no one can upload.

You can also set up your own authorization webhook!
(Check out our [ documentation ](https://docs.hasura.io/) for more!)

### Notify APIs

Check out the [ Learning center ](http://localhost:8080/learning-center) tab on the API Console for short tutorials on all the APIs!

## Add your own custom microservice

#### Docker microservice

```
$ hasura microservice add <service-name> -i <docker-image> -p <port>
```

#### git push microservice

```bash
$ hasura microservice add <service-name>
```

Once you have added a new service, you need to add a route to access the service.

#### Add route for the service created.

```bash
$ hasura route generate <service-name>
```

It will output the route information that needs to be put into conf/routes.yaml file.

#### Add a remote for the service

```bash
$ hasura remotes generate <service-name>
```

This will output the remotes configuration, which should be added to the conf/remotes.yaml file under the {{ cluster.name }} key.
    Make sure it is properly indented!

#### Apply your changes

```
$ git add .
$ git commit -m "Added a new service"
$ git push hasura master
```
# auth-api-quickstart

A blank template to be used as a starting point to build projects on Hasura. A "project" is a "gittable" directory in the file system, which captures all the information regarding clusters, services and migrations. It can also be used to keep source code for custom services that you write.

## Files and Directories

The project (a.k.a. project directory) has a particular directory structure and it has to be maintained strictly, else `hasura` cli would not work as expected. A representative project is shown below:

```
.
├── hasura.yaml
├── clusters.yaml
├── conf
│   ├── authorized-keys.yaml
│   ├── auth.yaml
│   ├── ci.yaml
│   ├── domains.yaml
│   ├── filestore.yaml
│   ├── gateway.yaml
│   ├── http-directives.conf
│   ├── notify.yaml
│   ├── postgres.yaml
│   ├── routes.yaml
│   └── session-store.yaml
├── migrations
│   ├── 1504788327_create_table_user.down.yaml
│   ├── 1504788327_create_table_user.down.sql
│   ├── 1504788327_create_table_user.up.yaml
│   └── 1504788327_create_table_user.up.sql
└── services
    ├── adminer
    │   └── k8s.yaml
    └── flask
        ├── src/
        ├── k8s.yaml
        └── Dockerfile
```

### `hasura.yaml`

This file contains some metadata about the project, namely a name, description and some keywords. Also contains `platformVersion` which says which Hasura platform version is compatible with this project.

### `clusters.yaml`

Info about the clusters added to this project can be found in this file. Each cluster is defined by it's name allotted by Hasura. While adding the cluster to the project you are prompted to give an alias, which is just hasura by default. The `kubeContext` mentions the name of kubernetes context used to access the cluster, which is also managed by hasura. The `config` key denotes the location of cluster's metadata on the cluster itself. This information is parsed and cluster's metadata is appended while conf is rendered. `data` key is for holding custom variables that you can define.

```yaml
- name: h34-ambitious93-stg
  alias: hasura
  kubeContext: h34-ambitious93-stg
  config:
    configmap: controller-conf
    namespace: hasura
  data: null
```
