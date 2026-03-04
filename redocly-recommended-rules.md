


# Redocly Recommended Ruleset — Complete Rules Reference

The `recommended` ruleset is one of the built-in rulesets provided by Redocly CLI. It contains rules grouped by severity: **Errors** and **Warnings**. A `recommended-strict` variant is also available, which elevates all warnings to errors.

**Configuration:**

```yaml
extends:
  - recommended
```

---

## 🔴 Error-Level Rules

These rules default to `error` severity in the recommended configuration.

---

### 1. `no-empty-servers`

**Description:** Requires the `servers` list to be defined in your API. An empty `servers` list defaults to `localhost`, which is not practical for API consumers. Define servers so that the "Try it" and code sample generator features in OpenAPI tools can produce functional API requests.



**Configuration:**

```yaml
rules:
  no-empty-servers: error
```

**❌ Incorrect:**

```yaml
server: []
```

**✅ Correct:**

```yaml
servers:
  - url: https://development.gigantic-server.com/v1
    description: Development server
```

---

### 2. `no-enum-type-mismatch`

**Description:** Requires that the contents of every `enum` value in your API description conform to the corresponding schema's specified `type`. If a property is defined for a certain type, then its corresponding enum values should comply with that type. Lack of compliance is most likely the result of a typo.


**Configuration:**

```yaml
rules:
  no-enum-type-mismatch: error
```

**❌ Incorrect — enum has integer/float values for a string type:**

```yaml
properties:
  huntingSkill:
    type: string
    description: The measured skill for hunting
    enum:
      - adventurous
      - 12
      - 3.14
```

**✅ Correct:**

```yaml
properties:
  huntingSkill:
    type: string
    description: The measured skill for hunting
    enum:
      - adventurous
      - aggressive
      - passive
```

---

### 3. `no-example-value-and-externalValue`

> ⚠️ **Deprecated** — This rule is deprecated in favor of `spec-example-values`.

**Description:** Ensures that `examples` object properties `externalValue` and `value` are mutually exclusive. According to the OpenAPI specification, the `value` field and `externalValue` field are mutually exclusive. The `value` field is for inline example values, while `externalValue` is for URIs referencing external examples.


**Configuration:**

```yaml
rules:
  no-example-value-and-externalValue: error
```

**❌ Incorrect — both `value` and `externalValue` are set:**

```yaml
requestBody:
  content:
    'application/json':
      schema:
        $ref: '#/components/schemas/Address'
      examples:
        foo:
          summary: A foo example
          value: {"foo": "bar"}
          externalValue: https://example.org/examples/foo-example.xml
```

**✅ Correct:**

```yaml
requestBody:
  content:
    'application/json':
      schema:
        $ref: '#/components/schemas/Address'
      examples:
        foo:
          summary: A foo example
          value: {"foo": "bar"}
        bar:
          summary: A bar example
          externalValue: https://example.org/examples/address-example.xml
```

---

### 4. `no-identical-paths`

**Description:** Ensures there are no identical paths in your API descriptions even when they have different path parameter names. For example, `/pets/{petId}` and `/pets/{name}` are considered identical and invalid per the OpenAPI specification.


**Configuration:**

```yaml
rules:
  no-identical-paths: error
```

**❌ Incorrect:**

```yaml
paths:
  /pets/{petId}:
    $ref: ./paths/petById.yaml
  /pets/{name}:
    $ref: ./paths/petByName.yaml
```

**✅ Correct:**

```yaml
paths:
  /pets/{petId}:
    $ref: ./paths/petById.yaml
  /pet-names/{name}:
    $ref: ./paths/petByName.yaml
```

---

### 5. `no-path-trailing-slash`

**Description:** Ensures that paths in your API do not end with a trailing slash (`/`). Some tooling treats `example.com/foo` and `example.com/foo/` as the same thing, but others do not. Enable this rule to avoid confusion and ensure consistency.


**Configuration:**

```yaml
rules:
  no-path-trailing-slash: error
```

**❌ Incorrect:**

```yaml
paths:
  /customers/:
    $ref: ./paths/customers.yaml
```

**✅ Correct:**

```yaml
paths:
  /customers:
    $ref: ./paths/customers.yaml
```

---

### 6. `no-schema-type-mismatch`

**Description:** Ensures that a schema's structural properties match its declared `type`. Specifically:

- A schema of type `object` must not include an `items` field.
- A schema of type `array` must not include a `properties` field.


**Configuration:**

```yaml
rules:
  no-schema-type-mismatch: error
```

**❌ Incorrect — object type with an `items` field:**

