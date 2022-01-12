---
title: "Schema Validation"
authors: [ "John S. Ryan <ryanjo@vmware.com>" ]
status: "draft"
approvers: []
---

# Schema Validations

## Problem Statement

## Terminology / Concepts
_Define any terms or concepts that are used throughout this proposal._

## Proposal


### Specification / Use Cases

- [Use Case: Required Input](#use-case-required-input)
  - [Required input for values with length](#required-input-for-types-with-length)
  - [Required input for types with empty value](#required-input-for-types-with-empty-value)
  - [Required input for types with no empty value](#required-input-for-types-with-no-empty-value)
- [Use Case: Enumeration](#use-case-enumeration)
- [Use Case: Union Structures](#use-case-union-structures)

#### Use Case: Required Input

When the author cannot/will not provide a default value for a data value and requires the consumer to provide it.

Where possible, authors are encouraged to use a type-specific "non-empty" validation. As a fallback, they may opt for nullable+not_null.

##### Required input for types with length

**String Values**

Data values of type `string` carry an automatic non-empty validation:

```yaml
#@data/values-schema
---
dex:
  #@schema/validate min_len=1
  namespace: ""
```
is identical to

```yaml
#@data/values-schema
---
dex:
  namespace: ""
```

The violation should be worded to convey needing a non-empty value:

> A value for "namespace" is required; it is empty.

**Array Values**

Similarly, a non-empty value of an array is having at least one element:

```yaml
#@data/values-schema
---
dex:
  oauth2:
    #@schema/validate min_len=1
    responseTypes:
    - ""
```

Violation error message:

> A value for "responseTypes" is required; it is empty.

##### Required input for types with empty value
 
##### Required input for types with no empty value

Some types have no natural (or easily detectable) "empty" value. In these cases, authors mark the value as "nullable" and constraint it to be "not null":

```yaml
#@data/values-schema
---
dex:
  #@schema/nullable
  #@schema/validate not_null=True
  someMap:
```

Which may be common enough to warrant some syntactic sugar:

```yaml
#@data/values-schema
---
dex:
  #@schema/unset
  someMap:
```

#### Use Case: Enumeration

It's quite common for string values to be constrained to a finite set: an enumeration.

```
data.values.dex.config.connector in ("oidc", "ldap") or \
  assert.fail("Dex connector should be oidc or ldap")
```

```yaml
#@data/values-schema
---
dex:
  config:
    #@schema/validate enum=["oidc", "ldap"]
    connector: ""
```


#### Use Case: Union Structures

Sometimes, design data values where two (or more) sibling keys are intended to be mutually exclusive. That is: it is invalid for more than one of the siblings to be non-null.

For example:
```yaml
#@data/values
---
dex:
  config:
    oidc:
      CLIENT_ID: null #! required if oidc enabled
      CLIENT_SECRET: null #! required if oidc enabled
      issuer: null #! <OIDC_IDP_URL> is required if oidc enabled
    ldap:
      host: null #! <LDAP_HOST> is required if ldap enabed
      bindDN: null
      bindPW: null
```
	
Only one authorization+authentication scheme can be active at a time.
	
```yaml
#@data/values-schema
---
dex:
  #@overlay/match by=overlay.one_of(["oidc", "ldap"])
  #@schema/validate 
  config:
    #@schema/nullable
    oidc:
      CLIENT_ID: null #! required if oidc enabled
      CLIENT_SECRET: null #! required if oidc enabled
      issuer: null #! <OIDC_IDP_URL> is required if oidc enabled
    #@schema/nullable
    ldap:
      host: null #! <LDAP_HOST> is required if ldap enabed
      bindDN: null
      bindPW: null
```


#### Consideration: Order of Validations

> I want to be able to affect the order in which violations are reported
> So that the user’s attention goes to the violations that have the most impact on how the invocation runs.

#### Consideration: Merging Schema Nodes

> I want to be able to further constrain a given data value
> So that my integration with another package/library meets my more strict needs.

> I want to be able to reset the constraints on a given data value
> So that my integration can work even though the author didn't expect my needs.


### Other Approaches Considered


## Open Questions


## Answered Questions


## Complete Examples

- https://github.com/vmware-tanzu/community-edition/tree/main/addons/packages
    - https://github.com/vmware-tanzu/community-edition/blob/cc7eb24b0e13875a06ac8578cd73f83a217ec4d9/addons/packages/harbor/2.2.3/bundle/config/values.star
    - https://github.com/vmware-tanzu/community-edition/blob/cc7eb24b0e13875a06ac8578cd73f83a217ec4d9/addons/packages/vsphere-cpi/1.22.4/bundle/config/values.star
    - https://github.com/vmware-tanzu/community-edition/blob/cc7eb24b0e13875a06ac8578cd73f83a217ec4d9/addons/packages/velero/1.6.3/bundle/config/values.star
    - https://github.com/vmware-tanzu/community-edition/blob/cc7eb24b0e13875a06ac8578cd73f83a217ec4d9/addons/packages/pinniped/0.12.0/bundle/config/upstream/_ytt_lib/supervisor/helpers.lib.yaml
- https://github.com/vmware-tanzu/tanzu-framework
    - https://github.com/vmware-tanzu/tanzu-framework/blob/2b3c557f5651a0bfe79dac1b19e12d5925178bef/pkg/v1/providers/ytt/lib/validate.star
        - e.g. `validate_configuration()` used by https://github.com/vrabbi/tkg-resources/blob/0e302afc87997bee8312cc7da25b76815ea01b0c/TKG%20Customization/tkg/providers/infrastructure-vsphere/v0.7.1/ytt/overlay.yaml
- https://github.com/cloudfoundry/cf-for-k8s/blob/develop/config/get_missing_parameters.star

### 

