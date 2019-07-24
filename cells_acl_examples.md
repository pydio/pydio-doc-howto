# ACL use cases 

## IP Restriction

### Deny access to a workspace to a list of IP

This is an example on how to restrict access to a workspace to a list of specific IP,
you could apply this rule to Cells, etc....

- Create a New Policy (you can put a Label and description of your choice) (as seen in screenshot number 1-2)

[:image-popup:/cells/acls_example/1.png]

[:image-popup:/cells/acls_example/2.png]

- Then put default rights (they are mandatory. It will give read/write to everyone, but with the next rule we are going to restrict them to a specific condition in your case IP) (screenshot 3)

[:image-popup:/cells/acls_example/3.png]

- Now we are going to define the IP restriction rule, so add a policy as seen in screenshot 4)

[:image-popup:/cells/acls_example/4.png]

- Give it a label, effect Deny, Actions Read Write(you can set them as you wish) (as seen in screenshot 5)

[:image-popup:/cells/acls_example/5.png]

- Now add a condition and choose RemoteAddress (as seen in screenshot 6)

[:image-popup:/cells/acls_example/6.png]

- Then write the condition (it's JSON) (screenshot number 7)


[:image-popup:/cells/acls_example/7.png]

So basically we want every IP that matches the list to be denied access

- Now let's apply this rule (you can choose, user, group or role. (in my example I chose a group, screenshot number 8)

[:image-popup:/cells/acls_example/8.png]

- Select the rule (the label was defined in the First Step)

[:image-popup:/cells/acls_example/9.png]

- Once the rule is selected save the changes (screenshot number 10)

[:image-popup:/cells/acls_example/10.png]


**You could also do the opposite of this rule and only give access to a list of IP by using `StringNotMatchCondition`**