```yaml
properties:
  user:
    type: object
    properties:
      id:
        type: string
    items:
      type: number
```

**❌ Incorrect — array type with a `properties` field:**

```yaml
properties:
  tags:
    type: array
    properties:
      name:
        type: string
```

**✅ Correct — object with proper `properties`:**

```yaml
properties:
  user:
    type: object
    properties:
      id:
        type: string
      name:
        type: string
```

**✅ Correct — array with proper `items`:**

```yaml
properties:
  tags:
    type: array
    items:
      type: string
```

---

### 7. `no-server-trailing-slash`

**Description:** Disallows servers with a trailing slash. The endpoint URL is the server URL joined with the path. If the server ends with a slash, it can produce double slashes like `https://example.com/api//pets`.


**Configuration:**

```yaml
rules:
  no-server-trailing-slash: error
```

**❌ Incorrect:**

```yaml
servers:
  - url: https://swift-squirrel.remockly.com/
    description: Mock server
```

**✅ Correct:**

```yaml
servers:
  - url: https://swift-squirrel.remockly.com
    description: Mock server
```

---

### 8. `no-server-variables-empty-enum`

**Description:** Disallows server variables without an `enum` list defined. If you use environment-driven server variables, you should predefine all possible values.


**Configuration:**

```yaml
rules:
  no-server-variables-empty-enum: error
```

**❌ Incorrect — no `enum` for variable:**

```yaml
servers:
  - url: 'https://{env}.example.com/api/v1'
    variables:
      env:
        default: api
        description: Environment
```

**✅ Correct:**

```yaml
servers:
  - url: 'https://{env}.example.com/api/v1'
    variables:
      env:
        default: api
        description: Environment
        enum:
          - api
          - sandbox
          - qa
          - test
          - dev
```

---

### 9. `no-undefined-server-variable`

**Description:** Disallows undefined server variables. If a `{variable}` is used in the server URL, it must be defined in the `variables` object.


**Configuration:**

```yaml
rules:
  no-undefined-server-variable: error
```

**❌ Incorrect — `{tenant}` is not defined:**

```yaml
servers:
  - url: 'https://{tenant}/api/v1'
```

**✅ Correct:**

```yaml
servers:
  - url: 'https://{tenant}/api/v1'
    variables:
      tenant:
        default: api.example.com
        description: Your server host
```

---

### 10. `no-unresolved-refs`

**Description:** Ensures that all `$ref` instances in your API descriptions are resolved. The `$ref` (reference object) is useful for keeping your OpenAPI descriptions DRY, but a typo can make a `$ref` unresolvable.


**Configuration:**

```yaml
rules:
  no-unresolved-refs: error
```

**❌ Incorrect — `Tires` (plural) doesn't exist, only `Tire`:**

```yaml
components:
  schemas:
    Car:
      type: object
      properties:
        color:
          type: string
        tires:
          $ref: '#/components/schemas/Tires'
    Tire:
      type: object
      properties:
        name:
          type: string
```

**✅ Correct:**

```yaml
components:
  schemas:
    Car:
      type: object
      properties:
        color:
          type: string
        tires:
          $ref: '#/components/schemas/Tire'
    Tire:
      type: object
      properties:
        name:
          type: string
```

---

### 11. `nullable-type-sibling`

**Description:** Ensures that all schemas with `nullable` field have a `type` field explicitly set. The `nullable` field is not allowed without the `type` field. *(OAS 3.0 only)*


**Configuration:**

```yaml
rules:
  nullable-type-sibling: error
```

**❌ Incorrect — `nullable` without `type`:**

```yaml
components:
  schemas:
    Incorrect:
      nullable: true
    ReferencingATypeButStillIncorrect:
      nullable: true
      allOf:
        - $ref: '#/components/schemas/SomeType'
```

**✅ Correct:**

```yaml
components:
  schemas:
    Correct:
      type: string
      nullable: true
    CorrectWithAllOf:
      type: object
      nullable: true
      allOf:
        - type: object
          properties:
            name:
              type: string
```

---

### 12. `operation-operationId-unique`

**Description:** Requires unique `operationId` values for each operation. The `operationId` is used by tooling to identify operations.


**Configuration:**

```yaml
rules:
  operation-operationId-unique: error
```

**❌ Incorrect — duplicate `operationId`:**

```yaml
paths:
  /cars:
    get:
      operationId: Car
    post:
      operationId: Car
```

**✅ Correct:**

```yaml
paths:
  /cars:
    get:
      operationId: getCar
    post:
      operationId: postCar
```

---

