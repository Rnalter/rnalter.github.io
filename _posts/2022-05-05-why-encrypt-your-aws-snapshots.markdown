---
layout: post
title:  "Why should you encrypt your AWS EBS snapshots!"
date:   2022-05-05 10:18:00
categories: Product-Security
---

Have you wondered whether encrypting AWS EBS snapshots, is it helpful in terms of security or is it just a best practice from AWS or a compliance check.
Here's a short story on why encryption of snapshot had been helpful in stopping/escalating an attack. 

While performing a penetration test for a client. I was able to gather their AWS keys with certain permissions. All the EBS snapshots were encrypted. 
Before jumping onto how to execute the attack , let's first dive into how EBS snapshots are shared with different AWS accounts.

1) Sharing plain/unencrypted snapshots
   - Modify permissions to share the snapshot with a AWS account id - [AWS docs][AWS-sharing-snapshots]
   
2) Sharing encrypted snapshots 
   - Create another snapshot with Customer Managed encryption encryption 
   - Share the CMK key and modify permissions of the snapshot - [AWS guide][AWS-sharing-encrypted-snapshot]


In this scenario, the keys were authorized to "modify snapshot permissions" to be shared with any other AWS account. 
But since the snapshot was encrypted with their default AWS KMS keys, I had to create another snapshot with CMK encryption and again modify the permissions to be shared
with my AWS account. Fortunately(for the client), the keys were not authorized to create another snapshot with the CMK keys. 
I was hoping to find the applications JAR,WAR files or codebases with other secrets in the snapshot.

Hence in this situation, encryption helped prevent further escalation of the attack. Which brings us to the conclusion

1) Always create roles/access keys with least privileges
   
2) Encrypt your snapshots in case, it has having sensitive data such as PII or codebase/jar/war etc 


[AWS-sharing-snapshots]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-modifying-snapshot-permissions.html
[AWS-sharing-encrypted-snapshot]: https://aws.amazon.com/premiumsupport/knowledge-center/share-ebs-volume/
