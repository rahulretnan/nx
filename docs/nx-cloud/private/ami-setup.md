# Setting up a dedicated NxCloud VM

## AWS EC2

1. Login to your AWS Console and select [the top image published here](https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#Images:visibility=public-images;imageName=nx-cloud;owner=623002322076;sort=desc:imageName)
2. Launch a new instance from that AMI
3. Recommended instance type: `t3.2xlarge`
4. You will need to SSH into the instance once it's created:
   1. Use an existing SSH key-pair that you already have installed locally.
   2. [Or create a new one and download the keys locally](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html?icmpid=docs_ec2_console#having-ec2-create-your-key-pair) 3. Then select it in the list
5. Networking: 6. Allow the instance to receive HTTP and HTTPS traffic 7. Allow SSH from your current IP
6. Leave the storage options as they are (external storage of 50GB)
7. "Launch instance"
8. Wait 10 minutes, then navigate to your instance's IP in the browser. You should see the NxCloud dashboard!

### Your NxCloud URL

1. At this point, you can consider your instance's IP address the URL of NxCloud. 2. You can use NxCloud and all the below steps will work for you.
2. You might want, however, to add a Load Balancer in front of the instance, with an explicit domain (e.g. https://my-nxcloud.my-org.com). 3. This is strongly recommended because you will be able to upgrade/restart/re-configure your NxCloud EC2 instance while keeping the NxCloud URL static. 3. Create an [application load balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-application-load-balancer.html) 4. You will need to create a certificate for your domain to assign to the LB

Once you have your NxCloud URL proceed to the below steps!

### Configuring your NxCloud instance

1. Create a new `myconfiguration.yaml` file with the below contents

```yaml
# This is all you need to get the baseline of your nx-cloud instance configured!

# Set the external URL your instance is running on. This is the URL from the previous step
nxCloudAppURL: 'https://nx-cloud.on.my-domain.ca'

secret:
  # set your initial admin password for logging into the app
  adminPassword: 'correcthorsebatterystaple'
```

2. Apply the configuration:

```bash
scp -r ./myconfiguration.yaml nx-cloud@<your-instance-ip>:~/config/user/update.yaml
```

That's it! After a few minutes, you should be able to log-in using your password!

### Connecting to your NxCloud instance

In your Nx workspace, you can enable NxCloud by running:

```bash
NX_CLOUD_API="https://nx-cloud.on.my-domain.ca" npx nx connect
```

### Advanced configuration and auth

You can optionally enable authentication using your preferred SSO provider:

- GitHub
- Bitbucket
- GitLab
- SAML (Okta, Azure AD etc.)

<details>
<summary>⤵️ Here are example configs for enabling auth:</summary>

```yaml
# This is all you need to get the baseline of your nx-cloud instance configured!

# only use this if you'd like to use any of the newer NxCloud version from here: https://nx.dev/nx-cloud/reference/release-notes#docker-containers
# global.imageTag: ''

# Set the external URL your instance is running on
nxCloudAppURL: 'https://nx-cloud.on.my-domain.ca'

# Uncomment (along with github secrets below) to enable working with GitHub pull requests or github auth
#github:
#  auth:
#    enabled: false

#gitlab:
#  apiUrl: '' # this is only needed if you have a self-hosted instance
#  auth:
#    enabled: false

#bitbucket:
#  auth:
#    enabled: false

#saml:
#  auth:
#    enabled: false

# Provide plaintext values for your application to use. We will extract them,
# store them within the application runtime, and scrub the plaintext ones from
# the filesystem
secret:
  # set your initial admin password for logging into the app
  # see here: https://nx.dev/nx-cloud/private-cloud/auth-single-admin
  adminPassword: 'correcthorsebatterystaple'

  # If you want to enable GitHub Login, just provide your client id & secret, we handle the rest
  # see here: https://nx.dev/nx-cloud/private-cloud/auth-github
  githubAuthClientId: 'my_client_id'
  githubAuthClientSecret: 'my_client_secret'

  # The same goes for GitLab authentication
  # see here: https://nx.dev/nx-cloud/private-cloud/auth-gitlab
  # gitlabAppId: 'my_gitlab_app_id'
  # gitlabAppSecret: 'my_gitlab_app_secret'

  # Bitbucket too! If these are uncommented, BB auth is automatically enabled
  # see here: https://nx.dev/nx-cloud/private-cloud/auth-bitbucket
  # bitbucketAppId: 'bitbucket_app_id'
  # bitbucketAppSecret: 'bitbucket_app_secret'

  # SAML auth
  # see here: https://nx.dev/nx-cloud/private-cloud/auth-saml
  # samlEntryPoint: 'your_saml_entry_point'
  # samlCert: 'saml_cert'
```

</details>

### Upgrades

We send out emails with every new NxCloud release to all our Enterprise customers:

1. You can view your current version at the `/version` route: https://your-nx-cloud-url.com/version
2. [And these are the latest NxCloud releases](https://nx.dev/nx-cloud/reference/release-notes#docker-containers)

To upgrade to a newer version, add the below line to your `myconfiguration.yml` file:

```yaml
global.imageTag: '2306.01.2' # set the version of nx-cloud you'd like
```

And apply the changes:

```bash
scp -r ./myconfiguration.yaml nx-cloud@<your-instance-ip>:~/config/user/update.yaml
```