### 13. `operation-operationId-url-safe`

**Description:** Requires the `operationId` value to be URL safe. Some tooling may use `operationId` in a URL path. This rule makes it possible to use the `operationId` in URLs without any transformation.


**Configuration:**

```yaml
rules:
  operation-operationId-url-safe: error
```

**❌ Incorrect:**

```yaml
paths:
  /cars:
    get:
      operationId: Car<>Wash
```

**✅ Correct:**

```yaml
paths:
  /cars:
    get:
      operationId: CarWash
```

---

### 14. `operation-parameters-unique`

**Description:** Verifies parameters are unique for any given operation. Parameters are considered identical if they have the same `name` and same `in` location.


**Configuration:**

```yaml
rules:
  operation-parameters-unique: error
```

**❌ Incorrect — duplicate parameter:**

```yaml
paths:
  /cars:
    get:
      parameters:
        - name: filter
          in: query
        - name: filter
          in: query
```

**✅ Correct:**

```yaml
paths:
  /cars:
    get:
      parameters:
        - name: filter
          in: query
        - name: another-filter
          in: query
```

---

### 15. `operation-summary`

**Description:** Enforces that every operation has a `summary`. Operation summaries are used to generate API docs — Redocly uses the summary as the header for the operation and the sidebar navigation text.


**Configuration:**

```yaml
rules:
  operation-summary: error
```

**❌ Incorrect — no summary:**

```yaml
post:
  tags:
    - Customers
  operationId: createCustomer
```

**✅ Correct:**

```yaml
post:
  summary: Create a customer
  tags:
    - Customers
  operationId: createCustomer
```

---

### 16. `path-declaration-must-exist`

**Description:** Requires definition of all path template variables. The path template variables must have a string inside `{}` — empty braces `{}` are not allowed.


**Configuration:**

```yaml
rules:
  path-declaration-must-exist: error
```

**❌ Incorrect:**

```yaml
paths:
  /customers/{}:
    post:
```

**✅ Correct:**

```yaml
paths:
  /customers/{id}:
    post:
      parameters:
        - name: id
          in: path
          required: true
          description: The customer's ID.
```

---

### 17. `path-not-include-query`

**Description:** Paths should not include query parameters. Query parameters belong in `parameters` with `in: query`.


**Configuration:**

```yaml
rules:
  path-not-include-query: error
```

**❌ Incorrect:**

```yaml
paths:
  /customers?id={id}:
    get:
      parameters:
        - name: id
          in: query
          required: true
```

**✅ Correct:**

```yaml
paths:
  /customers/{id}:
    get:
      parameters:
        - name: id
          in: path
          required: true
          description: The customer's ID.
```

---

### 18. `path-parameters-defined`

**Description:** Requires all path template variables are defined as path parameters. If you declare a `{variable}` in the path, you must define a corresponding parameter with `in: path`.


**Configuration:**

```yaml
rules:
  path-parameters-defined: error
```

**❌ Incorrect — `{id}` is in the path but not defined as a path parameter:**

```yaml
paths:
  /customers/{id}:
    post:
      parameters:
        - name: filter
          in: query
```

**✅ Correct:**

```yaml
paths:
  /customers/{id}:
    post:
      parameters:
        - name: id
          in: path
          required: true
          description: The customer's ID.
```

---

### 19. `security-defined`

**Description:** Verifies every operation or global security scheme is defined. Every security requirement must reference a security scheme defined in `securitySchemes`. Every API should have security defined, even if it is truly public.


**Configuration:**

```yaml
rules:
  security-defined: error
```

**❌ Incorrect — security references `OAuth` but only `JWT` is defined:**

```yaml
security:
  - OAuth: []
components:
  securitySchemes:
    JWT:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

**✅ Correct:**

```yaml
security:
  - JWT: []
components:
  securitySchemes:
    JWT:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

---

### 20. `spec-components-invalid-map-name`

**Description:** Requires that specific objects inside `components` use keys that match the regular expression: `^[a-zA-Z0-9\.\-_]+$`. No spaces or special characters are allowed.


**Configuration:**

```yaml
rules:
  spec-components-invalid-map-name: error
```

**❌ Incorrect — space in key:**

```yaml
components:
  examples:
    invalid identifier:
      description: invalid identifier
      value: 21
```

**✅ Correct:**

```yaml
components:
  examples:
    valid_identifier:
      description: valid identifier
      value: 21
```

---

### 21. `spec-example-values`

**Description:** Ensures that example objects have valid field combinations according to the OpenAPI specification. Fields `value`, `dataValue`, `serializedValue`, and `externalValue` are mutually exclusive.


