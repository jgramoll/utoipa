# utoipa - Autogenerate OpenAPI documentation

[![Utoipa build](https://github.com/juhaku/utoipa/actions/workflows/build.yaml/badge.svg)](https://github.com/juhaku/utoipa/actions/workflows/build.yaml)
[![crates.io](https://img.shields.io/static/v1?label=crates.io&message=0.1.0-beta.5&color=orange&logo=rust)](https://crates.io/crates/utoipa/0.1.0-beta.4)
[![docs.rs](https://img.shields.io/static/v1?label=docs.rs&message=utoipa&color=blue&logo=data:image/svg+xml;base64,PHN2ZyByb2xlPSJpbWciIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgdmlld0JveD0iMCAwIDUxMiA1MTIiPjxwYXRoIGZpbGw9IiNmNWY1ZjUiIGQ9Ik00ODguNiAyNTAuMkwzOTIgMjE0VjEwNS41YzAtMTUtOS4zLTI4LjQtMjMuNC0zMy43bC0xMDAtMzcuNWMtOC4xLTMuMS0xNy4xLTMuMS0yNS4zIDBsLTEwMCAzNy41Yy0xNC4xIDUuMy0yMy40IDE4LjctMjMuNCAzMy43VjIxNGwtOTYuNiAzNi4yQzkuMyAyNTUuNSAwIDI2OC45IDAgMjgzLjlWMzk0YzAgMTMuNiA3LjcgMjYuMSAxOS45IDMyLjJsMTAwIDUwYzEwLjEgNS4xIDIyLjEgNS4xIDMyLjIgMGwxMDMuOS01MiAxMDMuOSA1MmMxMC4xIDUuMSAyMi4xIDUuMSAzMi4yIDBsMTAwLTUwYzEyLjItNi4xIDE5LjktMTguNiAxOS45LTMyLjJWMjgzLjljMC0xNS05LjMtMjguNC0yMy40LTMzLjd6TTM1OCAyMTQuOGwtODUgMzEuOXYtNjguMmw4NS0zN3Y3My4zek0xNTQgMTA0LjFsMTAyLTM4LjIgMTAyIDM4LjJ2LjZsLTEwMiA0MS40LTEwMi00MS40di0uNnptODQgMjkxLjFsLTg1IDQyLjV2LTc5LjFsODUtMzguOHY3NS40em0wLTExMmwtMTAyIDQxLjQtMTAyLTQxLjR2LS42bDEwMi0zOC4yIDEwMiAzOC4ydi42em0yNDAgMTEybC04NSA0Mi41di03OS4xbDg1LTM4Ljh2NzUuNHptMC0xMTJsLTEwMiA0MS40LTEwMi00MS40di0uNmwxMDItMzguMiAxMDIgMzguMnYuNnoiPjwvcGF0aD48L3N2Zz4K)](https://docs.rs/utoipa/0.1.0-beta.5/utoipa/)

Want to have your API documented with OpenAPI? But you dont want to see the
trouble with manual yaml or json tweaking? Would like it to be so easy that it would almost
be like utopic? Don't worry utoipa is just there to fill this gap. It aims to do if not all then
the most of heavy lifting for you enabling you to focus writing the actual API logic instead of
documentation. It aims to be *minimal*, *simple* and *fast*. It uses simple proc macros which
you can use to annotate your code to have items documented.

Utoipa crate provides autogenerated OpenAPI documentation for Rust REST APIs. It treats
code first appoach as a first class citizen and simplifies API documentation by providing
simple macros for generating the documentation from your code.

It also contains Rust types of OpenAPI spec allowing you to write the OpenAPI spec only using
Rust if autogeneration is not your flavor or does not fit your purpose.

Long term goal of the library is to be the place to go when OpenAPI documentation is needed in Rust
codebase.

## What's up with the word play?

The name comes from words `utopic` and `api` where `uto` is the first three letters of utopic
and the `ipa` is api reversed.

## Features

* **default** Default enabled features are **json**.
* **json** Enables **serde_json** what allow to use json values in OpenAPI specification values. Thus is
  enabled by default.
* **swagger_ui** Enables the embedded Swagger UI to view openapi api documentation.
* **actix-web** Enables actix-web integration with pre-configured SwaggerUI service factory allowing
  users to use the Swagger UI without a hazzle.
* **actix_extras** Enhances actix-web intgration with being able to parse some documentation
  from actix web macro attributes and types.
* **debug** Add extra traits such as debug traits to openapi definitions and elsewhere.

## Install

Add minimal dependency declaration to Cargo.toml.
```
[dependencies]
utoipa = "0.1.0-beta.5"
```

To enable more features such as use of swagger together with actix-web framework you could define the
dependency as follows.
```
[dependencies]
utoipa = { version = "0.1.0-beta.5", features = ["swagger_ui", "actix-web", "actix_extras"] }
```

## Current project status

**This project is currently in active development and not ready for PRODUCTION use!** 

The basic features are nearly implemented and the it can handle the OpenApi generation in most typical situations. 
It is yet to be released to crates later on when the api and the functionalities gets mature enough. Before initial
release there are going to be still couple of beta releases and one or few rc releases.

See https://github.com/juhaku/utoipa/projects for more details about the progress of the project implementation.

## Examples

Create a struct or it could be an enum also. Add `Component` derive macro to it so it can be registered
as a component in OpenApi schema.
```rust
use utoipa::Component;

#[derive(Component)]
struct Pet {
   id: u64,
   name: String,
   age: Option<i32>,
}
```

Create an handler that would handle your business logic and add `path` proc attribute macro over it.
```rust
mod pet_api {
    /// Get pet by id
    ///
    /// Get pet from database by pet id  
    #[utoipa::path(
        get,
        path = "/pets/{id}"
        responses = [
            (status = 200, description = "Pet found succesfully", body = Pet),
            (status = 404, description = "Pet was not found")
        ],
        params = [
            ("id" = u64, path, description = "Pet database id to get Pet for"),
        ]
    )]
    async fn get_pet_by_id(pet_id: u64) -> Pet {
        Pet {
            id: pet_id,
            age: None,
            name: "lightning".to_string(),
        }
    }
}
```

Tie the `Component` and the api above to the OpenApi schema with following `OpenApi` derive proc macro.
```rust
use utoipa::OpenApi;
use crate::Pet;

#[derive(OpenApi)]
#[openapi(handlers = [pet_api::get_pet_by_id], components = [Pet])]
struct ApiDoc;

println!("{}", ApiDoc::openapi().to_pretty_json().unwrap());
```

This would produce api doc something similar to:
```json
{
  "openapi": "3.0.3",
  "info": {
    "title": "application name from Cargo.toml",
    "description": "description from Cargo.toml",
    "contact": {
      "name": "author name from Cargo.toml",
      "email":"author email from Cargo.toml"
    },
    "license": {
      "name": "license from Cargo.toml"
    },
    "version": "version from Cargo.toml"
  },
  "paths": {
    "/pets/{id}": {
      "get": {
        "tags": [
          "pet_api"
        ],
        "summary": "Get pet by id",
        "description": "Get pet by id\n\nGet pet from database by pet id\n",
        "operationId": "get_pet_by_id",
        "parameters": [
          {
            "name": "id",
            "in": "path",
            "description": "Pet database id to get Pet for",
            "required": true,
            "deprecated": false,
            "schema": {
              "type": "integer",
              "format": "int64"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Pet found succesfully",
            "content": {
              "application/json": {
                "schema": {
                  "$ref": "#/components/schemas/Pet"
                }
              }
            }
          },
          "404": {
            "description": "Pet was not found"
          }
        },
        "deprecated": false
      }
    }
  },
  "components": {
    "schemas": {
      "Pet": {
        "type": "object",
        "required": [
          "id",
          "name"
        ],
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64"
          },
          "name": {
            "type": "string"
          },
          "age": {
            "type": "integer",
            "format": "int32"
          }
        }
      }
    }
  }
}
```

If you have *swagger_ui* and the *actix-web* features enabled you can display the openapi documentation
as easily as follows:
```rust
HttpServer::new(move || {
        App::new()
            .service(
                SwaggerUi::new("/swagger-ui/{_:.*}")
                    .with_url("/api-doc/openapi.json", ApiDoc::openapi()),
            )
    })
    .bind(format!("{}:{}", Ipv4Addr::UNSPECIFIED, 8989))?
    .run()
    .await
```

See more details in `swagger_ui` module of [utoipa docs](https://docs.rs/utoipa/0.1.0-beta.5/utoipa/).
You can also browse to [examples](https://github.com/juhaku/utoipa/tree/master/examples) for 
more comprehensinve examples.

# License

Licensed under either of [Apache 2.0](LICENSE-APACHE) or [MIT](LICENSE-MIT) license at your option.

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in this crate 
by you, shall be dual licensed, without any additional terms or conditions. 
