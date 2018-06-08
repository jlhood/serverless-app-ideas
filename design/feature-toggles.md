# Feature Toggles

Feature Toggles is a serverless app for for maintaining [feature toggles/flags](https://martinfowler.com/articles/feature-toggles.html) (runtime switches that enable/disable features in a distributed system). It provides Lambda functions to load and update feature toggles.

## Architecture

![App Architecture](https://github.com/jlhood/serverless-app-ideas/raw/master/design/images/feature-toggles-architecture.png)

1. Feature toggles are stored in SSM Parameter Store using its hierarchy feature.
1. The LoadFeatureToggles Lambda function loads all feature toggles managed by this app. All feature toggles are loaded at once for better client-side caching support.
1. The UpdateFeatureToggles Lambda function updates one or more feature toggle values.

## Feature Toggle Structure

Feature toggles have a name and dimension. Name is the name of the feature. Dimensions are used to allow a feature to be enabled for some subset of requests. This is useful to partially enable a feature prior to enabling it for all users. This helps reduce blast radius if there is a problem with a feature. To do this, you could use an account id as a dimension value to represent a feature being enabled for that user account only. Then you can have a special string like "global" that you use as a dimension to represent a feature being enabled/disabled globally. In your application logic, you can check if a feature toggle value is set for the current user's account id. If it is, you use that value. Otherwise, you check the global dimension setting.

## Loading Feature Toggles

The LoadFeatureToggles Lambda function takes no input and returns a JSON structure representing the current settings of all feature toggles. All feature toggles are returned in a single call, because it's best practice to heavily cache feature toggles on the client side so you're not making a remote call to fetch toggle settings, possibly multiple times on each request. The output of LoadFeatureToggles has the following structure:

```json
{
    "feature_toggles": {
        "feature1": {
            "dimension1": true
        },
        "feature1": {
            "dimension2": false
        },
        "feature2": {
            "dimension3": true
        }
    }
}
```

In this example, `feature1` and `feature2` are feature names and `dimension1`, `dimension2`, and `dimension3` are dimensions.

## Updating Feature Toggles

The UpdateFeatureToggles Lambda function allows clients to update one or more feature toggle values. The request is expected to be a JSON object with the following structure (defined in JSONSchema):

```json
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Schema for update_feature_toggles request",
    "type": "object",
    "properties": {
        "operator_id": {
            "type": "string",
            "minLength": 1
        },
        "updates": {
            "type": "array",
            "minItems": 1,
            "items": {
                "$ref": "feature_toggle_update.json"
            }
        }
    },
    "required": [
        "operator_id",
        "updates"
    ]
}
```

Here is the JSONSchema of an individual update:

```json
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "title": "Schema for single feature toggle update",
    "type": "object",
    "properties": {
        "action": {
            "type": "string",
            "enum": [
                "SET",
                "CLEAR",
                "CLEAR_ALL"
            ]
        },
        "toggle_name": {
            "type": "string",
            "minLength": 1
        },
        "dimension": {
            "type": "string"
        },
        "value": {
            "type": "boolean"
        }
    },
    "required": [
        "action",
        "toggle_name"
    ]
}
```

The request takes a list of updates to perform and applies them in the order in which they appear in the request. An update indicates what action should be performed (SET, CLEAR, CLEAR_ALL) and includes the toggle name, dimension and value. Which fields are required depends on what action is being taken.

1. SET - Sets the value for the given feature toggle name and dimension to the given value.
1. CLEAR - Removes the feature toggles entry for the given feature toggle name and dimension. value is ignored for this action.
1. CLEAR_ALL - Removes feature toggle entries for all dimensions for the given feature toggle name. dimension and value are ignored for this action.

Here is an example request:

```json
{
    "operator_id": "janedeveloper",
    "updates": [
        {
            "action": "SET",
            "toggle_name": "feature1",
            "dimension": "dimension1",
            "value": true
        },
        {
            "action": "SET",
            "toggle_name": "feature1",
            "dimension": "dimension2",
            "value": false
        },
        {
            "action": "SET",
            "toggle_name": "feature2",
            "dimension": "dimension3",
            "value": true
        },
        {
            "action": "CLEAR",
            "toggle_name": "feature1",
            "dimension": "dimension1"
        }
    ]
}
```

Assuming no feature toggles were set (LoadFeatureToggles returns empty), invoking UpdateFeatureToggles with the above request and then invoking LoadFeatureToggles again would result in the following response:

```json
{
    "feature_toggles": {
        "feature1": {
            "dimension2": false
        },
        "feature2": {
            "dimension3": true
        }
    }
}
```

This is because feature1/dimension1 was added and set to true, then feature1/dimension2 was added and set to false, then feature2/dimension3 was added and set to true, then feature1/dimension1 was cleared (removed).

## Parameters

1. `LogLevel` - Log level for Lambda functions. Allows log level to be changed at runtime by updating Lambda environment variables. Default: INFO
1. `SSMParameterPathPrefix` - Path prefix to use when storing feature toggle values in SSM Parameter Store. Default: /FeatureToggles