**Configuration:**

```yaml
rules:
  spec-example-values: error
```

**❌ Incorrect — both `externalValue` and `value` are set:**

```yaml
components:
  examples:
    InvalidExternalValueAndValue:
      externalValue: https://example.com/user-example.json
      value:
        name: Jane Doe
```

**✅ Correct:**

```yaml
components:
  examples:
    ValidDataValue:
      dataValue:
        name: John Doe
    ValidSerializedValue:
      serializedValue: '{"name":"John Doe"}'
    ValidExternalValue:
      externalValue: https://example.com/user-example.json
```

---

### 22. `spec-no-invalid-encoding-combinations`

**Description:** Ensures that MediaType objects have valid combinations of encoding fields according to the OpenAPI 3.2.0 specification. Only valid encoding field combinations (`encoding`, `prefixEncoding`, `itemEncoding`) are allowed for a given media type.


**Configuration:**

```yaml
rules:
  spec-no-invalid-encoding-combinations: error
```

**❌ Incorrect — `encoding` and `prefixEncoding` used together on `multipart/form-data`:**

```yaml
requestBody:
  content:
    'multipart/form-data':
      schema:
        type: object
        properties:
          id:
            type: integer
      encoding:
        id:
          style: form
      prefixEncoding:
        - style: form
```

**✅ Correct:**

```yaml
requestBody:
  content:
    'multipart/form-data':
      schema:
        type: object
        properties:
          id:
            type: integer
      encoding:
        id:
          style: form
```

---

### 23. `spec-no-invalid-tag-parents`

**Description:** Validates that tag `parent` references are properly defined and don't create circular dependencies. Parent tags must exist in the `tags` array and tag parent relationships must not create cycles. *(OAS 3.2 only)*


**Configuration:**

```yaml
rules:
  spec-no-invalid-tag-parents: error
```

**❌ Incorrect — parent `products` is not defined:**

```yaml
tags:
  - name: books
    parent: products
```

**❌ Incorrect — circular reference:**

```yaml
tags:
  - name: comics
    parent: books
  - name: books
    parent: comics
```

**✅ Correct:**

```yaml
tags:
  - name: products
  - name: books
    parent: products
```

---

### 24. `struct`

**Description:** Ensures that your API document conforms to the structural requirements of the OpenAPI specification (also supports AsyncAPI, Arazzo, and Overlay specs). This is an essential rule that should not be turned off except in rare and special cases.


**Configuration:**

```yaml
rules:
  struct: error
```

**❌ Incorrect — missing required `title` in `info`:**

```yaml
openapi: 3.0.0
info:
  version: 1.0.0
paths: {}
```

**✅ Correct:**

```yaml
openapi: 3.0.0
info:
  title: Ultra API
  version: 1.0.0
paths: {}
```

---

## 🟡 Warning-Level Rules

These rules default to `warn` severity in the recommended configuration.

---

### 25. `info-license`

**Description:** Requires the license info in your API descriptions. By being upfront with the API license, you can reduce friction and encourage API adoption.


**Configuration:**

```yaml
rules:
  info-license: warn
```

**❌ Incorrect — no license:**

```yaml
info:
  version: v1.1
```

**✅ Correct:**

```yaml
info:
  license:
    name: Apache 2.0
```

---

### 26. `info-license-strict`

**Description:** Requires either the license `url` or `identifier` in your API descriptions — not just the license name alone.


**Configuration:**

```yaml
rules:
  info-license-strict: warn
```

**❌ Incorrect — only name, no `url` or `identifier`:**

```yaml
info:
  license:
    name: MIT
```

**✅ Correct (with URL):**

```yaml
info:
  license:
    name: Apache 2.0
    url: https://www.apache.org/licenses/LICENSE-2.0.html
```

**✅ Correct (with identifier):**

```yaml
info:
  license:
    name: Apache 2.0
    identifier: Apache-2.0
```

---

### 27. `no-ambiguous-paths`

**Description:** Ensures there are no ambiguous paths in your API descriptions. When templated paths support the same HTTP methods, they should not have ambiguous resolution. For example, `/{entity}/me` and `/books/{id}` are ambiguous because a single path like `/books/me` could satisfy both.


**Configuration:**

```yaml
rules:
  no-ambiguous-paths: warn
```

**❌ Incorrect — ambiguous paths:**

```yaml
paths:
  '/{entity}/me':
    $ref: ./paths/example.yaml
  '/books/{id}':
    $ref: ./paths/example.yaml
```

**✅ Correct:**

