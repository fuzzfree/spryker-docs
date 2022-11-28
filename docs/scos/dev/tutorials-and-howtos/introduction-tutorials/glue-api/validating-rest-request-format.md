---
title: Validating REST request format
description: Learn about REST request validation format and how to validate requests in Glue API.
last_updated: Jun 16, 2021
template: howto-guide-template
originalLink: https://documentation.spryker.com/2021080/docs/validating-rest-request-format
originalArticleId: 0e9175dc-05b2-4d72-9493-aac6220e27bd
redirect_from:
  - /2021080/docs/validating-rest-request-format
  - /2021080/docs/en/validating-rest-request-format
  - /docs/validating-rest-request-format
  - /docs/en/validating-rest-request-format
  - /v6/docs/validating-rest-request-format
  - /v6/docs/en/validating-rest-request-format
  - /v5/docs/validating-rest-request-format
  - /v5/docs/en/validating-rest-request-format
  - /v4/docs/validating-rest-request-format
  - /v4/docs/en/validating-rest-request-format
  - /v2/docs/validating-rest-request-format
  - /v2/docs/en/validating-rest-request-format
  - /v1/docs/validating-rest-request-format
  - /v1/docs/en/validating-rest-request-format
related:
  - title: Glue API installation and configuration
    link: docs/scos/dev/feature-integration-guides/page.version/glue-api/glue-api-installation-and-configuration.html
  - title: Glue infrastructure
    link: docs/scos/dev/glue-api-guides/page.version/glue-infrastructure.html
---

Glue API lets you validate requests sent to REST endpoints. It lets you check whether all required fields are present and whether the type and format of the fields are correct.

## Install the `RestRequestValidation` module

To enable validation of REST requests, install the `RestRequestValidation` module by following [Glue API: REST Schema Validation feature integration](/docs/scos/dev/feature-integration-guides/{{site.version}}/glue-api/glue-api-rest-schema-validation-feature-integration.html).

## Use default validation schema

Spryker Glue API comes with a default validation schema. The schema provides default validation rules for the REST APIs shipped with Spryker. You can find it in `vendor/spryker/spryker/Bundles/RestRequestValidator/config/validation.dist.yaml`. Use the schema as a sample and reference for your own validation implementations.

By default, the schema is not applied.

## Validate APIs

To apply validation rules to an API, follow these steps:

1. Copy the default validation schema to the project level and to each API module that you want to be validated.
  For example, to provide validation for the `StoresRestAPI` module, copy the schema to `src/Pyz/Glue/StoresRestAPI/Validation`.

2. Rename the file to the name of the API you are providing validation for.  
  For `StoresRestAPI`, the name is `STORES.validation.yaml`.

3. Specify validation for the endpoints provided by the API.
  Follow the format described in [Validation rule format](#validation-rule-format).

4. Generate validation cache:

```bash
vendor/bin/console rest-api:build-request-validation-cache
```

{% info_block warningBox %}

By default, a validation cache is generated for the current store. To generate it for a specific store, use the `APPLICATION_STORE` variable.
Example:
```bash
APPLICATION_STORE=AT vendor/bin/console rest-api:build-request-validation-cache
```

{% endinfo_block %}

{% info_block warningBox "Verification" %}

Make sure `src/Pyz/Generated/Glue/Validator/validation.cache` has been updated.

{% endinfo_block %}

The API endpoints that you've provided validation rules for validate all incoming requests.

## Validation rule format

The general format of validation YAML files is as follows:

```yaml
endpoint_name1:
  method:
    field1:
      - Constraint1
      - Constraint2
      - ConstraintWithParameters:
          parameterName: '/\w*/'
    field2:
      - Constraint1

endpoint_name2:
  method:
    field1:
    - Constraint1
```

The validation rules are the same as those of the Symfony Validator component. For details, see [Supported Constraints](https://symfony.com/doc/current/validation.html#supported-constraints).

## Deactivate validation of core-level fields

To deactivate validation of a field of an API shipped with Spryker, on a project or store level, override the field without any constraints.

For example, if an API is validated on the core level as follows:

```yaml
access-tokens:
  post:
    username:
      - NotBlank
      - Email
      - Regex:
          pattern: '/\w*/'

refresh-tokens:
  post:
    refresh_token:
    - Required
```

To remove the validation of the `username` field, override the field on the project or store level as follows:

```yaml
access-tokens:
  post:
    username:

refresh-tokens:
  post:
    refresh_token:
    - Required
```

## Make fields optional

By default, all the request fields are required. To make a field optional, write the schema as follows:

```yaml
refresh-tokens:
  post:
    refresh_token:
    - Optional:
        constraints:
        - NotBlank
```

## Validation cache

All validation rules in Spryker Glue API are cached.

To apply new or updated rules, rebuild the validation cache:

```bash
vendor/bin/console rest-api:build-request-validation-cache
```

{% info_block infoBox %}

By default, a validation cache is generated for the current store. To generate it for a specific store,  use the `APPLICATION_STORE` variable.
Example:
```bash
APPLICATION_STORE=AT vendor/bin/console rest-api:build-request-validation-cache
```

For validation, you can use a [Spryk](/docs/scos/dev/glue-api-guides/{{site.version}}/glue-spryks.html):

```bash
console spryk:run AddGlueValidation --mode=project --module=ResourcesRestApi --organization=Pyz --resourceType=resources
```

This command places the default `validation.yaml` file into the specified module. You need to add attributes manually.

{% endinfo_block %}