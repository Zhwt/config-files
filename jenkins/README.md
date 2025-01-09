## Export credentials stored in Jenkins:

**Warning: this method will expose all credentials stored globally to the web console!!!**

Navigate to: Dashboard -> Manage Jenkins -> Script Console and execute the following script:

```groovy
import com.cloudbees.plugins.credentials.CredentialsProvider
import com.cloudbees.plugins.credentials.SystemCredentialsProvider
import com.cloudbees.plugins.credentials.common.StandardUsernameCredentials
import com.cloudbees.plugins.credentials.domains.Domain
import jenkins.model.Jenkins

Jenkins.instance.getExtensionList('com.cloudbees.plugins.credentials.SystemCredentialsProvider')[0]
    .getStore()
    .getCredentials(Domain.global())
    .each { cred ->
        if (cred instanceof StandardUsernameCredentials) {
            println "ID: ${cred.id}"
            println "Username: ${cred.username}"
            if (cred.hasProperty('password')) {
                println "Password: ${cred.password}"
            }
            if (cred.hasProperty('privateKey')) {
                println "Private Key:\n${cred.privateKey}"
            }
            println "-----------------------------------"
        }
    }

```

Result will be like:

```
ID: Plain user info
Username: user1
Password: password
-----------------------------------
ID: User with SSH key
Username: user2
Private Key:
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAArAAAABNl
...
... omit key part ...
...
LNWrdG9hZAECAwQ=
-----END OPENSSH PRIVATE KEY-----
```