```yaml
paths:
  '/electronics/{id}':
    $ref: ./paths/example.yaml
  '/books/{id}':
    $ref: ./paths/example.yaml
```

---

### 28. `no-duplicated-tag-names`

**Description:** Ensures that tag names in the `tags` array are unique. Duplicate tags can cause confusion in documentation rendering tools and may lead to operations being grouped incorrectly.


**Configuration:**

```yaml
rules:
  no-duplicated-tag-names: warn
```

**Configuration with case-insensitive check:**

```yaml
rules:
  no-duplicated-tag-names:
    severity: warn
    ignoreCase: true
```

**❌ Incorrect — duplicate `store`:**

```yaml
tags:
  - name: pet
    description: Everything about your Pets
  - name: store
    description: Access to Petstore orders
  - name: store
    description: Duplicated store tag
```

**✅ Correct:**

```yaml
tags:
  - name: pet
    description: Everything about your Pets
  - name: store
    description: Access to Petstore orders
  - name: user
    description: Operations about user
```

---

### 29. `no-invalid-media-type-examples`

**Description:** Disallows invalid media type examples by ensuring they comply with the corresponding schema definitions.


**Configuration:**

```yaml
rules:
  no-invalid-media-type-examples:
    severity: warn
    allowAdditionalProperties: false
```

**❌ Incorrect — `year` should be integer, not string:**

```yaml
post:
  requestBody:
    content:
      application/json:
        schema:
          type: object
          properties:
            make:
              type: string
            model:
              type: string
            year:
              type: integer
        examples:
          tesla:
            summary: Red Tesla
            value:
              make: Tesla
              model: Y
              year: '2022'
```

**✅ Correct:**

```yaml
post:
  requestBody:
    content:
      application/json:
        schema:
          type: object
          properties:
            make:
              type: string
            model:
              type: string
            year:
              type: integer
        examples:
          tesla:
            summary: Red Tesla
            value:
              make: Tesla
              model: Y
              year: 2022
```

---

### 30. `no-invalid-parameter-examples`

**Description:** Disallows invalid parameter examples. Parameter examples must match schema constraints (type, maxLength, pattern, etc.).


**Configuration:**

```yaml
rules:
  no-invalid-parameter-examples:
    severity: warn
    allowAdditionalProperties: false
```

**❌ Incorrect — exceeds `maxLength` of 15:**

```yaml
parameters:
  - name: username
    in: query
    schema:
      type: string
      maxLength: 15
    description: Value to query the chess results against usernames
    example: ThisUsernameIsTooLong
```

**✅ Correct:**

```yaml
parameters:
  - name: username
    in: query
    schema:
      type: string
      maxLength: 10
    description: Value to query the chess results against usernames
    example: ella
```

---

### 31. `no-invalid-schema-examples`

**Description:** Disallows invalid schema examples. Schema examples must match the declared type.


**Configuration:**

```yaml
rules:
  no-invalid-schema-examples:
    severity: warn
    allowAdditionalProperties: false
```

**❌ Incorrect — example is a number for a string type:**

```yaml
components:
  schemas:
    Car:
      type: object
      properties:
        color:
          type: string
          example: 3.14
```

**✅ Correct:**

```yaml
components:
  schemas:
    Car:
      type: object
      properties:
        color:
          type: string
          example: red
```

---

### 32. `no-required-schema-properties-undefined`

**Description:** Ensures there are no required schema properties that are undefined. If a property is listed in `required`, it must also be defined in `properties`.


**Configuration:**

```yaml
rules:
  no-required-schema-properties-undefined: warn
```

**❌ Incorrect — `name` is required but not defined in properties:**

```yaml
schemas:
  Pet:
    type: object
    required:
      - id
      - name
    properties:
      id:
        type: integer
        format: int64
```

**✅ Correct:**

```yaml
schemas:
  Pet:
    type: object
    required:
      - id
      - name
    properties:
      id:
        type: integer
        format: int64
      name:
        type: string
        example: doggie
```

---

### 33. `no-server-example.com`

**Description:** Prevents using `example.com` as the value of the `servers.url` field. Although commonly used in documentation, `example.com` is not a real API server and consumers cannot use it to test APIs.


**Configuration:**

```yaml
rules:
  no-server-example.com: warn
```

**❌ Incorrect:**

```yaml
servers:
  - url: https://example.com
    description: Example server
```

**✅ Correct:**

```yaml
servers:
  - url: https://swift-squirrel.remockly.com
    description: Mock server
```

---

### 34. `no-unused-components`

