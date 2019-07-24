This how-to will display some real life use cases of the ACL feature.

## IP Restriction

### Deny access to a workspace to a list of IP

This is an example on how to restrict access to a **_workspace_** to a list of specific IP,
you could apply this rule to Cells, Share links and so on.

- Create a New Policy (you can put a **Name** and **Description** of your choice) (as seen in screenshot number 1-2)

[:image-popup:/cells/acls_example/1.png]

[:image-popup:/cells/acls_example/2.png]

- Then put default rights (They are mandatory otherwise other users will not have access. It will give **read/write** to everyone, but with the next rule we are going to filter them to a specific condition in your case IP) (screenshot 3)

[:image-popup:/cells/acls_example/3.png]

- Now we are going to define the _IP restriction rule_, let's add a policy (as seen in screenshot 4)

[:image-popup:/cells/acls_example/4.png]

- Give it a **Label**, **effect** `Deny`, **Actions** `Read Write` (you can set access as you wish) (as seen in screenshot 5)

[:image-popup:/cells/acls_example/5.png]

- Now add a condition and choose `RemoteAddress` (as seen in screenshot 6)

[:image-popup:/cells/acls_example/6.png]

- Then write the condition (it's using JSON) (screenshot number 7)


[:image-popup:/cells/acls_example/7.png]

So basically we want every IP that matches the list to be denied access (read and write as it is defined).

- Now let's apply this rule (you can choose, **user**, **group** or **role**. (In this example I chose a **group**, screenshot number 8)

[:image-popup:/cells/acls_example/8.png]

- Select the rule (the label was defined in the First Step)

[:image-popup:/cells/acls_example/9.png]

- Once the rule is selected save the changes (screenshot number 10)

[:image-popup:/cells/acls_example/10.png]


**You could also do the opposite of this rule and only give access to a list of IP by using `StringNotMatchCondition`**