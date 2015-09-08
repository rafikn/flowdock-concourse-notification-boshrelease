BOSH Release for flowdock-concourse-notification-resource
=========================================================

Send messages to your [Flowdock](https://flowdock.com) channels from Concourse:

![example](http://cl.ly/image/3L0w1r2t2V1H/Concourse_Flowdock_Notification.png)

This BOSH release packages Stark and Wayne's [flowdock-concourse-notification-resource](https://github.com/starkandwayne/flowdock-concourse-notification-resource.git) for Concourse, to make it simple to include the additional Concourse resource in your BOSH deployed Concourse system.

Final releases are automatically created based on any changes to the upstream flowdock-concourse-notification-resource

See the build pipeline http://ci.starkandwayne.com:8080/pipelines/flowdock-concourse-notification-resource-boshrelease for status.

Final releases are available on https://bosh.io/releases as well as this project's own [GitHub releases](https://github.com/cloudfoundry-community/flowdock-concourse-notification-resource-boshrelease/releases).

Installation
------------

To use this bosh release, first upload it to the BOSH/bosh-lite that is running Concourse:

```
bosh upload release https://bosh.io/d/github.com/cloudfoundry-community/flowdock-concourse-notification-resource-boshrelease
```

Next, update your Concourse deployment manifest to add the resource.

Add the `flowdock-concourse-notification-resource` release to the list:

```yaml
releases:
  - name: concourse
    version: latest
  - name: garden-linux
    version: latest
  - name: flowdock-concourse-notification-resource
    version: latest
```

Into the `worker` job, add the `{release: flowdock-concourse-notification-resource, name: install_flowdock_resource}` job template that will install the package:

```yaml
jobs:
- name: worker
  templates:
    ...
    - {release: flowdock-concourse-notification-resource, name: install_flowdock_resource}
```

The final change is to explicitly list all the resource types (they are implicit) and add the `flowdock-concourse-notification-resource` package to the list:

```yaml
jobs:
- name: worker
  ...
  properties:
    groundcrew:
      resource_types:
      - type: archive
        image: /var/vcap/packages/archive_resource
      - type: cf
        image: /var/vcap/packages/cf_resource
      - type: docker-image
        image: /var/vcap/packages/docker_image_resource
      - type: git
        image: /var/vcap/packages/git_resource
      - type: s3
        image: /var/vcap/packages/s3_resource
      - type: semver
        image: /var/vcap/packages/semver_resource
      - type: time
        image: /var/vcap/packages/time_resource
      - type: tracker
        image: /var/vcap/packages/tracker_resource
      - type: pool
        image: /var/vcap/packages/pool_resource
      - type: vagrant-cloud
        image: /var/vcap/packages/vagrant_cloud_resource
      - type: github-release
        image: /var/vcap/packages/github_release_resource
      - type: bosh-io-release
        image: /var/vcap/packages/bosh_io_release_resource
      - type: bosh-io-stemcell
        image: /var/vcap/packages/bosh_io_stemcell_resource
      - type: bosh-deployment
        image: /var/vcap/packages/bosh_deployment_resource
      - type: flowdock-notification
        image: /var/vcap/packages/flowdock-concourse-notification-resource
```

Note that it is the latter two lines that are specific to this BOSH release:

```yaml
- type: flowdock-notification
  image: /var/vcap/packages/flowdock-concourse-notification-resource
```

The former lines should be obtained from the Concourse BOSH release, not the documentation above which might be out of date. Use https://github.com/concourse/concourse/blob/master/jobs/groundcrew/spec#L69-L96

And `bosh deploy` your Concourse manifest.

Usage
-----

First, register a new webhook and select a default channel to send messages https://starkandwayne.slack.com/services/new/incoming-webhook

Slack will give you a URL like `https://hooks.slack.com/services/XXXX/XXX/XXXX`.

An example mini-pipeline that would send an alert:

```yaml
---
jobs:
- name: alert
  public: true
  plan:
  - put: alert
    params:
      text: Hi everybody!
      channel: "#general"
      username: concourse
      icon_url: http://cl.ly/image/3e1h0H3H2s0P/concourse-logo.png

resources:
- name: alert
  type: slack-notification
  source:
    url: https://hooks.slack.com/services/XXXX/XXX/XXXX
```

This will display the message with an icon for the avatar:

![example](http://cl.ly/image/1k44412g3i3E/slack_notification.png)

Setup pipeline in Concourse
---------------------------

```
fly -t snw c -c pipeline.yml --vars-from credentials.yml flowdock-concourse-notification-resource-boshrelease
```