**Description:** Ensures that every component specified in `components` is used (referenced with `$ref`) at least once. Helps prevent data leaks of schemas, parameters, and other properties that may be unused in exposed APIs.


**Configuration:**

```yaml
rules:
  no-unused-components: warn
```

**❌ Incorrect — `dealers` PathItem is never referenced:**

```yaml
openapi: 3.1.0
paths:
  /customers:
    $ref: '#/components/pathItems/customers'
components:
  pathItems:
    customers:
      # ...
    dealers:
      # ... ← unused!
```

**✅ Correct:**

```yaml
openapi: 3.1.0
paths:
  /customers:
    $ref: '#/components/pathItems/customers'
components:
  pathItems:
    customers:
      # ...
```

---

### 35. `operation-2xx-response`

**Description:** Ensures that every operation has at least one successful (2xx) HTTP response defined. Every operation should describe what a successful response looks like, even if it returns no content (204).


**Configuration:**

```yaml
rules:
  operation-2xx-response: warn
```

**❌ Incorrect — no 2xx response:**

```yaml
post:
  responses:
    '400':
      $ref: ../components/responses/Problem.yaml
```

**✅ Correct:**

```yaml
post:
  responses:
    '200':
      $ref: ../components/responses/Success.yaml
    '400':
      $ref: ../components/responses/Problem.yaml
```

---

### 36. `operation-4xx-response`

**Description:** Ensures that every operation has at least one error (4xx) HTTP response defined. What API doesn't have a problem from time to time?


**Configuration:**

```yaml
rules:
  operation-4xx-response: warn
```

**❌ Incorrect — no 4xx response:**

```yaml
post:
  responses:
    '200':
      $ref: ../components/responses/Success.yaml
```

**✅ Correct:**

```yaml
post:
  responses:
    '200':
      $ref: ../components/responses/Success.yaml
    '400':
      $ref: ../components/responses/Problem.yaml
```

---

### 37. `operation-operationId`

**Description:** Requires each operation to have an `operationId` defined. The `operationId` is used by tooling to identify operations. OpenAPI does not consider it required, but it is strongly recommended.


**Configuration:**

```yaml
rules:
  operation-operationId: warn
```

**❌ Incorrect — no `operationId`:**

```yaml
paths:
  /cars:
    get:
      responses:
        '200':
          $ref: ./Success.yaml
```

**✅ Correct:**

```yaml
paths:
  /cars:
    get:
      operationId: GetCar
      responses:
        '200':
          $ref: ./Success.yaml
```

---

### 38. `spec-discriminator-defaultMapping`

**Description:** Ensures that discriminator objects with optional `propertyName` (not in `required` array) include a `defaultMapping` field as required by the OpenAPI 3.2.0 specification. *(OAS 3.2 only)*


**Configuration:**

```yaml
rules:
  spec-discriminator-defaultMapping: warn
```

**❌ Incorrect — optional `propertyName` without `defaultMapping`:**

```yaml
components:
  schemas:
    Base:
      type: object
      discriminator:
        propertyName: type
        mapping:
          a: SomeType
      properties:
        type:
          type: string
```

**✅ Correct:**

```yaml
components:
  schemas:
    Base:
      type: object
      discriminator:
        propertyName: type
        defaultMapping: DefaultType
      properties:
        type:
          type: string
    DefaultType:
      allOf:
        - $ref: '#/components/schemas/Base'
```

---

### 39. `tag-description`

**Description:** Requires that all tags have a non-empty `description`. Good documentation needs descriptions for every tag.


**Configuration:**

```yaml
rules:
  tag-description: warn
```

**❌ Incorrect — no descriptions:**

```yaml
tags:
  - name: Partner APIs
  - name: Customer APIs
```

**✅ Correct:**

```yaml
tags:
  - name: Partner APIs
    description: Endpoints used for integrations with partners and external collaborators.
  - name: Customer APIs
    description: Endpoints used for integrations with customers.
```

---

## Quick Reference Table

