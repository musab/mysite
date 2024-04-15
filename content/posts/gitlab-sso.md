+++
title = 'Implementing SSO on GitLab Community Edition Deployed to Kubernetes'
date = 2024-04-15T16:40:29+09:00
draft = false
+++

I was recently tasked with setting up Single Sign-On (SSO) for our GitLab instance, which is deployed on our Kubernetes (EKS) instance. This deployment was carried out using [GitLab's Helm Charts](https://docs.gitlab.com/charts/).

The challenge I encountered was that most of the existing documentation provided instructions for setting up SSO on GitLab instances installed on a Linux box, and not for instances deployed using Helm Charts. I have documented the steps I took below:

1. First, ensure that your [identity provider is supported](https://docs.gitlab.com/ee/integration/omniauth.html#supported-providers). For this example, I will outline the steps required for AWS Cognito. However, these steps should be very similar for other providers.

2. The [GitLab AWS Cognito docs](https://docs.gitlab.com/ee/administration/auth/cognito.html) suggests setting up by manually editing the `/etc/gitlab/gitlab.rb` file. However, for users who have deployed GitLab using Helm Charts, this is not possible. In the Helm Chart documentation (https://docs.gitlab.com/charts/charts/globals.html#omniauth), under the 'Globals' section, they list the options that can be passed to `omniauth`. But does not include which key `omniauth` should be under. To enable `omniauth` in your Helm manifest, pass it nested under `appConfig`, as shown below:


```yaml
appConfig:
	omniauth:
		enabled: true
		allowSingleSignOn: ['cognito']
		providers:
		- secret: gitlab-cognito-provider
```

3. GitLab requires the provider information to be passed in as a Kubernetes secret. The secret should match the value from `gitlab_rails['omniauth_providers']` in the [AWS Cognito integration documentation](https://docs.gitlab.com/ee/administration/auth/cognito.html).
