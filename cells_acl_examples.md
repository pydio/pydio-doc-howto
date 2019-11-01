This how-to will display some real life use cases of the ACL feature.

At the bottom of this page you can find a glossary of all the possible values for the settings.

## IP Restrictions

### Deny Access on a workspace to a list of IP

This is an example on how to restrict access to a **_workspace_** to a list of specific IP,
you could apply this rule to Cells, Share links and so on.

- Create a New Policy (**Policy Type**: `Context-based ACLs` you can put a **Name** and a **Description** of your choice)

[:image-popup:/cells/acls_example/1.png]

[:image-popup:/cells/acls_example/2.png]

- Then put default rights (They are mandatory otherwise other users will not have access. It will give **read/write** to everyone, but with the next rule we are going to filter them to a specific condition in your case IP)

[:image-popup:/cells/acls_example/3.png]

- Now we are going to define the _IP restriction rule_, let's add a policy

[:image-popup:/cells/acls_example/4.png]

- Give it a **Label**, **effect** `Deny`, **Actions** `Read Write` (you can set access as you wish) (as seen in screenshot 5)

[:image-popup:/cells/acls_example/5.png]

- Now add a condition and choose `RemoteAddress` 

[:image-popup:/cells/acls_example/6.png]

- Then write the condition (it's using JSON)


[:image-popup:/cells/acls_example/7.png]

So basically we want every IP that matches the list to be denied access (read and write as it is defined).

- Now let's apply this rule (you can choose, **user**, **group** or **role**. (In this example we chose a **group**)

[:image-popup:/cells/acls_example/8.png]

- Select the rule (the label was defined in the First Step)

[:image-popup:/cells/acls_example/9.png]

- Once the rule is selected save the changes

[:image-popup:/cells/acls_example/10.png]


**You could also do the opposite of this rule and only give access to a list of IP by using `StringNotMatchCondition`**



### Allow access only to a specific IPs/range

- Create a New Policy (Policy Type: `Context-based ACLs`)
- Create the first rule that will **Allow Access** to specific **IP addresses** or a **range**

Allow:

```json
{
  "type": "StringMatchCondition",
  "options": {
    "matches": "192.168.2.*"
  }
}
```

_In this case we want every IP belonging to the sub network 192.168.2.0 to have R/W Access_
_You an add multiple IP/ranges by separating the with a pipe `192.168.0.*|192.168.3.2|etc....`_

- Now define the **Deny Access** rule.
Deny:

```json
{
  "type": "StringMatchCondition",
  "options": {
    "matches": "0.0.0.0/24"
  }
}
```

_This rule is just written as a default, Access will be denied to anyone but the addresses set with the first rule_

## Date/Time Restrictions

<!--TODO-->


## REST method Restrictions

<!--TODO-->

## ACLs values:

Actions:
| Action | Effect           | Example                                                                             |
| ------ | ---------------- | ----------------------------------------------------------------------------------- |
| Read   | read a resource  | for instance with a workspace it means that it's displayed in the list and readable |
| Write  | write a resource | for instance with a workspace you can upload resources or modify existing resources |

Query Context:
| Query          | Effect                       | Description                                                                                 |
| -------------- | ---------------------------- | ------------------------------------------------------------------------------------------- |
| Remote Address | The client's remote address  | this context is about the remote ip that requests access to the resource (usually a client) |
| Request Method | REST Methods                 | the context will be about a REST method such as (PUT, GET, DELETE, etc....)                 |
| Request URI    | A Pydio Cell's endpoint      | the context is about Cells Endpoints                                                        |
| Http Protocol  |                              | this context will be about the http protocol (http/https)                                   |
| UserAgent      | The agent type that requests | this context is about the UserAgent such as (browsers, mobile apps, etc...)                 |



Conditions:
| Type                    | Options     | Example                                                    | Description                                                     |
| ----------------------- | ----------- | ---------------------------------------------------------- | --------------------------------------------------------------- |
| StringMatchCondition    | `"matches`  | `"matches": "192.168.0.1"`                                 | condition is true if there is a match                           |
| StringNotMatchCondition | `"matches`  | `"matches": "192.168.2.1"`                                 | condition is true if there is no match                          |
| DateAfterCondition      | `"matches"` | `"matches": "2018-02-28T23:59+0100"`                       | condition is true if date is after the one defined in the match |
| WithinPeriodCondition   | `"matches"` | `"matches": "2018-02-01T00:00+0100/2018-04-01T00:00+0100"` | condition is true if date is within the range of match          |
| OfficeHoursCondition    | `"matches"` | `"matches": "Monday-Friday/09:00/18:30"`                   | condition is true if date & time are within the match           |