| # | Rule | Default Severity | Description |
|---|------|-----------------|-------------|
| 1 | `no-empty-servers` | error | Servers list must be defined |
| 2 | `no-enum-type-mismatch` | error | Enum values must match schema type |
| 3 | `no-example-value-and-externalValue` | error | `value` and `externalValue` are mutually exclusive *(deprecated)* |
| 4 | `no-identical-paths` | error | No identical paths with different param names |
| 5 | `no-path-trailing-slash` | error | Paths must not end with `/` |
| 6 | `no-schema-type-mismatch` | error | Schema structure must match declared type |
| 7 | `no-server-trailing-slash` | error | Server URLs must not end with `/` |
| 8 | `no-server-variables-empty-enum` | error | Server variables must have enum list |
| 9 | `no-undefined-server-variable` | error | Server variables must be defined |
| 10 | `no-unresolved-refs` | error | All `$ref` must resolve |
| 11 | `nullable-type-sibling` | error | `nullable` requires `type` sibling |
| 12 | `operation-operationId-unique` | error | `operationId` must be unique |
| 13 | `operation-operationId-url-safe` | error | `operationId` must be URL safe |
| 14 | `operation-parameters-unique` | error | Parameters must be unique per operation |
| 15 | `operation-summary` | error | Every operation must have a summary |
| 16 | `path-declaration-must-exist` | error | Path template variables must not be empty |
| 17 | `path-not-include-query` | error | Paths must not include query strings |
| 18 | `path-parameters-defined` | error | Path params must be defined |
| 19 | `security-defined` | error | Security schemes must be defined |
| 20 | `spec-components-invalid-map-name` | error | Component keys must match `^[a-zA-Z0-9\.\-_]+$` |
| 21 | `spec-example-values` | error | Example field combinations must be valid |
| 22 | `spec-no-invalid-encoding-combinations` | error | Valid encoding field combinations only |
| 23 | `spec-no-invalid-tag-parents` | error | Tag parents must be valid |
| 24 | `struct` | error | Document must conform to spec structure |
| 25 | `info-license` | warn | License info required |
| 26 | `info-license-strict` | warn | License URL or identifier required |
| 27 | `no-ambiguous-paths` | warn | No ambiguous path resolution |
| 28 | `no-duplicated-tag-names` | warn | Tag names must be unique |
| 29 | `no-invalid-media-type-examples` | warn | Media type examples must match schema |
| 30 | `no-invalid-parameter-examples` | warn | Parameter examples must match schema |
| 31 | `no-invalid-schema-examples` | warn | Schema examples must match type |
| 32 | `no-required-schema-properties-undefined` | warn | Required properties must be defined |
| 33 | `no-server-example.com` | warn | Don't use `example.com` as server |
| 34 | `no-unused-components` | warn | All components must be referenced |
| 35 | `operation-2xx-response` | warn | Must have a 2xx response |
| 36 | `operation-4xx-response` | warn | Must have a 4xx response |
| 37 | `operation-operationId` | warn | Every operation needs an `operationId` |
| 38 | `spec-discriminator-defaultMapping` | warn | Optional discriminator needs `defaultMapping` |
| 39 | `tag-description` | warn | Tags must have descriptions |

---

