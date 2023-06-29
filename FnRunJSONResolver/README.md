`FnRunJSONResolver` is a RapidIdentity Connect action set used to resolve a single value from a rule-based decision tree defined in a JSON file when evaluated against a RapidIdentity record object.


| Parameter | Type | Required | Description |
| -------- | ---- | ------- | ----------- |
|meta_record|`object`|Y|The LDAP record to evaluate against|
|resolver_file_path|`string`|Y|Full path to the JSON resolver file to be used for evaluation|
|resolver|`object`|N|*Used internally for recursion, do not pass any arg to this param*|
|meta_attribute_name|`string`|N|*Used internally for recursion, do not pass any arg to this param*|

These are the rules defined in the `ad-dn-resolver.json` file
```json
{
    "attr:idautoDisabled": {
        "TRUE": "cn=%idautoPersonSAMAccountName%,ou=Suspended,ou=Users,dc=local"
    },
    "attr:employeeType": {
        "Staff": {
            "attr:idautoPersonLocName": {
                "Building A": "cn=%idautoPersonSAMAccountName%,ou=Staff,ou=Building A,ou=Users,dc=local",
                "Building B": "cn=%idautoPersonSAMAccountName%,ou=Staff,ou=Building B,ou=Users,dc=local",
                "Building C": "cn=%idautoPersonSAMAccountName%,ou=Staff,ou=Building C,ou=Users,dc=local"
            }
        },
        "Student": {
            "attr:idautoPersonLocName": {
                "Building A": "cn=%idautoPersonSAMAccountName%,ou=Students,ou=Building A,ou=Users,dc=local",
                "Building B": "cn=%idautoPersonSAMAccountName%,ou=Students,ou=Building B,ou=Users,dc=local",
                "Building C": "cn=%idautoPersonSAMAccountName%,ou=Students,ou=Building C,ou=Users,dc=local"
            }
        }
    }
}
```

Example usage:

```sh
x = createRecordFromObject({
    "idautoDisabled":"TRUE",
    "idautoPersonSAMAccountName":"jdoe",
    "employeeType":"Staff",
    "idautoPersonLocName":"Building C"
})

y = FnRunJSONResolver(x, "ad-dn-resolver.json")
log(y)
# Expected output: "cn=jdoe,ou=Suspended,ou=Users,dc=local"
```

```sh
x = createRecordFromObject({
    "idautoPersonSAMAccountName":"jdoe",
    "employeeType":"Student",
    "idautoPersonLocName":"Building C"
})

y = FnRunJSONResolver(x, "ad-dn-resolver.json")
log(y)
# Expected output: "cn=jdoe,ou=Students,ou=Building C,ou=Users,dc=local"
```

```sh
x = createRecordFromObject({
    "idautoPersonSAMAccountName":"jdoe",
    "employeeType":"Staff",
    "idautoPersonLocName":"Building X"
})

y = FnRunJSONResolver(x, "ad-dn-resolver.json")
log(y)
# Expected output: null
# When a value cant be resolved, null is returned
```