> **Source:** [Redocly CLI — Recommended Ruleset](https://redocly.com/docs/cli/rules/recommended)

---

## 🔵 Custom & Configurable Rules

These rules are configured manually in `redocly.yaml` to enforce strict documentation standards.

---

### 40. `operation-description`

**Description:** Enforces that every operation (endpoint) has a `description` field. While `summary` is short, `description` allows for detailed explanations.

**Configuration:**

```yaml
rules:
  operation-description: error
```

**❌ Incorrect:**

```yaml
paths:
  /users:
    get:
      summary: Get users
```

**✅ Correct:**

```yaml
paths:
  /users:
    get:
      summary: Get users
      description: |
        Retrieves a list of all users.
        Supports filtering and pagination.
```

---

### 41. `parameter-description`

**Description:** Enforces that every parameter (path, query, header, cookie) has a `description`.

**Configuration:**

```yaml
rules:
  parameter-description: error
```

**❌ Incorrect:**

```yaml
parameters:
  - name: limit
    in: query
    schema:
      type: integer
```

**✅ Correct:**

```yaml
parameters:
  - name: limit
    in: query
    description: Maximum number of results to return.
    schema:
      type: integer
```

---

### 42. `rule/header-description`

**Description:** A custom configurable rule that enforces every `Header` object must have a `description`. This ensures that custom response headers are well-documented.

**Configuration:**

```yaml
rules:
  rule/header-description:
    subject:
      type: Header
      property: description
    assertions:
      defined: true
    message: Header must have a description.
```

**❌ Incorrect:**

```yaml
components:
  headers:
    X-Rate-Limit:
      schema:
        type: integer
```

**✅ Correct:**

```yaml
components:
  headers:
    X-Rate-Limit:
      description: The number of allowed requests in the current period.
      schema:
        type: integer
```

---

### 43. `rule/schema-description`

**Description:** A custom configurable rule that enforces every `Schema` object (including properties of objects) must have a `description`. This is critical for data models, ensuring every field in a request or response body is explained.

**Configuration:**

```yaml
rules:
  rule/schema-description:
    subject:
      type: Schema
      property: description
    assertions:
      defined: true
    message: Schema (request/response field) must have a description.
```

**❌ Incorrect — `email` property missing description:**

```yaml
components:
  schemas:
    User:
      type: object
      properties:
        username:
          type: string
          description: Unique user identifier
        email:
          type: string
```

**✅ Correct:**

```yaml
components:
  schemas:
    User:
      type: object
      description: Represents a registered user.
      properties:
        username:
          type: string
          description: Unique user identifier
        email:
          type: string
          description: User's contact email address
```

---

### 44. `rule/response-description`

**Description:** A custom configurable rule that enforces every `Response` object (including success and error responses) must have a `description`.

**Configuration:**

```yaml
rules:
  rule/response-description:
    subject:
      type: Response
      property: description
    assertions:
      defined: true
    message: Response (including error codes) must have a description.
```

**❌ Incorrect:**

```yaml
paths:
  /users: # Operation
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
```

**✅ Correct:**

```yaml
paths:
  /users: # Operation
    get:
      responses:
        '200':
          description: Successfully retrieved list of users.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
```




## Quick Reference Table

# | Rule | Default Severity | Description |
--- | --- | --- | --- |
1 | `no-empty-servers` | error | Servers list must be defined |
2 | `no-enum-type-mismatch` | error | Enum values must match schema type |
3 | `no-example-value-and-externalValue` | error | `value` and `externalValue` are mutually exclusive _(deprecated)_ |
4 | `no-identical-paths` | error | No identical paths with different param names |
5 | `no-path-trailing-slash` | error | Paths must not end with `/` |
6 | `no-schema-type-mismatch` | error | Schema structure must match declared type |
7 | `no-server-trailing-slash` | error | Server URLs must not end with `/` |
8 | `no-server-variables-empty-enum` | error | Server variables must have enum list |
9 | `no-undefined-server-variable` | error | Server variables must be defined |
10 | `no-unresolved-refs` | error | All `$ref` must resolve |
11 | `nullable-type-sibling` | error | `nullable` requires `type` sibling |
12 | `operation-operationId-unique` | error | `operationId` must be unique |
13 | `operation-operationId-url-safe` | error | `operationId` must be URL safe |
14 | `operation-parameters-unique` | error | Parameters must be unique per operation |
15 | `operation-summary` | error | Every operation must have a summary |
16 | `path-declaration-must-exist` | error | Path template variables must not be empty |
17 | `path-not-include-query` | error | Paths must not include query strings |
18 | `path-parameters-defined` | error | Path params must be defined |
19 | `security-defined` | error | Security schemes must be defined |
20 | `spec-components-invalid-map-name` | error | Component keys must match `^[a-zA-Z0-9\.\-_]+$` |
21 | `spec-example-values` | error | Example field combinations must be valid |
22 | `spec-no-invalid-encoding-combinations` | error | Valid encoding field combinations only |
23 | `spec-no-invalid-tag-parents` | error | Tag parents must be valid |
24 | `struct` | error | Document must conform to spec structure |
25 | `info-license` | warn | License info required |
26 | `info-license-strict` | warn | License URL or identifier required |
27 | `no-ambiguous-paths` | warn | No ambiguous path resolution |
28 | `no-duplicated-tag-names` | warn | Tag names must be unique |
29 | `no-invalid-media-type-examples` | warn | Media type examples must match schema |
30 | `no-invalid-parameter-examples` | warn | Parameter examples must match schema |
31 | `no-invalid-schema-examples` | warn | Schema examples must match type |
32 | `no-required-schema-properties-undefined` | warn | Required properties must be defined |
33 | `no-server-example.com` | warn | Don't use `example.com` as server |
34 | `no-unused-components` | warn | All components must be referenced |
35 | `operation-2xx-response` | warn | Must have a 2xx response |
36 | `operation-4xx-response` | warn | Must have a 4xx response |
37 | `operation-operationId` | warn | Every operation needs an `operationId` |
38 | `spec-discriminator-defaultMapping` | warn | Optional discriminator needs `defaultMapping` |
39 | `tag-description` | warn | Tags must have descriptions |
40 | `operation-description` | error | Every operation must have a `description` |
41 | `parameter-description` | error | Every parameter must have a `description` |
42 | `rule/header-description` | error | Every header must have a `description` |
43 | `rule/schema-description` | error | Every schema and property must have a `description` |
44 | `rule/response-description` | error | Every response must have a `description` |